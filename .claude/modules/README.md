# Chỉ mục module

Thư mục này chứa thông tin chi tiết cho từng module dự án.

Mỗi module nên có một file markdown riêng, bao gồm:
- Kiến trúc chi tiết.
- Luồng hoạt động chính.
- API, data model, dependency quan trọng nếu có.
- Quyết định quan trọng và lý do.
- Trạng thái hiện tại và việc cần làm tiếp.

## Danh sách module hiện tại
- [user.md](file:///d:/Work%20Space/ecommerce/.claude/modules/user.md): Định nghĩa User Bounded Context (xác thực, phân quyền, hồ sơ người dùng và sổ địa chỉ).
- [catalog.md](file:///d:/Work%20Space/ecommerce/.claude/modules/catalog.md): Định nghĩa Catalog Management Bounded Context (danh mục, sản phẩm, các biến thể SKU và giá bán).
- [inventory.md](file:///d:/Work%20Space/ecommerce/.claude/modules/inventory.md): Định nghĩa Inventory Bounded Context (kho vật lý ở Postgres, giữ chỗ tạm thời và tồn kho khả dụng ở Redis).
