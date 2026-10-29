```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "description": {
      "type": "string",
      "description": "Mô tả business tiếng Việt"
    },
    "dslRule": {
      "type": "string",
      "description": "DSL Rule stringified JSON. PHẢI bao gồm key 'detailsFields' (placeholder cố định theo yêu cầu hệ thống, xem rules_table_selection.md) ở cấp con trực tiếp, ngang hàng với 'groupBys'/'havings'/'orderBys'."
    },
    "name": {
      "type": "string",
      "pattern": "^[a-z]+(_[a-z0-9]+)*$"
    },
    "ruleComplexity": {
      "enum": [
        "low",
        "medium",
        "high"
      ]
    },
    "ruleComplexityScore": {
      "type": "integer",
      "minimum": 0,
      "maximum": 100
    },
    "configuration": {
      "type": "object",
      "properties": {
        "tagDefinition": {
          "type": "object",
          "properties": {
            "type": {
              "type": "string"
            },
            "description": {
              "type": "string"
            },
            "category": {
              "type": "string"
            },
            "subCategory": {
              "type": "string"
            }
          },
          "required": [
            "type",
            "description",
            "category",
            "subCategory"
          ]
        }
      },
      "required": [
        "tagDefinition"
      ]
    }
  },
  "required": [
    "description",
    "dslRule",
    "name",
    "ruleComplexity",
    "ruleComplexityScore",
    "configuration"
  ]
}
```

Chỉ xuất **chuỗi JSON hợp lệ** theo OUTPUT JSON SCHEMA. Không thêm giải thích.