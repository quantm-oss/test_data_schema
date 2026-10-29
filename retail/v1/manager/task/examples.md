## VÍ DỤ 1 — Gán task chăm sóc theo tag existing

**Câu hỏi user**: Gắn task chăm sóc cho những khách hàng đang có tag highvalue

**Solution**:

### Summary

- Gắn task chăm sóc cho những khách hàng đang có tag highvalue.

### Mục tiêu

- Đảm bảo những khách hàng giá trị cao được chăm sóc đặc biệt, từ đó tăng cường sự hài lòng và duy trì mối quan hệ lâu dài, thúc đẩy doanh số và lòng trung thành.

### Điều kiện lọc (logic)
- Khách hàng có tag `highvalue` active.
- `customer_tagging_staging.tagId` equals `highvalue`
- AND `customer_tagging_staging.status` like `active`

### Trường dữ liệu tham chiếu
- `customer_unified.rootId` (String): Định danh khách hàng (tham chiếu để gắn Task)
- `customer_tagging_staging.customerId` (String): Mã khách hàng trong log gắn rule/task
- `customer_tagging_staging.tagId` (String): ID của tag cần filter
- `customer_tagging_staging.status` (String): Trạng thái tag (chỉ active)

### Gợi ý tên task

assign_highvalue_customer_care_task


------------------------------

## VÍ DỤ 2 — Gán task cho khách mới được gắn tag trong tuần

**Câu hỏi user**: Gán task welcome cho khách mới được gắn tag new_signup trong 7 ngày qua

**Solution**:

### Summary

- Gán task welcome cho khách hàng vừa được gắn tag new_signup trong khoảng từ 14/07/2026 đến 20/07/2026 (GMT+7).

### Mục tiêu

- Onboarding khách mới trong khoảng thời gian vàng để tăng trải nghiệm ban đầu, tối ưu tỉ lệ retention và giá trị vòng đời khách hàng.

### Điều kiện lọc (logic)
- `customer_tagging_staging.tagId` equals `new_signup`
- AND `customer_tagging_staging.status` like `active`
- AND `customer_tagging_staging.createdAt` (DateTime64(3, UTC)) trong khoảng từ 14/07/2026 đến 20/07/2026 (GMT+7)

### Trường dữ liệu tham chiếu
- `customer_unified.rootId` (String): Định danh khách hàng (tham chiếu để gắn Task)
- `customer_tagging_staging.customerId` (String): Mã khách hàng
- `customer_tagging_staging.tagId` (String): ID của tag cần filter
- `customer_tagging_staging.status` (String): Trạng thái tag active
- `customer_tagging_staging.createdAt` (DateTime64(3, UTC)): Thời điểm gắn tag

### Gợi ý tên task

assign_new_signup_welcome_task

------------------------------

## VÍ DỤ 3 — matched=false vì schema thiếu field

**Câu hỏi user**: Gán task chăm sóc khách hàng có tổng chi tiêu trên 50 triệu

**Solution**:

### Không thể tạo task solution

- Schema Task hiện tại chỉ track log gắn tag (bảng customer_tagging_staging) và thông tin khách hàng đã hợp nhất (bảng customer_unified) — không có field track tổng chi tiêu.

### Lý do cụ thể
- Bảng `customer_unified` chỉ có fields identifier (rootId, name, source) — không có purchase history.
- Bảng `customer_tagging_staging` chỉ có metadata về tag đã gắn (tagId, status, channel, category...) — không có amount field cố định nào.
- Các field dạng Map (detailString, detailLong, detailDouble, detailBoolean, detailDate, detailArrayString, detailArrayLong, detailArrayDouble) hiện đang RỖNG 100% trong dữ liệu thực tế — không thể dùng để lưu/truy vấn thông tin chi tiêu.

### Đề xuất
Nếu tổng chi tiêu đã được tag hóa trước (VD tag "high_spender" hoặc "vip_over_50m" đã được hệ thống khác gắn sẵn cho khách hàng) → user cung cấp đúng tagId đó để filter qua `customer_tagging_staging.tagId`.
Hệ thống Task hiện chưa hỗ trợ filter trực tiếp theo giá trị số tiền chi tiêu — cần dữ liệu này được đưa vào dưới dạng tag trước khi Task có thể sử dụng.
------------------------------

## VÍ DỤ 4 — OR nhiều tag + business context (channel + category)

**Câu hỏi user**: Gán task follow-up cho khách có tag vip_gold hoặc vip_diamond, được gắn qua kênh Phone

**Solution**:

### Summary

- Gán task follow-up cho khách hàng có tag `vip_gold` hoặc `vip_diamond`, được gắn tag qua kênh Phone.

### Mục tiêu

- Đảm bảo nhóm khách hàng VIP (nhiều hạng khác nhau) được đội Sales/CSKH follow-up qua đúng kênh đã ghi nhận, tăng tỉ lệ chăm sóc kịp thời và duy trì trải nghiệm nhất quán.

### Điều kiện lọc (logic)
- (`customer_tagging_staging.tagId` equals `vip_gold` OR `customer_tagging_staging.tagId` equals `vip_diamond`)
- AND `customer_tagging_staging.status` like `active`
- AND `customer_tagging_staging.channelType` like `Phone`

### Trường dữ liệu tham chiếu
- `customer_unified.rootId` (String): Định danh khách hàng (tham chiếu để gắn Task)
- `customer_tagging_staging.customerId` (String): Mã khách hàng
- `customer_tagging_staging.tagId` (String): ID tag — vip_gold hoặc vip_diamond
- `customer_tagging_staging.status` (String): Trạng thái tag active
- `customer_tagging_staging.channelType` (String): Kênh gắn tag — Phone

### Gợi ý tên task

assign_vip_multi_tier_followup_task

------------------------------

## VÍ DỤ 5 — Tìm theo tên khách hàng + tag category (text search kết hợp)

**Câu hỏi user**: Gán task chăm sóc cho khách tên Nguyễn thuộc nhóm tag customer_segmentation

**Solution**:

### Summary

- Gán task chăm sóc cho khách hàng có tên chứa "Nguyễn" và đang thuộc nhóm phân loại tag `customer_segmentation`.

### Mục tiêu

- Kết hợp định danh cá nhân với nhóm phân loại tag để đội CSKH dễ dàng xác định và liên hệ đúng đối tượng cần chăm sóc theo chiến dịch phân khúc khách hàng.

### Điều kiện lọc (logic)
- `customer_unified.name` like `%Nguyễn%`
- AND `customer_tagging_staging.tagCategory` like `customer_segmentation`
- AND `customer_tagging_staging.status` like `active`

### Trường dữ liệu tham chiếu
- `customer_unified.rootId` (String): Định danh khách hàng (tham chiếu để gắn Task)
- `customer_unified.name` (String, nullable): Tên khách hàng, dùng tìm kiếm chứa "Nguyễn"
- `customer_tagging_staging.customerId` (String): Mã khách hàng
- `customer_tagging_staging.tagCategory` (String): Nhóm phân loại tag — customer_segmentation
- `customer_tagging_staging.status` (String): Trạng thái tag active

### Gợi ý tên task

assign_named_segmentation_care_task

------------------------------

## VÍ DỤ 6 — matched=false vì tagId không cụ thể (khác lý do thiếu field)

**Câu hỏi user**: Gán task chăm sóc cho khách hàng có tag phù hợp với chương trình khuyến mãi

**Solution**:

### Không thể tạo task solution

- Câu hỏi chưa cung cấp tagId cụ thể để filter khách hàng — "tag phù hợp với chương trình khuyến mãi" là mô tả chung chung, không xác định được giá trị `tagId` chính xác cần dùng.

### Lý do cụ thể
- Schema Task yêu cầu bắt buộc user cung cấp `tagId` cụ thể (VD: `promo_2026_summer`, `discount_eligible`) để build điều kiện lọc — không thể LLM tự suy đoán hoặc chọn đại 1 tagId ngẫu nhiên.
- Nếu tự bịa tagId, rule tạo ra có nguy cơ không match với tagId thực tế đang tồn tại trong hệ thống, khiến task không gán được cho ai.

### Đề xuất
Vui lòng cung cấp tagId cụ thể của tag khuyến mãi cần lọc (VD: tên tag đã được tạo cho chương trình đó trong hệ thống tagging), hoặc mô tả rõ hơn tên/mã chương trình khuyến mãi để xác định đúng tagId tương ứng.

------------------------------

## VÍ DỤ 7 — Filter theo expiredAt (task sắp hết hạn) + độ tin cậy

**Câu hỏi user**: Gán task gia hạn cho khách có tag membership sắp hết hạn trong 7 ngày tới, với độ tin cậy trên 0.7

**Solution**:

### Summary

- Gán task gia hạn cho khách hàng có tag `membership` sắp hết hạn trong 7 ngày tới, với độ tin cậy của tag trên 0.7.

### Mục tiêu

- Chủ động liên hệ khách hàng trước khi quyền lợi membership hết hạn, tăng tỉ lệ gia hạn và giảm churn, đồng thời chỉ ưu tiên các trường hợp có độ tin cậy gán tag cao để tránh liên hệ nhầm.

### Điều kiện lọc (logic)
- `customer_tagging_staging.tagId` equals `membership`
- AND `customer_tagging_staging.status` like `active`
- AND `customer_tagging_staging.expiredAt` (DateTime64(3, UTC)) trong khoảng từ 20/07/2026 đến 27/07/2026 (GMT+7)
- AND `customer_tagging_staging.confidence` gte 0.7

### Trường dữ liệu tham chiếu
- `customer_unified.rootId` (String): Định danh khách hàng (tham chiếu để gắn Task)
- `customer_tagging_staging.customerId` (String): Mã khách hàng
- `customer_tagging_staging.tagId` (String): ID tag — membership
- `customer_tagging_staging.status` (String): Trạng thái tag active
- `customer_tagging_staging.expiredAt` (DateTime64(3, UTC)): Thời điểm tag hết hạn, dùng filter khoảng thời gian sắp tới
- `customer_tagging_staging.confidence` (Double/Float64): Độ tin cậy gán tag, lọc >= 0.7

### Gợi ý tên task

assign_membership_renewal_task

## VÍ DỤ 9 — Ưu tiên xử lý theo thứ tự thời gian gắn tag (yêu cầu sort tường minh)

**Câu hỏi user**: Gán task liên hệ ngay cho khách có tag khach_hang_gia_tri_cao, ưu tiên xử lý khách được gắn tag gần đây nhất trước

**Solution**:

### Summary

- Gán task liên hệ ngay cho khách hàng có tag `khach_hang_gia_tri_cao`, sắp xếp ưu tiên khách được gắn tag gần đây nhất lên đầu.

### Mục tiêu

- Đảm bảo đội Sales/CSKH xử lý khách hàng theo đúng thứ tự thời gian gắn tag, ưu tiên các trường hợp mới phát sinh gần nhất để không bỏ lỡ cơ hội tương tác kịp thời.

### Điều kiện lọc (logic)
- `customer_tagging_staging.tagId` equals `khach_hang_gia_tri_cao`
- AND `customer_tagging_staging.status` like `active`
- Sắp xếp theo `customer_tagging_staging.createdAt` giảm dần (khách gắn tag gần đây nhất lên đầu)

### Trường dữ liệu tham chiếu
- `customer_unified.rootId` (String): Định danh khách hàng (tham chiếu để gắn Task)
- `customer_tagging_staging.customerId` (String): Mã khách hàng trong log gắn tag
- `customer_tagging_staging.tagId` (String): ID của tag cần filter
- `customer_tagging_staging.status` (String): Trạng thái tag (chỉ active)
- `customer_tagging_staging.createdAt` (DateTime64(3, UTC)): Thời điểm gắn tag, dùng để sắp xếp thứ tự ưu tiên xử lý

### Gợi ý tên task

assign_high_value_customer_recent_first_task