# CLAUDE.md - Hệ Thống Quản Lý Tín Dụng

## Harness: Credit Officer Tool

**Mục tiêu:** Hệ thống quản lý tín dụng toàn diện cho cán bộ tín dụng (CBTD) — quản lý dư nợ, khách hàng, và nguồn vốn.

**Kích hoạt:** Dùng `/credit-officer` hoặc mô tả nhu cầu liên quan đến tín dụng.

---

## Đội Ngũ Agent

| Agent | File | Vai trò |
|-------|------|---------|
| `can-bo-tin-dung` | `.claude/agents/can-bo-tin-dung.md` | Điều phối, dashboard, báo cáo tổng hợp |
| `quan-ly-khach-hang` | `.claude/agents/quan-ly-khach-hang.md` | Quản lý thông tin khách hàng, đánh giá tín dụng |
| `quan-ly-du-no` | `.claude/agents/quan-ly-du-no.md` | Quản lý khoản vay, dư nợ, thanh toán |
| `quan-ly-nguon-von` | `.claude/agents/quan-ly-nguon-von.md` | Quản lý nguồn vốn, NIM, phân bổ vốn |

## Skills

| Skill | File | Mô tả |
|-------|------|-------|
| `credit-officer` | `.claude/skills/credit-officer/SKILL.md` | Điểm vào chính của hệ thống |

## Dữ Liệu

| File | Nội dung |
|------|----------|
| `data/customers.json` | Thông tin khách hàng |
| `data/loans.json` | Dư nợ và khoản vay |
| `data/funding_sources.json` | Nguồn vốn |
| `data/transactions.json` | Lịch sử giao dịch |

## Tài Liệu Tham Khảo

| File | Nội dung |
|------|----------|
| `.claude/skills/credit-officer/references/data-schema.md` | Cấu trúc dữ liệu |
| `.claude/skills/credit-officer/references/customer-management.md` | Quy trình quản lý KH |
| `.claude/skills/credit-officer/references/debt-management.md` | Quy trình quản lý dư nợ |
| `.claude/skills/credit-officer/references/funding-management.md` | Quy trình quản lý nguồn vốn |

---

## Nhật Ký Thay Đổi

| Ngày | Thay đổi | File | Lý do |
|------|----------|------|-------|
| 2025-06-28 | Khởi tạo hệ thống | Toàn bộ | Yêu cầu mới |
| 2025-06-28 | Tạo 4 agents chuyên biệt | `.claude/agents/` | Phân tách nghiệp vụ |
| 2025-06-28 | Tạo skill và references | `.claude/skills/credit-officer/` | Hướng dẫn thao tác |
| 2025-06-28 | Tạo dữ liệu mẫu | `data/` | Demo và kiểm thử |
