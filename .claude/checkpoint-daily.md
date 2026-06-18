# Checkpoint hằng ngày

## 2026-06-18
### Đã làm
- Tạo cấu trúc `.claude` để lưu bộ nhớ làm việc của agent.
- Ghi nhận nguyên tắc: không tự ý sửa code; đọc lại markdown liên quan trước khi suy luận.
- Chuyển nội dung tài liệu `.claude` sang tiếng Việt có dấu.
- Xác định kiến trúc dự án (Microservices) và các phân hệ nghiệp vụ chính (User, Inventory, Product/Catalog, Sales/Order/Payment).
- Thống nhất stack công nghệ chính: Go, PostgreSQL (GORM), Kafka (segmentio/kafka-go).
- Phân tích miền nghiệp vụ và phân loại Subdomain (đưa User Management lên Supporting Domain). Lưu vào tài liệu [domain-analysis.md](file:///d:/Work%20Space/ecommerce/.claude/domain-analysis.md).
- Bổ sung nguyên tắc "Chủ động phản biện" vào luật làm việc của agent trong `project.md`.
- Phác thảo ranh giới và mô hình miền cho **User Bounded Context**, lưu trữ trong [user.md](file:///d:/Work%20Space/ecommerce/.claude/modules/user.md).
- Thảo luận phản biện và hoàn thành thống nhất thiết kế chi tiết cho User Bounded Context (sử dụng Khóa ngoại cho danh sách địa chỉ, sử dụng `AccountID` làm khóa chính thay vì `email` để đảm bảo hiệu năng và tính linh hoạt).
- Cập nhật tài liệu kiến trúc trong `project.md` và `modules/README.md`.
- Hoàn thành thiết kế chi tiết cho **Product Catalog Bounded Context** tại [product.md](file:///d:/Work%20Space/ecommerce/.claude/modules/product.md).
- Hoàn thành thiết kế chi tiết cho **Inventory Bounded Context** (với PostgreSQL làm nguồn tồn kho vật lý và Redis làm nguồn giữ chỗ/tồn kho khả dụng) tại [inventory.md](file:///d:/Work%20Space/ecommerce/.claude/modules/inventory.md).

### Đang làm
- Thiết kế ranh giới và mô hình miền cho **Ordering Bounded Context** (Đặt hàng).

### Cần tiếp tục
- Phân tích Context Mapping (Các service giao tiếp với nhau thế nào).
- Lựa chọn cấu trúc repository (Mono-repo vs Multi-repo) dựa trên ranh giới service.
- Thiết kế chi tiết từng service và cập nhật tài liệu vào `.claude/modules/`.

### Ghi chú
- Dự án đang ở pha phân tích nghiệp vụ chi tiết trước khi tiến hành tạo cấu trúc thư mục code.
