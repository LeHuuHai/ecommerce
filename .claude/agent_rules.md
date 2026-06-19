# Agent Entry Point & Core Rules

Đây là file quy định các nguyên tắc cốt lõi (Core Rules) mà AI Agent **phải luôn tuân thủ** trong suốt vòng đời dự án. Hãy đọc file này đầu tiên khi được yêu cầu kiểm tra luật (rules) của dự án.

## 1. Nguyên tắc Thao tác mã nguồn
- **Không tự ý sửa code**: Tuyệt đối không thay đổi mã nguồn hoặc cấu trúc dự án nếu chưa có yêu cầu và sự cho phép rõ ràng từ người dùng.
- **Giới hạn phạm vi (đối với Research)**: Các subagent phục vụ nghiên cứu chỉ được phép thao tác (tạo/sửa/xóa) trên các file trong thư mục `.claude/subagents/` hoặc tạo artifact, tuyệt đối **không** ghi đè mã nguồn sản phẩm.

## 2. Nguyên tắc Thiết kế & Kiến trúc
- **Đọc trước khi làm**: Trước khi đưa ra suy luận, lập kế hoạch hay viết code, bắt buộc phải đọc lại các file tài liệu liên quan trong `.claude/` (như `project.md`, `domain-analysis.md`, tài liệu trong `modules/`) để bám sát ngữ cảnh dự án.
- **Lưu vết mọi quyết định**: Bất kỳ quyết định nào liên quan đến kiến trúc, thiết kế hệ thống, công nghệ sử dụng đều phải được ghi chép lại đầy đủ kèm theo lý do cụ thể vào các file tài liệu tương ứng.
- **Lưu nguồn tham khảo**: Các tài liệu, bài viết, video, repo... quan trọng được sử dụng làm cơ sở cho quyết định thiết kế cần được dẫn link đầy đủ để dễ dàng tra cứu lại.

## 3. Quy trình Báo cáo & Cập nhật
- **Cập nhật Checkpoint**: Chủ động ghi lại tiến độ công việc, các chi tiết đáng lưu ý và các quyết định quan trọng và lý do vào các file checkpoint (`.claude/checkpoint/checkpoint-daily.md` và `.claude/checkpoint/checkpoint-project.md`) khi hoàn thành một mốc quan trọng hoặc khi kết thúc phiên làm việc.
- **Đồng bộ Metadata**: Khi tạo hoặc chỉnh sửa file tài liệu (Markdown), cần đồng bộ metadata (ngày giờ, người/agent sửa...) để theo dõi dễ dàng.

## 4. Tư duy Phản biện (Critical Thinking) - MỤC QUAN TRỌNG NHẤT
- **Không chỉ "gật đầu"**: Tuyệt đối không tự động đồng ý với mọi ý kiến hay giải pháp của người dùng mà không qua suy nghĩ.
- **Phân tích rủi ro**: Phải thẳng thắn phân tích, chỉ ra các điểm mâu thuẫn, kẽ hở logic, rủi ro bảo mật hoặc điểm chưa tối ưu trong kiến trúc từ các yêu cầu được đưa ra.
- **Đề xuất giải pháp**: Cùng người dùng thảo luận để tìm ra giải pháp tối ưu nhất, thay vì mù quáng làm theo một hướng dẫn có thể dẫn đến nợ kỹ thuật (technical debt) sau này.

## 5. Ngôn ngữ giao tiếp & Tài liệu
- **Sử dụng Tiếng Việt**: Toàn bộ tài liệu viết trong thư mục `.claude/` và các trao đổi với người dùng phải được viết bằng tiếng Việt (có dấu), rõ ràng, mạch lạc.
- **Thuật ngữ chuyên ngành**: Có thể giữ nguyên gốc tiếng Anh đối với các thuật ngữ kỹ thuật chuyên ngành để đảm bảo tính chính xác (ví dụ: Microservices, Context Mapping, Redis, Broker...).

## 6. Sơ đồ Tài liệu & Thứ tự đọc (Document Map)
Để hiểu rõ toàn cảnh dự án, tôi (và các Agent khác) cần đọc các tài liệu sau theo thứ tự và ngữ cảnh tương ứng:

### 📖 Đọc đầu tiên (Core Project Context)
- [project.md](file:///c:/Users/hailh22/WorkSpace/ecommerce/.claude/project/project.md): Kiến trúc tổng quan, mục tiêu dự án, công nghệ sử dụng và nguyên tắc làm việc.
- [domain-analysis.md](file:///c:/Users/hailh22/WorkSpace/ecommerce/.claude/project/domain-analysis.md): Phân tích chi tiết miền nghiệp vụ (Domain-Driven Design) và phân rã các Subdomains.

### 🧩 Đọc khi làm việc với từng Module cụ thể (Modules)
- [README.md (Modules)](file:///c:/Users/hailh22/WorkSpace/ecommerce/.claude/modules/README.md): Tổng quan về thư mục modules.
- **Lưu ý**: Các file module cụ thể (catalog.md, inventory.md, user.md...) sẽ được tạo sau khi bắt đầu code, chứa kiến trúc chi tiết và plan triển khai từng module.

### ⏱️ Đọc để nắm tiến độ hiện tại (Checkpoints)
- [checkpoint-project.md](file:///c:/Users/hailh22/WorkSpace/ecommerce/.claude/checkpoint/checkpoint-project.md): Tiến độ tổng quan của toàn bộ dự án.
- [checkpoint-daily.md](file:///c:/Users/hailh22/WorkSpace/ecommerce/.claude/checkpoint/checkpoint-daily.md): Tiến độ công việc hằng ngày và các task đang dang dở.

### 🔍 Đọc khi thực hiện nghiên cứu (Subagents)
- [research_agent.md](file:///c:/Users/hailh22/WorkSpace/ecommerce/.claude/subagents/research/research_agent.md): Quy định riêng cho hoạt động nghiên cứu.
- [reservation_flow_research.md](file:///c:/Users/hailh22/WorkSpace/ecommerce/.claude/subagents/research/reservation_flow_research.md): Các nghiên cứu về luồng giữ chỗ kho hàng.
