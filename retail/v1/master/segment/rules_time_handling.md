## NGUYÊN TẮC HYBRID — 2 approaches cho time constraint

Master Segment tier hỗ trợ 2 approach xử lý time, đều dùng **offset ngày dạng số nguyên nhỏ** — KHÔNG dùng epoch millis tuyệt đối (13 chữ số) vì đây là phép tính LLM dễ sai, đã xác nhận qua nhiều lần test thực tế.

### RULE DECISION (STRICT — TUYỆT ĐỐI TUÂN THỦ)

**Nếu solution có date cụ thể theo format `DD/MM/YYYY`** → Approach 2 (`gte_relative_day` / `between_relative_day` với offset tính từ THỜI GIAN HIỆN TẠI)

**Nếu solution KHÔNG có `DD/MM/YYYY`** (chỉ có wording relative như "30 ngày gần đây", "6 tháng qua") → Approach 1 (`relative_time` với step count theo đơn vị Month/Week/...)

**Tại sao Approach 2 không còn dùng epoch millis tuyệt đối**: LLM tính sai epoch millis (13 chữ số) rất thường xuyên dù có bảng tra cứu hỗ trợ — đã xác nhận qua test thực tế (VD ngày trùng khớp bảng tham chiếu vẫn có lúc bị tính sai). Offset ngày (số nguyên nhỏ, phép trừ 2 ngày) là phép tính LLM thực hiện đáng tin cậy hơn nhiều.

### Time fields có sẵn trong schema unified_profile_model

Tất cả time field trong `unified_profile_model` là `Long` (epoch millis) — **không phân theo collection** vì Segment chỉ có 1 nguồn duy nhất.

| Field | Schema type | DSL dataType | Ghi chú |
|---|---|---|---|
| `createdAt` | Long (epoch millis) | `"Timestamp"` | Thời điểm tạo record |
| `updatedAt` | Long (epoch millis) | `"Timestamp"` | Thời điểm cập nhật record gần nhất |
| `birthday` | Long (epoch millis) | `"Timestamp"` | Ngày sinh đầy đủ, = 0 nếu chưa có dữ liệu |
| `firstActivityAt` | Long (epoch millis) | `"Timestamp"` | Thời điểm hoạt động đầu tiên |
| `lastActivityAt` | Long (epoch millis) | `"Timestamp"` | Thời điểm hoạt động gần nhất |
| `lastTouchedAt` | Long (epoch millis) | `"Timestamp"` | Thời điểm hồ sơ được cập nhật gần nhất (do job hệ thống) |
| `firstPurchasedAt` | Long (epoch millis) | `"Timestamp"` | Thời điểm đơn hàng đầu tiên, = 0 nếu chưa mua |
| `lastPurchasedAt` | Long (epoch millis) | `"Timestamp"` | Thời điểm đơn hàng gần nhất |
| `attributeLongs.MembershipIssuedAt` | Long (epoch millis, Map động) | `"Timestamp"` | Thời điểm cấp thẻ/hạng membership |
| `attributeLongs.lastInteractOA` | Long (epoch millis, Map động) | `"Timestamp"` | Thời điểm tương tác Zalo OA gần nhất |
| `lastSegmentAt.<KeyName>` | Long (epoch millis, Map động) | `"Timestamp"` | Thời điểm cập nhật gần nhất cho logic segment cụ thể |
| `fieldUpdatedAt.<fieldName>` | Long (epoch millis, Map động) | `"Timestamp"` | Thời điểm field cụ thể được cập nhật gần nhất |
| `enrichmentModels.updatedAt` | Long (epoch millis, trong Array<Map>) | `"Timestamp"` | Thời điểm mô hình eRFM được tính |

**Tất cả time fields** dùng `dataType: "Timestamp"` trong DSL rule.

**Field KHÔNG phải time dù có Long, tránh nhầm lẫn**: `lastPurchasedDays` (Long, đếm SỐ NGÀY recency — không phải epoch millis, dùng `dataType: "Long"` với `gt`/`lt`/`gte`/`lte` trực tiếp, KHÔNG dùng `relative_time`/`*_relative_day` cho field này).

---

## APPROACH 1 — `relative_time` (rolling window theo đơn vị lớn: Week/Month/Year)

Dùng khi solution KHÔNG có DD/MM/YYYY explicit, chỉ có wording tương đối dạng tuần/tháng/năm/giờ/phút.

### Semantics `values` và `extra1`

- `values = [startOffset, endOffset]` (integer array, 2 elements)
- Cả 2 tính bằng đơn vị của `extra1`
- `startOffset` <= `endOffset`
- Value `0` = hiện tại; value âm = quá khứ; value dương = tương lai
- `extra1` enum: `"Minute"`, `"Hour"`, `"Day"`, `"Week"`, `"Month"`, `"Year"`
- `extra2` = `null`

### Format DSL leaf

```json
{
  "field": "<timeField>",
  "operator": "relative_time",
  "values": [<startOffset>, <endOffset>],
  "dataType": "Timestamp",
  "extra1": "<Minute|Hour|Day|Week|Month|Year>",
  
}
```

### Test cases relative_time

**T4 — "khách hàng mua hàng trong 30 ngày gần đây"**:
```json
{"field": "lastPurchasedAt", "operator": "relative_time", "values": [-30, 0], "dataType": "Timestamp", "extra1": "Day", }
```

**T5 — "khách hàng hoạt động trong 6 tháng gần nhất"**:
```json
{"field": "lastActivityAt", "operator": "relative_time", "values": [-6, 0], "dataType": "Timestamp", "extra1": "Month", }
```

**T6 — "hồ sơ cập nhật tuần này"**:
```json
{"field": "updatedAt", "operator": "relative_time", "values": [0, 0], "dataType": "Timestamp", "extra1": "Week", }
```

**T7 — "khách hàng tương tác Zalo OA hôm nay"**:
```json
{"field": "attributeLongs.lastInteractOA", "operator": "relative_time", "values": [0, 0], "dataType": "Timestamp", "extra1": "Day", }
```

**T8 — "hồ sơ được tạo trong 3 giờ gần nhất"**:
```json
{"field": "createdAt", "operator": "relative_time", "values": [-3, 0], "dataType": "Timestamp", "extra1": "Hour", }
```

**T8b — "khách hàng được cấp thẻ thành viên trong 90 ngày qua"**:
```json
{"field": "attributeLongs.MembershipIssuedAt", "operator": "relative_time", "values": [-90, 0], "dataType": "Timestamp", "extra1": "Day", }
```

---

## APPROACH 2 — `gte_relative_day` / `between_relative_day` (absolute date → offset day)

Dùng khi solution có DD/MM/YYYY explicit. **KHÔNG dùng `gte`/`lte` với epoch millis tuyệt đối** — thay vào đó quy đổi ngày tuyệt đối thành offset SỐ NGÀY (số nguyên nhỏ) tính từ THỜI GIAN HIỆN TẠI, rồi dùng operator ngày tương đối.

### Semantics operator

- `gte_relative_day: [N]` → field >= (today - N ngày). Dùng khi end date của khoảng = THỜI GIAN HIỆN TẠI (offset_end = 0).
- `lte_relative_day: [N]` → field <= (today - N ngày).
- `between_relative_day: [A, B]` → field trong khoảng [today - B, today - A], với A < B (A = offset nhỏ hơn/gần hiện tại hơn, B = offset lớn hơn/xa quá khứ hơn).

### Cách tính offset — PHẢI làm theo đúng thứ tự sau

**Bước 1**: Tính `offset = số ngày dương lịch giữa THỜI GIAN HIỆN TẠI và ngày trong description` (làm tròn xuống, không cộng/trừ thêm 1). Ngày = hôm nay → offset = `0`. Ngày = N ngày trước hôm nay → offset = `N`.

**Bước 2**: Chọn operator dựa theo offset của ngày KẾT THÚC (end date) trong khoảng solution đưa ra:
- offset_end = `0` (end date = hôm nay hoặc rất gần hôm nay) → dùng `gte_relative_day: [offset_start]`.
- offset_end > `0` (end date là mốc quá khứ cụ thể, khác hôm nay) → dùng `between_relative_day: [offset_end, offset_start]` (offset_end nhỏ hơn đứng trước theo đúng thứ tự A<B).

**TUYỆT ĐỐI KHÔNG output epoch millis tuyệt đối (số 13 chữ số) cho bất kỳ leaf nào trong Approach 2** — chỉ output offset ngày (số nguyên nhỏ, thường 1-4 chữ số).

### Format DSL leaf

```json
{
  "field": "<timeField>",
  "operator": "gte_relative_day",
  "values": [<offset>],
  "dataType": "Timestamp"
}
```

hoặc combo AND cho `between_relative_day`:

```json
{
  "field": "<timeField>",
  "operator": "between_relative_day",
  "values": [<offset_end>, <offset_start>],
  "dataType": "Timestamp"
}
```

### Test cases (giả định THỜI GIAN HIỆN TẠI = 22/08/2026)

**T1 — "khách hàng mua hàng trong khoảng từ 01/08/2026 đến 31/08/2026"** (`lastPurchasedAt`, end date 31/08/2026 > hôm nay → coi như "tới hiện tại" nếu solution ý là "tháng này đang diễn ra"; nếu solution ghi rõ range đã qua, tính offset_end thực tế):
- offset_start (01/08/2026) = 21 ngày trước
- offset_end (31/08/2026) — nếu là ngày tương lai gần, coi offset_end ≈ 0 (tới hiện tại) → dùng `gte_relative_day: [21]`

```json
{"field": "lastPurchasedAt", "operator": "gte_relative_day", "values": [21], "dataType": "Timestamp"}
```

**T2 — "hồ sơ tạo trong khoảng từ 01/01/2026 đến 31/07/2026"** (`createdAt`, end date đã qua so với hiện tại 22/08/2026):
- offset_start (01/01/2026) ≈ 233 ngày trước
- offset_end (31/07/2026) ≈ 22 ngày trước (offset_end > 0, không phải hôm nay)
- → `between_relative_day: [22, 233]`

```json
{"field": "createdAt", "operator": "between_relative_day", "values": [22, 233], "dataType": "Timestamp"}
```

**T3 — "khách hàng tương tác trong khoảng từ 01/06/2025 đến 22/08/2026"** (`lastActivityAt`, end date = hôm nay):
- offset_start (01/06/2025) ≈ 447 ngày trước
- offset_end (22/08/2026) = 0 (= hôm nay)
- → `gte_relative_day: [447]`

```json
{"field": "lastActivityAt", "operator": "gte_relative_day", "values": [447], "dataType": "Timestamp"}
```

---

## MULTIPLE TIME FIELDS VỚI OR

Solution có 2+ time fields với OR → wrap OR ngoài, mỗi field 1 leaf riêng (relative_day hoặc relative_time tùy Approach).

### T9 — "hồ sơ tạo HOẶC cập nhật trong khoảng từ 01/06/2025 đến 22/08/2026 (hôm nay)"

```json
{
  "type": "or",
  "children": [
    {"field": "createdAt", "operator": "gte_relative_day", "values": [447], "dataType": "Timestamp"},
    {"field": "updatedAt", "operator": "gte_relative_day", "values": [447], "dataType": "Timestamp"}
  ]
}
```

### T10 — "khách hàng mua đơn đầu tiên HOẶC đơn gần nhất trong 30 ngày gần đây" (relative_time, không có DD/MM/YYYY)

```json
{
  "type": "or",
  "children": [
    {"field": "firstPurchasedAt", "operator": "relative_time", "values": [-30, 0], "dataType": "Timestamp", "extra1": "Day", },
    {"field": "lastPurchasedAt", "operator": "relative_time", "values": [-30, 0], "dataType": "Timestamp", "extra1": "Day", }
  ]
}
```

---

## KHÔNG CÓ TIME CONSTRAINT

### T11 — "khách hàng hạng Gold có điểm loyalty trên 1000"

Solution KHÔNG mention time → **KHÔNG thêm** time leaf condition vào rule.

```json
{
  "type": "and",
  "children": [
    {"field": "attributes.MembershipLevel", "operator": "equals", "values": ["Gold"], "dataType": "String"},
    {"field": "attributeInts.LoyaltyPoint", "operator": "gt", "values": [1000], "dataType": "Long"}
  ]
}
```

---

## EDGE CASE — Wording "hiện tại" không convert DD/MM/YYYY

### T12 — "trong tháng hiện tại" (Manager không convert)

Vì không có DD/MM/YYYY → dùng Approach 1 (`relative_time`):
```json
{"field": "lastPurchasedAt", "operator": "relative_time", "values": [0, 0], "dataType": "Timestamp", "extra1": "Month", }
```

### T13 — "hồ sơ tạo trong quý này" (Manager không convert)

```json
{"field": "createdAt", "operator": "relative_time", "values": [0, 0], "dataType": "Timestamp", "extra1": "Month", }
```

Note: `relative_time` không có `extra1: "Quarter"` — dùng `"Month"` với values `[-2, 0]` (3 tháng gần nhất).

---

## TUYỆT ĐỐI KHÔNG

- **Mix cả 2 approach trong 1 leaf**: Leaf hoặc dùng `relative_time` (có `extra1`), hoặc `gte_relative_day`/`lte_relative_day`/`between_relative_day` (không có `extra1`).
- **Dùng `relative_time` không có `extra1`**: `extra1` REQUIRED khi operator là `relative_time`.
- **Output epoch millis tuyệt đối (13 chữ số) cho bất kỳ leaf nào** — TUYỆT ĐỐI KHÔNG dùng operator `gte`/`lte` với giá trị epoch millis cho time field trong Approach 2. Chỉ dùng offset ngày (số nguyên nhỏ) với `gte_relative_day`/`lte_relative_day`/`between_relative_day`.
- **Suy diễn thời gian**: Nếu solution KHÔNG mention time → KHÔNG thêm time leaf (không tự bịa "30 ngày qua").
- **Đảo ngược thứ tự values trong `between_relative_day`**: PHẢI là `[offset_nhỏ_hơn, offset_lớn_hơn]` (A < B).
- **Set `extra1` khi dùng `gte_relative_day`/`lte_relative_day`/`between_relative_day`**: `extra1` = `null`.
- **Convert DD/MM/YYYY sai timezone khi tính offset**: Bắt buộc GMT+7 (Vietnam), không dùng UTC.
- **Dùng `relative_time`/`*_relative_day` cho `lastPurchasedDays`**: field này là số ngày đếm sẵn (recency), KHÔNG phải epoch millis — so sánh trực tiếp bằng `gt`/`lt`/`gte`/`lte` với `dataType: "Long"`.
- **Bịa field không có trong `unified_profile_model`**: Chỉ dùng các field đã liệt kê ở bảng trên, hoặc field Map động đã biết key cụ thể.