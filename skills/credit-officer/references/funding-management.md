# Hướng Dẫn Quản Lý Nguồn Vốn

## Tổng Quan Về Nguồn Vốn

### Nguyên Tắc Quản Lý Vốn

1. **Phù hợp kỳ hạn (Maturity Matching)**
   - Vốn ngắn hạn (< 1 năm) → cho vay ngắn hạn (< 12 tháng)
   - Vốn trung hạn (1-5 năm) → cho vay trung hạn (12-60 tháng)
   - Vốn dài hạn (> 5 năm) → cho vay dài hạn

2. **Đảm Bảo Thanh Khoản**
   - Luôn duy trì tỷ lệ thanh khoản tối thiểu: `available_amount / total_amount ≥ 10%`
   - Cảnh báo khi utilization_rate > 90%

3. **Tối Ưu Chi Phí Vốn**
   - Ưu tiên nguồn vốn chi phí thấp phù hợp với đối tượng vay
   - Duy trì spread tối thiểu 1.5% (lending_rate - cost_rate)

4. **Tuân Thủ Điều Kiện Nhà Cung Cấp**
   - NV003 (Ngân sách NN): Chỉ cho hộ nông dân, ngư dân
   - NV002 (NHNN tái cấp): Ưu tiên vay tiêu dùng thiết yếu
   - NV004 (Trái phiếu): Ưu tiên vay trung dài hạn

---

## Quy Trình Phân Bổ Vốn

### Khi Tạo Khoản Vay Mới

Agent `quan-ly-nguon-von` nhận yêu cầu từ `quan-ly-du-no`:

```
Input:
  loan_type: "Vay nông nghiệp"
  loan_amount: 100,000,000 VND
  loan_term: 24 tháng
  customer_type: "Nông dân"

Quy trình:
1. Lọc nguồn vốn: status = "active" AND available_amount >= loan_amount
2. Áp dụng quy tắc ưu tiên theo loại vay:
   - Nông nghiệp → NV003 (ưu tiên 1), NV001 (dự phòng)
3. Trong cùng ưu tiên → chọn nguồn có cost_rate thấp nhất
4. Kiểm tra: loan_interest_rate >= min_lending_rate nguồn đó
5. Xác nhận với CBTD
6. Cập nhật: allocated_amount += loan_amount, available_amount -= loan_amount
7. Tính lại utilization_rate = allocated_amount / total_amount × 100%
8. Cập nhật summary
```

### Khi Tất Toán Khoản Vay
```
funding_source.allocated_amount -= loan.principal
funding_source.available_amount += loan.principal
Cập nhật utilization_rate và summary
```

---

## Bảng Quy Tắc Chọn Nguồn Vốn

| Loại vay | Ưu tiên 1 | Ưu tiên 2 | Ưu tiên 3 |
|----------|-----------|-----------|-----------|
| Vay nông nghiệp | NV003 (NN hỗ trợ) | NV001 (Tiết kiệm) | NV004 (Trái phiếu) |
| Vay tiêu dùng | NV002 (NHNN) | NV001 (Tiết kiệm) | NV004 (Trái phiếu) |
| Vay mua nhà | NV004 (Trái phiếu) | NV001 (Tiết kiệm) | NV002 (NHNN) |
| Vay kinh doanh | NV001 (Tiết kiệm) | NV004 (Trái phiếu) | NV002 (NHNN) |
| Vay đầu tư | NV004 (Trái phiếu) | NV001 (Tiết kiệm) | - |

*Nếu nguồn ưu tiên 1 không đủ hoặc không phù hợp → chuyển ưu tiên 2*

---

## Phân Tích Hiệu Quả Nguồn Vốn

### Tính NIM Theo Nguồn

Với mỗi nguồn vốn NVxxx:
1. Lọc loans.json: `funding_source_id = "NVxxx"` AND `status ≠ "closed"`
2. Tính lãi suất cho vay bình quân: WACC = Σ(outstanding_balance_i × interest_rate_i) / Σ(outstanding_balance_i)
3. NIM = WACC - cost_rate

```
Ví dụ NV001:
  VV001: 420,000,000 × 8.5% = 35,700,000
  Lãi suất BQ = 35,700,000 / 420,000,000 = 8.5%
  NIM = 8.5% - 5.5% = 3.0% ✅ (Tốt)
```

### Báo Cáo Hiệu Quả Nguồn Vốn

```
PHÂN TÍCH HIỆU QUẢ NGUỒN VỐN
Ngày: DD/MM/YYYY
═══════════════════════════════════════════════════════════════
Nguồn  │ Loại      │ Chi phí │ LS CV BQ │ NIM    │ Sử dụng
───────┼───────────┼─────────┼──────────┼────────┼─────────
NV001  │ Tiết kiệm │  5.5%   │   8.5%   │  3.0%  │  75.0%
NV002  │ Tái cấp   │  4.0%   │  10.5%   │  6.5%  │  64.0%
NV003  │ NSNN      │  2.0%   │   6.5%   │  4.5%  │  72.5%
NV004  │ Trái phiếu│  6.0%   │     -    │    -   │  62.5%
───────┼───────────┼─────────┼──────────┼────────┼─────────
Tổng   │           │  4.8%   │   8.6%   │  3.8%  │  68.6%
═══════════════════════════════════════════════════════════════
Chi phí vốn BQ (WACC): 4.8%/năm
NIM bình quân toàn danh mục: 3.8%/năm ✅
```

---

## Cảnh Báo Và Khuyến Nghị Tự Động

### Cảnh Báo Thanh Khoản
```
⚠️  CẢNH BÁO THANH KHOẢN
NV001 - Vốn tiết kiệm dân cư
Tỷ lệ sử dụng: 90% (Ngưỡng an toàn: 85%)
Vốn còn: 1,000,000,000 VND
Đề xuất: Cần huy động thêm ≥ 500,000,000 VND
```

### Cảnh Báo Hết Hạn
```
⏰  NGUỒN VỐN SẮP HẾT HẠN
NV003 - Ngân sách NN hỗ trợ nông nghiệp
Ngày hết hạn: 31/12/2025 (còn 187 ngày)
Dư nợ đang dùng: 75,000,000 VND
Hành động: Liên hệ Bộ Tài chính gia hạn hoặc chuyển nguồn vốn thay thế
```

### Cảnh Báo NIM Thấp
```
❌  CẢNH BÁO NIM ÂM
NVxxx - [Tên nguồn vốn]
Chi phí vốn: 8.0%
Lãi suất CV BQ: 7.5%
NIM: -0.5% ⚠️ (Đang lỗ trên nguồn vốn này)
Hành động khẩn: Xem xét tăng lãi suất cho vay hoặc ngừng phân bổ
```

---

## Thêm Nguồn Vốn Mới

### Thu Thập Thông Tin
```
Tên nguồn vốn     : ___________
Loại              : ___________
Đơn vị cung cấp  : ___________
Tổng hạn mức     : ___________ VND
Chi phí vốn      : ____%/năm
Lãi suất CV tối thiểu: ____%/năm
Lãi suất CV tối đa: ____%/năm
Ngày bắt đầu     : DD/MM/YYYY
Ngày kết thúc    : DD/MM/YYYY (hoặc không có hạn)
Điều kiện đặc biệt: ___________
```

### Kiểm Tra Tự Động
1. `min_lending_rate >= cost_rate + 1.5%` → đảm bảo spread tối thiểu
2. Nếu không đạt → cảnh báo: "Spread quá thấp. Đề nghị điều chỉnh lãi suất CV tối thiểu lên X%"
3. Tạo ID tự động (NV + số tiếp theo)

---

## Tối Ưu Hóa Danh Mục Nguồn Vốn

### Phân Tích Chi Phí Vốn Biên
Khi cần thêm vốn mới, so sánh các lựa chọn:

```
PHÂN TÍCH CÁC PHƯƠNG ÁN HUY ĐỘNG VỐN
────────────────────────────────────────────────────
Phương án     │ Chi phí │ Hạn mức   │ Kỳ hạn │ Ưu tiên
──────────────┼─────────┼───────────┼─────────┼────────
Tăng TK 6th  │  5.0%   │ Không giới │  6th    │  3
Phát hành TP │  6.2%   │  hạn       │  5 năm  │  2  
Vay LNH      │  4.8%   │  hạn       │  Quay đêm│ 1 (ngắn)
NHNN tái cấp │  4.0%   │  hạn       │  2 năm  │  1 (ưu)
────────────────────────────────────────────────────
Đề xuất: [Phương án tối ưu dựa trên nhu cầu cụ thể]
```

### Cân Đối Kỳ Hạn (ALM - Asset-Liability Management)
Đảm bảo:
- Tổng dư nợ ngắn hạn (< 12 tháng) ≤ Tổng nguồn vốn ngắn hạn × 110%
- Tổng dư nợ trung dài hạn (> 12 tháng) ≤ Tổng nguồn vốn trung dài hạn × 120%
- Cảnh báo nếu có rủi ro mất cân đối kỳ hạn (maturity mismatch)
