# Kiến trúc dự án

## Trạng thái hiện tại
Dự án mới, workspace đang trống. Chưa có source code, framework, module, database schema, hay cấu trúc thư mục ứng dụng.

## Mục tiêu dự án
Xây dựng hệ thống ecommerce áp dụng kiến trúc Microservices với các năng lực nghiệp vụ cốt lõi:
- **Quản lý người dùng** (User Management)
- **Quản lý kho hàng** (Inventory Management)
- **Quản lý danh mục mặt hàng** (Catalog Management)
- **Quản lý bán hàng** (Sales Management): Đặt hàng (Order) và Thanh toán (Payment)

## Nguyên tắc làm việc của agent
- Không tự ý sửa code khi chưa có yêu cầu rõ ràng.
- Trước khi suy luận, lập kế hoạch, hoặc code, phải đọc lại các file markdown liên quan trong `.claude`.
- Mọi quyết định kiến trúc quan trọng cần được ghi lại kèm lý do.
- Cập nhật checkpoint khi hoàn thành một mốc công việc hoặc khi kết thúc ngày làm việc.
- Tài liệu trong `.claude` viết bằng tiếng Việt có dấu, trừ khi thuật ngữ kỹ thuật cần giữ nguyên tiếng Anh.
- **Chủ động phản biện**: Phải phân tích sâu sắc, thẳng thắn phản biện và chỉ ra các điểm mâu thuẫn, kẽ hở logic, hoặc rủi ro kiến trúc trong các ý kiến/yêu cầu của người dùng để cùng thảo luận giải pháp tối ưu nhất (tránh việc chỉ gật đầu đồng ý mà không phân tích rủi ro).

## Công nghệ sử dụng
- **Ngôn ngữ lập trình**: Go (Golang)
- **Cơ sở dữ liệu**: PostgreSQL
- **Caching & High-speed Storage**: Redis (sử dụng để lưu trữ dữ liệu giữ chỗ kho và tồn kho khả dụng)
- **ORM**: GORM (Go Object Relational Mapping)
- **Message Broker**: Kafka (sử dụng thư viện của `segmentio` - `segmentio/kafka-go`)
- **Các công nghệ khác**: Sẽ quyết định và bổ sung sau khi có nhu cầu cụ thể.

## Kiến trúc tổng quan
- **Mô hình**: Microservices.
- **Phân rã service**: Hệ thống được phân rã thành các dịch vụ độc lập dựa trên kết quả phân tích miền nghiệp vụ trong [domain-analysis.md](file:///d:/Work%20Space/ecommerce/.claude/domain-analysis.md). Chi tiết mỗi service khi được thiết kế hoặc tạo ra sẽ lưu trữ tại thư mục `.claude/modules/`.

## Cấu trúc tài liệu
- `project.md`: kiến trúc tổng quan, mục tiêu, công nghệ, nguyên tắc làm việc.
- `domain-analysis.md`: phân tích chi tiết miền nghiệp vụ (Domain) và phân loại Subdomain.
- `modules/`: mỗi file markdown mô tả một module của dự án.
- `checkpoint-project.md`: checkpoint tiến độ toàn dự án.
- `checkpoint-daily.md`: checkpoint hằng ngày.
