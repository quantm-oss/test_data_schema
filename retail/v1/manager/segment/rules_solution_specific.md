Solution có giá trị KHI VÀ CHỈ KHI điều kiện lọc đủ cụ thể để phân nhóm ĐÚNG đối tượng — không phải phân tất cả khách hàng trong tenant.

## Phân biệt rõ "câu chung chung" vs "thiếu schema"

- "Câu chung chung" = câu hỏi của USER không đủ chi tiết để xây dựng điều kiện cụ thể.
- "Thiếu schema" = câu hỏi của user đã ĐỦ chi tiết, nhưng không có field phù hợp trong `unified_profile_model`. Trường hợp này thuộc `rules_no_hallucination.md` (không hallucinate).

## NGUYÊN TẮC VÀNG (đọc TRƯỚC mọi quy tắc khác)

Mặc định của bạn là CỐ GẮNG TẠO SOLUTION (matched=true). Reject (matched=false) chỉ là phương án cuối cùng khi thực sự không có cách nào map.

Nếu câu hỏi có BẤT KỲ thông tin cụ thể nào dưới đây → BẮT BUỘC trả matched=true, KHÔNG được reject vì "chung chung":

- **Ngưỡng số rõ ràng** (VD: "trên 5 triệu", "chi tiêu trên 50M", "trên 3 đơn", "điểm loyalty trên 1000").
- **Giới tính / demographics cụ thể** (VD: "khách nam", "khách nữ", "độ tuổi 25-35").
- **Hạng thành viên cụ thể** (VD: "Gold", "Silver", "membership Diamond").
- **Khoảng thời gian cụ thể** (VD: "30 ngày", "6 tháng gần nhất", "tháng này", "quý này").
- **Segment/Tag đã gán cụ thể** (VD: "thuộc segment PRODUCT-INTEREST", "có tag high_value_customer").
- **Nguồn dữ liệu cụ thể** (VD: "nguồn từ CustomerMembership", "cập nhật từ ZaloProfile").
- **Kênh liên hệ cụ thể** (VD: "có số điện thoại", "có Zalo", "có Facebook").
- **Chỉ số RFM cụ thể** (VD: "recency dưới 30 ngày", "monetary score cao", "frequency trên 5 lần").
- **Thuộc tính mở rộng cụ thể** (VD: "customerId bắt đầu bằng X", "đã follow Zalo OA").

Khi câu hỏi VỪA có yếu tố "chung chung" VỪA có yếu tố cụ thể, BẮT BUỘC ưu tiên map các yếu tố cụ thể đó sang field — KHÔNG được reject vì phần "chung chung" còn lại.

## Hướng dẫn map thông tin cụ thể sang field (unified_profile_model)

### Định danh & demographics
- "khách nam" → `gender` (String) = "male".
- "khách nữ" → `gender` (String) = "female". Chú ý: phần lớn dữ liệu là "unknown".
- "độ tuổi X-Y" → `ageGroup` (String), match theo nhóm tuổi có sẵn (VD "25-35").
- "sinh năm X" → `birthYear` (Long).
- "có số điện thoại" / "channel Phone" → `mainPhone` (String) IS NOT NULL/khác rỗng.
- "có Zalo" → `mainZaloUserId` (String) khác rỗng.
- "có Facebook" → `mainFacebookPsid` (String) khác rỗng.
- "sống tại thành phố X" → `city` (String).

### Hành vi mua hàng (aggregate, KHÔNG dùng field 1-giao-dịch)
- "chi tiêu trên X" / "tổng chi tiêu" → `totalPurchasedAmount` (Long) >= X. **KHÔNG dùng `firstPurchasedAmount`/`lastPurchasedAmount` cho tổng chi tiêu — 2 field đó chỉ phản ánh 1 giao dịch đơn lẻ.**
- "đã mua trên X đơn" / "số lần mua" → `totalPurchasedCount` (Long) >= X.
- "giá trị đơn hàng trung bình trên X" → `avgPurchasedAmount` (Long) >= X.
- "mua gần đây trong N ngày" / "recency" → `lastPurchasedDays` (Long) <= N, hoặc `lastPurchasedAt` (Long, epoch millis) trong khoảng thời gian.
- "chưa từng mua hàng" → `totalPurchasedCount` (Long) = 0 hoặc `firstPurchasedAt` (Long) = 0.
- "khách hàng mới" (đơn đầu tiên gần đây) → `firstPurchasedAt` (Long, epoch millis) trong khoảng thời gian gần.
- "đã mua sản phẩm X" → `lastPurchasedProducts` (Array<String>) hoặc `firstPurchasedProducts` (Array<String>), dùng contains_any.

### Hạng thành viên & Loyalty (Map động `attributes`/`attributeInts`/`attributeLongs`)
- "hạng thành viên Gold/Silver/Diamond" / "membership level" → `attributes.MembershipLevel` (String).
- "điểm loyalty trên X" → `attributeInts.LoyaltyPoint` (Long) >= X.
- "điểm tích lũy (earn point) trên X" → `attributeInts.EarnPoint` (Long) >= X.
- "cấp thẻ thành viên trong N ngày qua" → `attributeLongs.MembershipIssuedAt` (Long, epoch millis) trong khoảng thời gian.
- "mã khách hàng CRM/POS là X" → `attributes.customerId` (String).
- "đã follow Zalo OA" → `attributes.followedOA` (String) = "true".
- "tương tác Zalo OA gần đây" → `attributeLongs.lastInteractOA` (Long, epoch millis) trong khoảng thời gian. Lưu ý có biến thể `attributeLongs.LastInteractedOA` (viết hoa khác) — kiểm tra dữ liệu thực tế trước khi chọn key.

### Segment & Tag đã gán
- "thuộc segment X" / "đã được gán segment X" → `userSegmentKeys` (Array<String>), dùng contains_any với giá trị segment key cụ thể.
- "có tag X" → `coreTagIds` (Array<String>), dùng contains_any.
- "nhóm khách hàng theo Social CRM là X" → `lastSegmentKeys.SocialCRM_CustomerGroup` (String).
- "được phân công cửa hàng chăm sóc X" → `lastSegmentKeys.SocialCRM_AssignStore` (String).
- "quan tâm sản phẩm X" (phân khúc) → `lastSegmentKeys.PRODUCT_INTEREST` (String), dùng like nếu cần match rộng.
- "khách hàng ngừng hoạt động / lapsed" → `lastSegmentKeys.customer_lapsed` (String) khác rỗng.
- "segment X được cập nhật trong N ngày qua" → field tương ứng trong `lastSegmentAt.*` (Long, epoch millis).

### Mô hình phân tích RFM (`enrichmentModels`)
- "recency dưới X ngày" → phần tử `enrichmentModels` có `model="erfm"`, field `erfm_recency` (Long) <= X.
- "điểm recency cao" → `erfm_recency_score` (Long).
- "tần suất mua trên X lần" → `erfm_frequency` (Long) >= X.
- "điểm frequency cao" → `erfm_frequency_score` (Long).
- "chi tiêu theo mô hình RFM trên X" → `erfm_monetary` (Long) >= X.
- "điểm monetary cao" / "khách hàng giá trị cao theo RFM" → `erfm_monetary_score` (Long).
- "tương tác gần đây (không phải mua hàng)" → `erfm_recency_event` (Long).
- "tần suất tương tác cao" → `erfm_frequency_event` (Long) hoặc `erfm_frequency_event_score` (Long).

### Nguồn & metadata
- "hồ sơ được tạo/cập nhật từ nguồn X" → `lastUpdatedSource` (String) hoặc `firstUpdatedSource` (String), VD "PurchaseHistory", "CustomerMembership", "ZaloProfile", "Customer", "CRM".
- "field X được cập nhật từ nguồn Y" → `fieldSourceMap.<fieldName>` (String) = Y.
- "hồ sơ tạo/cập nhật trong N ngày qua" → `createdAt`/`updatedAt` (Long, epoch millis) trong khoảng thời gian.
- "hoạt động gần nhất trong N ngày qua" → `lastActivityAt` (Long, epoch millis) trong khoảng thời gian.
- "nguồn dữ liệu hợp nhất từ X" → `sources` (Array<String>), dùng contains_any.

## VÍ DỤ ÁP DỤNG NGUYÊN TẮC VÀNG

**Ví dụ 1**: "Phân khúc khách hàng có tổng chi tiêu trên 50 triệu"
- Phân tích: "50 triệu" là ngưỡng số cụ thể.
- Quyết định: matched=TRUE. Map: `totalPurchasedAmount` (Long) >= 50,000,000.

**Ví dụ 2**: "Phân nhóm khách hàng hạng Gold có điểm loyalty trên 1000"
- Phân tích: "Gold" là hạng thành viên cụ thể + "1000" là ngưỡng số cụ thể.
- Quyết định: matched=TRUE. Map: `attributes.MembershipLevel` = "Gold" AND `attributeInts.LoyaltyPoint` >= 1000.

**Ví dụ 3**: "Khách hàng chưa mua hàng trong 90 ngày qua"
- Phân tích: "90 ngày" là khoảng thời gian cụ thể, "chưa mua" ánh xạ recency.
- Quyết định: matched=TRUE. Map: `lastPurchasedDays` (Long) > 90.

**Ví dụ 4**: "Khách hàng thuộc segment quan tâm sản phẩm Apple"
- Phân tích: "quan tâm sản phẩm Apple" là segment/phân khúc cụ thể.
- Quyết định: matched=TRUE. Map: `userSegmentKeys` contains_any giá trị segment key liên quan tới Apple (VD `PRODUCT-INTEREST_apple-product-enthusiasts-new`), hoặc `lastSegmentKeys.PRODUCT_INTEREST` like '%apple%'.

**Ví dụ 5**: "Nhóm khách hàng đang trong giai đoạn phát triển chưa đạt mức độ trung thành cao"
- Phân tích: "đang phát triển" và "trung thành thấp" chung chung, KHÔNG có ngưỡng số / hạng thành viên / khoảng thời gian cụ thể.
- Quyết định: matched=FALSE. Gợi ý user bổ sung: ngưỡng điểm loyalty, tổng chi tiêu, hoặc khoảng thời gian hoạt động cụ thể.

## Chỉ trả 'matched: false' khi câu hỏi THỰC SỰ rỗng thông tin cụ thể

- "Phân nhóm khách hàng tiềm năng" (KHÔNG có ngưỡng, KHÔNG có segment, KHÔNG có hành vi cụ thể).
- "Nhóm khách hàng thân thiết" (KHÔNG có ngưỡng chi tiêu/đơn hàng, KHÔNG có time).
- "Khách hàng chất lượng cao" (hoàn toàn trừu tượng).
- "Nhóm tiền VIP" (KHÔNG có tiêu chí xác định "tiền VIP" là gì).
- Điều kiện lọc dự kiến quá chung → không filter được gì hữu ích.

## Khi đã quyết định trả 'matched: false'

- Ở phần "Đề xuất cải thiện", hỏi user bổ sung thông tin cụ thể nào.
- Đưa ra 2-3 gợi ý cụ thể dựa trên fields thực tế có sẵn trong `unified_profile_model`.
- VD: "Bạn có thể clarify: (1) Ngưỡng chi tiêu cụ thể (totalPurchasedAmount > bao nhiêu)? (2) Hạng thành viên nào (attributes.MembershipLevel = Gold/Silver/Diamond)? (3) Khoảng thời gian hoạt động nào (lastPurchasedDays, lastActivityAt)?".