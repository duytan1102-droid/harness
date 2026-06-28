# Hướng Dẫn Quản Lý Khách Hàng

## Quy Trình Thêm Khách Hàng Mới

### Bước 1: Thu Thập Thông Tin
Hỏi tuần tự hoặc nhận form điền sẵn:
```
Họ và tên    : ___________
Số CCCD      : ___________
Điện thoại   : ___________
Địa chỉ      : ___________
Email        : ___________
Nghề nghiệp  : ___________
Thu nhập/tháng: ___________ VND
```

### Bước 2: Kiểm Tra Trùng Lặp
Trước khi thêm mới, kiểm tra trong customers.json:
- CCCD đã tồn tại chưa?
- Số điện thoại đã tồn tại chưa?
→ Nếu trùng: thông báo và hỏi xem có phải cùng người không

### Bước 3: Tính Điểm Tín Dụng Sơ Bộ
Áp dụng khi chưa có lịch sử:

| Yếu tố | Điểm cộng |
|--------|-----------|
| Thu nhập > 20 triệu/tháng | +50 |
| Thu nhập 10-20 triệu/tháng | +30 |
| Thu nhập 5-10 triệu/tháng | +10 |
| Công nhân viên chức nhà nước | +30 |
| Doanh nghiệp tư nhân, ổn định | +20 |
| Lao động phổ thông | 0 |
| Có tài sản đảm bảo | +50 |
| Khách hàng giới thiệu | +20 |

Điểm khởi đầu: 500 + tổng điểm cộng

### Bước 4: Lưu Và Xác Nhận
Hiển thị tóm tắt thông tin → chờ xác nhận → lưu vào customers.json

---

## Đánh Giá Điều Kiện Vay Vốn

### Kiểm Tra Đầu Vào
Trước khi cho vay, agent thu thập và phân tích:

**1. Điểm tín dụng**
| Điểm | Kết quả |
|------|---------|
| ≥ 700 | Đủ điều kiện |
| 600-699 | Đủ điều kiện, cần tài sản đảm bảo |
| 500-599 | Cần xem xét đặc biệt |
| < 500 | Không đủ điều kiện |

**2. Tỷ lệ nợ/thu nhập (DTI)**
- Tính tổng `monthly_payment` của các khoản vay active hiện tại
- Cộng thêm `monthly_payment` dự kiến của khoản vay mới
- DTI = Tổng / income_monthly × 100%
- DTI ≤ 40%: Chấp thuận
- DTI 40-50%: Cần tài sản đảm bảo bổ sung
- DTI > 50%: Từ chối hoặc giảm hạn mức

**3. Tài sản đảm bảo**
Tỷ lệ tối thiểu (LTV): Giá trị khoản vay / Giá trị TSĐB
- Vay mua nhà: LTV ≤ 70%
- Vay kinh doanh: LTV ≤ 80%
- Vay tiêu dùng: LTV ≤ 90% (hoặc không cần TSĐB nếu DTI thấp)

### Output Đánh Giá
```
═══════════════════════════════════════════
  KẾT QUẢ ĐÁNH GIÁ ĐIỀU KIỆN VAY VỐN
═══════════════════════════════════════════
Khách hàng : Nguyễn Văn An (KH001)
Khoản vay  : 300,000,000 VND / 24 tháng
───────────────────────────────────────────
✅ Điểm tín dụng  : 780 (Xuất sắc)
✅ DTI hiện tại   : 32% (≤ 40%)
✅ DTI sau vay    : 47% (≤ 50%, cần TSĐB)
✅ Tài sản ĐB     : Có (LTV = 65%)
───────────────────────────────────────────
KẾT LUẬN: ĐỦ ĐIỀU KIỆN - Cần tài sản ĐB
Lãi suất đề xuất : 8.5%/năm
Trả hàng tháng   : 13,635,000 VND
═══════════════════════════════════════════
```

---

## Tìm Kiếm Khách Hàng

### Cách Tìm
Hỗ trợ tìm theo:
- Mã KH: `KH001`
- Tên (không phân biệt hoa thường, tìm gần đúng)
- Số điện thoại
- Số CCCD
- Địa chỉ (từ khóa)

### Hiển Thị Kết Quả
Nếu tìm thấy 1 người → hiển thị chi tiết đầy đủ
Nếu tìm thấy nhiều người → hiển thị danh sách để chọn

---

## Cập Nhật Thông Tin

### Các Trường Cho Phép Cập Nhật
- Số điện thoại
- Địa chỉ
- Email
- Nghề nghiệp
- Thu nhập hàng tháng
- Điểm tín dụng (chỉ CBTD có quyền cập nhật)
- Trạng thái
- Ghi chú

### Các Trường Không Cho Cập Nhật
- Mã KH (id)
- Số CCCD (cần quy trình đặc biệt)
- Ngày tạo

### Khi Tín Dụng Thay Đổi Đáng Kể
Nếu cập nhật credit_score thay đổi > 50 điểm → ghi chú lý do vào `notes`

---

## Xem Lịch Sử Vay

Khi xem lịch sử vay của KH001:
1. Lọc loans.json theo `customer_id = "KH001"`
2. Lọc transactions.json theo `customer_id = "KH001"`
3. Hiển thị theo timeline từ cũ đến mới

Format:
```
LỊCH SỬ VAY VỐN - Nguyễn Văn An (KH001)
──────────────────────────────────────────
📋 VV001 - Vay kinh doanh (2024-02-01)
   Vốn gốc  : 500,000,000 VND
   Dư nợ HT : 420,000,000 VND
   Trạng thái: ✅ Đang hoạt động

📋 [Các khoản vay khác nếu có...]

💳 GIAO DỊCH GẦN ĐÂY
   01/06/2025: Thanh toán 15,750,000 VND (CK20250601001)
   [...]
```

---

## Quản Lý Blacklist

Khi đặt KH vào blacklist (`status: "blacklist"`):
- Ghi rõ lý do vào `notes`
- Ghi ngày blacklist
- KH trong blacklist không được tạo khoản vay mới
- Cảnh báo màu đỏ khi tra cứu

Format cảnh báo:
```
⛔ CẢNH BÁO: Khách hàng này đang trong danh sách đen
Lý do: [lý do]
Ngày blacklist: DD/MM/YYYY
Liên hệ cấp trên trước khi xử lý.
```
