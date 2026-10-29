DSL Task Rule có structure cố định như sau. Bạn PHẢI tuân thủ chính xác.

## Structure top-level của 'dslRule' (LLM gen)

```json
{
  "systemFields": {
    "rootUserId": "_customer_id_",
    "taggingId": "<tự quyết dựa vào context>",
    "labelId": ""
  },
  "targetModels": [
    {"model": "customer_tagging_staging_{orgId}", "alias": "tag"},
    {"model": "customer_unified_{orgId}", "alias": "cuf"}
  ],
  "rootModel": {
    "alias": "tag",
    "selectFields": ["tagId", "createdAt", "customerId", "channelId"]
  },
  "aliasFields": { ... optional ... },
  "selectFields": [
    {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
    {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
    {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
  ],
  "preFilters": { ... optional theo alias ... },
  "joins": [
    {
      "modelAlias": "cuf",
      "type": "inner",
      "on": "cuf.rootId = tag.customerId",
      "selectFields": ["curRootId as rootId"]
    }
  ],
  "conditions": { ... optional ... },
  "groupBys": [],
  "havings": [],
  "orderBys": []
}
```

## Nguyên tắc BẮT BUỘC

### 1. targetModels — 2 bảng cố định

Task luôn dùng cả 2:
- 'customer_tagging_staging_{orgId}' với alias 'tag' (bảng chính, rootModel)
- 'customer_unified_{orgId}' với alias 'cuf' (bảng join inner)

Model name pattern: '<table_name>_<orgId>'. VD:
- 'customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea'
- 'customer_unified_bdf66a48c3ebfaec10d2feb3492099ea'

### 2. rootModel — Fixed selectFields

```json
"rootModel": {
  "alias": "tag",
  "selectFields": ["tagId", "createdAt", "customerId", "channelId"]
}
```

**Bắt buộc** 4 fields: 'tagId', 'createdAt', 'customerId', 'channelId' — không thêm không bớt.

### 3. selectFields — Fixed 3 fields output

```json
"selectFields": [
  {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
  {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
  {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
]
```

**Bắt buộc** 3 output aliases: '_customer_id_', 'dtf_channel', 'dtf_tag'.

### 4. Joins

```json
"joins": [
  {
    "modelAlias": "cuf",
    "type": "inner",
    "on": "cuf.rootId = tag.customerId",
    "selectFields": ["curRootId as rootId"]
  }
]
```

Join key: 'cuf.rootId = tag.customerId'. Selected field: 'curRootId as rootId'.

### 5. preFilters — Filter trước JOIN (nested theo alias)

Format:
```json
"preFilters": {
  "tag": {
    "type": "and",
    "children": [
      {"field": "tagId", "operator": "equals", "values": ["<tag_value>"]}
    ]
  }
}
```

Dùng preFilters (plural, top-level) — KHÔNG dùng 'preFilter' (singular inside rootModel).

Filter theo 'tagId', 'createdAt', 'channelId', hoặc combinations.

### 6. conditions — Filter sau JOIN (top-level)

Format:
```json
"conditions": {
  "type": "and",
  "children": [
    {"field": "rootId", "operator": "equals", "values": ["<customer_id>"]}
  ]
}
```

Dùng 'conditions' khi cần filter theo field từ bảng joined ('cuf.rootId').

Nếu không có filter sau join → '"conditions": {}' (empty object).

### 7. aliasFields — Optional aggregation

Nếu solution cần count records (VD nhiều tagged events per customer):

```json
"aliasFields": {
  "dtf_tag_count": {
    "model": "tag",
    "function": "count",
    "field": "*"
  }
}
```

Nếu không cần → '"aliasFields": {}' (empty object).

### 8. orderBys — Mặc định RỖNG, chỉ thêm khi user yêu cầu rõ ràng

**Mặc định: `"orderBys": []`.** KHÔNG tự ý thêm sort nếu câu hỏi/description không có yêu cầu thứ tự cụ thể.

### 9. systemFields.taggingId — LLM tự quyết

Dựa vào context solution để chọn (các giá trị dưới đây CHỈ là ví dụ minh họa cách đặt tên, KHÔNG phải giá trị cố định phải dùng):

- Filter theo tagId cụ thể (VD 'Retail_HOT') → 'taggingId = "Retail_HOT"' (giống tagId)
- Filter theo customerId cụ thể (không có tagId filter) → 'taggingId = <generic_intent>' (VD '"social_lead"', '"customer_engagement"')
- Multi-tag hoặc complex filter → 'taggingId = <camelCase intent name>' (VD '"vip_recent_activity"')

**Nguyên tắc**: taggingId là ID định danh business intent của task, có thể match tagId hoặc tự đặt theo ngữ cảnh solution thực tế.

### 10. groupBys / havings — Thường empty

Task ít khi cần aggregation:
- '"groupBys": []'
- '"havings": []'

Chỉ dùng khi solution có yêu cầu tính toán (count tag events > N per customer).

## Fields KHÔNG được LLM gen (BE inject sau)

Những fields sau **BE sẽ tự inject**, LLM KHÔNG cần output:

- 'status', 'schedule', 'version', 'limit', 'debug', 'forceUpdate' — DSL metadata constants
- 'deduplication' — hardcoded template với 'dedupKeyStrategy=TYPE_ROOT_USER', windowTime.value=30

LLM chỉ focus vào **logic core**: systemFields, targetModels, rootModel, aliasFields, selectFields, preFilters, joins, conditions, groupBys, havings, orderBys.

### 11. groupBys khi có aggregation — PHẢI bao gồm đủ field non-aggregate trong selectFields

Khi DSL có `groupBys` (aggregation với havings), theo chuẩn SQL, MỌI field xuất hiện trong `selectFields` top-level mà KHÔNG phải aggregate PHẢI có mặt trong `groupBys`, nếu không ClickHouse sẽ raise lỗi `NOT_AN_AGGREGATE` (Code 215).

Vì `selectFields` là contract cố định 3 field (`cuf.rootId → _customer_id_`, `tag.channelId → dtf_channel`, `tag.tagId → dtf_tag`), khi có `groupBys`, PHẢI thêm cả 3 field gốc này vào `groupBys` cùng với field grouping chính (`tag.customerId`):

```json
"groupBys": ["tag.customerId", "cuf.rootId", "tag.tagId", "tag.channelId"]
```

Đã xác nhận qua data thực tế: mỗi `customerId` luôn có đúng 1 giá trị `channelId` cố định (không có trường hợp 1 khách hàng có nhiều channelId khác nhau) — nên thêm `tag.channelId` vào GROUP BY AN TOÀN, không làm sai lệch/phân mảnh kết quả `COUNT(*)`.

**Quy tắc chung**: bất kỳ khi nào DSL có `groupBys`, LUÔN thêm đủ 4 field này (`tag.customerId`, `cuf.rootId`, `tag.tagId`, `tag.channelId`) vào `groupBys`, bất kể field grouping chính do solution yêu cầu là gì — vì `selectFields` luôn cố định 3 field output không đổi.