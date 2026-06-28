# Hướng Dẫn Quản Lý Dư Nợ

## Quy Trình Tạo Khoản Vay Mới

### Bước 1: Thẩm Định Khách Hàng
Phối hợp với `quan-ly-khach-hang`:
- Xác nhận KH tồn tại và đang `active`
- Kiểm tra credit_score ≥ 600
- Tính DTI sau khi có khoản vay mới ≤ 50%
- Kiểm tra không có nợ nhóm 3,4,5 hiện tại

### Bước 2: Chọn Nguồn Vốn
Phối hợp với `quan-ly-nguon-von`:
- Xác định loại vay → nguồn vốn phù hợp
- Kiểm tra `available_amount >= loan_amount`
- Xác nhận `interest_rate >= min_lending_rate` của nguồn vốn đó

### Bước 3: Tính Toán Khoản Vay

```
Ví dụ: Vay 300 triệu, 8.5%/năm, 24 tháng

monthly_rate = 8.5% / 12 = 0.7083%
monthly_payment = 300,000,000 × 0.007083 / (1 - (1.007083)^(-24))
               = 2,124,900 / 0.1546
               = 13,743,000 VND/tháng (làm tròn)

Tổng phải trả = 13,743,000 × 24 = 329,832,000 VND
Tổng lãi = 329,832,000 - 300,000,000 = 29,832,000 VND
```

### Bước 4: Xác Nhận Và Tạo
1. Hiển thị tóm tắt khoản vay để CBTD xác nhận
2. Tạo ID mới (VV + số tiếp theo)
3. Cập nhật loans.json
4. Cập nhật funding_sources.json (allocated_amount, available_amount)
5. Tạo giao dịch giải ngân trong transactions.json

---

## Lịch Trả Nợ (Amortization Schedule)

### Tạo Lịch Đầy Đủ
Format output bảng lịch trả:

```
LỊCH TRẢ NỢ - VV001 (Nguyễn Văn An)
Vốn gốc: 500,000,000 | Lãi suất: 8.5%/năm | 36 tháng
Monthly Payment: 15,750,000 VND
══════════════════════════════════════════════════════════
Kỳ  Ngày          Trả gốc      Trả lãi      Dư nợ còn
──────────────────────────────────────────────────────────
 1  01/03/2024  12,208,333    3,541,667   487,791,667
 2  01/04/2024  12,294,792    3,455,208   475,496,875
 3  01/05/2024  12,381,867    3,368,133   463,114,508
...
36  01/02/2027  15,638,750      111,250           0
══════════════════════════════════════════════════════════
Tổng gốc: 500,000,000  |  Tổng lãi: 67,000,000
```

### Tính Nhanh Kỳ Hiện Tại
Dựa vào `outstanding_balance` hiện tại:
```
interest_this_month = outstanding_balance × (interest_rate/100/12)
principal_this_month = monthly_payment - interest_this_month
```

---

## Ghi Nhận Thanh Toán

### Quy Trình
1. Xác nhận khoản vay (VV ID)
2. Nhập thông tin: ngày thanh toán, số tiền, phương thức, số tham chiếu
3. Tính phân bổ gốc/lãi
4. Kiểm tra: số tiền = monthly_payment? Hay khác?

### Xử Lý Các Trường Hợp

**Thanh toán đúng hạn:**
```
outstanding_balance -= principal_this_month
paid_principal += principal_this_month
paid_interest += interest_this_month
last_payment_date = payment_date
next_payment_date = payment_date + 1 month
overdue_days = 0 (nếu đang quá hạn)
```

**Thanh toán trước hạn (partial prepayment):**
- Số tiền > monthly_payment
- Phần dư → giảm trực tiếp vào outstanding_balance
- Tính lại lịch trả (giảm kỳ hạn hoặc giảm monthly_payment)
- Hỏi CBTD: "Giảm kỳ hạn hay giảm số tiền trả hàng tháng?"

**Tất toán sớm (full prepayment):**
- Tính phí tất toán trước hạn (nếu có, thường 1-2% dư nợ trong 1-2 năm đầu)
- Cập nhật `status = "closed"`
- Giải phóng vốn trong funding_sources.json

**Thanh toán thiếu:**
- Ghi nhận phần đã thanh toán
- Phần thiếu → tính vào nợ quá hạn kỳ tiếp
- Cảnh báo CBTD

---

## Theo Dõi Và Xử Lý Nợ Quá Hạn

### Phân Loại Và Ưu Tiên

```
⚠️  DANH SÁCH NỢ QUÁ HẠN (Sắp xếp theo mức độ ưu tiên)
═══════════════════════════════════════════════════════════
NHÓM 5 - NỢ CÓ KHẢ NĂNG MẤT VỐN (> 360 ngày)
   [Danh sách + số tiền]

NHÓM 4 - NỢ NGHI NGỜ (181-360 ngày)
   [Danh sách + số tiền]

NHÓM 3 - DƯỚI TIÊU CHUẨN (91-180 ngày)
   [Danh sách + số tiền]

NHÓM 2 - NỢ CẦN CHÚ Ý (10-90 ngày)
   ⚠️  VV003 - Lê Minh Tuấn - 75,000,000 VND (32 ngày QH)
       Điện thoại: 0933456789
       TSĐB: Đất nông nghiệp 5000m2

NHÓM 2 - SẮP QUÁ HẠN (trong 7 ngày)
   ⏰  VV002 - Trần Thị Bích - đến hạn 15/07/2025
═══════════════════════════════════════════════════════════
Tổng nợ xấu (nhóm 3-5): 0 VND
NPL Ratio: 0.0%
```

### Hành Động Theo Nhóm

| Nhóm | Hành động đề xuất |
|------|------------------|
| Sắp QH (< 7 ngày) | Nhắc nhở khách hàng qua điện thoại |
| Nhóm 2 (10-90 ngày) | Gặp khách hàng, tìm hiểu nguyên nhân |
| Nhóm 3 (91-180 ngày) | Đề nghị tái cơ cấu nợ, gửi thư nhắc nợ chính thức |
| Nhóm 4 (181-360 ngày) | Khởi động thủ tục xử lý TSĐB |
| Nhóm 5 (> 360 ngày) | Chuyển sang bộ phận xử lý nợ, cân nhắc khởi kiện |

---

## Tái Cơ Cấu Nợ

### Điều Kiện Được Tái Cơ Cấu
- Khách hàng có khó khăn khách quan (thiên tai, dịch bệnh, mất việc)
- Nợ quá hạn < 90 ngày (nhóm 2)
- Có tài sản đảm bảo đầy đủ
- Được trưởng phòng phê duyệt

### Các Hình Thức

**Gia hạn kỳ hạn (Extension):**
```
Trước: 36 tháng còn lại 20 kỳ
Sau:   Gia hạn thêm 12 tháng → còn 32 kỳ
→ monthly_payment giảm xuống
→ Cập nhật maturity_date, monthly_payment
→ status = "restructured"
```

**Hạ lãi suất (Rate Reduction):**
```
Trước: 8.5%/năm
Sau:   7.0%/năm (giảm 1.5%)
→ Tính lại monthly_payment
→ Ghi rõ ngày bắt đầu áp dụng lãi mới
```

**Khoanh nợ gốc:**
```
Tạm thời chỉ thu lãi, không thu gốc trong N tháng
→ outstanding_balance không đổi trong kỳ khoanh nợ
→ Tự động kết thúc sau N tháng
```

---

## Báo Cáo Dư Nợ

### Báo Cáo Tháng
```
BÁO CÁO DƯ NỢ THÁNG 06/2025
═══════════════════════════════════════════════
THEO LOẠI VAY:
  Vay kinh doanh : 420,000,000 VND (61.8%)
  Vay tiêu dùng  : 185,000,000 VND (27.2%)
  Vay nông nghiệp:  75,000,000 VND (11.0%)
  Tổng           : 680,000,000 VND

THEO NHÓM NỢ:
  Nhóm 1 (Đủ TC) : 605,000,000 VND (89.0%)
  Nhóm 2 (Chú ý) :  75,000,000 VND (11.0%)
  Nhóm 3-5 (NPL) :           0 VND (0.0%)
  NPL Ratio       : 0.0% ✅

THEO NGUỒN VỐN:
  NV001 (Tiết kiệm)   : 420,000,000 VND
  NV002 (NHNN)        : 185,000,000 VND
  NV003 (Nông nghiệp) :  75,000,000 VND

BIẾN ĐỘNG SO VỚI THÁNG TRƯỚC:
  Dư nợ tăng/giảm: -15,750,000 VND (đã trả kỳ)
  Giải ngân mới  :           0 VND
  Tất toán       :           0 VND
═══════════════════════════════════════════════
```
