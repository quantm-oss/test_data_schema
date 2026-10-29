Phần "Trường dữ liệu tham chiếu" trong solution PHẢI liệt kê **TẤT CẢ các fields** đã xuất hiện trong "Điều kiện lọc".

## Nguyên tắc

1. **Đầy đủ**: Field nào xuất hiện trong "Điều kiện lọc" → PHẢI có trong "Trường dữ liệu tham chiếu".
2. **KHÔNG prefix collection**: Segment CHỈ có 1 nguồn duy nhất (`unified_profile_model`). Ghi tên field TRỰC TIẾP (VD: `totalPurchasedAmount`, không phải `purchase_history.totalCost`). KHÔNG có khái niệm nhiều collection như Tag.
3. **Đúng type**: Note type theo schema `unified_profile_model` (VD: `String`, `Long`, `Double`, `Array<String>`, `Map[String,String]`).
4. **Ngắn gọn**: Mô tả 1 dòng ngắn về cách dùng field, không copy nguyên description trong schema.
5. **Time field**: Toàn bộ field thời gian trong `unified_profile_model` là `Long` dạng epoch millis — note `(Long, epoch millis)` (VD: `createdAt`, `updatedAt`, `lastPurchasedAt`, `attributeLongs.MembershipIssuedAt`). KHÔNG có field nào là `DateTime`/`Date`/`Timestamp` trong schema này.
6. **Map field (dynamic key)**: Note đầy đủ key path cụ thể (VD: `attributes.MembershipLevel` không phải chỉ `attributes`; `attributeInts.LoyaltyPoint` không phải chỉ `attributeInts`). Key phải là key đã biết có thật trong schema — không bịa key mới.
7. **Array field**: Note dùng operator `contains_any`/`contains_all` (VD: `userSegmentKeys` (Array<String>): dùng contains_any để filter theo segment key).
8. **Cảnh báo amount field**: `firstPurchasedAmount`/`lastPurchasedAmount` chỉ phản ánh 1 giao dịch đơn lẻ, KHÔNG dùng để suy luận tổng chi tiêu — nếu solution cần tổng chi tiêu, dùng `totalPurchasedAmount`.

## Format mỗi field

```
- `fieldName` (Type): Mô tả ngắn cách dùng.
```

## Ví dụ đúng

```markdown
### Trường dữ liệu tham chiếu
- `totalPurchasedAmount` (Long): Tổng giá trị đã mua, cộng dồn tất cả đơn hàng, đơn vị VND.
- `lastPurchasedAt` (Long, epoch millis): Thời điểm mua hàng gần nhất, dùng filter khoảng thời gian.
- `mainPhone` (String): Số điện thoại chính của khách hàng, dùng làm channelIdField.
- `gender` (String): Giới tính — giá trị quan sát được: male, female, unknown.
- `attributes.MembershipLevel` (String): Hạng thành viên (membership tier), ví dụ Gold/Silver.
- `attributeInts.LoyaltyPoint` (Long): Số điểm loyalty hiện có của khách hàng.
- `userSegmentKeys` (Array<String>): Danh sách segment khách hàng hiện đang thuộc về, dùng contains_any để filter.
- `coreTagIds` (Array<String>): Danh sách tag chính đã gán cho khách hàng, dùng contains_any để filter.
- `attributeLongs.lastInteractOA` (Long, epoch millis): Thời điểm tương tác Zalo OA gần nhất.
```

## Ví dụ sai

```markdown
### Trường dữ liệu tham chiếu
- purchase_history.totalCost (Long): Chi phí đơn hàng.              ← SAI: field/prefix này thuộc ClickHouse (Tag), KHÔNG tồn tại trong unified_profile_model
- lead_info.classification (String): Phân loại lead.                ← SAI: field/prefix này thuộc ClickHouse (Tag), KHÔNG áp dụng cho Segment
- attributes (Map[String,String]): Thông tin mở rộng.                ← SAI: thiếu key cụ thể, phải ghi rõ attributes.MembershipLevel hoặc key tương ứng
- lastPurchasedAt (DateTime): Ngày mua gần nhất.                     ← SAI type: phải là (Long, epoch millis), không phải DateTime
- totalPurchasedAmount: Tổng chi tiêu.                                ← SAI: thiếu note type trong ngoặc
```