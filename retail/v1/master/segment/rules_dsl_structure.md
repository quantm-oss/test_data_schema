DSL Segment Rule STRICT SCHEMA:

## Top-level structure

DSL rule là 1 JSON object gồm 2 keys chính: `information` và `dslRule`.

```json
{
  "information": {
    "assignSegment": "",
    "assignGroup": "",
    "priority": 1,
    "color": "<hex>"
  },
  "dslRule": {
    "rule": <RuleCondition>,
    "status": "Inactive",
    "windowTime": {"type": "Day", "value": 30},
    "sort": [<optional sort items>],
    "assignSegment": "",
    "assignGroup": "",
    "priority": 0,
    "forceUpdate": true
  }
}
```

## Template values BẮT BUỘC (hardcode)

Các fields SAU đây LUÔN có giá trị cố định — TUYỆT ĐỐI không thay đổi:

| Field | Value | Ghi chú |
|---|---|---|
| `information.assignSegment` | `""` (empty string) | Luôn empty |
| `information.assignGroup` | `""` (empty string) | Luôn empty |
| `information.priority` | `1` (integer) | Luôn 0 |
| `dslRule.status` | `"Inactive"` | Luôn Inactive |
| `dslRule.windowTime` | `{"type": "Day", "value": 30}` | 30 ngày |
| `dslRule.assignSegment` | `""` (empty string) | Duplicate của `information` |
| `dslRule.assignGroup` | `""` (empty string) | Duplicate của `information` |
| `dslRule.priority` | `0` (integer) | Duplicate của `information` |
| `dslRule.forceUpdate` | `true` (boolean) | Luôn true |

## Dynamic values (LLM generate)

| Field | Rule |
|---|---|
| `information.color` | Hex pastel color, xem "QUY TẮC CHỌN MÀU" |
| `dslRule.rule` | Main logic — xem "RuleCondition" bên dưới |
| `dslRule.sort` | `[]` (empty array) nếu solution không yêu cầu sort; nếu có thì `[{"field": "<fieldName>", "order": "asc"/"desc"}]` |

## RuleCondition structure

`RuleCondition` là **oneOf**:

**Type 1 — Nested condition (AND/OR)**:
```json
{
  "type": "and" | "or",
  "children": [<RuleCondition>, ...]
}
```
- `children` phải có `minItems=1`
- Có thể nest nhiều lớp AND/OR

**Type 2 — Leaf condition**:
```json
{
  "field": "<fieldName>",
  "operator": "<operator>",
  "values": [...],
  "dataType": "String" | "Integer" | "Long" | "Double" | "Boolean" | "Timestamp"
}
```

### Leaf condition fields

**Required**:
- `field` (string): Tên field trong schema `unified_profile_model` (KHÔNG có prefix collection name — Segment chỉ có 1 nguồn duy nhất). Case-sensitive.
- `operator` (string): Phải nằm trong WHITELIST OPERATORS
- `values` (array): min 1 item
- `dataType` (string): enum `["String", "Integer", "Long", "Double", "Boolean", "Timestamp"]`

**Optional (nullable)**:
- `extra1` (string, nullable): Chỉ dùng với operator `relative_time` — enum `["Minute", "Hour", "Day", "Week", "Month", "Year"]`
- `extra2` (string, nullable): Mốc giờ cụ thể `HH:mm` — dùng với `relative_time` nếu cần

**Rules về extra1/extra2**:
- Với operator `relative_time`: `extra1` REQUIRED
- Với operator KHÁC (gte, lte, eq, ...): `extra1` và `extra2` = `null`

## Field references

- LUÔN dùng field name từ schema `unified_profile_model` (case-sensitive).
- KHÔNG có prefix collection name — Segment chỉ dùng 1 nguồn duy nhất (`unified_profile_model`, khác Tag có nhiều collection):
  - Solution ghi `totalPurchasedAmount` → DSL field ghi `totalPurchasedAmount`
  - Solution ghi `gender` → DSL field ghi `gender`
  - Solution ghi `lastPurchasedDays` → DSL field ghi `lastPurchasedDays`
- **Map field động — PHẢI GIỮ NGUYÊN dot-notation** (đây KHÔNG phải prefix collection, mà là key thật bên trong Map, bắt buộc phải có để trỏ đúng field trong ES):
  - `attributes.MembershipLevel`, `attributes.customerId`, `attributes.followedOA`
  - `attributeInts.LoyaltyPoint`, `attributeInts.EarnPoint`
  - `attributeLongs.MembershipIssuedAt`, `attributeLongs.lastInteractOA`
  - `lastSegmentKeys.PRODUCT_INTEREST`, `lastSegmentKeys.SocialCRM_CustomerGroup`
  - `fieldSourceMap.<fieldName>`, `fieldUpdatedAt.<fieldName>`
  - **TUYỆT ĐỐI KHÔNG bỏ phần key sau dấu chấm** (VD viết nhầm `attributes` thay vì `attributes.MembershipLevel` sẽ khiến query trỏ sai/rỗng).
- Array<Map> field (`enrichmentModels`): reference field con của phần tử phù hợp qua key con, VD `enrichmentModels.erfm_monetary_score` (kèm điều kiện lọc phần tử có `model = "erfm"` nếu cần phân biệt nhiều model).

## dataType mapping

Schema `unified_profile_model` dùng các types sau → DSL `dataType` mapping:

| Schema type | DSL dataType | Ví dụ fields | Ghi chú |
|---|---|---|---|
| `String` | `"String"` | `gender`, `name`, `city`, `attributes.MembershipLevel`, `lastSegmentKeys.PRODUCT_INTEREST` | Text/enum fields |
| `Long` (time field) | `"Timestamp"` | `createdAt`, `updatedAt`, `lastPurchasedAt`, `firstPurchasedAt`, `lastActivityAt`, `attributeLongs.MembershipIssuedAt`, `attributeLongs.lastInteractOA`, `lastSegmentAt.*`, `fieldUpdatedAt.*` | Epoch millis — Long nhưng là time → dùng Timestamp |
| `Long` (non-time) | `"Long"` | `totalPurchasedAmount`, `totalPurchasedCount`, `avgPurchasedAmount`, `lastPurchasedDays`, `attributeInts.LoyaltyPoint`, `attributeInts.EarnPoint`, `enrichmentModels.erfm_monetary`, `enrichmentModels.erfm_monetary_score` | Số nguyên/đếm/điểm số (KHÔNG phải time) |
| `Double` | `"Double"` | `firstPurchasedAmount`, `lastPurchasedAmount` | Giá trị 1 giao dịch đơn lẻ, KHÔNG dùng cho tổng chi tiêu |
| `Array<String>` | `"String"` | `userSegmentKeys`, `coreTagIds`, `sources`, `firstPurchasedProducts`, `lastPurchasedProducts` | Dùng với `contains_any` / `contains_all` |
| `Map[String,String]` | `"String"` | `attributes.*` | Reference qua key cụ thể: `attributes.MembershipLevel` |
| `Map[String,Long]` | `"Long"` hoặc `"Timestamp"` | `attributeInts.*`, `attributeLongs.*`, `lastSegmentAt.*`, `fieldUpdatedAt.*`, `coreTagIdsAudit.*` | Tùy semantic key cụ thể — xem mục "Phân biệt Long time vs non-time" bên dưới |

**Lưu ý**: `attributeDoubles`, `attributeBooleans`, `attributeTimestamps` luôn rỗng trong dữ liệu thực tế — tránh dùng trừ khi có lý do rõ ràng. `membership` (Object) luôn `null` — KHÔNG dùng.

### QUAN TRỌNG — Phân biệt Long time vs Long non-time

Cách nhận biết Long là time hay non-time trong `unified_profile_model`:
- Field có description chứa "epoch millis", "thời điểm", "thời gian" → **Timestamp**
- Field có tên kết thúc bằng `At` (`createdAt`, `updatedAt`, `lastPurchasedAt`, `firstActivityAt`, `lastActivityAt`, `lastTouchedAt`, `MembershipIssuedAt`, `lastInteractOA`) → **Timestamp**
- Field có tên là score, count, amount, point, days → **Long** (non-time), VD `totalPurchasedCount`, `avgPurchasedAmount`, `lastPurchasedDays`, `attributeInts.LoyaltyPoint`, `attributeInts.EarnPoint`

Ví dụ cụ thể:
- `attributeLongs.MembershipIssuedAt` (Long) → description "thời điểm cấp thẻ/hạng membership" → `dataType: "Timestamp"`
- `attributeInts.LoyaltyPoint` (Long) → description "số điểm loyalty hiện có" → `dataType: "Long"`
- `lastPurchasedDays` (Long) → description "số ngày kể từ lần mua gần nhất" (đếm ngày, KHÔNG phải epoch millis) → `dataType: "Long"`
- `enrichmentModels.erfm_monetary_score` (Long) → description "điểm số Monetary đã chuẩn hóa" → `dataType: "Long"`
- `enrichmentModels.updatedAt` (Long) → description "thời điểm mô hình được tính/cập nhật" → `dataType: "Timestamp"`