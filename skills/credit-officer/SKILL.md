---
name: credit-officer
description: Hệ thống quản lý tín dụng toàn diện cho cán bộ tín dụng (CBTD). Kích hoạt khi cần quản lý dư nợ, khách hàng vay vốn, nguồn vốn cho vay, báo cáo tín dụng, xử lý nợ quá hạn. Trigger: /credit-officer, cán bộ tín dụng, quản lý dư nợ, quản lý khách hàng vay, nguồn vốn cho vay, CBTD portal.
---

# Skill: Hệ Thống Quản Lý Tín Dụng

Skill này kích hoạt đội ngũ agent chuyên biệt giúp cán bộ tín dụng quản lý toàn diện hoạt động tín dụng.

## Đội Ngũ Agent

| Agent | Vai trò | Kích hoạt khi |
|-------|---------|---------------|
| `can-bo-tin-dung` | Điều phối, dashboard, báo cáo tổng hợp | Mặc định, báo cáo, không rõ yêu cầu |
| `quan-ly-khach-hang` | CRUD khách hàng, đánh giá tín dụng | Yêu cầu về khách hàng |
| `quan-ly-du-no` | Quản lý khoản vay, dư nợ, thanh toán | Yêu cầu về khoản vay/dư nợ |
| `quan-ly-nguon-von` | Quản lý nguồn vốn, NIM, phân bổ | Yêu cầu về nguồn vốn |

## Khởi Động Nhanh

```
/credit-officer              → Mở dashboard tổng quan
/credit-officer khách hàng   → Quản lý khách hàng
/credit-officer dư nợ        → Quản lý dư nợ  
/credit-officer nguồn vốn    → Quản lý nguồn vốn
/credit-officer báo cáo      → Báo cáo tổng hợp
```

## Luồng Làm Việc Điển Hình

### Tạo Khoản Vay Mới
1. Tìm/xác nhận khách hàng → `quan-ly-khach-hang`
2. Đánh giá điều kiện vay → `quan-ly-khach-hang`
3. Chọn nguồn vốn phù hợp → `quan-ly-nguon-von`
4. Tạo và giải ngân khoản vay → `quan-ly-du-no`

### Ghi Nhận Thanh Toán
1. Tìm khoản vay cần thanh toán → `quan-ly-du-no`
2. Ghi nhận giao dịch → `quan-ly-du-no`
3. Cập nhật dư nợ và lịch sử → tự động

### Xử Lý Nợ Quá Hạn
1. Kiểm tra danh sách quá hạn → `quan-ly-du-no`
2. Xem thông tin khách hàng → `quan-ly-khach-hang`
3. Quyết định tái cơ cấu hoặc xử lý → `quan-ly-du-no`

## Dữ Liệu

Tất cả dữ liệu lưu trong thư mục `data/`:
- `customers.json` — Thông tin khách hàng
- `loans.json` — Dư nợ và khoản vay
- `funding_sources.json` — Nguồn vốn
- `transactions.json` — Lịch sử giao dịch

## Tham Khảo Chi Tiết

→ `references/customer-management.md` — Hướng dẫn quản lý khách hàng
→ `references/debt-management.md` — Hướng dẫn quản lý dư nợ
→ `references/funding-management.md` — Hướng dẫn quản lý nguồn vốn
→ `references/data-schema.md` — Cấu trúc dữ liệu đầy đủ
