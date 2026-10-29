**Ví dụ 1 — customer_information simple filter**:

```json
{
  "channelType": "Phone",
  "channelIdField": "phone",
  "targetModels": [
    {
      "model": "customer_information_<orgId>",
      "alias": "ci"
    }
  ],
  "rootModel": {
    "alias": "ci",
    "selectFields": [
      "phone",
      "memberLevel"
    ]
  },
  "selectFields": [
    {
      "modelAlias": "ci",
      "field": "phone"
    }
  ],
  "conditions": {
    "type": "and",
    "children": [
      {
        "field": "memberLevel",
        "operator": "in",
        "values": [
          "Vàng",
          "Kim Cương"
        ]
      }
    ]
  }
}
```

Note: `rootModel.selectFields` = [phone, memberLevel] — ĐỦ fields được reference.

**Ví dụ 2 — lead_info với multi-condition**:

```json
{
  "channelType": "Phone",
  "channelIdField": "phone",
  "targetModels": [
    {
      "model": "lead_info_<orgId>",
      "alias": "li"
    }
  ],
  "rootModel": {
    "alias": "li",
    "selectFields": [
      "phone",
      "classification",
      "state"
    ]
  },
  "selectFields": [
    {
      "modelAlias": "li",
      "field": "phone"
    }
  ],
  "conditions": {
    "type": "and",
    "children": [
      {
        "field": "classification",
        "operator": "equals",
        "values": [
          "Hot"
        ]
      },
      {
        "field": "state",
        "operator": "in",
        "values": [
          "Ready-to-Buy",
          "High Intent"
        ]
      }
    ]
  }
}
```

**Ví dụ 3 — purchase_history aggregation với COUNT**:

```json
{
  "channelType": "Phone",
  "channelIdField": "customerPhone",
  "targetModels": [
    {
      "model": "purchase_history_<orgId>",
      "alias": "ph"
    }
  ],
  "rootModel": {
    "alias": "ph",
    "selectFields": [
      "customerPhone"
    ]
  },
  "aliasFields": {
    "order_count": {
      "modelAlias": "ph",
      "function": "count",
      "field": "*"
    }
  },
  "selectFields": [
    {
      "modelAlias": "ph",
      "field": "customerPhone"
    },
    {
      "alias": "order_count"
    }
  ],
  "groupBys": [
    "ph.customerPhone"
  ],
  "havings": [
    {
      "field": {
        "function": "count",
        "field": "*"
      },
      "modelAlias": "ph",
      "operator": "gte",
      "values": [
        3
      ]
    }
  ]
}
```

Note: aliasFields dùng `field: "*"` → `rootModel.selectFields` chỉ cần `["customerPhone"]`.

**Ví dụ 4 — purchase_history với channel + city filter (rootModel PHẢI include đủ)**:

```json
{
  "channelType": "Phone",
  "channelIdField": "customerPhone",
  "targetModels": [
    {
      "model": "purchase_history_<orgId>",
      "alias": "ph"
    }
  ],
  "rootModel": {
    "alias": "ph",
    "selectFields": [
      "customerPhone",
      "channel",
      "storeCity"
    ]
  },
  "selectFields": [
    {
      "modelAlias": "ph",
      "field": "customerPhone"
    }
  ],
  "conditions": {
    "type": "and",
    "children": [
      {
        "field": "channel",
        "operator": "equals",
        "values": [
          "offline"
        ]
      },
      {
        "field": "storeCity",
        "operator": "in",
        "values": [
          "Hà Nội",
          "Hồ Chí Minh"
        ]
      }
    ]
  }
}
```

Note: `rootModel.selectFields` include CẢ 3: `customerPhone` (channelIdField), `channel` + `storeCity` (conditions).

**Ví dụ 5 — lead_info với like fuzzy matching**:

```json
{
  "channelType": "Phone",
  "channelIdField": "phone",
  "targetModels": [
    {
      "model": "lead_info_<orgId>",
      "alias": "li"
    }
  ],
  "rootModel": {
    "alias": "li",
    "selectFields": [
      "phone",
      "source",
      "modelInterest",
      "state"
    ]
  },
  "selectFields": [
    {
      "modelAlias": "li",
      "field": "phone"
    }
  ],
  "conditions": {
    "type": "and",
    "children": [
      {
        "field": "source",
        "operator": "in",
        "values": [
          "Web",
          "FB"
        ]
      },
      {
        "field": "modelInterest",
        "operator": "like",
        "values": [
          "%SUV%"
        ]
      },
      {
        "field": "state",
        "operator": "in",
        "values": [
          "Consideration",
          "High Intent",
          "Ready-to-Buy"
        ]
      }
    ]
  }
}
```

Note: dùng `like` cho `modelInterest` (case-insensitive) để match cả `"P_SUV"` và `"[P_SUV]"`.

**Ví dụ 6 — customer_profile so sánh giá trị 2 field (khách vừa mua hàng lần đầu: firstPurchasedAt = lastPurchasedAt)**:

```json
{
  "channelType": "UserId",
  "channelIdField": "userId",
  "targetModels": [
    {
      "model": "customer_profile_<orgId>",
      "alias": "cp"
    }
  ],
  "rootModel": {
    "alias": "cp",
    "selectFields": [
      "userId",
      "mainPhone",
      "firstPurchasedAt",
      "lastPurchasedAt"
    ]
  },
  "selectFields": [
    {
      "modelAlias": "cp",
      "field": "userId"
    },
    {
      "modelAlias": "cp",
      "field": "mainPhone"
    }
  ],
  "conditions": {
    "type": "and",
    "children": [
      {
        "field": "firstPurchasedAt",
        "operator": "eq",
        "values": [
          "lastPurchasedAt"
        ]
      },
      {
        "field": "firstPurchasedAt",
        "operator": "is_not_null",
        "values": []
      }
    ]
  }
}
```

Note: solution "vừa mới mua hàng lần đầu / chỉ mới mua 1 lần" = thời điểm mua đầu tiên TRÙNG thời điểm mua gần nhất → so sánh trực tiếp value của 2 field trong cùng bảng `customer_profile`: `field: "firstPurchasedAt"`, `operator: "eq"`, `values: ["lastPurchasedAt"]`. Luôn kèm điều kiện `firstPurchasedAt` `is_not_null` (`values: []`) để loại khách chưa từng mua hàng; 2 điều kiện kết hợp bằng AND (`type: "and"`). Tên field ở vế phải `values` viết TRỰC TIẾP, KHÔNG bọc dấu single quotes (bọc `''` là literal datetime, ví dụ `"'2026-09-01'"`), để được hiểu là tham chiếu cột thay vì chuỗi. Vì cả `firstPurchasedAt` lẫn `lastPurchasedAt` đều được reference trong `conditions` nên `rootModel.selectFields` PHẢI include CẢ 2 + `mainPhone` (channelIdField) — thiếu 1 trong 2 sẽ bị `UNKNOWN_IDENTIFIER`. KHÔNG dùng `gte_relative_day`/`lte_relative_day` cho phép so sánh này vì đây là so sánh field với field, không phải so với mốc thời gian.

**Ví dụ 7 — JOIN 2 bảng customer_profile + sms_interaction qua số điện thoại (khách hàng nữ đã click SMS trong 30 ngày qua)**:

```json
{
  "channelType": "Phone",
  "channelIdField": "mainPhone",
  "targetModels": [
    {
      "model": "customer_profile_<orgId>",
      "alias": "cp"
    },
    {
      "model": "sms_interaction_<orgId>",
      "alias": "sms"
    }
  ],
  "rootModel": {
    "alias": "cp",
    "selectFields": [
      "mainPhone",
      "gender"
    ]
  },
  "preFilters": {
    "cp": {
      "type": "and",
      "children": [
        {
          "field": "gender",
          "operator": "in",
          "values": [
            "female",
            "Nữ"
          ]
        }
      ]
    }
  },
  "joins": [
    {
      "modelAlias": "sms",
      "type": "inner",
      "on": "sms.phoneNumber = cp.mainPhone",
      "selectFields": [
        "phoneNumber",
        "event",
        "eventAt"
      ],
      "preFilter": {
        "type": "and",
        "children": [
          {
            "field": "phoneNumber",
            "operator": "not_equals",
            "values": [
              ""
            ]
          },
          {
            "field": "event",
            "operator": "equals",
            "values": [
              "clicked"
            ]
          },
          {
            "field": "eventAt",
            "operator": "gte",
            "values": [
              "toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 30 DAY)"
            ]
          }
        ]
      }
    }
  ],
  "selectFields": [
    {
      "modelAlias": "cp",
      "field": "mainPhone"
    }
  ],
  "groupBys": [
    "cp.mainPhone"
  ]
}
```

Note:
- `rootModel` = `customer_profile` (đối tượng được gắn tag) → `channelIdField` = `mainPhone`. `sms_interaction` là bảng join, khai báo trong `targetModels` với alias `sms`.
- Phần tử `joins` CHỈ dùng các key `modelAlias`, `type`, `on`, `selectFields`, `preFilter` — `modelAlias: "sms"` trỏ tới alias trong `targetModels`, KHÔNG dùng `model`/`alias`.
- `on` là chuỗi SQL thô nối bằng số điện thoại: `sms.phoneNumber` có trong `joins[0].selectFields`, `cp.mainPhone` có trong `rootModel.selectFields`.
- Điều kiện của `cp` (`gender`) đặt trong `preFilters.cp`; điều kiện của `sms` (`phoneNumber` khác rỗng, `event`, `eventAt`) đặt trong `joins[0].preFilter` — KHÔNG đặt điều kiện của `sms` trong `preFilters`.
- 1 khách có nhiều dòng trong `sms_interaction` → `groupBys: ["cp.mainPhone"]` để mỗi khách chỉ còn 1 dòng.
- "30 ngày qua" → `gte` với `toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 30 DAY)` (mốc 00:00 giờ Việt Nam, KHÔNG bọc dấu nháy), KHÔNG dùng `gte_relative_day`.

**Ví dụ 8 — purchase_history với khoảng ngày tuyệt đối (người dùng nói GMT+7, dữ liệu UTC)**:

Câu hỏi: "gắn tag khách hàng mua hàng từ 01/09/2026 đến 15/09/2026"

Quy đổi: `01/09/2026 00:00:00` GMT+7 = UTC `2026-08-31 17:00:00`; hết ngày 15/09 GMT+7 = `16/09 00:00:00` GMT+7 = UTC `2026-09-15 17:00:00` (dùng `lt`). Dùng `transCreatedAt` (DateTime), KHÔNG dùng `transDate` (Date, không quy đổi UTC được).

```json
{
  "channelType": "Phone",
  "channelIdField": "customerPhone",
  "targetModels": [
    {
      "model": "purchase_history_<orgId>",
      "alias": "ph"
    }
  ],
  "rootModel": {
    "alias": "ph",
    "selectFields": [
      "customerPhone",
      "transCreatedAt"
    ]
  },
  "preFilters": {
    "ph": {
      "type": "and",
      "children": [
        {
          "field": "transCreatedAt",
          "operator": "gte",
          "values": [
            "'2026-08-31 17:00:00'"
          ]
        },
        {
          "field": "transCreatedAt",
          "operator": "lt",
          "values": [
            "'2026-09-15 17:00:00'"
          ]
        }
      ]
    }
  },
  "selectFields": [
    {
      "modelAlias": "ph",
      "field": "customerPhone"
    }
  ],
  "groupBys": [
    "ph.customerPhone"
  ]
}
```

Note:
- Mốc người dùng nêu không có hậu tố = GMT+7 → TRỪ 7 giờ để ra UTC. Mốc có `Z` chỉ bỏ `Z`; mốc `+07:00` trừ 7 giờ rồi bỏ offset.
- "Đến hết ngày B" → `lt` với `B 17:00:00` UTC (= 00:00 ngày B+1 GMT+7), KHÔNG dùng `lte 'B 00:00:00'` hay `lte 'B'`.
- Giá trị chuỗi bọc dấu nháy đơn BÊN TRONG chuỗi JSON; KHÔNG dùng `gte_relative_day` cho mốc tuyệt đối.
- `transCreatedAt` dùng trong `preFilters` nên PHẢI có trong `rootModel.selectFields`.
