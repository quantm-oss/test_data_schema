## Ví dụ 1 — purchase_history với payment method

**Câu hỏi**: "gắn tag khách hàng đã mua hàng qua momo trong 30 ngày qua"

**Bảng chọn**: purchase_history

**Solution**:

### Summary
Tag khách hàng đã thanh toán qua Momo trong 30 ngày gần nhất.

### Mục tiêu
Phân loại khách hàng sử dụng phương thức thanh toán Momo để triển khai chiến dịch marketing riêng cho nhóm này, khuyến khích tiếp tục dùng Momo hoặc chuyển sang phương thức thanh toán điện tử khác.

### Điều kiện lọc (logic)
- Phương thức thanh toán chính là `momo`.
- Thời gian giao dịch trong 30 ngày gần nhất tính đến thời điểm hiện tại.

### Trường dữ liệu tham chiếu
- `purchase_history.customerPhone` (String): Số điện thoại khách hàng.
- `purchase_history.paymentMethod` (String): Phương thức thanh toán chính của giao dịch.
- `purchase_history.transCreatedAt` (DateTime): Thời điểm diễn ra giao dịch.

### Gợi ý tên tag
momo_purchaser_last_30_days

---

## Ví dụ 2 — lead_info với classification + source

**Câu hỏi**: "gắn tag lead từ Facebook có phân loại Hot"

**Bảng chọn**: lead_info

**Solution**:

### Summary
Tag lead từ kênh Facebook được đánh giá phân loại Hot.

### Mục tiêu
Xác định nhóm lead chất lượng cao đến từ Facebook để ưu tiên nurture, chuyển đổi thành khách hàng thực tế.

### Điều kiện lọc (logic)
- Nguồn lead là FB (Facebook).
- Phân loại lead là Hot.

### Trường dữ liệu tham chiếu
- `lead_info.phone` (String): Số điện thoại lead.
- `lead_info.source` (String): Nguồn thu thập lead.
- `lead_info.classification` (String): Phân loại mức độ ưu tiên của lead.

### Gợi ý tên tag
hot_facebook_lead

---

## Ví dụ 3 — sms_interaction với campaign + time window

**Câu hỏi**: "gắn tag khách hàng đã nhận SMS chiến dịch flash sale trong 7 ngày qua"

**Bảng chọn**: sms_interaction

**Solution**:

### Summary
Tag khách hàng đã nhận SMS thuộc chiến dịch có tên chứa "flash sale" trong 7 ngày gần nhất.

### Mục tiêu
Phân loại khách hàng đã tiếp nhận thông điệp flash sale để đo lường hiệu quả chiến dịch và triển khai retargeting.

### Điều kiện lọc (logic)
- Tên chiến dịch chứa cụm "flash sale" (không phân biệt hoa/thường).
- Thời gian tương tác SMS trong 7 ngày gần nhất.

### Trường dữ liệu tham chiếu
- `sms_interaction.phoneNumber` (String): Số điện thoại khách hàng nhận SMS.
- `sms_interaction.campaignName` (String): Tên chiến dịch SMS.
- `sms_interaction.eventAt` (DateTime): Thời điểm sự kiện tương tác.

### Gợi ý tên tag
flash_sale_sms_receiver_last_7_days

---

## Ví dụ 4 — zma_log_cf với appType + platform

**Câu hỏi**: "gắn tag người dùng có tương tác PangoForm trong tuần qua"

**Bảng chọn**: zma_log_cf

**Solution**:

### Summary
Tag người dùng đã tương tác với ứng dụng PangoForm trong tuần gần nhất.

### Mục tiêu
Xác định nhóm user đã trải nghiệm sản phẩm PangoForm để nurture họ sử dụng các Pango app khác hoặc mua gói premium.

### Điều kiện lọc (logic)
- Loại ứng dụng là PangoForm.
- Thời gian ghi nhận log trong 7 ngày gần nhất.

### Trường dữ liệu tham chiếu
- `zma_log_cf.zaloUserAppId` (String): Định danh Zalo user trong ứng dụng.
- `zma_log_cf.appType` (String): Loại ứng dụng Pango.
- `zma_log_cf.eventAt` (DateTime): Thời điểm log sự kiện.

### Gợi ý tên tag
pangoform_engaged_user_last_7_days

---

## Ví dụ 5 — purchase_history aggregation với ngưỡng chi tiêu

**Câu hỏi**: "gắn tag khách hàng VIP có tổng chi tiêu trên 10 triệu trong 6 tháng qua"

**Bảng chọn**: purchase_history

**Solution**:

### Summary
Tag khách hàng VIP có tổng chi tiêu trên 10 triệu VND trong 6 tháng gần nhất.

### Mục tiêu
Xác định nhóm khách hàng chi tiêu cao trong period gần đây để triển khai chương trình VIP, cross-sell sản phẩm cao cấp, hoặc chăm sóc đặc biệt.

### Điều kiện lọc (logic)
- Tổng chi tiêu (aggregate SUM totalCost) theo khách hàng trong 6 tháng gần nhất > 10.000.000 VND.

### Trường dữ liệu tham chiếu
- `purchase_history.customerPhone` (String): Số điện thoại khách hàng — dùng làm định danh grouping.
- `purchase_history.totalCost` (UInt64): Tổng giá trị đơn hàng — dùng để tính tổng chi tiêu.
- `purchase_history.transCreatedAt` (DateTime): Thời điểm giao dịch — dùng để giới hạn khoảng thời gian.

### Gợi ý tên tag
vip_high_spender_last_6_months

---

## Ví dụ 6 — customer_profile so sánh thời điểm mua đầu tiên với mua gần nhất

**Câu hỏi**: "gắn tag khách hàng vừa mua hàng lần đầu"

**Bảng chọn**: customer_profile

**Solution**:

### Summary
Tag khách hàng vừa mua hàng lần đầu — thời điểm mua hàng đầu tiên trùng với thời điểm mua hàng gần nhất (mới chỉ phát sinh đúng 1 lần mua).

### Mục tiêu
Xác định nhóm khách hàng mới chỉ mua hàng 1 lần để triển khai chăm sóc sau bán, gửi lời cảm ơn và kích thích khách quay lại mua lần thứ hai.

### Điều kiện lọc (logic)
- Thời điểm mua hàng đầu tiên (`firstPurchasedAt`) bằng thời điểm mua hàng gần nhất (`lastPurchasedAt`) — so sánh trực tiếp giá trị của 2 field trong cùng bảng, không so với mốc thời gian cố định.

### Trường dữ liệu tham chiếu
- `customer_profile.mainPhone` (String): Số điện thoại chính của khách hàng — dùng làm định danh.
- `customer_profile.firstPurchasedAt` (DateTime): Thời điểm khách hàng mua hàng lần đầu tiên.
- `customer_profile.lastPurchasedAt` (DateTime): Thời điểm khách hàng mua hàng gần nhất — dùng làm vế so sánh với `firstPurchasedAt`.

### Gợi ý tên tag
first_purchase_customer

---

## Ví dụ 7 — customer_profile + sms_interaction (JOIN 2 bảng qua số điện thoại)

**Câu hỏi**: "gắn tag khách hàng nữ đã click SMS trong 30 ngày qua"

**Bảng chọn**: customer_profile, sms_interaction

**Solution**:

### Summary
Tag khách hàng nữ đã click vào tin nhắn SMS trong 30 ngày gần nhất.

### Mục tiêu
Xác định nhóm khách hàng nữ có phản hồi tích cực với kênh SMS để ưu tiên gửi các chiến dịch dành riêng cho nhóm này.

### Điều kiện lọc (logic)
- Giới tính khách hàng là nữ (`customer_profile.gender` thuộc `female`, `Nữ`).
- AND Khách hàng có sự kiện SMS là `clicked` (`sms_interaction.event` equals `clicked`).
- AND Thời điểm sự kiện SMS trong 30 ngày gần nhất (`sms_interaction.eventAt`).
- Nối 2 bảng bằng số điện thoại: `customer_profile.mainPhone` = `sms_interaction.phoneNumber`.

### Trường dữ liệu tham chiếu
- `customer_profile.mainPhone` (String): Số điện thoại chính của khách hàng — dùng làm định danh và khóa nối.
- `customer_profile.gender` (String): Giới tính khách hàng.
- `sms_interaction.phoneNumber` (String): Số điện thoại nhận SMS — khóa nối với `customer_profile.mainPhone`.
- `sms_interaction.event` (String): Loại sự kiện SMS.
- `sms_interaction.eventAt` (DateTime): Thời điểm sự kiện SMS.

### Gợi ý tên tag
female_sms_clicker_last_30_days
