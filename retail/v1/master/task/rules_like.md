Cho MỌI fuzzy string matching, dùng `like` (case-insensitive) hoặc `not_like`.

## Nguyên tắc — 3 nhóm String (đồng bộ với Manager Task)

**Nhóm 1 — ID hệ thống (exact match)**: `tagId`, `ruleId`, `customerId`, `requestId`, `dslResultId`
→ Dùng `equals` hoặc `in`. Đây là ID/mã định danh có nhiều giá trị khác nhau, cần match chính xác tuyệt đối để tránh nhầm lẫn giữa các ID gần giống nhau.

**Nhóm 2 — Enum ít giá trị (rủi ro case-sensitivity)**: `status`, `channelType`, `identifyStatus`, `tagCategory`, `tagSubCategory`, `sourceType`
→ Dùng `like` với values KHÔNG có wildcard `%` (VD `values: ["active"]`). Data thực tế đã xác nhận các field này lưu chữ thường (VD `status = "active"`, không phải `"ACTIVE"`) — dùng `equals`/`in` có rủi ro fail do sai case, `like` an toàn hơn vì case-insensitive nhưng vẫn match chính xác toàn bộ chuỗi.

**Nhóm 3 — String tự do**: `name` (customer_unified), `note` (customer_tagging_staging)
→ Dùng `like`/`not_like` với wildcard `%value%` — fuzzy match chứa chuỗi con.

## Ví dụ

**ĐÚNG**:
```json
{"field": "tagId", "operator": "equals", "values": ["Retail_HOT"]}
{"field": "status", "operator": "like", "values": ["active"]}
{"field": "channelType", "operator": "like", "values": ["Phone"]}
{"field": "name", "operator": "like", "values": ["%Nguyễn%"]}
```

**SAI**:
```json
{"field": "tagId", "operator": "like", "values": ["%Retail%"]}     ← nên equals vì tagId là ID hệ thống (Nhóm 1)
{"field": "status", "operator": "equals", "values": ["ACTIVE"]}     ← SAI CASE (giá trị thật chữ thường "active") + nên dùng like (Nhóm 2) để tránh rủi ro case tương tự
{"field": "status", "operator": "in", "values": ["ACTIVE", "PENDING"]}  ← tương tự, sai case và sai operator cho field enum
```

## Wildcard cho like

- Chứa (contains): `%value%` — dùng cho Nhóm 3 (text tự do)
- Bắt đầu (startswith): `value%`
- Kết thúc (endswith): `%value`
- Không wildcard: `value` — tương đương exact match nhưng case-insensitive, dùng cho Nhóm 2 (enum)

Không escape wildcard trong values.