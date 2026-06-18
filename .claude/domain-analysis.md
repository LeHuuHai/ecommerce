# Phân tích Miền nghiệp vụ (Domain Analysis) & Subdomains

Tài liệu này phân tích chi tiết các năng lực nghiệp vụ và phân loại chúng thành các Subdomain (Miền con) theo phương pháp Domain-Driven Design (DDD) làm cơ sở để phân rã hệ thống thành các Microservices.

---

## 1. Bản đồ Năng lực Nghiệp vụ (Business Capabilities)

Năng lực nghiệp vụ mô tả những gì hệ thống cần làm để đáp ứng nhu cầu kinh doanh:

*   **Quản lý người dùng (User Management)**:
    *   Xác thực (Authentication) & Ủy quyền (Authorization) truy cập hệ thống.
    *   Quản lý thông tin hồ sơ khách hàng (Profile, thông tin liên hệ, tùy chọn).
    *   Quản lý sổ địa chỉ giao hàng (Delivery Address Book) phục vụ quá trình đặt hàng.
    *   Phân quyền người dùng (Khách hàng, Admin, Thủ kho, Nhân viên bán hàng).
*   **Quản lý danh mục mặt hàng (Catalog Management)**:
    *   Quản lý cây danh mục sản phẩm (Categories) và thương hiệu (Brands).
    *   Quản lý thông tin chi tiết sản phẩm (Mô tả, hình ảnh, thông số kỹ thuật, biến thể sản phẩm như kích thước, màu sắc).
    *   Quản lý giá bán (Giá niêm yết, giá khuyến mãi).
*   **Quản lý kho hàng (Inventory Management)**:
    *   Quản lý số lượng tồn kho (Stock quantity) của từng mặt hàng tại các kho.
    *   Quy trình nhập kho (Stock-in) và xuất kho (Stock-out).
    *   Cơ chế giữ chỗ kho (Stock Reservation) tạm thời khi đơn hàng được tạo, nhằm tránh tình trạng bán vượt quá tồn kho thực tế (overselling).
*   **Quản lý bán hàng (Sales Management)**:
    *   **Đặt hàng (Ordering)**: Quản lý giỏ hàng (Shopping Cart), tạo đơn hàng (Order creation), tính toán giá trị đơn hàng (áp dụng thuế, phí vận chuyển), và quản lý vòng đời trạng thái đơn hàng (Chờ thanh toán -> Đang xử lý -> Đang giao -> Hoàn thành / Hủy).
    *   **Thanh toán (Payment)**: Tích hợp và xử lý giao dịch tiền tệ với các cổng thanh toán bên thứ ba, theo dõi trạng thái giao dịch (Thành công, Thất bại, Hoàn tiền).

---

## 2. Phân loại Subdomain (Miền con)

Chúng ta phân loại các năng lực nghiệp vụ trên thành các Subdomain để áp dụng chiến lược phát triển và đầu tư nguồn lực phù hợp:

### 2.1. Core Domain (Miền cốt lõi)
Đây là phần quan trọng nhất, tạo ra sự khác biệt và lợi thế cạnh tranh của sản phẩm. Cần đầu tư tối đa tài nguyên và viết custom code chất lượng cao nhất ở đây.
*   **Sales & Ordering (Đặt hàng & Xử lý đơn hàng)**:
    *   *Lý do phân loại*: Luồng đặt hàng, tính toán giá trị, quản lý trạng thái đơn hàng và các luật nghiệp vụ (business rules) liên quan đến mua bán chính là cốt lõi tạo ra doanh thu và trải nghiệm khách hàng cho hệ thống ecommerce.

### 2.2. Supporting Domain (Miền hỗ trợ)
Đây là các phần nghiệp vụ bắt buộc phải có, mang tính chất đặc thù của doanh nghiệp nhưng không trực tiếp tạo ra lợi thế cạnh tranh cốt lõi. Cần tự xây dựng (custom-built) nhưng mô hình thiết kế có thể đơn giản hơn Core Domain.
*   **User Management (Quản lý người dùng)**:
    *   *Lý do phân loại*: Không chỉ là xác thực đăng nhập thông thường (Authentication), quản lý người dùng ở đây gắn liền với hồ sơ khách hàng, phân quyền nội bộ (Admin/Thủ kho) và quản lý sổ địa chỉ giao hàng tương tác trực tiếp với luồng checkout đơn hàng.
*   **Catalog Management (Quản lý danh mục mặt hàng)**:
    *   *Lý do phân loại*: Cung cấp cấu trúc dữ liệu sản phẩm hỗ trợ hiển thị trên storefront. Nghiệp vụ lưu trữ danh mục sản phẩm, biến thể và thuộc tính tương đối phức tạp và mang đặc thù của ngành hàng ecommerce.
*   **Inventory Management (Quản lý kho hàng)**:
    *   *Lý do phân loại*: Hỗ trợ việc lưu trữ và vận hành logistics. Cơ chế giữ chỗ kho (reservation) và đồng bộ số lượng tồn kho với các kênh bán hàng là cầu nối quan trọng hỗ trợ cho Core Domain (Sales).

### 2.3. Generic Domain (Miền phổ thông)
Đây là các bài toán tiêu chuẩn, không mang đặc thù doanh nghiệp, có thể giải quyết bằng các giải pháp hoặc dịch vụ bên thứ ba đã có sẵn trên thị trường.
*   **Payment Integration (Tích hợp thanh toán)**:
    *   *Lý do phân loại*: Nghiệp vụ thanh toán thực tế được thực hiện bởi các cổng thanh toán (Momo, VNPay, Stripe). Hệ thống của chúng ta chỉ thực hiện tích hợp API để gửi yêu cầu thanh toán và nhận webhook callback xác nhận trạng thái.

---

## 3. Các bước tiếp theo
1.  **Xác định Bounded Contexts**: Phác thảo ranh giới dữ liệu và hành vi cho từng miền con.
2.  **Context Mapping**: Thiết kế cách thức các miền con giao tiếp với nhau (Đồng bộ qua REST/gRPC hay Bất đồng bộ qua Kafka).
3.  **Decomposition**: Xác định danh sách các Microservices chính thức và cơ sở dữ liệu tương ứng.
