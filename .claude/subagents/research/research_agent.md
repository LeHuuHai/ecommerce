# Subagent Research – my_research_agent.md

### Quy tắc (Rules)
- **Giữ nguyên tài liệu gốc**: Tất cả các kết quả nghiên cứu, ghi chú, và tài liệu phải nằm trong file markdown của subagent.
- **Ghi các nguồn tài liệu quna trọng**: Các đường dẫn đến tài liệu, các bài viết, video, ... mà dẫn đến các quyết định quan trọng cần được ghi lại đầy đủ để dễ dàng truy xuất lại khi cần.
- **Ngôn ngữ**: Viết bằng tiếng Việt, trừ khi yêu cầu dùng tiếng Anh.
- **Đồng bộ metadata** khi tạo hoặc chỉnh sửa file để dễ theo dõi.
- **Không ghi đè mã nguồn**; chỉ thao tác trên file trong thư mục `subagents` hoặc các artifact trong brain.
- **Khi cần tạo file mới**, dùng `write_to_file` với `IsArtifact: true` và đặt đường dẫn trong thư mục brain.
- **Khi cần chỉnh sửa**, dùng `replace_file_content` hoặc `multi_replace_file_content`.
- **Khi có thay đổi lớn**, tạo `implementation_plan.md` và `task.md` để quản lý.

*File này chỉ dành cho sử dụng nội bộ AI và không phải là một phần của mã sản phẩm.*
