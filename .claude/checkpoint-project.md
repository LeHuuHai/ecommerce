# Checkpoint tiến độ toàn dự án

## 2026-06-18
- Khởi tạo bộ tài liệu `.claude` theo yêu cầu.
- Xác định kiến trúc tổng quan là Microservices và định hình 4 nhóm nghiệp vụ chính: User, Inventory, Product/Catalog, Sales (Order & Payment).
- Thống nhất stack công nghệ chính: Go, PostgreSQL (GORM), Kafka (segmentio/kafka-go).
- Hoàn thành phân tích miền nghiệp vụ (Domain & Subdomains) và lưu trữ trong [domain-analysis.md](file:///d:/Work%20Space/ecommerce/.claude/domain-analysis.md).
- Hoàn thành thiết kế chi tiết và thống nhất các quyết định kiến trúc cho **User Bounded Context** tại [user.md](file:///d:/Work%20Space/ecommerce/.claude/modules/user.md).
- Thiết lập ranh giới và mô hình miền chi tiết cho **Product Catalog Bounded Context** tại [product.md](file:///d:/Work%20Space/ecommerce/.claude/modules/product.md).
- Thiết lập ranh giới và mô hình miền chi tiết cho **Inventory Bounded Context** tại [inventory.md](file:///d:/Work%20Space/ecommerce/.claude/modules/inventory.md).
- Trạng thái repo: workspace đang trống, chưa phải git repository, chưa có source code.
- Chuẩn hóa tài liệu `.claude` sang tiếng Việt có dấu.

## Việc cần làm tiếp
- Thiết lập các Bounded Context tiếp theo: Ordering, Payment.
- Phân tích Context Mapping và ranh giới tương tác giữa các Bounded Context.
- Quyết định phương án phân rã service (gộp chung Sales hay tách riêng Order & Payment).
- Lựa chọn cấu trúc repository (Mono-repo vs Multi-repo).
- Thiết kế kiến trúc tổng thể (API Gateway, communication protocol như REST/gRPC, v.v.).
- Tạo cấu trúc thư mục và khởi tạo Git repository cho dự án.
