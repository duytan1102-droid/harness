# Cấu Trúc Dữ Liệu - Hệ Thống Quản Lý Tín Dụng

## customers.json

```json
{
  "customers": [
    {
      "id": "KH001",              // Mã KH (KH + số thứ tự 3 chữ số)
      "name": "Họ và tên đầy đủ",
      "cccd": "012345678901",     // 12 số CCCD hoặc 9 số CMND
      "phone": "0901234567",      // 10 số, bắt đầu 0
      "address": "Số nhà, đường, phường/xã, quận/huyện, tỉnh/thành",
      "email": "email@domain.com",
      "occupation": "Nghề nghiệp cụ thể",
      "income_monthly": 15000000, // VND, thu nhập ròng hàng tháng
      "credit_score": 720,        // 300-850
      "status": "active",         // active | inactive | blacklist
      "created_at": "YYYY-MM-DD",
      "notes": "Ghi chú của CBTD"
    }
  ],
  "last_updated": "YYYY-MM-DD",
  "total_count": 3
}
```

### Quy Tắc Validation
- `id`: Tự động tăng, format KH + 3 số (KH001, KH002, ...)
- `cccd`: 9 hoặc 12 chữ số, không trùng
- `phone`: 10 số, bắt đầu 0, không trùng
- `income_monthly`: > 0, đơn vị VND
- `credit_score`: 300-850
- `status`: Chỉ nhận 3 giá trị trên

---

## loans.json

```json
{
  "loans": [
    {
      "id": "VV001",                  // Mã vay (VV + số 3 chữ số)
      "customer_id": "KH001",
      "customer_name": "Họ tên KH",  // Denormalized để truy vấn nhanh
      "funding_source_id": "NV001",
      "loan_type": "Vay kinh doanh",  // Xem bảng loại vay bên dưới
      "purpose": "Mục đích cụ thể",
      "principal": 500000000,          // Vốn gốc ban đầu (VND)
      "interest_rate": 8.5,           // %/năm
      "term_months": 36,              // Kỳ hạn (tháng)
      "disbursement_date": "YYYY-MM-DD",
      "maturity_date": "YYYY-MM-DD",
      "outstanding_balance": 420000000, // Dư nợ hiện tại
      "paid_principal": 80000000,      // Đã trả gốc tích lũy
      "paid_interest": 32500000,       // Đã trả lãi tích lũy
      "status": "active",              // active | overdue | closed | restructured
      "overdue_days": 0,              // Số ngày quá hạn hiện tại
      "last_payment_date": "YYYY-MM-DD",
      "next_payment_date": "YYYY-MM-DD",
      "monthly_payment": 15750000,     // Số tiền trả hàng kỳ (annuity)
      "collateral": "Mô tả TSĐB",
      "officer_id": "CBTD001",
      "approved_by": "Tên người phê duyệt",
      "notes": "Ghi chú"
    }
  ],
  "last_updated": "YYYY-MM-DD",
  "total_count": 3,
  "summary": {
    "total_outstanding": 680000000,  // Tổng dư nợ
    "total_overdue": 75000000,       // Tổng nợ quá hạn
    "active_loans": 2,
    "overdue_loans": 1
  }
}
```

### Loại Vay (loan_type)
- `Vay kinh doanh` — Phục vụ SXKD, thương mại
- `Vay tiêu dùng` — Mua sắm, sửa nhà, mua xe
- `Vay nông nghiệp` — Nông/lâm/ngư nghiệp
- `Vay mua nhà` — BĐS để ở
- `Vay đầu tư` — Đầu tư tài chính, chứng khoán
- `Vay khác` — Các mục đích khác

### Phân Nhóm Nợ (CIC)
| Nhóm | Tên | Tiêu chí | Dự phòng |
|------|-----|----------|----------|
| 1 | Đủ tiêu chuẩn | Đúng hạn, quá hạn < 10 ngày | 0% |
| 2 | Cần chú ý | Quá hạn 10-90 ngày | 5% |
| 3 | Dưới tiêu chuẩn | Quá hạn 91-180 ngày | 20% |
| 4 | Nghi ngờ | Quá hạn 181-360 ngày | 50% |
| 5 | Mất vốn | Quá hạn > 360 ngày | 100% |

---

## funding_sources.json

```json
{
  "funding_sources": [
    {
      "id": "NV001",              // Mã nguồn vốn (NV + số 3 chữ số)
      "name": "Tên nguồn vốn",
      "type": "Tiết kiệm",        // Xem bảng loại nguồn vốn
      "provider": "Đơn vị cung cấp",
      "total_amount": 10000000000,    // Tổng hạn mức (VND)
      "allocated_amount": 7500000000, // Đã phân bổ cho vay (VND)
      "available_amount": 2500000000, // Còn khả dụng (VND)
      "cost_rate": 5.5,           // Chi phí vốn (%/năm)
      "min_lending_rate": 7.5,    // Lãi suất cho vay tối thiểu
      "max_lending_rate": 12.0,   // Lãi suất cho vay tối đa
      "currency": "VND",          // VND | USD
      "status": "active",         // active | inactive | expired
      "start_date": "YYYY-MM-DD",
      "end_date": null,           // null = không có hạn
      "description": "Mô tả nguồn vốn",
      "notes": "Ghi chú, điều kiện đặc biệt"
    }
  ],
  "last_updated": "YYYY-MM-DD",
  "total_count": 4,
  "summary": {
    "total_capital": 25000000000,
    "total_allocated": 17150000000,
    "total_available": 7850000000,
    "utilization_rate": 68.6       // %
  }
}
```

### Loại Nguồn Vốn (type)
- `Tiết kiệm` — Tiền gửi tiết kiệm dân cư
- `Tái cấp vốn` — Vay NHNN, NHTW
- `Ngân sách Nhà nước` — Chương trình hỗ trợ chính phủ
- `Trái phiếu` — Phát hành trái phiếu
- `Vay liên ngân hàng` — Vay từ các TCTD khác
- `Vốn chủ sở hữu` — Vốn điều lệ, thặng dư

---

## transactions.json

```json
{
  "transactions": [
    {
      "id": "GD001",              // Mã GD (GD + số 3 chữ số)
      "loan_id": "VV001",
      "customer_id": "KH001",
      "type": "payment",          // payment | disbursement | penalty | fee
      "amount": 15750000,         // Tổng số tiền (VND)
      "principal_amount": 11083333, // Phần gốc
      "interest_amount": 4666667,   // Phần lãi
      "payment_date": "YYYY-MM-DD",
      "recorded_by": "CBTD001",
      "method": "Chuyển khoản",   // Chuyển khoản | Tiền mặt | Khấu trừ TK
      "reference": "CK20250601001", // Số tham chiếu giao dịch
      "notes": "Ghi chú"
    }
  ],
  "last_updated": "YYYY-MM-DD",
  "total_count": 3
}
```

### Loại Giao Dịch (type)
- `payment` — Thanh toán gốc + lãi định kỳ
- `disbursement` — Giải ngân khoản vay
- `penalty` — Phạt trả chậm
- `fee` — Phí quản lý khoản vay
- `prepayment` — Trả trước hạn (có thể có phí)

---

## Quan Hệ Giữa Các Bảng

```
customers (KH001) ──┬── loans (VV001) ──── funding_sources (NV001)
                    └── loans (VV002)           │
                                               └── loans (VV002)

loans (VV001) ──── transactions (GD001, GD002, ...)
```

- 1 KH có thể có nhiều khoản vay
- 1 khoản vay thuộc 1 nguồn vốn
- 1 nguồn vốn có thể phục vụ nhiều khoản vay
- 1 khoản vay có nhiều giao dịch

---

## Công Thức Tính Toán Cốt Lõi

### Monthly Payment (Annuity)
```
r = interest_rate / 100 / 12
monthly_payment = principal × r / (1 - (1 + r)^(-term_months))
```

### Phân Bổ Gốc/Lãi Từng Kỳ
```
interest_this_month = outstanding_balance × r
principal_this_month = monthly_payment - interest_this_month
new_balance = outstanding_balance - principal_this_month
```

### Tỷ Lệ Nợ/Thu Nhập (DTI)
```
DTI = Σ(monthly_payment các khoản vay active) / income_monthly × 100%
Giới hạn cho vay thêm: DTI sau vay mới ≤ 40-50%
```

### NPL Ratio
```
NPL = Tổng dư nợ nhóm 3+4+5 / Tổng dư nợ × 100%
Mục tiêu: NPL < 3%
Cảnh báo: NPL > 5%
```
