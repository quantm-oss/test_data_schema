Ví dụ DSL Segment

## VÍ DỤ 1 — Mua hàng trong khoảng thời gian tuyệt đối, end date = hôm nay (gte_relative_day)

**Giả định THỜI GIAN HIỆN TẠI = 22/08/2026.**

**Solution input** (từ Manager Segment):

### Summary
- Phân khúc khách hàng đã có giao dịch mua hàng trong khoảng từ 01/07/2026 đến 22/08/2026 (GMT+7).
### Mục tiêu
- Nhận diện những khách hàng phát sinh giao dịch gần đây để phục vụ chiến dịch marketing, tri ân, khuyến mãi.
### Điều kiện lọc (logic)
- `lastPurchasedAt` (Long, epoch millis) trong khoảng từ 01/07/2026 đến 22/08/2026 (GMT+7)
### Trường dữ liệu tham chiếu
- `lastPurchasedAt` (Long, epoch millis): Thời điểm mua hàng gần nhất.
- `mainPhone` (String): Số điện thoại chính của khách hàng.
### Gợi ý tên segment
'recent_purchasers_jul_aug_2026'

**Output DSL**:

```json
{
  "success": true,
  "name": "recent_purchasers_jul_aug_2026",
  "description": "Phân khúc khách hàng đã thực hiện giao dịch mua hàng gần đây (từ 01/07/2026 đến hiện tại), phục vụ chiến dịch marketing và tri ân khách hàng.",
  "dslRule": {
    "information": {
      "assignSegment": "",
      "assignGroup": "",
      "priority": 1,
      "color": "#ADD8E6"
    },
    "dslRule": {
      "rule": {
        "field": "lastPurchasedAt",
        "operator": "gte_relative_day",
        "values": [52],
        "dataType": "Timestamp",
        
        
      },
      "status": "Inactive",
      "windowTime": {"type": "Day", "value": 30},
      "sort": [],
      "assignSegment": "",
      "assignGroup": "",
      "priority": 0,
      "forceUpdate": true
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 15
}
```

**Note**:
- End date (22/08/2026) = THỜI GIAN HIỆN TẠI → offset_end = 0 → dùng `gte_relative_day`, KHÔNG dùng `between_relative_day`.
- offset_start = số ngày từ 01/07/2026 đến 22/08/2026 = 52 ngày.
- **TUYỆT ĐỐI KHÔNG** output epoch millis tuyệt đối (VD `1751328000000`) cho leaf này — chỉ output offset ngày `[52]`.
- 1 leaf (không wrap AND/OR vì chỉ 1 điều kiện) = 15 → `"low"`.

---

## VÍ DỤ 2 — Single condition aggregate (không wrap AND/OR, không có time)

**Solution input**:
### Summary
- Phân khúc khách hàng có tổng giá trị đã mua trên 50 triệu VND.
### Mục tiêu
- Xác định nhóm khách hàng có giá trị cao để triển khai chương trình VIP, upsell sản phẩm cao cấp.
### Điều kiện lọc (logic)
- `totalPurchasedAmount` (Long) >= 50,000,000 (VND)
### Trường dữ liệu tham chiếu
- `totalPurchasedAmount` (Long): Tổng giá trị đã mua, cộng dồn tất cả đơn hàng.
- `mainPhone` (String): Số điện thoại chính của khách hàng.
### Gợi ý tên segment
'high_value_customers_50m'

**Output DSL**:

```json
{
  "success": true,
  "name": "high_value_customers_50m",
  "description": "Phân khúc khách hàng có tổng giá trị đã mua từ 50 triệu VND trở lên, phục vụ chương trình chăm sóc VIP và upsell.",
  "dslRule": {
    "information": {
      "assignSegment": "",
      "assignGroup": "",
      "priority": 1,
      "color": "#FFDEAD"
    },
    "dslRule": {
      "rule": {
        "field": "totalPurchasedAmount",
        "operator": "gte",
        "values": [50000000],
        "dataType": "Long",
        
        
      },
      "status": "Inactive",
      "windowTime": {"type": "Day", "value": 30},
      "sort": [],
      "assignSegment": "",
      "assignGroup": "",
      "priority": 0,
      "forceUpdate": true
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 15
}
```

**Note**:
- Field không phải time → dùng `gte` bình thường (không liên quan Approach 1/2 xử lý thời gian).
- `totalPurchasedAmount` type Long trong schema → `dataType: "Long"`, value là số nguyên.
- 1 leaf + no nesting = 15 → `"low"`.

---

## VÍ DỤ 3 — Hạng thành viên + Loyalty (Map động, AND + equals, không có time)

**Solution input**:
### Summary
- Phân nhóm khách hàng có hạng thành viên Gold và điểm loyalty trên 1000.
### Mục tiêu
- Xác định nhóm thành viên cao cấp có mức độ gắn kết loyalty cao để ưu tiên chăm sóc VIP, tặng quà tri ân.
### Điều kiện lọc (logic)
- `attributes.MembershipLevel` (String) = "Gold"
- AND `attributeInts.LoyaltyPoint` (Long) > 1000
### Trường dữ liệu tham chiếu
- `attributes.MembershipLevel` (String): Hạng thành viên (membership tier).
- `attributeInts.LoyaltyPoint` (Long): Số điểm loyalty hiện có của khách hàng.
- `mainPhone` (String): Số điện thoại chính của khách hàng.
### Gợi ý tên segment
'gold_member_loyalty_1000'

**Output DSL**:

```json
{
  "success": true,
  "name": "gold_member_loyalty_1000",
  "description": "Phân nhóm khách hàng có hạng thành viên Gold và điểm loyalty tích lũy trên 1000, phục vụ chăm sóc VIP.",
  "dslRule": {
    "information": {
      "assignSegment": "",
      "assignGroup": "",
      "priority": 1,
      "color": "#FF6347"
    },
    "dslRule": {
      "rule": {
        "type": "and",
        "children": [
          {"field": "attributes.MembershipLevel", "operator": "equals", "values": ["Gold"], "dataType": "String"},
          {"field": "attributeInts.LoyaltyPoint", "operator": "gt", "values": [1000], "dataType": "Long"}
        ]
      },
      "status": "Inactive",
      "windowTime": {"type": "Day", "value": 30},
      "sort": [],
      "assignSegment": "",
      "assignGroup": "",
      "priority": 0,
      "forceUpdate": true
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 24
}
```

**Note**:
- `attributes.MembershipLevel` và `attributeInts.LoyaltyPoint` là key thật trong Map động — PHẢI giữ nguyên dot-notation, KHÔNG được rút gọn thành `MembershipLevel`/`LoyaltyPoint`.
- Không có time constraint → không thêm leaf thời gian.
- 2 leaves (20) + 1 level AND (+0) + 2 field trong Map động (+4) = 24 → `"low"`.

---

## VÍ DỤ 4 — Segment key + thời gian tương đối wording (Array field + relative_time, Approach 1)

**Solution input**:
### Summary
- Phân nhóm khách hàng thuộc segment quan tâm sản phẩm Apple, có hoạt động trong 30 ngày gần đây.
### Mục tiêu
- Đánh giá nhóm khách hàng quan tâm Apple đang hoạt động tích cực để triển khai chiến dịch remarketing sản phẩm liên quan.
### Điều kiện lọc (logic)
- `userSegmentKeys` (Array<String>) chứa giá trị liên quan "apple" (contains_any)
- AND `lastActivityAt` (Long, epoch millis) trong 30 ngày gần đây
### Trường dữ liệu tham chiếu
- `userSegmentKeys` (Array<String>): Danh sách key segment khách hàng hiện đang thuộc về.
- `lastActivityAt` (Long, epoch millis): Thời điểm hoạt động gần nhất.
- `mainPhone` (String): Số điện thoại chính của khách hàng.
### Gợi ý tên segment
'apple_interest_active_30_days'

**Output DSL**:

```json
{
  "success": true,
  "name": "apple_interest_active_30_days",
  "description": "Phân nhóm khách hàng thuộc segment quan tâm sản phẩm Apple, có hoạt động trong 30 ngày gần đây, phục vụ chiến dịch remarketing.",
  "dslRule": {
    "information": {
      "assignSegment": "",
      "assignGroup": "",
      "priority": 1,
      "color": "#87CEEB"
    },
    "dslRule": {
      "rule": {
        "type": "and",
        "children": [
          {"field": "userSegmentKeys", "operator": "contains_any", "values": ["PRODUCT-INTEREST_apple-product-enthusiasts-new", "PRODUCT-INTEREST-New_apple-product-enthusiasts-new-yeah"], "dataType": "String"},
          {
            "field": "lastActivityAt",
            "operator": "relative_time",
            "values": [-30, 0],
            "dataType": "Timestamp",
            "extra1": "Day",
            
          }
        ]
      },
      "status": "Inactive",
      "windowTime": {"type": "Day", "value": 30},
      "sort": [],
      "assignSegment": "",
      "assignGroup": "",
      "priority": 0,
      "forceUpdate": true
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 30
}
```

**Note**:
- Solution có "30 ngày gần đây" (KHÔNG có DD/MM/YYYY) → dùng Approach 1: `relative_time` với values=[-30, 0], extra1="Day".
- **Khác Ví dụ 1**: Approach 1 (`relative_time`) dùng cho wording tương đối thuần túy, KHÔNG cần tính offset từ ngày tuyệt đối — LLM chỉ cần đọc thẳng con số "30" trong câu hỏi, không phải tự tính chênh lệch ngày.
- `userSegmentKeys` là Array<String> → dùng `contains_any` với các giá trị segment key cụ thể đã biết chứa "apple" (không dùng `like` vì Array không hỗ trợ fuzzy match).
- 2 leaves (20) + 1 level AND (+0) + `contains_any` (+3) + `relative_time` (+3) = 26 → làm tròn 30 → `"low"`.

---

## VÍ DỤ 5 — Complex nested: RFM score + hạng thành viên OR + thời gian tương đối wording (nested AND/OR, Approach 1)

**Solution input**:
### Summary
- Phân nhóm khách hàng có điểm Monetary theo mô hình RFM trên 4, hạng thành viên Gold hoặc Diamond, hồ sơ được tạo trong 90 ngày qua.
### Mục tiêu
- Nhận diện nhóm khách hàng giá trị cao (điểm RFM cao + hạng thành viên cao cấp + hồ sơ còn mới) để đội chăm sóc khách hàng tập trung nguồn lực.
### Điều kiện lọc (logic)
- `enrichmentModels.erfm_monetary_score` (Long, trong phần tử có `model` = "erfm") >= 4
- AND (`attributes.MembershipLevel` (String) = "Gold" OR `attributes.MembershipLevel` = "Diamond")
- AND `createdAt` (Long, epoch millis) trong 90 ngày gần đây
### Trường dữ liệu tham chiếu
- `enrichmentModels.erfm_monetary_score` (Long): Điểm số Monetary đã chuẩn hóa theo mô hình eRFM.
- `attributes.MembershipLevel` (String): Hạng thành viên.
- `createdAt` (Long, epoch millis): Thời điểm tạo record.
- `mainPhone` (String): Số điện thoại chính của khách hàng.
### Gợi ý tên segment
'high_rfm_premium_members'

**Output DSL**:

```json
{
  "success": true,
  "name": "high_rfm_premium_members",
  "description": "Phân nhóm khách hàng có điểm Monetary theo mô hình RFM trên 4, hạng thành viên Gold hoặc Diamond, hồ sơ được tạo trong 90 ngày gần đây — nhóm khách hàng giá trị cao ưu tiên chăm sóc.",
  "dslRule": {
    "information": {
      "assignSegment": "",
      "assignGroup": "",
      "priority": 1,
      "color": "#DDA0DD"
    },
    "dslRule": {
      "rule": {
        "type": "and",
        "children": [
          {"field": "enrichmentModels.erfm_monetary_score", "operator": "gte", "values": [4], "dataType": "Long"},
          {
            "type": "or",
            "children": [
              {"field": "attributes.MembershipLevel", "operator": "equals", "values": ["Gold"], "dataType": "String"},
              {"field": "attributes.MembershipLevel", "operator": "equals", "values": ["Diamond"], "dataType": "String"}
            ]
          },
          {
            "field": "createdAt",
            "operator": "relative_time",
            "values": [-90, 0],
            "dataType": "Timestamp",
            "extra1": "Day",
            
          }
        ]
      },
      "status": "Inactive",
      "windowTime": {"type": "Day", "value": 30},
      "sort": [],
      "assignSegment": "",
      "assignGroup": "",
      "priority": 0,
      "forceUpdate": true
    }
  },
  "ruleComplexity": "high",
  "ruleComplexityScore": 75
}
```

**Note**:
- `attributes.MembershipLevel` OR wrap trong AND ngoài — nested 1 level.
- `enrichmentModels.erfm_monetary_score` nằm trong Array<Map> — reference trực tiếp field con.
- `createdAt` type Long (epoch millis) → "90 ngày gần đây" (wording tương đối, KHÔNG có DD/MM/YYYY) → dùng Approach 1 `relative_time` values=[-90, 0].
- 4 leaves (50) + 1 level nested OR (+10) + `relative_time` (+3) + 3 field trong Map/nested động (+6) = 69 → làm tròn lên 75.

---

## VÍ DỤ 6 — Absolute date range, end date KHÔNG phải hôm nay (between_relative_day)

**Giả định THỜI GIAN HIỆN TẠI = 22/08/2026.**

**Solution input**:
### Summary
- Phân khúc khách hàng có hồ sơ được tạo trong khoảng từ 01/01/2026 đến 31/03/2026 (Q1 2026).
### Mục tiêu
- Xác định nhóm khách hàng đăng ký trong quý 1/2026 để đánh giá hiệu quả chiến dịch acquisition đầu năm.
### Điều kiện lọc (logic)
- `createdAt` (Long, epoch millis) trong khoảng từ 01/01/2026 đến 31/03/2026 (GMT+7)
### Trường dữ liệu tham chiếu
- `createdAt` (Long, epoch millis): Thời điểm tạo record.
- `mainPhone` (String): Số điện thoại chính của khách hàng.
### Gợi ý tên segment
'q1_2026_signups'

**Output DSL**:

```json
{
  "success": true,
  "name": "q1_2026_signups",
  "description": "Phân khúc khách hàng có hồ sơ được tạo trong quý 1/2026 (01/01 - 31/03/2026), phục vụ đánh giá hiệu quả chiến dịch acquisition đầu năm.",
  "dslRule": {
    "information": {
      "assignSegment": "",
      "assignGroup": "",
      "priority": 1,
      "color": "#B0E0E6"
    },
    "dslRule": {
      "rule": {
        "field": "createdAt",
        "operator": "between_relative_day",
        "values": [144, 233],
        "dataType": "Timestamp",
        
        
      },
      "status": "Inactive",
      "windowTime": {"type": "Day", "value": 30},
      "sort": [],
      "assignSegment": "",
      "assignGroup": "",
      "priority": 0,
      "forceUpdate": true
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 15
}
```

**Note**:
- End date (31/03/2026) **KHÁC** THỜI GIAN HIỆN TẠI (22/08/2026) → offset_end > 0 → dùng `between_relative_day`, KHÔNG dùng `gte_relative_day`.
- offset_start (01/01/2026) = 233 ngày trước 22/08/2026.
- offset_end (31/03/2026) = 144 ngày trước 22/08/2026.
- `values = [144, 233]` — offset NHỎ HƠN (144, gần hiện tại hơn) đứng TRƯỚC, offset LỚN HƠN (233, xa quá khứ hơn) đứng SAU — đúng thứ tự `[A, B]` với A < B.
- **TUYỆT ĐỐI KHÔNG** output epoch millis tuyệt đối cho case này.
- 1 leaf + no nesting = 15 → `"low"`.

---

## TỔNG HỢP COVERAGE

| Ví dụ | Pattern | Field nhóm | Operators | Complexity |
|---|---|---|---|---|
| E1 | Absolute date, end=hôm nay (Approach 2, gte_relative_day) | Hành vi mua hàng (`lastPurchasedAt`) | gte_relative_day | low (15) |
| E2 | Single condition amount (không time) | Hành vi mua hàng aggregate (`totalPurchasedAmount`) | gte | low (15) |
| E3 | Enum equals + Map động 2 cấp (không time) | Hạng thành viên/Loyalty (`attributes.*`, `attributeInts.*`) | equals, gt | low (24) |
| E4 | Array contains_any + wording tương đối (Approach 1, relative_time) | Segment key + hoạt động (`userSegmentKeys`, `lastActivityAt`) | contains_any, relative_time | low (30) |
| E5 | Nested AND/OR + Array<Map> + wording tương đối (Approach 1, relative_time) | RFM + hạng thành viên (`enrichmentModels.*`, `attributes.*`, `createdAt`) | gte, equals, relative_time | high (75) |
| E6 | Absolute date, end date KHÁC hôm nay (Approach 2, between_relative_day) | Hồ sơ (`createdAt`) | between_relative_day | low (15) |

**Nguyên tắc cốt lõi Approach 2 (E1, E6)**: LUÔN quy đổi ngày tuyệt đối trong solution thành **offset ngày (số nguyên nhỏ)** tính từ THỜI GIAN HIỆN TẠI, dùng `gte_relative_day`/`between_relative_day`. TUYỆT ĐỐI KHÔNG output epoch millis tuyệt đối (13 chữ số) — đây là nguồn gốc lỗi sai lệch ngày/năm đã xảy ra thực tế nhiều lần khi LLM tự tính epoch.

---

**LLM phải adapt theo solution input thực tế, KHÔNG copy nguyên văn ví dụ.**