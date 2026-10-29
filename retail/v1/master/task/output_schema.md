LLM PHẢI output JSON hợp lệ theo schema sau. Chỉ 6 top-level fields.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "description", "dslRule", "configuration", "ruleComplexity", "ruleComplexityScore"],
  "additionalProperties": false,
  "properties": {
    "name": {
      "type": "string",
      "pattern": "^[a-z]+(_[a-z0-9]+)*$",
      "description": "Tên rule snake_case, theo pattern assign_[intent]_[target]_task, extract từ 'Gợi ý tên task' trong solution nếu có, hoặc tự đặt."
    },
    "description": {
      "type": "string",
      "description": "Mô tả business bằng tiếng Việt, dựa vào Summary + Mục tiêu của solution."
    },
    "dslRule": {
      "type": "object",
      "description": "DSL Task Rule core structure. LLM chỉ gen logic fields; BE inject metadata constants (limit, expireAfter, deduplication, version) sau.",
      "required": ["channelType", "channelIdField", "targetModels", "rootModel", "selectFields", "systemFields"],
      "properties": {
        "channelType": {
          "type": "string",
          "enum": ["Phone", "Email", "UserId"],
          "description": "Kênh định danh khách hàng để thực thi task. Với các bảng hiện có của tenant, chỉ Phone/Email/UserId có field hỗ trợ đáng tin cậy. KHÔNG dùng cặp Phone + mainPhone — định danh qua customer_profile.mainPhone PHẢI dùng channelType UserId + channelIdField userId."
        },
        "channelIdField": {
          "type": "string",
          "description": "Tên field chứa giá trị định danh, PHẢI là field có thật trong rootModel hoặc 1 model trong joins (ví dụ contactPhone, phoneNumber, phone, userId). KHÔNG dùng mainPhone — thay bằng userId kèm channelType UserId."
        },
        "systemFields": {
          "type": "object",
          "description": "Mapping field hệ thống cho downstream. rootUserId PHẢI trỏ tới tên alias output của field userId trong selectFields.",
          "required": ["rootUserId"],
          "properties": {
            "rootUserId": {
              "type": "string",
              "description": "Tên alias output chứa userId. Luôn = 'userId' vì selectFields luôn có {\"field\": \"cp.userId\", \"alias\": \"userId\"} hoặc {\"field\": \"le.curRootId\", \"alias\": \"userId\"}."
            }
          }
        },
        "targetModels": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "object",
            "required": ["model", "alias"],
            "properties": {
              "model": {"type": "string"},
              "alias": {"type": "string"}
            }
          },
          "description": "Danh sách TẤT CẢ bảng dùng trong query, bao gồm rootModel và các bảng join thêm (nếu có)."
        },
        "rootModel": {
          "type": "object",
          "required": ["alias", "selectFields"],
          "properties": {
            "alias": {
              "type": "string",
              "description": "Alias tự chọn, khớp với 1 phần tử trong targetModels."
            },
            "selectFields": {
              "type": "array",
              "items": {"type": "string"},
              "description": "Field cần lấy từ rootModel, dùng cho subquery projection. PHẢI include TẤT CẢ field được reference trong preFilter/conditions/groupBys/havings/orderBys/selectFields top-level (xem rules_rootmodel.md). Được phép thêm biểu thức 'toString(<field mảng>) AS <field>Text' (xem mục field mảng trong rules_rootmodel.md)."
            },
            "preFilter": {
              "type": "object",
              "description": "Điều kiện lọc field của CHÍNH rootModel, áp dụng TRƯỚC khi join."
            }
          }
        },
        "aliasFields": {
          "type": "object",
          "description": "Aggregate output. Key = output alias, value = aggregate definition với modelAlias, function, field, condition. Key PHẢI có marker trong selectFields[].alias."
        },
        "selectFields": {
          "type": "array",
          "minItems": 1,
          "description": "Field output top-level, dạng {modelAlias, field} hoặc {alias} cho aggregate marker. Hỗ trợ function 'extract_array_string_prefix' cho field Array(String).",
          "items": {
            "type": "object",
            "properties": {
              "modelAlias": {"type": "string"},
              "field": {"type": "string"},
              "alias": {"type": "string"},
              "function": {"type": "string", "description": "Chỉ hỗ trợ 'extract_array_string_prefix'."},
              "prefix": {"type": "string", "description": "Dùng kèm function 'extract_array_string_prefix'."}
            }
          }
        },
        "joins": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["modelAlias", "type", "on", "selectFields"],
            "properties": {
              "modelAlias": {"type": "string"},
              "type": {"type": "string", "enum": ["inner", "left"]},
              "on": {"type": "string", "description": "Chuỗi SQL thô, dạng '<aliasJoin>.<field> = <aliasRoot>.<field>'."},
              "selectFields": {"type": "array", "items": {"type": "string"}, "description": "Field cần lấy từ bảng join. Được phép thêm biểu thức 'toString(<field mảng>) AS <field>Text' (xem mục field mảng trong rules_rootmodel.md)."}
            }
          },
          "description": "Danh sách bảng join thêm vào rootModel. Bỏ qua hoàn toàn nếu single-table."
        },
        "conditions": {
          "type": "object",
          "description": "Điều kiện áp dụng SAU khi join, dùng cho field của bảng join. Nếu không cần, để {}."
        },
        "groupBys": {
          "type": "array",
          "items": {"type": "string"},
          "description": "Khi có joins, liệt kê tên field output (dùng alias nếu có) để tránh trùng dòng."
        },
        "havings": {
          "type": "array",
          "description": "HAVING clause. field.function hỗ trợ aggregate functions và 'alias' (reference output alias)."
        },
        "orderBys": {
          "type": "array",
          "description": "ORDER BY clause. Mỗi phần tử có alias và direction ('ASC'/'DESC')."
        },
        "detailsFields": {
          "type": "object",
          "description": "Metadata downstream, KHÔNG tạo SQL. BẮT BUỘC có trong output dưới dạng placeholder rỗng.",
          "properties": {
            "detailString": {"type": "array", "items": {"type": "string"}},
            "detailLong": {"type": "array", "items": {"type": "string"}},
            "detailDouble": {"type": "array", "items": {"type": "string"}},
            "detailBoolean": {"type": "array", "items": {"type": "string"}},
            "detailDate": {"type": "array", "items": {"type": "string"}},
            "detailArrayString": {"type": "array", "items": {"type": "string"}},
            "detailArrayLong": {"type": "array", "items": {"type": "string"}},
            "detailArrayDouble": {"type": "array", "items": {"type": "string"}}
          }
        }
      }
    },
    "configuration": {
      "type": "object",
      "required": ["taskTemplate"],
      "properties": {
        "taskTemplate": {
          "type": "object",
          "required": ["type", "description", "category", "subType", "templateConfigData", "priority"],
          "properties": {
            "type": {"type": "string", "description": "Chọn từ danh sách task_types khả dụng (runtime var)."},
            "description": {"type": "string", "maxLength": 500},
            "category": {"type": "string", "description": "Chọn từ danh sách task_categories khả dụng (runtime var)."},
            "subType": {
              "type": "string",
              "enum": ["Information", "Remind", "Opportunity", "Warning", "Risk"]
            },
            "templateConfigData": {
              "type": "object",
              "required": ["title", "content"],
              "properties": {
                "title": {"type": "string", "maxLength": 1000},
                "content": {"type": "string", "maxLength": 1000}
              }
            },
            "priority": {
              "type": "string",
              "enum": ["low", "normal", "high"],
              "description": "Default 'normal'"
            }
          }
        }
      }
    },
    "ruleComplexity": {"type": "string", "enum": ["low", "medium", "high"]},
    "ruleComplexityScore": {"type": "integer", "minimum": 0, "maximum": 100}
  }
}
```