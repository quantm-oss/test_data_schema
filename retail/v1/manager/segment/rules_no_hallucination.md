Solution CHỈ được reference đến các fields có trong schema đã cung cấp (load từ unifiedProfileUrl — schema `unified_profile_model`).

## Nguyên tắc bắt buộc

1. **Tồn tại**: Mỗi field trong "Điều kiện lọc" và "Trường dữ liệu tham chiếu" PHẢI tồn tại trong schema `unified_profile_model`.
2. **KHÔNG prefix collection**: Segment CHỈ có 1 nguồn duy nhất (`unified_profile_model`), KHÔNG có khái niệm nhiều collection như Tag. Ghi tên field TRỰC TIẾP: `totalPurchasedAmount`, `gender`, `mainPhone` — KHÔNG ghi prefix dạng `purchase_history.totalCost` hay `lead_info.classification` (đó là field của ClickHouse, dùng cho Tag, KHÔNG áp dụng cho Segment).
3. **Đúng tên**: Field reference theo đúng tên trong schema (case-sensitive, VD: `mainPhone` không phải `MainPhone` hay `main_phone`).
4. **Đúng type**: Toàn bộ field thời gian trong `unified_profile_model` là `Long` (epoch millis) — VD `createdAt`, `updatedAt`, `lastPurchasedAt`, `attributeLongs.MembershipIssuedAt`. KHÔNG được dùng format `DateTime`/`Date` cho các field này.
5. **Map access (dynamic key)**: Fields dạng `Map[String,...]` (`attributes`, `attributeInts`, `attributeLongs`, `attributeDoubles`, `attributeBooleans`, `attributeTimestamps`, `fieldSourceMap`, `fieldUpdatedAt`, `lastSegmentKeys`, `lastSegmentAt`, `lastSegmentPrioritys`, `coreTagIdsAudit`) → reference qua dot-notation `fieldName.KeyName` (VD: `attributes.MembershipLevel`, `attributeInts.LoyaltyPoint`, `lastSegmentKeys.PRODUCT_INTEREST`). Key PHẢI là 1 trong các key đã biết có thật trong schema — KHÔNG được tự bịa key mới.
6. **Array access**: Fields dạng `Array<String>` (`userSegmentKeys`, `coreTagIds`, `sources`, `firstPurchasedProducts`, `lastPurchasedProducts`) → dùng operator `contains_any`/`contains_all`.
7. **Cảnh báo amount fields**: `firstPurchasedAmount`/`lastPurchasedAmount` chỉ phản ánh 1 giao dịch đơn lẻ (đầu tiên/gần nhất) — KHÔNG dùng để suy luận "tổng chi tiêu". Dùng `totalPurchasedAmount` cho mục đích đó.
8. **`membership` (Object) luôn null** trong dữ liệu thực tế — KHÔNG dùng field này, dùng `attributes.MembershipLevel` + `attributeInts.*`/`attributeLongs.*` liên quan thay thế.

## Nguồn dữ liệu duy nhất

Segment CHỈ dùng schema `unified_profile_model` (Elasticsearch index `p_connext_user_profile_{orgId}`) — hồ sơ khách hàng 360° hợp nhất từ CRM, E-commerce, Social, Mobile App. Các field chính gồm:

- **Định danh & demographics**: `mainPhone`, `mainEmail`, `mainZaloUserId`, `mainFacebookPsid`, `name`, `gender`, `birthday`, `ageGroup`, `city`, `address`
- **Hành vi mua hàng**: `totalPurchasedAmount`, `totalPurchasedCount`, `avgPurchasedAmount`, `lastPurchasedAt`, `lastPurchasedDays`, `firstPurchasedAt`
- **Segment/Tag đã gán**: `userSegmentKeys` (Array), `coreTagIds` (Array), `lastSegmentKeys.*` (Map động)
- **Thuộc tính mở rộng (dynamic Map)**: `attributes.*` (String), `attributeInts.*` (Long), `attributeLongs.*` (Long, thường là timestamp)
- **Mô hình phân tích**: `enrichmentModels` (Array<Map>, chứa eRFM: `erfm_recency`, `erfm_frequency`, `erfm_monetary` và score tương ứng)
- **Metadata nguồn/thời gian cập nhật**: `fieldSourceMap.*`, `fieldUpdatedAt.*`, `createdAt`, `updatedAt`

## Khi user yêu cầu field không có trong schema

- KHÔNG được tự bịa field mới.
- KHÔNG được suy đoán field có tên tương tự.
- TRƯỚC KHI kết luận field không tồn tại, PHẢI kiểm tra kỹ cả field top-level LẪN các key con trong Map động (`attributes.*`, `attributeInts.*`, `attributeLongs.*`, `lastSegmentKeys.*`) — nhiều thông tin nghiệp vụ (VD hạng thành viên, điểm loyalty, phân khúc quan tâm sản phẩm) nằm trong các Map này, không phải field top-level.
- Trả `matched: false` với giải thích rõ field nào bị thiếu, chỉ SAU KHI đã kiểm tra kỹ toàn bộ schema (bao gồm cả dynamic key đã biết).
- Gợi ý field có sẵn trong `unified_profile_model` có thể dùng thay thế nếu có.

## Ví dụ áp dụng

**Sai**: User hỏi "khách VIP có điểm loyalty > 1000" → LLM output field `loyaltyPoints` (tên không khớp schema).

**Đúng**:
- Kiểm tra schema → không có field top-level `loyaltyPoints`, nhưng có `attributeInts.LoyaltyPoint` (Long) — đúng semantic "điểm loyalty".
- Dùng `attributeInts.LoyaltyPoint > 1000`.

**Sai**: User hỏi "khách có memberLevel Gold" → LLM output field `memberLevel` (sai tên, không kiểm tra kỹ Map `attributes`).

**Đúng**:
- Kiểm tra schema → không có field top-level `memberLevel`, nhưng có `attributes.MembershipLevel` (String) — đúng semantic "hạng thành viên".
- Dùng `attributes.MembershipLevel = "Gold"`.

**Sai**: User hỏi "khách đã tương tác Zalo OA trong 30 ngày qua" → LLM output field `zaloInteractionDate` (bịa field không tồn tại).

**Đúng**:
- Kiểm tra schema → không có field `zaloInteractionDate`, nhưng có `attributeLongs.lastInteractOA` (Long, epoch millis) — đúng semantic "thời điểm tương tác Zalo OA gần nhất".
- Dùng `attributeLongs.lastInteractOA >= (now - 30 days)`.

**Sai**: User hỏi "khách thuộc phân khúc doanh thu cao theo mô hình RFM" → LLM tự bịa field `revenueSegment`.

**Đúng**:
- Kiểm tra schema → không có field `revenueSegment`, nhưng có `enrichmentModels` (Array<Map>) chứa `erfm_monetary_score` — đúng semantic.
- Dùng điều kiện trên phần tử `enrichmentModels` có `model = "erfm"` và `erfm_monetary_score` ở ngưỡng cao.

**Trường hợp thực sự không có field tương đương**: User hỏi "khách có số CMND/CCCD là X" → Kiểm tra toàn bộ schema (bao gồm `attributes.*` đã biết: `customerId`, `MembershipLevel`, `followedOA`) → không có field nào lưu CMND/CCCD → Trả `matched: false`, giải thích rõ schema hiện tại không có field lưu thông tin định danh này, gợi ý user cung cấp tiêu chí khác (VD lọc theo `mainPhone`, `attributes.customerId`).