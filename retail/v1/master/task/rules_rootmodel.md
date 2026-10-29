'rootModel' là bảng chính (root) của DSL Task Rule. Task luôn dùng 'customer_tagging_staging' làm rootModel.

## Cấu trúc

```json
"rootModel": {
  "alias": "tag",
  "selectFields": ["tagId", "createdAt", "customerId", "channelId"]
}
```

## Nguyên tắc BẮT BUỘC

### 1. Alias cố định = 'tag'

Không dùng tên khác. Mọi Task rule alias rootModel = 'tag'.

### 2. selectFields — 4 fields cố định

BẮT BUỘC selectFields = **exactly 4 fields**:
- 'tagId'
- 'createdAt'
- 'customerId'
- 'channelId'

**Không thêm, không bớt**. Đây là contract cố định cho Task rule.

### 3. Không có 'preFilter' singular inside rootModel

Sai:
```json
"rootModel": {
  "alias": "tag",
  "selectFields": [...],
  "preFilter": { ... }   ← SAI: singular inside rootModel
}
```

Đúng: dùng 'preFilters' (plural) top-level:
```json
"rootModel": {
  "alias": "tag",
  "selectFields": ["tagId", "createdAt", "customerId", "channelId"]
},
"preFilters": {
  "tag": { ... }         ← ĐÚNG: plural top-level, nested theo alias
}
```

## Lý do

- Alias 'tag' + 4 selectFields = SQL subquery: 'SELECT tagId, createdAt, customerId, channelId FROM customer_tagging_staging_{orgId} [WHERE preFilters]) tag'
- INNER JOIN với 'cuf' sau đó match 'cuf.rootId = tag.customerId'
- Output aliases ('_customer_id_', 'dtf_channel', 'dtf_tag') reference từ 'tag.channelId', 'tag.tagId', và 'cuf.rootId'

## Field kiểu mảng (Array) — chuẩn DSL thống nhất

Áp dụng cho MỌI field có kiểu `Array(...)` hoặc `Array[...]` trong schema. Field kiểu text dùng thẳng `like`; field kiểu mảng chuyển về text trước rồi mới `like`:

1. Trong `selectFields` của CHÍNH bảng chứa mảng (`rootModel.selectFields` hoặc `joins[i].selectFields`) thêm đúng 1 biểu thức `"toString(<field>) AS <field>Text"` (alias LUÔN là `<field>Text`, ví dụ `featureClickText`, `tagsText`).
2. Lọc bằng `like` trên alias đó: `{"field": "<field>Text", "operator": "like", "values": ["%<text>%"]}`, đặt ở `preFilters.tag` (rootModel) hoặc `conditions`.
3. Alias `<field>Text` CHỈ để lọc: KHÔNG đưa vào `selectFields` top-level và `groupBys`. Biểu thức này được tính là đã có trong `selectFields` của bảng (không vi phạm quy tắc "thiếu field").
4. KHÔNG dùng `like`/`not_like` trực tiếp trên field mảng (BE báo ILLEGAL_TYPE_OF_ARGUMENT). KHÔNG dùng `contains_any`/`contains_all`/`extract_array_string_prefix` để lọc theo giá trị mảng (`extract_array_string_prefix` chỉ để TRÍCH giá trị ra cột output).
5. `<text>` lấy ĐÚNG như Solution nêu (không đổi hoa/thường). Nhiều giá trị: mỗi giá trị 1 điều kiện `like` riêng, nối `or` (hoặc `and` nếu Solution yêu cầu có đủ).

⚠️ Áp dụng với rootModel `customer_tagging_staging` (field mảng duy nhất: `sourceIds`): khi Solution lọc theo `sourceIds`, đây là NGOẠI LỆ DUY NHẤT của quy tắc "selectFields — 4 fields cố định": thêm đúng 1 biểu thức `"toString(sourceIds) AS sourceIdsText"` vào `rootModel.selectFields` (giữ nguyên 4 field cố định). Không thêm field nào khác.

VÍ DỤ ĐÚNG:

```json
"rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId", "toString(sourceIds) AS sourceIdsText"]},
"preFilters": {"tag": {"type": "and", "children": [
  {"field": "sourceIdsText", "operator": "like", "values": ["%<mã nguồn>%"]}
]}}
```

VÍ DỤ SAI: `{"field": "sourceIds", "operator": "contains_any", ...}` hoặc `{"field": "sourceIds", "operator": "like", ...}`.
