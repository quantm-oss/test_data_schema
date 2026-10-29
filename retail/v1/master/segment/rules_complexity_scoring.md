## QUY TẮC CHẤM ĐIỂM RULE COMPLEXITY

Master Segment output có 2 fields liên quan complexity:

1. `ruleComplexity`: string enum — `"low"` | `"medium"` | `"high"`
2. `ruleComplexityScore`: integer 0-100

## Thang điểm ruleComplexityScore

| Range | Meaning | ruleComplexity |
|---|---|---|
| **10-30** | Đơn giản, 1-2 điều kiện đơn | `"low"` |
| **31-70** | Trung bình, 3-4 điều kiện, có kết hợp AND/OR đơn giản | `"medium"` |
| **71-100** | Phức tạp, nhiều điều kiện lồng nhau, logic nâng cao | `"high"` |

## Hướng dẫn chấm điểm chi tiết

### Yếu tố tăng điểm

**1. Số lượng leaf conditions**:
- 1 leaf: base 10 điểm
- 2 leaves: base 20 điểm
- 3 leaves: base 35 điểm
- 4-5 leaves: base 50 điểm
- 6+ leaves: base 70 điểm

**2. Độ sâu nesting (AND/OR nested)**:
- 0 level (chỉ leaf hoặc 1 level AND/OR): +0 điểm
- 1 level nested (VD OR chứa AND): +10 điểm
- 2 level nested (VD OR chứa AND chứa OR): +20 điểm
- 3+ level nested: +30 điểm

**3. Loại operators dùng**:
- Operators đơn giản (`equals`, `in`, `gt`, `lt`, `gte`, `lte`): +0 điểm/mỗi
- Fuzzy (`like`, `not_like`): +2 điểm/mỗi
- Time relative (`relative_time`, `gte_relative_day`, `lte_relative_day`, `between_relative_day`): +3 điểm/mỗi
- Array/Set (`contains_any`, `contains_all`): +3 điểm/mỗi
- Existence (`is_null`, `is_not_null`, `exist`, `not_exist`): +1 điểm/mỗi

**4. Multiple time fields với OR**:
- +5 điểm nếu có (VD 2 time ranges OR)

**5. Reference vào Map động nhiều cấp** (`attributes.*`, `attributeInts.*`, `attributeLongs.*`, `lastSegmentKeys.*`, `enrichmentModels.*`):
- +2 điểm/mỗi field (nesting logic truy vấn dynamic key phức tạp hơn field top-level)

### Yếu tố giảm điểm

- KHÔNG có yếu tố giảm điểm — chỉ tăng dần theo phức tạp

## Ví dụ chấm điểm

### Ví dụ 1 — Single field aggregate

Rule: `totalPurchasedAmount >= 50000000`
- 1 leaf: 10 điểm
- No nesting: +0
- Operator `gte`: +0
- **Total: 10 điểm** → `"low"`

### Ví dụ 2 — Two fields AND (Map động)

Rule: `attributes.MembershipLevel = "Gold" AND attributeInts.LoyaltyPoint gt 1000`
- 2 leaves: 20 điểm
- 1 level AND: +0 (không tính là nesting)
- Operator `equals`: +0
- Operator `gt`: +0
- 2 field trong Map động: +4 điểm
- **Total: 24 điểm** → `"low"`

### Ví dụ 3 — Recency + demographics

Rule: `gender = "female" AND lastPurchasedDays lte 30`
- 2 leaves: 20 điểm
- 1 level AND: +0
- Operators `equals`, `lte`: +0
- **Total: 20 điểm** → `"low"`

### Ví dụ 4 — Segment key + time relative

Rule: `userSegmentKeys contains_any ["PRODUCT-INTEREST_apple..."] AND lastActivityAt relative_time [-30, 0]`
- 2 leaves: 20 điểm
- 1 level AND: +0
- Operator `contains_any`: +3
- Operator `relative_time`: +3
- **Total: 26 điểm** → `"low"`

### Ví dụ 5 — Complex nested AND/OR

Rule: `attributeInts.LoyaltyPoint gte 1000 AND (attributes.MembershipLevel = "Gold" OR attributes.MembershipLevel = "Diamond") AND createdAt relative_time [-90, 0]`
- 4 leaves: 50 điểm
- 1 level nested (OR trong AND): +10 điểm
- Operator `gte`: +0
- Operator `equals` x2: +0
- Operator `relative_time`: +3 điểm
- 3 field trong Map động (`attributeInts.LoyaltyPoint`, `attributes.MembershipLevel` x2): +6 điểm
- **Total: 69 điểm** → `"medium"`

### Ví dụ 6 — Complex multi-condition RFM pattern

Rule: `totalPurchasedAmount gte 20000000 AND lastPurchasedDays lte 60 AND gender = "female" AND enrichmentModels chứa phần tử erfm_monetary_score gte 4`
- 4 leaves: 50 điểm
- 1 level AND: +0
- Operators `gte`, `lte`, `equals`, `gte`: +0
- 1 field trong Array<Map> động (`enrichmentModels.erfm_monetary_score`): +2 điểm
- **Total: 52 điểm** → `"medium"`

## Consistency check

**BẮT BUỘC** `ruleComplexity` và `ruleComplexityScore` phải consistent theo bảng:

| Score | ruleComplexity |
|---|---|
| 0-30 | `"low"` |
| 31-70 | `"medium"` |
| 71-100 | `"high"` |

Nếu score = 25 → ruleComplexity phải là `"low"` (không phải `"medium"`).

## TUYỆT ĐỐI KHÔNG

- Score < 0 hoặc > 100
- ruleComplexity không khớp với score range
- Chấm điểm dựa trên fields không tồn tại trong schema `unified_profile_model`