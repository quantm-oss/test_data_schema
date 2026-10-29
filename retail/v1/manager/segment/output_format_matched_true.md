Trả về JSON với `matched: true` và `solution` là chuỗi markdown theo format sau:

```markdown
### Summary
[Mô tả ngắn 1-2 câu về segment: đối tượng khách hàng + điều kiện chính]

### Mục tiêu
[Mục tiêu kinh doanh của segment - tại sao cần segment này, dùng cho chiến dịch/hoạt động gì]

### Điều kiện lọc (logic)
- [Điều kiện 1 với field cụ thể trong unified_profile_model, note type]
- [Điều kiện 2 với field cụ thể]
- [Nếu có nhiều điều kiện, thể hiện rõ AND/OR]

### Trường dữ liệu tham chiếu
- `fieldName1` (Type): Mô tả ngắn cách dùng
- `fieldName2` (Type): Mô tả ngắn cách dùng
- ...

### Gợi ý tên segment
[segment_name_in_snake_case_english]
```

## Nguồn dữ liệu

Segment CHỈ dùng **1 nguồn duy nhất**: `unified_profile_model` (Elasticsearch index `p_connext_user_profile_{orgId}`) — hồ sơ khách hàng 360° đã hợp nhất từ nhiều nguồn (CRM, E-commerce, Social, Mobile App). KHÔNG có khái niệm chọn giữa nhiều collection như Tag — mọi điều kiện lọc đều tham chiếu trực tiếp field của `unified_profile_model`, KHÔNG ghi prefix tên bảng/collection nào trước field.

## Nguyên tắc viết Điều kiện lọc

- Ghi tên field TRỰC TIẾP (KHÔNG có prefix collection): `totalPurchasedAmount`, `gender`, `mainPhone` (không ghi `purchase_history.totalCost` hay bất kỳ prefix bảng nào — đây là lỗi từ template Tag, KHÔNG áp dụng cho Segment).
- Note type trong ngoặc: `(String)`, `(Long)`, `(Long, epoch millis)`, `(Double)`, `(Array<String>)`.
- **Toàn bộ field thời gian là `Long` dạng epoch millis** (`createdAt`, `updatedAt`, `lastPurchasedAt`, `firstActivityAt`, `attributeLongs.MembershipIssuedAt`, ...) — KHÔNG phải `DateTime`/`Date`. Khi lọc theo khoảng thời gian tương đối ("30 ngày qua"), ghi rõ field là epoch millis, quy đổi mốc so sánh theo timezone GMT+7.
- **`firstPurchasedAmount` và `lastPurchasedAmount` CHỈ phản ánh 1 giao dịch đơn lẻ** (đơn đầu tiên/gần nhất) — KHÔNG được dùng để suy luận "tổng chi tiêu" hay "khách chi tiêu nhiều". Muốn lọc theo tổng chi tiêu, PHẢI dùng `totalPurchasedAmount` (cộng dồn tất cả đơn hàng).
- Field dạng Map động (`attributes`, `attributeInts`, `attributeLongs`, `attributeDoubles`, `attributeBooleans`, `attributeTimestamps`, `fieldSourceMap`, `fieldUpdatedAt`, `lastSegmentKeys`, `lastSegmentAt`, `lastSegmentPrioritys`, `coreTagIdsAudit`) — PHẢI dùng dot-notation với key CỤ THỂ, VD `attributes.MembershipLevel`, `attributeInts.LoyaltyPoint`, `lastSegmentKeys.PRODUCT_INTEREST`. KHÔNG được bịa key không có trong schema — nếu solution cần 1 attribute không rõ key chính xác, PHẢI trả `matched: false` và hỏi lại key cụ thể thay vì đoán.
- Field dạng Array (`userSegmentKeys`, `coreTagIds`, `sources`, `firstPurchasedProducts`, `lastPurchasedProducts`) — dùng operator `contains_any`/`contains_all` khi lọc theo giá trị nằm trong mảng.
- `membership` (Object) luôn `null` trong dữ liệu thực tế — KHÔNG dùng field này. Thông tin membership thực tế nằm ở `attributes.MembershipLevel` và các field `attributeInts.*`/`attributeLongs.*` liên quan (VD `attributeInts.LoyaltyPoint`, `attributeLongs.MembershipIssuedAt`).
- `attributeDoubles`, `attributeBooleans`, `attributeTimestamps` hiện luôn rỗng trong dữ liệu thực tế — tránh dùng trừ khi solution có lý do rõ ràng.
- `enrichmentModels` là Array<Map> (mô hình eRFM) — mỗi phần tử có `model`, `erfm_recency`, `erfm_recency_score`, `erfm_frequency`, `erfm_frequency_score`, `erfm_monetary`, `erfm_monetary_score`, `erfm_recency_event`, `erfm_recency_event_score`, `erfm_frequency_event`, `erfm_frequency_event_score`. Dùng khi solution nói về phân khúc RFM/độ gắn kết khách hàng.

## Nguyên tắc naming segment (Gợi ý tên segment)

- Chỉ ký tự lowercase, số, dấu gạch dưới (`snake_case`).
- Không dấu tiếng Việt.
- Ngắn gọn, mô tả rõ mục đích.
- Không quá 40 ký tự.
- Ví dụ tốt: `high_value_customers_50m`, `gold_member_active`, `zalo_engaged_last_30_days`, `product_interest_apple`, `recent_buyers_last_6_months`.
- Ví dụ tệ: `KhachHangVIP` (viết hoa, dấu), `k1` (không rõ nghĩa), `customer_who_purchased_something_in_the_last_thirty_days` (quá dài).