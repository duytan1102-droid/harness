# Hệ Thống Quản Lý Tín Dụng - CBTD Portal

Công cụ quản lý tín dụng toàn diện cho cán bộ tín dụng, tích hợp vào Claude Code.

## Tính Năng

### 👤 Quản Lý Khách Hàng
- Thêm, tìm kiếm, cập nhật thông tin khách hàng
- Đánh giá điểm tín dụng và điều kiện vay vốn
- Xem lịch sử vay vốn toàn diện
- Quản lý danh sách đen (blacklist)

### 💰 Quản Lý Dư Nợ
- Tạo khoản vay mới với tính toán tự động (annuity)
- Ghi nhận thanh toán, tính phân bổ gốc/lãi
- Tạo lịch trả nợ đầy đủ (amortization schedule)
- Theo dõi và phân loại nợ quá hạn (CIC 5 nhóm)
- Tái cơ cấu nợ khi khách hàng gặp khó khăn

### 🏦 Quản Lý Nguồn Vốn
- Xem tổng quan nguồn vốn, tỷ lệ sử dụng
- Phân bổ vốn thông minh theo loại vay
- Phân tích NIM (Net Interest Margin)
- Cảnh báo thanh khoản, nguồn vốn sắp hết hạn
- Tính chi phí vốn bình quân (WACC)

### 📊 Báo Cáo & Phân Tích
- Dashboard tổng quan theo thời gian thực
- Báo cáo dư nợ theo nhóm nợ, loại vay, nguồn vốn
- Tỷ lệ NPL (nợ xấu)
- Hiệu quả sử dụng nguồn vốn

## Cài Đặt

```bash
# 1. Copy agents vào dự án của bạn
cp -r .claude/agents/* ~/.claude/agents/

# 2. Copy skill vào dự án
cp -r .claude/skills/credit-officer ~/.claude/skills/

# 3. Tạo thư mục dữ liệu
mkdir -p data

# 4. Khởi tạo dữ liệu trống (hoặc dùng dữ liệu mẫu)
cp data/*.json /path/to/your/project/data/
```

## Sử Dụng

### Khởi Động
```
/credit-officer
```
Hoặc chỉ gõ lệnh tự nhiên:
```
Cho tôi xem tổng quan dư nợ hôm nay
Thêm khách hàng mới tên Nguyễn Văn B
KH001 muốn vay thêm 200 triệu
Danh sách nợ quá hạn?
Nguồn vốn còn lại bao nhiêu?
```

### Các Lệnh Thường Dùng

| Mục đích | Ví dụ lệnh |
|----------|-----------|
| Xem dashboard | "tổng quan", "dashboard" |
| Thêm KH | "thêm khách hàng mới" |
| Tìm KH | "tìm KH001" hoặc "tìm Nguyễn Văn An" |
| Tạo khoản vay | "tạo khoản vay cho KH001, 300 triệu, 24 tháng" |
| Ghi thanh toán | "ghi nhận VV001 đã thanh toán hôm nay" |
| Nợ quá hạn | "danh sách nợ quá hạn" |
| Nguồn vốn | "xem nguồn vốn NV001" |
| Báo cáo | "báo cáo dư nợ tháng này" |

## Cấu Trúc Dữ Liệu

```
data/
├── customers.json       # Khách hàng
├── loans.json           # Khoản vay / Dư nợ
├── funding_sources.json # Nguồn vốn
└── transactions.json    # Lịch sử giao dịch
```

## Kiến Trúc Agent

```
        [Người dùng / CBTD]
               │
               ▼
    ┌─── can-bo-tin-dung ───┐
    │   (Điều phối trung tâm)│
    └──────────┬────────────┘
               │
    ┌──────────┼──────────────┐
    ▼          ▼              ▼
quan-ly-   quan-ly-       quan-ly-
khach-hang  du-no         nguon-von
    │          │              │
    ▼          ▼              ▼
customers  loans.json  funding_sources
 .json    transactions      .json
               .json
```

## Yêu Cầu

- Claude Code v2.x trở lên
- Tính năng Agent Teams (experimental): `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## Giấy Phép

Apache 2.0 - Xem file [LICENSE](LICENSE)
