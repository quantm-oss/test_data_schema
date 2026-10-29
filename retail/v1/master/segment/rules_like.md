**LUÔN dùng `like` (case-insensitive) cho MỌI fuzzy matching.

**Whitelist có `like` và `not_like`**.

## Cách dùng like

`like` dùng cho text field khi cần match "chứa chuỗi con", "bắt đầu bằng", "kết thúc bằng":

**Values pattern**:
- `%keyword%` — chứa keyword ở bất kỳ vị trí
- `keyword%` — bắt đầu bằng keyword
- `%keyword` — kết thúc bằng keyword
- `keyword` — match chính xác keyword (không có % thì tương đương equals)

## Ví dụ — unified_profile_model use cases

**Match segment key chứa "apple" (userSegmentKeys — lưu ý field này là Array, xem thêm ở phần "Khi nào KHÔNG dùng like" cho cách xử lý đúng với contains_any; ví dụ dưới minh họa cho trường hợp field String đơn lẻ tương tự)**:

**Match phân khúc quan tâm sản phẩm chứa keyword (lastSegmentKeys.PRODUCT_INTEREST)**:

```json
{
  "field": "lastSegmentKeys.PRODUCT_INTEREST",
  "operator": "like",
  "values": ["%apple%"],
  "dataType": "String"
}
```

**Match tên khách hàng chứa "Nguyễn" (name)**:

```json
{
  "field": "name",
  "operator": "like",
  "values": ["%Nguyễn%"],
  "dataType": "String"
}
```

**Match hạng thành viên chứa keyword không phân biệt hoa/thường (attributes.MembershipLevel)**:

```json
{
  "field": "attributes.MembershipLevel",
  "operator": "like",
  "values": ["%gold%"],
  "dataType": "String"
}
```

**Match nguồn cập nhật hồ sơ chứa keyword (lastUpdatedSource)**:

```json
{
  "field": "lastUpdatedSource",
  "operator": "like",
  "values": ["%CustomerMembership%"],
  "dataType": "String"
}
```

## Khi nào KHÔNG dùng like

- **Enum values rõ ràng**: dùng `equals` / `in` thay vì `like`
  - `gender` = "male" → dùng `equals` (enum quan sát được: male, female, unknown)
  - `attributes.followedOA` = "true" → dùng `equals` (giá trị dạng chuỗi "true"/"false")

- **Số hoặc time**: dùng `gt`, `gte`, `lt`, `lte`, `relative_time`
  - `totalPurchasedAmount` >= 50000000 → dùng `gte`
  - `attributeInts.LoyaltyPoint` >= 1000 → dùng `gte`
  - `lastPurchasedAt` trong 30 ngày → dùng `relative_time`
  - `lastPurchasedDays` <= 30 → dùng `lte`

- **Array fields**: dùng `contains_any` / `contains_all`
  - `userSegmentKeys` chứa segment key cụ thể → dùng `contains_any`
  - `coreTagIds` chứa tag cụ thể → dùng `contains_any`
  - Nếu chỉ biết 1 phần của segment key (không biết chính xác toàn bộ chuỗi) và cần fuzzy match trên Array, cân nhắc dùng field thay thế dạng String tương đương nếu có (VD `lastSegmentKeys.PRODUCT_INTEREST` cho phân khúc quan tâm sản phẩm), vì `contains_any` yêu cầu match giá trị chính xác trong mảng, không hỗ trợ fuzzy như `like`.