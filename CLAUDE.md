# Harness — AI Text Management System

Dự án này chứa hệ thống harness cho Claude Code, bao gồm agent team quản lý văn bản.

## Hệ thống quản lý văn bản (Text Management)

### Khi nào kích hoạt
Bất kỳ yêu cầu nào liên quan đến văn bản/tài liệu — tạo, chỉnh sửa, phân tích, tổ chức, kiểm tra:
- "quản lý văn bản", "tạo tài liệu", "viết văn bản"
- "phân tích tài liệu", "tóm tắt", "nghiên cứu văn bản"
- "tổ chức tài liệu", "phân loại tài liệu"
- "kiểm tra chất lượng", "review tài liệu"

→ Kích hoạt skill `/manage-text` hoặc gọi trực tiếp agent `text-manager`.

### Agents
| Agent | File | Vai trò |
|-------|------|---------|
| text-manager | `.claude/agents/text-manager.md` | Orchestrator trung tâm |
| text-researcher | `.claude/agents/text-researcher.md` | Nghiên cứu & phân tích |
| text-writer | `.claude/agents/text-writer.md` | Sáng tác nội dung mới |
| text-editor | `.claude/agents/text-editor.md` | Biên tập & cải thiện |
| text-organizer | `.claude/agents/text-organizer.md` | Tổ chức & phân loại |
| text-reviewer | `.claude/agents/text-reviewer.md` | Kiểm tra chất lượng |

### Workspace
Kết quả lưu tại `_workspace/text-management/`.

## Lịch sử thay đổi
- 2026-06-30: Khởi tạo hệ thống quản lý văn bản — 5 agent + 1 skill + orchestrator
