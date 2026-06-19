# Domain Analysis (Thuần Nghiệp vụ)

Tài liệu này tập trung 100% vào nghiệp vụ kinh doanh của hệ thống E-commerce, gạt bỏ hoàn toàn các yếu tố công nghệ (Database, API, Message Broker). Chúng ta sẽ phân tích từ lúc bắt đầu hành trình của khách hàng cho đến khi kết thúc giao dịch.

---

## 1. Hành trình Khách hàng (Core Customer Journey)

Đây là kịch bản mua hàng "Happy Path" (luồng thành công) cốt lõi của một hệ thống E-commerce chuẩn mực:

**Giai đoạn 1: Khám phá & Lựa chọn (Discovery & Selection)**
- Khách hàng (Customer) lướt xem Danh mục sản phẩm.
- Khách hàng xem thông tin chi tiết sản phẩm (Tên, Hình ảnh, Giá bán, Mô tả).
- Khách hàng quyết định số lượng và **thêm sản phẩm vào Giỏ hàng**.
- *(Giỏ hàng lúc này chỉ mang tính chất ghi nhớ mong muốn của khách, chưa tác động gì đến kho).*

**Giai đoạn 2: Lập Đơn hàng (Checkout / Order Placement)**
- Khách hàng vào Giỏ hàng, kiểm tra lại danh sách sản phẩm và tổng tiền tạm tính.
- Khách hàng tiến hành **Đặt hàng (Checkout)**.
- Khách hàng cung cấp thông tin Giao hàng (Tên, Số điện thoại, Địa chỉ).
- Hệ thống tính toán Tổng tiền cuối cùng.
- Hệ thống ghi nhận đây là một **Đơn hàng vừa được lập**. Giỏ hàng bị làm trống.

**Giai đoạn 3: Giữ chỗ & Thanh toán (Reservation & Payment)**
- Ngay khi Đơn hàng được lập, hệ thống lập tức phải **Giữ chỗ tồn kho (Reserve Inventory)** để đảm bảo khách không bị mất hàng (hết hàng) trong lúc đang thao tác thanh toán. Việc giữ chỗ này có thời hạn (ví dụ: 15 phút).
- Nếu việc giữ chỗ thất bại (kho thực sự hết hàng), Đơn hàng bị đánh dấu là Hủy do hết hàng.
- Nếu giữ chỗ thành công, hệ thống yêu cầu khách hàng thanh toán.
- Khách hàng tiến hành trả tiền qua Cổng thanh toán.
- Nếu thanh toán thành công, hệ thống xác nhận **Đơn hàng đã thanh toán**, đồng thời **Khấu trừ tồn kho vĩnh viễn** (từ hàng giữ chỗ chuyển thành hàng đã bán).
- Nếu quá hạn 15 phút mà chưa thanh toán, hệ thống tự động **Hủy đơn hàng** và **Giải phóng tồn kho giữ chỗ** (trả hàng lại lên kệ cho người khác mua).

---

## 2. Event Storming: Bóc tách Sự kiện & Lệnh

Từ kịch bản trên, chúng ta bóc tách ra các Hành động (Command - Thể hiện ý định) và Sự kiện (Domain Event - Những gì đã xảy ra trong quá khứ không thể thay đổi).

| Actor (Ai làm?) | Command (Lệnh/Ý định) | Domain Event (Sự kiện đã xảy ra) | Ý nghĩa nghiệp vụ |
| :--- | :--- | :--- | :--- |
| **Customer** | Thêm vào Giỏ hàng (`AddToCart`) | `CartUpdated` | Giỏ hàng thay đổi nội dung, cần tính lại tiền tạm. |
| **Customer** | Tiến hành Đặt hàng (`PlaceOrder`) | `OrderPlaced` | Một đơn hàng mới vừa được hình thành. |
| **System** | Làm trống Giỏ hàng (`ClearCart`) | `CartCleared` | Giỏ hàng của user trở về trống. |
| **System** | Giữ chỗ Tồn kho (`ReserveInventory`) | `InventoryReserved` | Hàng đã được cất riêng cho đơn hàng này. |
| | | `InventoryReservationFailed` | Không đủ hàng trong kho. |
| **System** | Yêu cầu Thanh toán (`RequestPayment`) | `PaymentRequested` | Chờ khách hàng thực hiện trả tiền. |
| **Customer** | Xác nhận Thanh toán (`ConfirmPayment`) | `PaymentConfirmed` | Tiền đã vào tài khoản công ty. |
| | | `PaymentExpired` | Khách không thanh toán trong thời gian quy định. |
| **System** | Khấu trừ Tồn kho (`CommitInventory`) | `InventoryCommitted` | Hàng chính thức rời khỏi kho (chuyển cho giao vận). |
| **System** | Hoàn trả Tồn kho (`ReleaseInventory`) | `InventoryReleased` | Trả hàng về lại kệ do đơn bị hủy. |
| **System** | Đánh dấu Đơn bị Hủy (`CancelOrder`) | `OrderCanceled` | Đơn hàng bị hủy (do hết hàng hoặc chưa thanh toán). |

---

## 3. Bản đồ Năng lực Nghiệp vụ (Business Capabilities)

Từ hành trình và các sự kiện trên, để hệ thống vận hành được, chúng ta cần xây dựng các năng lực nghiệp vụ (Business Capabilities) độc lập. Dưới góc nhìn thuần Business, hệ thống cần "Biết làm những việc gì"?:

### 3.1. Năng lực Trưng bày & Phân loại (Catalog Capability)
- **Mục đích**: Trả lời câu hỏi "Chúng ta có gì để bán?". 
- **Trách nhiệm**: Định nghĩa cấu trúc hàng hóa (Sản phẩm, Biến thể/SKU). Cung cấp thông tin hiển thị (Tên, Hình ảnh, Mô tả, Giá bán) để thu hút khách hàng ở Giai đoạn Khám phá.

### 3.2. Năng lực Bán hàng (Sales & Ordering Capability)
- **Mục đích**: Trả lời câu hỏi "Khách hàng muốn mua gì và thỏa thuận mua bán là gì?".
- **Trách nhiệm**: Lưu trữ ý định mua tạm thời (Giỏ hàng). Chuyển đổi ý định thành một cam kết pháp lý (Đơn hàng). Tính toán số tiền khách phải trả (dựa trên giá của Catalog). Theo dõi tiến trình của Đơn hàng từ lúc tạo đến lúc hoàn tất.

### 3.3. Năng lực Quản lý Hàng hóa thực tế (Inventory Capability)
- **Mục đích**: Trả lời câu hỏi "Chúng ta thực sự còn bao nhiêu hàng trong kho?".
- **Trách nhiệm**: Tách biệt hoàn toàn với khái niệm "Trưng bày". Ở đây chỉ quan tâm đến các con số: `Tồn kho vật lý`, `Đang giữ chỗ`, `Có thể bán`. Nó phải đảm bảo nguyên tắc kinh doanh tuyệt đối: **Không bao giờ bán hàng không có thực**.

### 3.4. Năng lực Giao dịch Tài chính (Payment Capability)
- **Mục đích**: Trả lời câu hỏi "Khách hàng đã đưa tiền chưa?".
- **Trách nhiệm**: Ghi nhận các khoản thanh toán, đối soát (reconciliation) với các kênh thu hộ (ngân hàng, ví điện tử). Đảm bảo một Đơn hàng chỉ được xác nhận thành công khi tiền đã vào túi.

### 3.5. Năng lực Định danh (Identity / Customer Capability)
- **Mục đích**: Trả lời câu hỏi "Ai đang mua hàng?".
- **Trách nhiệm**: Ghi nhớ khách hàng là ai để không phải hỏi lại địa chỉ giao hàng nhiều lần. Đảm bảo tính riêng tư cho Đơn hàng của họ.

---

## 4. Bounded Contexts & Ngôn ngữ thống nhất (Ubiquitous Language)

> **Quy tắc vàng:** *Nếu một từ vựng có nhiều ý nghĩa khác nhau hoặc chịu các luật nghiệp vụ (business rules) khác nhau, chúng ta BẮT BUỘC phải chia nhỏ Context ban đầu để bảo vệ tính nhất quán của dữ liệu.*

Dựa trên 5 Năng lực Nghiệp vụ và sự nhập nhằng của ngôn ngữ tự nhiên, chúng ta vẽ ra các ranh giới ngữ cảnh (Bounded Contexts) để bảo vệ ý nghĩa của từ vựng:

### Ví dụ điển hình: Từ "Sản phẩm" (Product)
Từ "Sản phẩm" là từ nguy hiểm nhất. Nó mang ý nghĩa hoàn toàn khác nhau tùy vào ngữ cảnh:
- Trong góc nhìn của người làm Marketing/Trưng bày: "Sản phẩm" là Hình ảnh, Tên hay, Đoạn văn quảng cáo -> Khai sinh ra **Catalog Context**.
- Trong góc nhìn của người Thủ kho: "Sản phẩm" là cục hàng nằm ở kệ A, nặng 2kg, số lượng còn 5 cái -> Buộc phải đổi tên thành `StockItem` / `SKU` và khai sinh ra **Inventory Context**.
- Trong góc nhìn của người Bán hàng (Kế toán): "Sản phẩm" là món đồ khách đã chốt mua với giá 100k, không được phép thay đổi giá dù Catalog có đổi giá -> Buộc phải đổi tên thành `OrderItem` và khai sinh ra **Ordering Context**.

### Ánh xạ Năng lực -> Bounded Context

| Năng lực (Capability) | Bounded Context | Ngôn ngữ thống nhất (Ubiquitous Language) | Từ vựng bị CẤM sử dụng |
| :--- | :--- | :--- | :--- |
| **Trưng bày & Phân loại** | **Catalog Context** | `Product` (Sản phẩm), `Category` (Danh mục), `Price` (Giá niêm yết). | Cấm dùng `Stock` (Vì Catalog không quan tâm còn hàng hay không). |
| **Bán hàng & Đặt hàng** | **Ordering Context** | `Cart` (Giỏ hàng), `CartItem` (Ý định mua - giá trôi nổi, tự do sửa xóa), `Order` (Đơn hàng), `OrderItem` (Cam kết mua - giá khóa cứng, bất biến). | Cấm dùng `Product`. `CartItem` ≠ `OrderItem` (khác luật nghiệp vụ). |
| **Hàng hóa thực tế** | **Inventory Context** | `StockItem` (Hàng trong kho), `Reservation` (Cục hàng đang giữ chỗ). | Cấm dùng `Product` (Phải gọi là `StockItem` / `SKU`). |
| **Giao dịch Tài chính** | **Payment Context** | `Transaction` (Giao dịch), `PaymentGateway` (Cổng thanh toán). | Cấm dùng `Order` (Payment không biết Order là gì, chỉ biết `ReferenceID`). |
| **Định danh** | **User Context** | `Customer` (Khách hàng), `ShippingAddress` (Sổ địa chỉ). | |

---

## 5. Tactical Design: Catalog Context

> **Mục đích của Context này**: Trả lời câu hỏi *"Chúng ta có gì để bán?"*

### 5.1. Aggregates (Cụm thực thể)

#### Product Aggregate

**Aggregate Root: `Product`**

Một `Product` (Sản phẩm) đại diện cho một mặt hàng mà doanh nghiệp muốn trưng bày và bán cho khách hàng. Product là đơn vị quản lý trung tâm của Catalog.

**Thành phần bên trong:**
- **`Variant`** (Biến thể): Một Product có thể có nhiều biến thể. Mỗi biến thể là một phiên bản cụ thể có thể mua được (ví dụ: "Áo thun Đỏ - Size L", "Áo thun Xanh - Size M"). Mỗi Variant mang một mã `SKUCode` duy nhất trên toàn hệ thống — đây chính là "chìa khóa" để các Context khác (Inventory, Ordering) tham chiếu đến món hàng cụ thể. Mỗi Variant chứa danh sách `VariantImage` (hình ảnh) riêng của mình.

**Quan hệ bên ngoài Aggregate:**
- Một Product **thuộc về** một hoặc nhiều `Category`.
- Một Product **thuộc về** một `Brand` (nếu có).

#### Category Aggregate

**Aggregate Root: `Category`**

Một `Category` (Danh mục) dùng để phân loại sản phẩm theo nhóm, giúp khách hàng duyệt và tìm kiếm dễ dàng hơn.

**Đặc điểm:**
- Category có cấu trúc cây (Cha - Con). Ví dụ: `Thời trang` -> `Áo` -> `Áo thun`.
- Một Category có thể chứa nhiều Category con.
- Một Category không bắt buộc phải có cha (nếu là Category gốc / root).

### 5.2. Value Objects (Đối tượng giá trị)

| Value Object | Mô tả | Ví dụ |
| :--- | :--- | :--- |
| `SKUCode` | Mã định danh duy nhất của một Variant trên toàn hệ thống. Là cầu nối ngôn ngữ giữa Catalog, Inventory và Ordering. | `"AOTHUN-DO-L"` |
| `Money` | Một khoản tiền gắn liền với đơn vị tiền tệ. Không bao giờ tách `amount` ra khỏi `currency`. | `Money(100000, "VND")` |
| `VariantImage` | Một hình ảnh gắn với biến thể cụ thể, bao gồm đường dẫn và thứ tự hiển thị. | `VariantImage(url, altText, displayOrder)` |

### 5.3. Business Invariants (Luật bất biến)

1. **Một Product phải có ít nhất 1 Variant.** *(Không thể bán "khái niệm", phải bán một phiên bản cụ thể có mã SKU.)*
2. **Mỗi Variant phải có một `SKUCode` duy nhất trên toàn hệ thống.** *(SKUCode là chìa khóa để Inventory và Ordering tham chiếu chính xác món hàng.)*
3. **Giá bán (Price) của Variant phải > 0.** *(Không cho phép bán hàng miễn phí qua luồng mua hàng thông thường.)*
4. **Một Product phải có Tên (Name) không rỗng.**
5. **Một Product phải thuộc ít nhất 1 Category.** *(Sản phẩm không thuộc danh mục nào sẽ "vô hình" với khách hàng khi duyệt.)*

### 5.4. Commands & Events

| Command (Lệnh) | Domain Event (Sự kiện) | Ghi chú |
| :--- | :--- | :--- |
| `CreateProduct` | `ProductCreated` | Tạo sản phẩm mới kèm ít nhất 1 Variant. |
| `UpdateProductInfo` | `ProductInfoUpdated` | Thay đổi tên, mô tả, hình ảnh. |
| `AddVariant` | `VariantAdded` | Thêm biến thể mới (SKU mới) cho sản phẩm. |
| `UpdateVariantPrice` | `VariantPriceChanged` | Thay đổi giá bán của một biến thể. |
| `CreateCategory` | `CategoryCreated` | Tạo danh mục mới. |
| `AssignProductToCategory` | `ProductCategoryAssigned` | Gắn sản phẩm vào danh mục. |

---

## 6. Tactical Design: Ordering Context

> **Mục đích của Context này**: Trả lời câu hỏi *"Khách hàng muốn mua gì và thỏa thuận mua bán là gì?"*

Context này chứa **2 Aggregates riêng biệt** trong cùng 1 ranh giới: Cart (ý định) và Order (cam kết). Chúng chia sẻ ngữ cảnh nhưng tuyệt đối không dùng lẫn từ vựng của nhau.

### 6.1. Aggregates (Cụm thực thể)

#### Cart Aggregate (Ý định mua)

**Aggregate Root: `Cart`**

`Cart` (Giỏ hàng) là nơi lưu trữ ý định mua của khách hàng. Dữ liệu trong Cart mang tính tạm thời, khách có toàn quyền thay đổi bất kỳ lúc nào.

**Thành phần bên trong:**
- **`CartItem`**: Một dòng trong giỏ hàng. Chỉ chứa `SKUCode` (tham chiếu đến Variant bên Catalog) và `Quantity` (số lượng muốn mua). **Không lưu giá** — giá luôn được lấy mới nhất từ Catalog mỗi khi hiển thị. Điều này đảm bảo khách hàng luôn thấy giá chính xác nhất.

**Đặc điểm:**
- Mỗi Customer chỉ có **đúng 1 Cart** active tại một thời điểm.
- Cart có thể rỗng (vừa tạo, chưa thêm gì).
- Cart bị làm trống sau khi Checkout thành công.

#### Order Aggregate (Cam kết mua)

**Aggregate Root: `Order`**

`Order` (Đơn hàng) là một cam kết pháp lý giữa khách hàng và doanh nghiệp. Khi đơn hàng được lập, mọi thông tin đều bị **"đóng băng" (snapshot)** tại thời điểm đó và không được phép thay đổi.

**Thành phần bên trong:**
- **`OrderItem`**: Một dòng trong đơn hàng. Chứa `SKUCode`, `Quantity` và **`UnitPrice` (giá chốt cứng tại thời điểm Checkout)**. Dù Catalog thay đổi giá sau đó, OrderItem vẫn giữ nguyên giá cũ.
- **`ShippingInfo`** (Value Object): Snapshot thông tin giao hàng (Tên, SĐT, Địa chỉ) tại thời điểm đặt hàng. Nếu khách sửa địa chỉ trong hồ sơ cá nhân sau đó, đơn hàng này không bị ảnh hưởng.

**Máy trạng thái (State Machine) của Order:**
```
                    ┌──────────────────┐
                    │     PENDING      │ ← Vừa tạo
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼                             ▼
┌─────────────────────┐        ┌────────────────────────┐
│ RESERVATION_FAILED  │        │   AWAITING_PAYMENT     │
│ (Hết hàng)          │        │   (Chờ thanh toán)     │
└─────────────────────┘        └────────────┬───────────┘
                                            │
                                ┌───────────┼───────────┐
                                ▼                       ▼
                        ┌──────────┐           ┌────────────┐
                        │   PAID   │           │  CANCELED  │
                        │ (Đã trả) │           │ (Quá hạn)  │
                        └──────────┘           └────────────┘
```

### 6.2. Value Objects (Đối tượng giá trị)

| Value Object | Mô tả | Ví dụ |
| :--- | :--- | :--- |
| `SKUCode` | Mã tham chiếu đến Variant bên Catalog. Dùng chung cho cả CartItem và OrderItem. | `"AOTHUN-DO-L"` |
| `Money` | Số tiền + đơn vị tiền tệ. Dùng cho `UnitPrice`, `SubTotal`, `TotalAmount`. | `Money(100000, "VND")` |
| `Quantity` | Số nguyên dương. Đại diện cho số lượng muốn mua. | `Quantity(2)` |
| `ShippingInfo` | Snapshot thông tin giao hàng tại thời điểm đặt hàng: Tên người nhận, SĐT, Địa chỉ. | `ShippingInfo("Nguyễn Văn A", "0901234567", "123 Lê Lợi, Q1, HCM")` |
| `OrderStatus` | Trạng thái đơn hàng. Giá trị giới hạn: PENDING, RESERVATION_FAILED, AWAITING_PAYMENT, PAID, CANCELED. | `OrderStatus.PAID` |

### 6.3. Business Invariants (Luật bất biến)

**Cart:**
1. **Mỗi Customer chỉ có đúng 1 Cart active.** *(Không tồn tại 2 giỏ hàng song song cho cùng 1 người.)*
2. **Quantity của CartItem phải > 0.** *(Muốn bỏ thì xóa CartItem, không được để qty = 0.)*
3. **Không được trùng SKUCode trong cùng 1 Cart.** *(Nếu thêm SKU đã có, tăng Quantity thay vì tạo dòng mới.)*

**Order:**
4. **Một Order phải có ít nhất 1 OrderItem.** *(Không tồn tại đơn hàng rỗng.)*
5. **UnitPrice của OrderItem bất biến sau khi tạo.** *(Đây là giá chốt — không ai được sửa dù Catalog thay đổi.)*
6. **TotalAmount phải bằng tổng SubTotal của tất cả OrderItem.** *(Tính toàn vẹn số liệu.)*
7. **Trạng thái Order chỉ được chuyển đổi theo đúng State Machine.** *(Ví dụ: Không được nhảy từ CANCELED sang PAID.)*

### 6.4. Commands & Events

| Command (Lệnh) | Domain Event (Sự kiện) | Ghi chú |
| :--- | :--- | :--- |
| `AddToCart` | `CartUpdated` | Thêm hoặc tăng số lượng CartItem. |
| `RemoveFromCart` | `CartUpdated` | Xóa CartItem khỏi giỏ. |
| `UpdateCartItemQty` | `CartUpdated` | Thay đổi số lượng. |
| `PlaceOrder` | `OrderPlaced` | Chuyển Cart thành Order. Giá được chốt tại đây. |
| `ClearCart` | `CartCleared` | Làm trống giỏ hàng sau Checkout. |
| `MarkReservationFailed` | `OrderReservationFailed` | Inventory báo hết hàng. |
| `MarkAwaitingPayment` | `OrderAwaitingPayment` | Giữ chỗ kho thành công, chờ thanh toán. |
| `ConfirmPaid` | `OrderPaid` | Thanh toán thành công. |
| `CancelOrder` | `OrderCanceled` | Hủy do quá hạn hoặc khách chủ động hủy. |
---

## 7. Tactical Design: Inventory Context

> **Mục đích của Context này**: Trả lời câu hỏi *“Chúng ta thực sự còn bao nhiêu hàng trong kho?”*  
> **Giới hạn**: Không quan tâm đến mô tả sản phẩm, chỉ quản lý số lượng, giữ chỗ và các quy tắc bảo toàn tồn kho.

### 7.1. Aggregates (Cụm thực thể)

#### StockItem Aggregate
- **Aggregate Root:** `StockItem`
- **Thành phần:** `SKUCode` (VO), `QuantityOnHand`, `QuantityReserved`, `Location` (VO)
- **Behavior (phương thức):**
  - `IncreaseStock(amount)` – tăng `QuantityOnHand`.
  - `DecreaseStock(amount)` – giảm `QuantityOnHand` khi reservation được commit.
  - `Reserve(quantity)` – giảm tạm `QuantityOnHand`, tăng `QuantityReserved`.
  - `ReleaseReservation(quantity)` – trả lại `QuantityOnHand`, giảm `QuantityReserved`.
- **Business Invariants:**
  1. `QuantityOnHand >= 0`.
  2. `QuantityReserved >= 0`.
  3. `QuantityOnHand + QuantityReserved = Tổng nhập kho`.
  4. Không cho phép `DecreaseStock` khi `QuantityOnHand < amount`.

#### Reservation Aggregate
- **Aggregate Root:** `Reservation`
- **Thành phần:** `ReservationId`, `SKUCode`, `Quantity`, `ExpiresAt`, `Status` (`PENDING`, `COMMITTED`, `CANCELLED`)
- **Behavior:**
  - `Confirm()` → `Status = COMMITTED` và gọi `StockItem.DecreaseStock`.
  - `Cancel()` → `Status = CANCELLED` và gọi `StockItem.ReleaseReservation`.
  - `Expire()` (cron) → tự động chuyển thành `CANCELLED` nếu chưa `COMMITTED`.
- **Business Invariants:**
  1. `Quantity > 0`.
  2. `ExpiresAt` luôn ở tương lai khi tạo.
  3. Khi `Status = COMMITTED` thì `Quantity` đã được giảm khỏi `StockItem`.
  4. Khi chuyển trạng thái phải đồng bộ với StockItem.

### 7.2. Value Objects

| Value Object | Mô tả | Ví dụ |
| :--- | :--- | :--- |
| `SKUCode` | Mã định danh duy nhất cho một Variant. | `"AOTHUN-DO-L"` |
| `Quantity` | Số lượng nguyên dương. | `Quantity(5)` |
| `Location` | Địa điểm vật lý trong kho (Warehouse, Aisle, Shelf). | `Location("WH1","A3","S7")` |
| `ReservationId` | UID của một giữ chỗ. | `ReservationId("R-20230619-001")` |
| `ReservationStatus` | Enum: `PENDING`, `COMMITTED`, `CANCELLED`. | `ReservationStatus.PENDING` |

### 7.3. Business Invariants (Tổng hợp)
1. **Không bao giờ** `StockItem.QuantityOnHand` < 0.
2. **Không bao giờ** `StockItem.QuantityReserved` > `StockItem.QuantityOnHand` (trước khi commit).
3. Mỗi `SKUCode` chỉ có **một StockItem** duy nhất.
4. Khi `Reservation` chuyển sang `COMMITTED`, `StockItem.DecreaseStock` phải thành công; nếu không, reservation chuyển thành `CANCELLED`.
5. Khi `Reservation` hết hạn, hệ thống tự động `ReleaseReservation` (cron).

### 7.4. Commands & Events

| Command (Lệnh) | Domain Event (Sự kiện) | Ghi chú |
| :--- | :--- | :--- |
| `AddStock` | `StockAdded` | Tăng `QuantityOnHand`. |
| `RemoveStock` | `StockRemoved` | Khi xuất kho (giao hàng). |
| `CreateReservation` | `ReservationCreated` | Khi Order được tạo, tạo Reservation. |
| `ReserveStock` | `StockReserved` | Gọi `StockItem.Reserve`. |
| `ConfirmReservation` | `ReservationCommitted` | Khi thanh toán thành công, commit. |
| `CancelReservation` | `ReservationCancelled` | Khi khách hủy hoặc hết thời gian. |
| `ExpireReservation` | `ReservationExpired` | Cron tự động hủy. |

---

## 8. Tactical Design: Payment Context

> **Mục đích của Context này**: Trả lời câu hỏi *"Khách hàng đã đưa tiền chưa?"*  
> **Giới hạn**: Chỉ quan tâm đến việc ghi nhận, đối soát thanh toán. Không liên quan đến kiến trúc kho hàng hay danh mục sản phẩm.

### 8.1. Aggregates (Cụm thực thể)

#### Payment Aggregate
- **Aggregate Root**: `Payment`
- **Thành phần**: `PaymentId`, `OrderReferenceId` (VO - không phải Order ID trực tiếp, chỉ là reference), `Amount` (VO), `PaymentMethod` (VO), `Status` (VO), `TransactionId` (VO - từ cổng thanh toán)
- **Behavior (phương thức)**:
  - `InitiatePayment()` → `Status = INITIATED`. Gọi Payment Gateway để tạo transaction.
  - `ConfirmPayment(gatewayTransactionId)` → `Status = CONFIRMED`. Đối soát với cổng thanh toán thành công.
  - `RejectPayment(reason)` → `Status = REJECTED`. Cổng thanh toán từ chối.
  - `RefundPayment()` → `Status = REFUNDED`. Hoàn tiền cho khách (nếu giao hàng rồi khách hủy).
  
- **State Machine**:
  ```
  INITIATED → CONFIRMED → (REFUNDED hoặc hoàn thành)
           → REJECTED
  ```

### 8.2. Value Objects

| Value Object | Mô tả | Ví dụ |
| :--- | :--- | :--- |
| `PaymentId` | UID của một payment record. | `PaymentId("PAY-20230619-001")` |
| `OrderReferenceId` | Reference ID đến Order từ Ordering Context. **Payment không nên import/biết Order class**, chỉ cần một string identifier. | `OrderReferenceId("ORD-20230619-0001")` |
| `Amount` | Khoản tiền + đơn vị tiền tệ. | `Money(250000, "VND")` |
| `PaymentMethod` | Enum hoặc Value Object: CREDIT_CARD, BANK_TRANSFER, E_WALLET (momo, zalo pay), COD. | `PaymentMethod.CREDIT_CARD` |
| `PaymentStatus` | Enum: INITIATED, CONFIRMED, REJECTED, REFUNDED. | `PaymentStatus.CONFIRMED` |
| `TransactionId` | ID được cấp bởi cổng thanh toán bên ngoài (VNPay, Stripe, ...). Dùng để đối soát. | `TransactionId("VNP-20230619-ABC123")` |
| `GatewayErrorCode` | Mã lỗi từ cổng thanh toán nếu bị reject. | `GatewayErrorCode("ERR_INSUFFICIENT_BALANCE")` |

### 8.3. Business Invariants (Luật bất biến)

1. **Một Payment chỉ có thể tạo một lần**.  *(Một Order chỉ sinh ra một Payment record.)*
2. **Amount phải > 0**.
3. **OrderReferenceId bắt buộc phải có**.  *(Không tồn tại payment "nổi" không liên kết đến order nào.)*
4. **Status chỉ được chuyển theo quy tắc State Machine**. *(Ví dụ: Không được nhảy từ CONFIRMED sang INITIATED.)*
5. **Một khi Status = CONFIRMED, Amount bất biến**. *(Không được sửa số tiền sau khi xác nhận.)*
6. **Khi Status = CONFIRMED, phải có TransactionId từ gateway**. *(Cần để đối soát sau.)*

### 8.4. Commands & Events

| Command (Lệnh) | Domain Event (Sự kiện) | Ghi chú |
| :--- | :--- | :--- |
| `InitiatePayment` | `PaymentInitiated` | Gọi cổng thanh toán, tạo session/transaction. |
| `ConfirmPayment` | `PaymentConfirmed` | Gateway xác nhận tiền đã vào. Phát ra event để Ordering Context xử lý. |
| `RejectPayment` | `PaymentRejected` | Gateway từ chối. Ordering Context nhận event này để hủy Order. |
| `RefundPayment` | `PaymentRefunded` | Hoàn tiền cho khách. |

---

## 9. Tactical Design: User Context

> **Mục đích của Context này**: Trả lời câu hỏi *"Ai đang mua hàng? Họ muốn giao hàng đến đâu?"*  
> **Giới hạn**: Chỉ quản lý thông tin cơ bản khách hàng (Authentication không phải trách nhiệm context này — có thể sử dụng system ngoài). Trọng tâm là lưu địa chỉ giao hàng, hồ sơ cá nhân.

### 9.1. Aggregates (Cụm thực thể)

#### Customer Aggregate
- **Aggregate Root**: `Customer`
- **Thành phần**:
  - `CustomerId` (VO): UID của khách hàng.
  - `Email` (VO): Email duy nhất, dùng để đăng nhập.
  - `PhoneNumber` (VO): Số điện thoại.
  - `PersonalInfo` (VO): Tên, Ngày sinh (tuỳ chọn).
  - `ShippingAddresses` (Value Object Collection): Danh sách địa chỉ giao hàng. Mỗi Customer có thể lưu nhiều địa chỉ, có một "default" để dùng khi checkout.
  
- **Behavior (phương thức)**:
  - `RegisterCustomer(email, phone, name)` → Tạo khách hàng mới.
  - `AddShippingAddress(address)` → Thêm địa chỉ giao hàng.
  - `SetDefaultShippingAddress(addressId)` → Đặt địa chỉ mặc định.
  - `UpdatePersonalInfo(name, phone)` → Cập nhật hồ sơ (không ảnh hưởng đến Order đã tạo).

### 9.2. Value Objects

| Value Object | Mô tả | Ví dụ |
| :--- | :--- | :--- |
| `CustomerId` | UID của khách hàng. | `CustomerId("CUST-001")` |
| `Email` | Địa chỉ email duy nhất. | `Email("user@example.com")` |
| `PhoneNumber` | Số điện thoại. Có validation format (VN: 10 chữ số). | `PhoneNumber("+84901234567")` |
| `PersonalInfo` | Họ tên + Ngày sinh (tuỳ chọn). | `PersonalInfo("Nguyễn Văn A", LocalDate.of(1990, 5, 15))` |
| `ShippingAddress` | Tên người nhận, SĐT, Địa chỉ chi tiết (Số nhà, Phố, Phường, Quận, TP), Postal code. | `ShippingAddress("Nguyễn Văn A", "0901234567", "123 Lê Lợi, P. Bến Nghé, Q.1, TP.HCM", "700000")` |
| `AddressId` | UID của một địa chỉ giao hàng trong danh sách. | `AddressId("ADDR-001")` |

### 9.3. Business Invariants (Luật bất biến)

1. **Email duy nhất trên toàn hệ thống**. *(Mỗi khách đăng ký một email.)*
2. **Mỗi Customer phải có ít nhất 1 ShippingAddress sau khi đăng ký**. *(Không tồn tại khách "lơ lửng" không có địa chỉ.)*
3. **Luôn phải có một địa chỉ "default"**. *(Khi checkout, nếu không chọn, sẽ dùng địa chỉ default.)*
4. **PhoneNumber phải hợp lệ** (validate format quốc gia).
5. **Cập nhật PersonalInfo không ảnh hưởng đến Order cũ**. *(Order lưu snapshot ShippingInfo tại thời điểm đặt hàng.)*

### 9.4. Commands & Events

| Command (Lệnh) | Domain Event (Sự kiện) | Ghi chú |
| :--- | :--- | :--- |
| `RegisterCustomer` | `CustomerRegistered` | Tạo khách hàng mới (có thể gắn liền với Auth Service tạo account). |
| `AddShippingAddress` | `ShippingAddressAdded` | Thêm địa chỉ mới vào danh sách. |
| `SetDefaultAddress` | `DefaultAddressChanged` | Thay đổi địa chỉ mặc định. |
| `UpdatePersonalInfo` | `CustomerInfoUpdated` | Cập nhật tên, SĐT. |

---

## 10. Tóm tắt Inter-Context Communication (Giao tiếp giữa các Contexts)

Dưới đây là cách các Context tương tác với nhau thông qua Domain Events:

```
┌────────────────┐
│   Ordering     │ 
│   Context      │
└───────┬────────┘
        │
        │ PlaceOrder Command
        │   ↓
        │ OrderPlaced Event
        │   ↓
        ├─────────→ Inventory Context: CreateReservation
        │              │
        │              └─→ ReservationCreated Event
        │
        │ PaymentConfirmed Event (from Payment)
        │   ↓
        └─────────→ Inventory Context: ConfirmReservation
                         │
                         └─→ ReservationCommitted Event
                              (Inventory khấu trừ kho vĩnh viễn)
```

**Quy tắc giao tiếp**:
1. **Ordering Context** là "trái tim" — nó orchestrate (điều phối) luồng xử lý.
2. **Inventory Context** lắng nghe `OrderPlaced` từ Ordering, tạo Reservation.
3. **Payment Context** lắng nghe `OrderAwaitingPayment` từ Ordering, tạo Payment record.
4. Khi `PaymentConfirmed` từ Payment, Ordering chuyển sang trạng thái `PAID` và báo cho Inventory commit reservation.
5. Khi `ReservationExpired` từ Inventory (hết thời hạn thanh toán), Ordering nhận event này để tự động cancel Order.
6. **User Context** đóng vai trò "reference data" — Ordering không phải subscribe events từ User, chỉ lấy thông tin khi cần qua API hoặc query.

---

## 11. Glossary (Từ điển Ubiquitous Language)

| Thuật ngữ | Định nghĩa | Ngữ cảnh sử dụng |
| :--- | :--- | :--- |
| **Product** | Một mặt hàng mà doanh nghiệp muốn bán, có một hoặc nhiều biến thể. | Catalog Context |
| **Variant (SKU)** | Một phiên bản cụ thể có thể mua được của một Product (ví dụ: Áo Đỏ size L). Mang một mã SKUCode duy nhất. | Catalog, Ordering, Inventory |
| **Category** | Nhóm phân loại sản phẩm, hỗ trợ duyệt hàng. | Catalog Context |
| **Cart** | Giỏ hàng lưu ý định mua (tạm thời, có thể thay đổi). | Ordering Context |
| **Order** | Đơn hàng chính thức, là cam kết pháp lý (bất biến sau khi tạo). | Ordering Context |
| **Reservation** | Cục hàng được "cất riêng" trong kho để phục vụ một Order cụ thể, có thời hạn. | Inventory Context |
| **StockItem** | Kỷ lục tồn kho của một SKU. Ghi nhận `QuantityOnHand`, `QuantityReserved`. | Inventory Context |
| **Payment** | Ghi chép về một giao dịch thanh toán liên kết với một Order. | Payment Context |
| **Customer** | Khách hàng, lưu thông tin cá nhân và danh sách địa chỉ. | User Context |
| **ShippingAddress** | Địa chỉ giao hàng của khách. Snapshot vào Order tại thời điểm Checkout. | User Context, Ordering Context |

---

## 12. Non-Functional Requirements by Context

Các yêu cầu phi chức năng quan trọng cho mỗi Context:

| Context | NFR | Lý do |
| :--- | :--- | :--- |
| **Catalog** | Read-heavy (CQS), caching dữ liệu. | Khách liên tục lướt danh mục, ít thay đổi. |
| **Inventory** | **Strong Consistency** (ACID). | Không được tính sai số lượng tồn kho — tuyệt đối chính xác. |
| **Ordering** | **Eventual Consistency** giữa Ordering & Inventory. | Ordering không khoá Inventory, chỉ gửi event; Inventory xử lý asynchronously. |
| **Payment** | Có **Event Sourcing / Audit Trail** để trace mọi giao dịch. | Cần để kiểm toán và điều tra tranh chấp. |
| **User** | Có caching. Dữ liệu không thay đổi thường xuyên. | Tăng tốc độ checkout. |