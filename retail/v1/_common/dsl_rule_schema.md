# SCHEMA DSL RULE (STRICT COMPLIANCE)

Master phải sinh DSL Rule tuân thủ JSON schema sau:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "definitions": {
    "operator": {
      "type": "string",
      "enum": [
        "equals",
        "not_equals",
        "in",
        "not_in",
        "like",
        "not_like",
        "is_null",
        "is_not_null",
        "eq",
        "neq",
        "gt",
        "gte",
        "lt",
        "lte",
        "gte_relative_day",
        "lte_relative_day",
        "between_relative_day",
        "not_between_relative_day",
        "contains_any",
        "contains_all",
        "exist",
        "not_exist"
      ]
    },
    "logicalCondition": {
      "type": "object",
      "required": [
        "type",
        "children"
      ],
      "properties": {
        "type": {
          "type": "string",
          "enum": [
            "and",
            "or"
          ]
        },
        "children": {
          "type": "array",
          "items": {
            "type": "object",
            "required": [
              "field",
              "operator",
              "values"
            ],
            "properties": {
              "field": {
                "type": "string"
              },
              "operator": {
                "$ref": "#/definitions/operator"
              },
              "values": {
                "type": "array"
              },
              "modelAlias": {
                "type": "string"
              }
            }
          }
        }
      }
    },
    "aggFunction": {
      "type": "string",
      "enum": [
        "count",
        "count_distinct",
        "count_if",
        "sum",
        "sum_if",
        "avg",
        "avg_if",
        "max",
        "min",
        "group_uniq_array",
        "arg_max",
        "arg_min"
      ]
    }
  },
  "properties": {
    "channelType": {
      "type": "string",
      "enum": [
        "Phone",
        "Email",
        "ZaloUId",
        "ZaloUAId",
        "FbPsid",
        "UserId"
      ]
    },
    "channelIdField": {
      "type": "string"
    },
    "targetModels": {
      "type": "array",
      "items": {
        "type": "object",
        "required": [
          "model",
          "alias"
        ],
        "properties": {
          "model": {
            "type": "string"
          },
          "alias": {
            "type": "string"
          }
        }
      }
    },
    "rootModel": {
      "type": "object",
      "required": [
        "alias",
        "selectFields"
      ],
      "properties": {
        "alias": {
          "type": "string"
        },
        "selectFields": {
          "type": "array",
          "items": {
            "type": "string"
          }
        }
      }
    },
    "aliasFields": {
      "type": "object"
    },
    "selectFields": {
      "type": "array",
      "minItems": 1
    },
    "preFilters": {
      "type": "object"
    },
    "joins": {
      "type": "array",
      "items": {
        "type": "object",
        "required": [
          "modelAlias",
          "type",
          "on",
          "selectFields"
        ],
        "properties": {
          "modelAlias": {
            "type": "string",
            "description": "Alias của bảng join, phải khớp 1 alias trong targetModels"
          },
          "type": {
            "type": "string",
            "description": "Kiểu join, mặc định inner"
          },
          "on": {
            "type": "string",
            "description": "Chuỗi SQL thô, VD: sms.phoneNumber = cp.mainPhone"
          },
          "selectFields": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "preFilter": {
            "$ref": "#/definitions/logicalCondition"
          }
        }
      }
    },
    "conditions": {
      "$ref": "#/definitions/logicalCondition"
    },
    "groupBys": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "havings": {
      "type": "array"
    },
    "orderBys": {
      "type": "array"
    }
  },
  "required": [
    "channelType",
    "channelIdField",
    "targetModels",
    "rootModel",
    "selectFields"
  ]
}
```

## Các fields chính

- **channelType**: loại channel identifier (Phone/Email/Zalo/FB/UserId). KHÔNG BAO GIỜ dùng cặp `channelType: "Phone"` + `channelIdField: "mainPhone"` — khi định danh khách hàng qua `customer_profile.mainPhone` PHẢI dùng `channelType: "UserId"` + `channelIdField: "userId"` (`userId` PHẢI có trong `rootModel.selectFields` hoặc `joins[].selectFields`, và trong `selectFields` top-level). Các cặp khác (VD `Phone` + `phone`) giữ nguyên.
- **channelIdField**: tên field chứa giá trị identifier trong bảng
- **targetModels**: danh sách bảng dùng trong query
- **rootModel**: bảng gốc + `selectFields` (dùng cho subquery projection). Được phép thêm biểu thức `'toString(<field mảng>) AS <field>Text'` để lọc field kiểu Array bằng `like` (xem mục "Field kiểu mảng" ở `rules_rootmodel.md`); `joins[i].selectFields` cũng vậy.
- **selectFields**: fields cần return
- **conditions**: filter logic (and/or nested)
- **preFilters**: filter của `rootModel` (key = alias của `rootModel`)
- **joins**: các bảng nối thêm; điều kiện của bảng join đặt trong `joins[i].preFilter` của chính bảng đó
- **aliasFields**: aggregation functions với alias
- **groupBys**, **havings**, **orderBys**: SQL clauses tương ứng