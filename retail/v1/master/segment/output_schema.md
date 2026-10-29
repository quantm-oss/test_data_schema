Output PHẢI là 1 JSON object hợp lệ theo schema strict sau. TUYỆT ĐỐI KHÔNG bọc code fence, KHÔNG thêm text giải thích ngoài JSON.

## Full JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "success": {
      "type": "boolean",
      "const": true,
      "description": "LUÔN true nếu build DSL thành công. Nếu không match, throw error thay vì set false."
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "description": "Tên segment snake_case, không dấu tiếng Việt. Từ 'Gợi ý tên segment' trong solution."
    },
    "description": {
      "type": "string",
      "minLength": 1,
      "description": "Mô tả segment bằng tiếng Việt có dấu. Từ 'Summary' + 'Mục tiêu' trong solution, viết cô đọng 1-2 câu."
    },
    "dslRule": {
      "type": "object",
      "description": "DSL Segment rule object với 2 keys: 'information' và 'dslRule'.",
      "properties": {
        "information": {
          "type": "object",
          "properties": {
            "assignSegment": {"type": "string", "const": ""},
            "assignGroup": {"type": "string", "const": ""},
            "priority": {"type": "integer", "const": 1},
            "color": {
              "type": "string",
              "pattern": "^#[0-9a-fA-F]{6}$",
              "description": "Hex pastel color, xem QUY TẮC CHỌN MÀU"
            }
          },
          "required": ["assignSegment", "assignGroup", "priority", "color"],
          "additionalProperties": false
        },
        "dslRule": {
          "type": "object",
          "properties": {
            "rule": {
              "description": "RuleCondition — oneOf: nested (type+children) hoặc leaf (field+operator+values+dataType)"
            },
            "status": {"type": "string", "const": "Inactive"},
            "windowTime": {
              "type": "object",
              "properties": {
                "type": {"type": "string", "const": "Day"},
                "value": {"type": "integer", "const": 30}
              },
              "required": ["type", "value"]
            },
            "sort": {
              "type": "array",
              "description": "Empty array [] nếu solution KHÔNG yêu cầu sort. Nếu có, mỗi item: {field, order: 'asc'|'desc'}.",
              "items": {
                "type": "object",
                "properties": {
                  "field": {"type": "string"},
                  "order": {"type": "string", "enum": ["asc", "desc"]}
                },
                "required": ["field", "order"]
              }
            },
            "assignSegment": {"type": "string", "const": ""},
            "assignGroup": {"type": "string", "const": ""},
            "priority": {"type": "integer", "const": 0},
            "forceUpdate": {"type": "boolean", "const": true}
          },
          "required": [
            "rule", "status", "windowTime", "sort",
            "assignSegment", "assignGroup", "priority", "forceUpdate"
          ]
        }
      },
      "required": ["information", "dslRule"]
    },
    "ruleComplexity": {
      "type": "string",
      "enum": ["low", "medium", "high"]
    },
    "ruleComplexityScore": {
      "type": "integer",
      "minimum": 0,
      "maximum": 100
    }
  },
  "required": ["success", "name", "description", "dslRule", "ruleComplexity", "ruleComplexityScore"],
  "additionalProperties": false
}
```

## Example output đúng format

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
          {
            "field": "attributes.MembershipLevel",
            "operator": "equals",
            "values": ["Gold"],
            "dataType": "String",
            
            
          },
          {
            "field": "attributeInts.LoyaltyPoint",
            "operator": "gt",
            "values": [1000],
            "dataType": "Long",
            
            
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
  "ruleComplexityScore": 25
}
```

## Field name trong DSL rule

Fields trong `rule.children[].field` là tên field từ schema `unified_profile_model`, ghi ĐÚNG NGUYÊN VĂN theo cách Manager Segment đã ghi trong solution (Manager Segment KHÔNG dùng prefix collection — đây là điểm khác với Tag/Task, vốn có nhiều bảng cần prefix). Ví dụ:
- Solution ghi `totalPurchasedAmount` → DSL field ghi `totalPurchasedAmount`
- Solution ghi `attributes.MembershipLevel` → DSL field ghi `attributes.MembershipLevel` (giữ nguyên dot-notation vì đây là key thật trong Map `attributes`, không phải prefix collection)
- Solution ghi `attributeInts.LoyaltyPoint` → DSL field ghi `attributeInts.LoyaltyPoint`
- Solution ghi `userSegmentKeys` → DSL field ghi `userSegmentKeys`

**Lưu ý quan trọng phân biệt 2 loại dot-notation**:
- `attributes.MembershipLevel`, `attributeInts.LoyaltyPoint`, `attributeLongs.MembershipIssuedAt`, `lastSegmentKeys.PRODUCT_INTEREST`, `fieldSourceMap.<fieldName>` — đây là **key thật bên trong Map động**, PHẢI giữ nguyên dot-notation khi ghi vào `field`.
- KHÔNG có khái niệm "prefix collection.fieldName" ở Segment (khác Tag) — vì Segment chỉ có 1 nguồn `unified_profile_model`.

## TUYỆT ĐỐI KHÔNG

- Bọc JSON trong code fence markdown (```json ... ```)
- Thêm text giải thích trước/sau JSON
- Set `success: false` — nếu không thể build DSL, throw error thay vì set false
- Bỏ qua bất kỳ field REQUIRED nào trong template
- Đổi template values (`assignSegment`, `assignGroup`, `priority`, `status`, `windowTime`, `forceUpdate`)
- Set `sort` khác `[]` nếu solution không yêu cầu sort
- Bịa field không tồn tại trong schema `unified_profile_model`
- Xóa nhầm dot-notation của key Map động (VD viết `MembershipLevel` thay vì `attributes.MembershipLevel` — sẽ làm sai field path, ES query fail)