---
name: manage-text
description: "Kích hoạt hệ thống quản lý văn bản AI — điều phối team agent chuyên biệt để tạo, chỉnh sửa, phân tích, tổ chức và kiểm tra chất lượng tài liệu. Sử dụng khi: bất kỳ yêu cầu nào liên quan đến văn bản (tạo mới, sửa, phân tích, tóm tắt, phân loại, kiểm tra). Trigger: 'quản lý văn bản', 'tạo tài liệu', 'viết văn bản', 'chỉnh sửa tài liệu', 'phân tích văn bản', 'tổ chức tài liệu', 'kiểm tra văn bản', 'tóm tắt tài liệu', 'manage text', 'manage document'."
---

# Manage Text — Hệ thống quản lý văn bản AI

Skill này kích hoạt và điều phối toàn bộ hệ thống quản lý văn bản với team 5 agent chuyên biệt.

## Kiến trúc hệ thống

```
Người dùng
    ↓
text-manager (Orchestrator/Supervisor)
    ├── text-researcher  → Nghiên cứu & phân tích
    ├── text-writer      → Sáng tác nội dung mới
    ├── text-editor      → Biên tập & cải thiện
    ├── text-organizer   → Tổ chức & phân loại
    └── text-reviewer    → Kiểm tra chất lượng
```

**Pattern:** Supervisor — `text-manager` nhận yêu cầu, phân tích, điều phối pipeline phù hợp.

## Pipeline theo tác vụ

| Tác vụ | Pipeline |
|--------|---------|
| Nghiên cứu/phân tích tài liệu | researcher → (organizer) → báo cáo |
| Tạo văn bản mới | researcher → writer → reviewer → editor |
| Chỉnh sửa văn bản | researcher → editor → reviewer |
| Tổ chức bộ tài liệu | researcher → organizer → reviewer |
| Kiểm tra chất lượng | reviewer → (editor nếu cần sửa) |
| Tóm tắt | researcher → writer (tóm tắt) → reviewer |

## Bắt đầu

Khi skill này được kích hoạt, hãy:

1. **Khởi động text-manager** bằng cách gọi agent `text-manager` với yêu cầu của người dùng.
2. **text-manager** sẽ tự phân tích và điều phối team phù hợp.
3. **Workspace:** Mọi kết quả trung gian và cuối lưu tại `_workspace/text-management/`.

## Lệnh gọi nhanh

```
Phân tích tài liệu:  → text-researcher
Viết tài liệu mới:   → text-manager (writer + reviewer)
Chỉnh sửa:          → text-editor
Tổ chức tài liệu:   → text-organizer
Kiểm tra chất lượng: → text-reviewer
Toàn bộ workflow:    → text-manager
```

## File đầu ra

Các file được lưu tại `_workspace/text-management/` theo quy ước:
- `{nn}_research_{topic}.md` — Kết quả nghiên cứu
- `{nn}_draft_{topic}.md` — Draft văn bản mới
- `{nn}_edited_{topic}.md` — Văn bản sau biên tập
- `{nn}_organized_{topic}.md` — Bộ tài liệu đã tổ chức
- `{nn}_review_{topic}.md` — Báo cáo review
- `{nn}_final_{topic}.md` — Kết quả cuối cùng
