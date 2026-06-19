# Context Mapping - Bản đồ tương tác Bounded Contexts

Tài liệu này mô tả cách các Bounded Contexts giao tiếp, chia sẻ dữ liệu, và xử lý sự kiện trong hệ thống.

---

## 1. Tổng quan Relationship Patterns

Dựa trên 5 Bounded Contexts chính, chúng ta xác định các mẫu tương tác như sau:

| Context A | Relationship | Context B | Pattern | Ghi chú |
| :--- | :--- | :--- | :--- | :--- |
| **Ordering** | Orchestrates | **Inventory** | Event-Driven | Ordering phát sự kiện, Inventory lắng nghe |
| **Ordering** | Orchestrates | **Payment** | Event-Driven | Ordering phát sự kiện, Payment lắng nghe |
| **Ordering** | Consumes | **Catalog** | API Call / Query | Lấy giá, thông tin sản phẩm khi cần |
| **Ordering** | Consumes | **User** | API Call / Query | Lấy thông tin khách hàng |
| **Inventory** | Publishes | **Ordering** | Event-Driven | Báo kết quả giữ chỗ hàng |
| **Payment** | Publishes | **Ordering** | Event-Driven | Báo kết quả thanh toán |
| **User** | Reference Data | Cả hệ thống | Shared Identity | CustomerId là identifier dùng chung |
| **Catalog** | Reference Data | Cả hệ thống | Read-Only Cache | SKUCode là identifier dùng chung |

---

## 2. Luồng Sự kiện Chính (Mua hàng)

Luồng chính được điều chỉnh theo nguyên tắc: **đặt giữ chỗ kho trước, sau đó yêu cầu thanh toán**. Cụ thể:

1) **Ordering** (PENDING) phát `OrderPlaced` → **Inventory** để yêu cầu giữ chỗ.
2) **Inventory** phản hồi bằng `StockReserved` (nếu thành công) hoặc `InventoryReservationFailed` (nếu hết hàng).
   - Nếu **StockReserved**: **Ordering** chuyển state → `AWAITING_PAYMENT` và tiếp tục phát event `OrderReserved` → **Payment** để bắt đầu quy trình thanh toán.
   - Nếu **InventoryReservationFailed**: **Ordering** chuyển state → `CANCELED` và dừng luồng (không gọi Payment).
3) **Ordering** phát `OrderReserved` → **Payment** để yêu cầu thanh toán.
4) **Payment** xử lý (khách thanh toán) và phát `PaymentConfirmed` hoặc `PaymentRejected` tới **Ordering**.
5) Khi **Ordering** nhận `PaymentConfirmed`, nó sẽ phát event `CommitReservation` tới **Inventory** để khấu trừ tồn kho vĩnh viễn.
   - Nếu Inventory trả về `ReservationCommitted` → `Ordering` chuyển state → `PAID` và tiến hành các bước giao hàng.
   - Nếu Inventory báo không thể commit (ví dụ: trạng thái reservation đã chuyển Expired) → `Ordering` phát `RefundPayment` tới **Payment** để hoàn tiền.

5) Nếu `PaymentRejected`, **Ordering** phát `ReleaseReservation` (hoặc `CancelReservation`) tới **Inventory** để trả hàng về kệ, và chuyển Order → `CANCELED`.

Quy tắc quan trọng:
- **Ordering** giữ vai trò orchestrator: quyết định bước tiếp theo dựa trên response từ Inventory và Payment.
- **Inventory** luôn là authoritative cho trạng thái vật lý của kho; mọi commit phải được xác nhận bởi Inventory.
- Mọi command/event liên quan phải idempotent và có `OrderId` để đảm bảo an toàn khi retry.



## 3. Event-to-Command Mapping

| Bước | Bounded Context Phát (Source) | Sự Kiện (Domain Event) | Bounded Context Nhận (Target) | Lệnh Được Thực Thi (Command) | Thay Đổi Trạng Thái (State Change) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | **Ordering** | `OrderPlaced` | **Inventory** | `ReserveStock` | Đơn hàng: `PENDING` |
| 2a | **Inventory** | `StockReserved` | **Ordering** | `AcceptReservation` | Đơn hàng: `AWAITING_PAYMENT`<br>Giữ kho: `PENDING` (TTL 15m) |
| 2b | **Inventory** | `InventoryReservationFailed` | **Ordering** | `CancelOrder` | Đơn hàng: `CANCELED` |
| 3 | **Ordering** | `OrderReserved` | **Payment** | `ProcessPayment` | Chờ cổng thanh toán phản hồi |
| 4a | **Payment** | `PaymentConfirmed` | **Ordering** | `ConfirmOrderPayment` | Ghi nhận thanh toán thành công <br> Đơn hàng: `PAID`|
| 4b | **Payment** | `PaymentRejected` | **Ordering** | `CancelOrder` | Đơn hàng: `CANCELED` |
| 5 | **Ordering** | `CommitReservation` (Command) | **Inventory** | `DeductStock` | Giữ kho: `COMMITTED`<br>Kho vật lý: Khấu trừ |
| 6a | **Inventory** | `ReservationCommitted` | Ordering | `MarkOrderAsPaid` | Đơn hàng: `CONFIRMED` (Hoàn tất) |
| 6b | **Inventory** | `ReservationCommitFailed` | Ordering | `InitiateRefund` | Đơn hàng: `CANCELED` |
| 7 | **Ordering** | `OrderExpired` | **Payment** | `ProcessRefund` | Chờ cổng thanh toán phản hồi |
| 8 | **Inventory** (Cron) | `ReservationExpired` | Ordering | `ExpireOrder` | Giữ kho: `EXPIRED` (Hàng về kệ)<br>Đơn hàng: `CANCELED` |

---

## 4. Integration Patterns & Anti-Corruption Layers

### 4.1. Ordering ↔ Inventory

**Pattern:** Event-Driven + Orchestration

**Communication:**
- **Ordering → Inventory:** `OrderPlaced` event (async via Kafka)
- **Inventory → Ordering:** `StockReserved`, `InventoryReservationFailed`, `ReservationCommitted` events (async via Kafka)

**Anti-Corruption Layer:** ❌ Không cần
- Cả hai contexts đều dùng `SKUCode` và `OrderId` chung
- Vocabulary không xung đột

---

### 4.2. Ordering ↔ Payment

**Pattern:** Event-Driven + Orchestration

**Communication:**
- **Ordering → Payment:** `OrderPlaced` event (chứa Amount) (async via Kafka)
- **Payment → Ordering:** `PaymentConfirmed`, `PaymentRejected` events (async via Kafka)

**Anti-Corruption Layer:** ❌ Không cần
- Payment không cần biết Order details, chỉ cần `OrderReferenceId` + `Amount`
- Vocabulary tách biệt sạch sẽ

---

### 4.3. Ordering ↔ Catalog (Read-Only)

**Pattern:** Direct API Call (Synchronous)

**Communication:**
- **Ordering → Catalog:** `GET /products/{skuCode}` để lấy giá hiện tại khi hiển thị giỏ hàng
- Ordering lưu `UnitPrice` vào OrderItem tại lúc Checkout (snapshot giá)

**Anti-Corruption Layer:** ⚠️ Cần cảnh báo
- **Nếu Catalog thay đổi cấu trúc:**
  - Ví dụ: Catalog thêm field mới `discountPrice`
  - Ordering không cần biết, vì nó chỉ lấy `price` của SKU cụ thể
- **Khuyến nghị:** Tạo API version riêng hoặc Facade layer nếu Catalog API thay đổi quá thường xuyên

---

### 4.4. Ordering ↔ User (Read-Only)

**Pattern:** Direct API Call (Synchronous)

**Communication:**
- **Ordering → User:** `GET /customers/{customerId}` để lấy thông tin khách hàng
- Ordering snapshot `ShippingInfo` từ User vào Order tại lúc Checkout

**Anti-Corruption Layer:** ❌ Không cần
- User chỉ cung cấp `CustomerId`, `Name`, `Phone`, `Address` — rất đơn giản
- Không có xung đột từ vựng

---

### 4.5. System-wide Shared Identifiers

**Identifier:** `CustomerId`
- Dùng chung bởi: Ordering, User, Payment, Inventory (ghi chú trong Reservation)
- **Authority:** User Context (định nghĩa, quản lý lifecycle)

**Identifier:** `SKUCode`
- Dùng chung bởi: Catalog, Ordering, Inventory, (có thể Payment nếu cần trace)
- **Authority:** Catalog Context (định nghĩa, quản lý lifecycle)

**Identifier:** `OrderId`
- Dùng chung bởi: Ordering, Inventory (tham chiếu), Payment (tham chiếu)
- **Authority:** Ordering Context

**Identifier:** `ReservationId`, `PaymentId`
- Dùng riêng bởi từng Context, chỉ tham chiếu thông qua `OrderId`

---

## 5. Deployment & Communication Infrastructure

### 5.1. Kafka Topics Layout

```
Topic: order.events
  Partitions: 5 (phân tán theo OrderId % 5 để đảm bảo order events của cùng order song hành)
  Subscribers: Inventory, Payment, Ordering (cho logging)
  Events: OrderPlaced, OrderCanceled, OrderReadyForShipping

Topic: inventory.events
  Partitions: 5 (phân tán theo SKUCode % 5)
  Subscribers: Ordering
  Events: StockReserved, InventoryReservationFailed, ReservationCommitted, ReservationExpired

Topic: payment.events
  Partitions: 5 (phân tán theo PaymentId % 5)
  Subscribers: Ordering, Inventory (hoặc tập trung vào Ordering orchestration)
  Events: PaymentConfirmed, PaymentRejected, PaymentRefunded
```

### 5.2. API Calls (Synchronous)

```
Ordering Service
  ├── GET /catalog/{skuCode}           → Catalog Service
  ├── GET /customers/{customerId}      → User Service
  └── POST /payments/initiate          → Payment Service (hoặc qua events)

Inventory Service
  └── GET /stock/{skuCode}             → Internal (không expose ra ngoài)

Catalog Service
  └── GET /products/{skuCode}          → External (published API)

User Service
  └── GET /customers/{customerId}      → External (published API)
```

---

## 6. Failure Scenarios & Compensation

### 6.1. Inventory Reservation Fails

**Scenario:** Order được tạo, nhưng Inventory báo hết hàng

**Flow:**
1. Ordering nhận `InventoryReservationFailed` event
2. Chuyển Order status → `RESERVATION_FAILED`
3. Payment không được tạo (vì Inventory đã fail)
4. Customer nhận thông báo "hết hàng"
5. Không cần compensation — Order được cancel ngay

---

### 6.2. Payment Fails or Timeout

**Scenario:** Order được tạo, Inventory giữ chỗ, nhưng Payment thất bại

**Flow:**
1. Inventory đã tạo Reservation với `ExpiresAt = now + 15min`
2. Payment Service báo `PaymentRejected` hoặc timeout
3. Ordering nhận → chuyển Order status → `CANCELED`
4. **Compensation (Inventory):** Cron job tự động release Reservation sau 15 phút
   - Hoặc: Ordering phát event `OrderCanceled` → Inventory lắng nghe và ngay lập tức release

---

### 6.3. Kafka Message Delivery Guarantees

**Requirement:** At-least-once delivery
- Mỗi service phải idempotent khi xử lý events (sử dụng `OrderId` + `EventType` + `Version` làm composite key trong outbox pattern)

**Outbox Pattern** (để đảm bảo không mất event):
1. Service A lưu aggregate state + event record **trong cùng 1 transaction**
2. Outbox Polling Service theo dõi và phát event lên Kafka
3. Nếu Kafka fail, event vẫn ở Outbox, sẽ retry

---

## 7. Tóm tắt Context Map

```
                    ┌─────────────────────┐
                    │   Catalog Context   │
                    │   (Read-Only Ref)   │
                    └──────────────────────┘
                              ▲
                              │ API call: GET /products/{skuCode}
                              │
        ┌─────────────────────┴──────────────────────┐
        │                                            │
        ▼                                            ▼
┌──────────────────────┐                ┌──────────────────────┐
│  Ordering Context    │                │   User Context       │
│  (Orchestrator)      │                │   (Shared Identity)  │
└─────────┬──────────┬─┘                └──────────────────────┘
          │          │
    Events│          │ Events
          │          │
          ▼          ▼
┌──────────────────────┐        ┌──────────────────────┐
│ Inventory Context    │        │  Payment Context     │
│ (Stock Management)   │        │ (Payment Processing) │
└──────────────────────┘        └──────────────────────┘
```

---

## 8. Quyết định tiếp theo

- [ ] Xác nhận Kafka topology và partition strategy
- [ ] Quyết định Mono-repo hay Multi-repo
- [ ] Thiết kế API Gateway và routing rules
- [ ] Chọn lựa gRPC vs REST vs GraphQL cho inter-service communication
- [ ] Cấu hình Outbox/Inbox pattern cho mỗi service

---

## 9. Local Caching Strategy (Lưu bản sao local để giảm gọi sync)

Mục tiêu: Giảm số lần gọi đồng bộ tới các services tham chiếu (Catalog, User) bằng cách giữ bản sao đọc local, đồng thời đảm bảo tính nhất quán đủ dùng cho trải nghiệm người dùng.

Principles:
- Giữ local copy chỉ cho dữ liệu "read-heavy" và không yêu cầu strong consistency (ví dụ: Catalog, User profile).
- Dùng event-driven invalidation: các Context phát event khi dữ liệu thay đổi, các dịch vụ có cache lắng nghe để cập nhật/invalidade.
- Kết hợp TTL (time-to-live) và cơ chế "stale-while-revalidate" để cân bằng độ tươi và hiệu năng.
- Dùng idempotency & versioning (version hoặc updatedAt) để áp đặt ordering khi apply update từ event stream.

Pattern suggestions:
- Catalog cache in Ordering service:
  - Local cache (in-memory LRU hoặc embedded persistent store như SQLite/RocksDB) để phục vụ `GET /catalog/{sku}` với TTL ~5-60s.
  - Ordering subscribe vào `catalog.events` (ProductUpdated, VariantPriceChanged) để invalidate hoặc apply delta update ngay.
  - Fallback: nếu cache miss, Ordering gọi sync `Catalog` API và lưu kết quả vào cache.

- User cache:
  - Lưu bản sao các trường cần thiết cho checkout (default shipping address, name, phone) với TTL dài hơn.
  - Khi user thay đổi profile, `User` emit `CustomerUpdated` để Ordering cập nhật cache.

- Eventual consistency guarantees:
  - Chấp nhận độ trễ ngắn giữa thay đổi và phản ánh trên các cache local.
  - Trường hợp cần strong correctness (ví dụ: kiểm tra tồn kho trước commit), luôn gọi sync tới authoritative Context (Inventory) hoặc dùng Reservation pattern.

Implementation notes:
- Đảm bảo cache update xử lý idempotently (dựa trên `eventId`/`version`) để tránh race condition.
- Sử dụng Outbox → Kafka để đảm bảo change events được phát nhất quán.
- Cân nhắc một small-topic `catalog.updates` để chỉ phát các thay đổi cần thiết cho cache.

Example quick plan (Catalog):
1. Thêm subscriber `catalog.cache-updater` trong Ordering deployment.
2. Khi nhận `ProductUpdated` hoặc `VariantPriceChanged`, cập nhật local cache và publish telemetry.
3. Thêm TTL + metrics (hit/miss) để điều chỉnh TTL phù hợp.

---

Những bước tiếp theo có thể là: tạo prototype cache cho `Catalog` (Ordering service) hoặc tạo spec API cho `catalog.events`.