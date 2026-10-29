'rootModel.selectFields' là danh sách fields dùng cho **SUBQUERY PROJECTION** trong SQL. SQL converter build query dạng:
FROM (SELECT <rootModel.selectFields> FROM <table>) alias
WHERE <conditions>
GROUP BY <groupBys>
HAVING <havings>

Nếu 1 field xuất hiện trong 'conditions/preFilters/aliasFields/groupBys/havings/orderBys' NHƯNG KHÔNG có trong 'rootModel.selectFields' → SQL converter sẽ ném UNKNOWN_IDENTIFIER error.

**'rootModel.selectFields' PHẢI include TẤT CẢ fields của rootModel table được reference ở BẤT KỲ ĐÂU**:

1. 'conditions' (bao gồm children recursively)
2. 'preFilters' (nếu có filter cho rootModel alias)
3. 'aliasFields[*].field' và 'aliasFields[*].field2' (nếu aggregation trên rootModel)
4. 'groupBys' (bỏ prefix alias để lấy field name)
5. 'havings'
6. 'orderBys'
7. 'selectFields' top-level (nếu reference field của rootModel)

**VÍ DỤ SAI** (SQL converter fail):

```json
"rootModel": {"alias": "ph", "selectFields": ["customerPhone"]},
"conditions": {"type": "and", "children": [
  {"field": "channel", "operator": "equals", "values": ["offline"]},
  {"field": "storeCity", "operator": "in", "values": ["Hà Nội"]}
]}
```

→ Error: 'Unknown identifier 'channel'' — vì 'channel' không trong subquery projection.

**VÍ DỤ ĐÚNG**:

```json
"rootModel": {"alias": "ph", "selectFields": ["customerPhone", "channel", "storeCity"]},
"conditions": {"type": "and", "children": [
  {"field": "channel", "operator": "equals", "values": ["offline"]},
  {"field": "storeCity", "operator": "in", "values": ["Hà Nội"]}
]}
```

**NGUYÊN TẮC**: khi build 'rootModel.selectFields', LUÔN scan tất cả fields được reference trong DSL và include vào. Không có field thừa nhưng cũng KHÔNG được thiếu field nào.

## Field kiểu mảng (Array) — chuẩn DSL thống nhất

Áp dụng cho MỌI field có kiểu `Array(...)` hoặc `Array[...]` trong schema. Field kiểu text dùng thẳng `like`; field kiểu mảng chuyển về text trước rồi mới `like`:

1. Trong `selectFields` của CHÍNH bảng chứa mảng (`rootModel.selectFields` hoặc `joins[i].selectFields`) thêm đúng 1 biểu thức `"toString(<field>) AS <field>Text"` (alias LUÔN là `<field>Text`, ví dụ `featureClickText`, `tagsText`).
2. Lọc bằng `like` trên alias đó: `{"field": "<field>Text", "operator": "like", "values": ["%<text>%"]}`, đặt ở `preFilters.<alias rootModel>` (bảng gốc), `joins[i].preFilter` (bảng join, ưu tiên: lọc trước khi join) hoặc `conditions` (kèm `"modelAlias"` nếu có nhiều bảng).
3. Alias `<field>Text` CHỈ để lọc: KHÔNG đưa vào `selectFields` top-level và `groupBys`. Biểu thức này được tính là đã có trong `selectFields` của bảng (không vi phạm quy tắc "thiếu field").
4. KHÔNG dùng `like`/`not_like` trực tiếp trên field mảng (BE báo ILLEGAL_TYPE_OF_ARGUMENT). KHÔNG dùng `contains_any`/`contains_all`/`extract_array_string_prefix` để lọc theo giá trị mảng (`extract_array_string_prefix` chỉ để TRÍCH giá trị ra cột output).
5. `<text>` lấy ĐÚNG như Solution nêu (không đổi hoa/thường). Nhiều giá trị: mỗi giá trị 1 điều kiện `like` riêng, nối `or` (hoặc `and` nếu Solution yêu cầu có đủ).

VÍ DỤ ĐÚNG (bảng gốc `lead_info`):

```json
"rootModel": {"alias": "li", "selectFields": ["phone", "featureClick", "toString(featureClick) AS featureClickText"]},
"conditions": {"type": "and", "children": [
  {"field": "featureClickText", "operator": "like", "values": ["%<tên tính năng>%"]}
]}
```

VÍ DỤ SAI: `{"field": "featureClick", "operator": "like", ...}` (like trực tiếp trên mảng) hoặc `{"field": "featureClick", "operator": "contains_any", ...}`.
