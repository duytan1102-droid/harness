---
name: second-brain
description: "Dựng và truy vấn 'bộ não thứ hai' từ một kho tài liệu Google Drive. Điều phối đội agent (corpus-librarian → text-researcher song song → knowledge-synthesizer) để quét toàn bộ thư mục, phân tích từng tài liệu và tổng hợp tri thức đối chiếu chéo. Dùng khi người dùng nói: 'nghiên cứu toàn bộ văn bản/tài liệu', 'dựng bộ não thứ hai', 'quét cả thư mục Drive', 'tổng hợp cả kho', 'có mâu thuẫn gì giữa các tài liệu', và cả các yêu cầu tiếp theo: 'cập nhật bộ não', 'thêm tài liệu mới', 'hỏi lại dựa trên kho', 'nghiên cứu lại'."
---

# Second Brain — Orchestrator nghiên cứu cả kho tài liệu

Điều phối một đội agent để biến một thư mục Google Drive thành "bộ não thứ hai" — tri thức có cấu trúc, truy vấn được, mọi câu trả lời truy ngược về tài liệu nguồn.

## Đội ngũ & vai trò

| Agent | Vai trò | Mode |
|-------|---------|------|
| `corpus-librarian` | Quét Drive, lập danh mục toàn bộ tài liệu | Sub-agent đơn |
| `text-researcher` | Phân tích sâu TỪNG tài liệu (tái sử dụng) | Sub-agent, **song song (fan-out)** |
| `knowledge-synthesizer` | Đối chiếu chéo, dựng bộ não, trả lời truy vấn | Sub-agent đơn |

**Mode tổng thể: hybrid pipeline** — đơn → fan-out song song → đơn. Mỗi Agent gọi với `model: "opus"`.

## Phase 0: Xác định ngữ cảnh

Trước khi chạy, xác định đây là lần đầu hay tiếp theo:
- `_workspace/brain/` **chưa có** → **Dựng mới** (chạy Phase 1→4).
- `_workspace/brain/` **đã có** + người dùng thêm tài liệu mới → **Cập nhật tăng trưởng** (chỉ chạy librarian cho file mới + text-researcher cho file mới + synthesizer hợp nhất lại).
- `_workspace/brain/` **đã có** + người dùng chỉ hỏi → **Chế độ truy vấn** (bỏ qua Phase 1-2, gọi thẳng `knowledge-synthesizer` để trả lời).

Hỏi người dùng đường dẫn/ID thư mục Drive nếu chưa cung cấp.

## Phase 1: Lập danh mục (corpus-librarian)

Gọi `corpus-librarian` với thư mục Drive mục tiêu. Nó xuất `_workspace/00_librarian_catalog.md` và trả về danh sách mã `Dxx` + Drive ID đọc được.

> Nếu librarian báo không kết nối được Drive → dừng, yêu cầu người dùng bật MCP Google Drive. Không tiếp tục với danh mục rỗng.

## Phase 2: Phân tích song song (text-researcher × N — fan-out)

Với mỗi tài liệu đọc được trong catalog, spawn **một** `text-researcher` chạy nền song song:

- Gọi `Agent(subagent_type="text-researcher", model="opus", run_in_background=true)` cho từng tài liệu.
- Yêu cầu mỗi instance đọc tài liệu và **trả về** bản phân tích theo định dạng mặc định của text-researcher (TL;DR → luận điểm + trích dẫn → số liệu → lưu ý).
- **Kiểm soát lô:** kho lớn thì chạy theo lô 5–8 tài liệu/lần để tránh quá tải; thu kết quả từng lô rồi tiếp lô sau.

> Fan-out vì các tài liệu độc lập, phân tích song song nhanh hơn nhiều. Mỗi instance chỉ lo đúng một tài liệu.

> **Quan trọng — text-researcher là agent CHỈ ĐỌC** (tools: Read, Glob, Grep, WebFetch, WebSearch; không có Write). Vì vậy nó **không tự ghi** `_workspace/docs/{Dxx}.md` — nó **trả bản phân tích về orchestrator**, và **orchestrator ghi file** (return-value based). Đừng giao việc ghi cho agent không có quyền ghi. (Cách khác: cấp thêm `Write` cho một bản text-researcher chuyên dùng cho corpus, nhưng giữ bản gốc chỉ-đọc để an toàn khi phân tích file lẻ.)

> **Đọc Google Drive trong fan-out:** nếu tên server MCP Drive không khớp `tools:` của text-researcher, orchestrator đọc nội dung Drive trước, lưu tạm `_workspace/source/{Dxx}.*`, rồi giao text-researcher đọc bản cục bộ đó.

## Phase 3: Tổng hợp (knowledge-synthesizer)

Sau khi tất cả bản phân tích đã có trong `_workspace/docs/`, gọi `knowledge-synthesizer`:
- Đọc catalog + toàn bộ `_workspace/docs/`.
- Dựng `_workspace/brain/`: `index.md`, `themes.md`, `entities.md`, `contradictions.md`, `open-questions.md`.
- Trả về cho người dùng: tổng quan bộ não + các phát hiện nổi bật (chủ đề lớn, mâu thuẫn chính, khoảng trống).

## Phase 4: Truy vấn (lặp lại)

Người dùng đặt câu hỏi xuyên tài liệu → gọi `knowledge-synthesizer` ở chế độ truy vấn:
- Trả lời tổng hợp từ bộ não, mỗi ý kèm nguồn `[Dxx, vị trí]` + mức độ chắc chắn.
- Nếu câu hỏi cần đào sâu một tài liệu cụ thể → có thể gọi lại `text-researcher` cho tài liệu đó.

## Dữ liệu & cấu trúc workspace

```
_workspace/
├── 00_librarian_catalog.md     # danh mục toàn kho (corpus-librarian)
├── docs/
│   ├── D01.md                  # phân tích từng tài liệu (text-researcher)
│   └── D02.md
└── brain/                      # bộ não thứ hai (knowledge-synthesizer)
    ├── index.md
    ├── themes.md
    ├── entities.md
    ├── contradictions.md
    └── open-questions.md
```

Truyền dữ liệu: **file-based** (qua `_workspace/`) cho sản phẩm + **return-value** để orchestrator thu tiến độ. Mã `Dxx` là khóa liên kết xuyên các phase.

## Xử lý lỗi

- **File không đọc được (khóa quyền/định dạng lạ):** librarian đánh dấu, bỏ qua ở Phase 2, synthesizer ghi vào `open-questions.md`. Không để một file lỗi chặn cả pipeline.
- **Một text-researcher thất bại:** thử lại 1 lần; vẫn lỗi thì tiếp tục, ghi rõ tài liệu thiếu phân tích trong báo cáo cuối.
- **Mâu thuẫn giữa tài liệu:** KHÔNG tự hòa giải — đưa vào `contradictions.md` kèm cả hai nguồn.
- **Kho quá lớn vượt khả năng một lượt:** chạy Phase 2 theo lô; synthesizer tổng hợp tăng trưởng.

## Lưu ý môi trường

Tên công cụ MCP Google Drive trong agent (`mcp__Google_Drive__*`) phụ thuộc tên server MCP trong cấu hình của bạn. Nếu server tên khác, chỉnh lại trường `tools:` trong `corpus-librarian.md` cho khớp, hoặc bỏ trường `tools:` để agent kế thừa toàn bộ công cụ sẵn có.

## Test scenario

- **Luồng thường:** thư mục Drive 10 file PDF → librarian lập catalog 10 mã → 10 text-researcher song song (2 lô) → synthesizer dựng brain → người dùng hỏi "các tài liệu có thống nhất về mốc thời gian X không?" → trả lời kèm `[Dxx]`.
- **Luồng lỗi:** 1 file khóa quyền → librarian đánh dấu ⚠️ → Phase 2 bỏ qua → `open-questions.md` ghi "D07 chưa phân tích (khóa quyền)" → pipeline vẫn hoàn tất 9 file còn lại.
