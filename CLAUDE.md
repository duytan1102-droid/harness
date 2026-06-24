# CLAUDE.md

## Agent: text-researcher

**Mục tiêu:** Nghiên cứu và phân tích văn bản — đọc hiểu, tóm tắt, trích xuất ý chính, đối chiếu nhiều nguồn, trả lời câu hỏi dựa trên tài liệu, luôn kèm trích dẫn truy được về nguồn.

**Trigger:** Khi người dùng cần đọc/phân tích/tóm tắt một tài liệu (PDF, txt, md, docx, bài viết, hợp đồng, báo cáo…), hãy giao việc cho agent `text-researcher`. Câu hỏi đơn giản có thể trả lời trực tiếp.

**Vị trí định nghĩa:**
- Bản dùng được trong session: `.claude/agents/text-researcher.md` (lưu ý: `.claude/` bị `.gitignore`)
- Bản đã commit (để copy sang dự án khác): `examples/agents/text-researcher.md`

**Đặc điểm:** Format sub-agent chuẩn của Claude Code. Công cụ: `Read, Glob, Grep, WebFetch, WebSearch` (chỉ đọc, không ghi file). Model: `opus`. Đầu ra mặc định: TL;DR → luận điểm chính kèm vị trí trích dẫn → dẫn chứng/số liệu → khoảng trống/lưu ý → bảng đối chiếu (khi nhiều nguồn).

**Lịch sử thay đổi:**
| Ngày | Thay đổi | Đối tượng | Lý do |
|------|----------|-----------|-------|
| 2026-06-24 | Tạo agent nghiên cứu văn bản | examples/agents/text-researcher.md | Yêu cầu người dùng |
| 2026-06-24 | Chuyển sang format sub-agent chuẩn Claude Code (thêm tools, bỏ phần team) | examples/agents/text-researcher.md | Dùng trong Claude Code CLI/IDE |
