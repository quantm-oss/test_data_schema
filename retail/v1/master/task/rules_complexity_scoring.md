Chấm điểm rule complexity dựa vào cấu trúc DSL.

## 3 tiers

- **low** (score 10-30): Rule đơn giản — 1-2 conditions, không aliasFields, filter cơ bản
- **medium** (score 31-70): Rule trung bình — multiple conditions, có time constraint, có aliasFields count
- **high** (score 71-100): Rule phức tạp — nested AND/OR, aggregations với havings, groupBys, complex time logic

## Scoring guide

### Low (10-30)

Đặc điểm:
- 1 preFilter với 1 condition
- Không có `conditions` top-level (hoặc empty)
- Không có `aliasFields` (hoặc empty)
- Không có `groupBys` / `havings`
- Filter đơn giản (tagId equals, customerId equals)

Ví dụ:
```json
{
  "preFilters": {"tag": {"type": "and", "children": [{"field": "tagId", "operator": "equals", "values": ["Retail_HOT"]}]}},
  "aliasFields": {},
  "conditions": {},
  "groupBys": [],
  "havings": []
}
```
→ Score 15-25

### Medium (31-70)

Đặc điểm:
- preFilter với 2-3 conditions (AND/OR)
- Có time constraint (`gte`/`lte` với biểu thức mốc thời gian)
- Có 'aliasFields' count đơn giản
- Filter với 'in' (multi-values)

Ví dụ:
```json
{
  "preFilters": {
    "tag": {
      "type": "and",
      "children": [
        {"field": "tagId", "operator": "in", "values": ["Retail_HOT"]},
        {"field": "createdAt", "operator": "gte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 180 DAY)"]}
      ]
    }
  },
  "aliasFields": {"dtf_tag_count": {"model": "tag", "function": "count", "field": "*"}}
}
```
→ Score 35-45

### High (71-100)

Đặc điểm:
- Nested logic (`or` chứa `and`, hoặc ngược lại)
- Multiple time constraints kết hợp
- `groupBys` + `havings` với aggregation
- Multiple `aliasFields` với sum/avg/max
- Complex string matching (multiple like)

Ví dụ:
```json
{
  "preFilters": {
    "tag": {
      "type": "or",
      "children": [
        {"field": "tagId", "operator": "in", "values": ["Retail_HOT", "Retail_WARM"]},
        {
          "type": "and",
          "children": [
            {"field": "tagCategory", "operator": "equals", "values": ["premium"]},
            {"field": "createdAt", "operator": "gte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 30 DAY)"]},
            {"field": "createdAt", "operator": "lte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 7 DAY)"]}
          ]
        }
      ]
    }
  },
  "groupBys": ["tag.customerId"],
  "havings": [
    {"field": {"function": "count", "field": "*"}, "operator": "gte", "values": [3]}
  ]
}
```
→ Score 75-90

**Lưu ý cấu trúc nested condition**: Mỗi phần tử trong `children` PHẢI là một trong 2 dạng — nested (`{"type": "and"|"or", "children": [...]}`) HOẶC leaf (`{"field": ..., "operator": ..., "values": [...]}`). TUYỆT ĐỐI KHÔNG trộn lẫn 2 dạng trong cùng 1 object (VD KHÔNG được viết `{"field": "type", "operator": "and", "children": [...]}` — đây là cấu trúc sai, không parse được).

## Consistency

- `ruleComplexity` và `ruleComplexityScore` PHẢI consistent:
  - `low` → score 10-30
  - `medium` → score 31-70
  - `high` → score 71-100

- Không được output `ruleComplexity: "low"` với `ruleComplexityScore: 80` (inconsistent).