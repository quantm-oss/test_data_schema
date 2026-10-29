## VÍ DỤ 1 — Chăm sóc khách có tag Retail_HOT (basic)

**Input description**:

### Summary
Gán task chăm sóc cho khách có tag "Retail_HOT" để đội Sales/CSKH liên hệ ngay.

### Điều kiện lọc (logic)
customer_tagging_staging.tagId = Retail_HOT

### Gợi ý tên task
'assign_hot_customer_interaction_task'

**Output LLM**:
```json
{
  "name": "assign_hot_customer_interaction_task",
  "description": "Tạo Task yêu cầu đội ngũ Sales/CSKH liên hệ ngay với khách hàng có mức độ quan tâm cao (tag Retail_HOT) nhằm tăng khả năng chuyển đổi và chốt đơn nhanh chóng.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "Retail_HOT", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [{"field": "tagId", "operator": "equals", "values": ["Retail_HOT"]}]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": [],
    "havings": [],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "Interaction_Hot_Customer",
      "description": "Task chăm sóc khách hàng có mức độ quan tâm cao nhằm tăng khả năng chuyển đổi và chốt đơn nhanh chóng.",
      "category": "care",
      "subType": "Opportunity",
      "templateConfigData": {
        "title": "Khách đang quan tâm cao cần chăm sóc ngay",
        "content": "Đội ngũ Sales/CSKH liên hệ ngay với khách hàng qua điện thoại hoặc nhắn tin trực tiếp để tăng khả năng chốt đơn."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 20
}
```
---

## VÍ DỤ 2 — Gán task cho customerId cụ thể

**Input description**:

### Summary
Gán task chăm sóc cho khách có customerId P1769593800230690449741506492 từ nguồn Social.

### Điều kiện lọc (logic)
customer_unified.rootId = "P1769593800230690449741506492"

**Output LLM**:
```json
{
  "name": "assign_social_lead_engagement_task",
  "description": "Tạo task chăm sóc cho khách hàng cụ thể (customerId P1769593800230690449741506492) từ nguồn Social nhằm theo dõi và quản lý tương tác hiệu quả.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "social_lead", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {},
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {
      "type": "and",
      "children": [{"field": "rootId", "operator": "equals", "values": ["P1769593800230690449741506492"]}]
    },
    "groupBys": [],
    "havings": [],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "customer",
      "description": "Task liên quan đến tương tác và quản lý khách hàng cụ thể.",
      "category": "care",
      "subType": "Information",
      "templateConfigData": {
        "title": "Chăm sóc khách hàng tiềm năng từ nguồn Social",
        "content": "Chăm sóc khách hàng tiềm năng từ nguồn Social qua kênh phù hợp để duy trì tương tác."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 15
}
```

---

## VÍ DỤ 3 — Nurturing Cold Lead (Automation)

**Input description**:
### Summary
Gán task automation gửi tin cho khách có tag "Cold_PAuto" để nurturing thành Hot.

### Điều kiện lọc (logic)
customer_tagging_staging.tagId = "Cold_PAuto"

**Output LLM**:
```json
{
  "name": "assign_cold_lead_nurturing_task",
  "description": "Tạo task automation gửi chương trình chăm sóc và khuyến mãi cho khách đang ở trạng thái Cold để nurturing thành Hot.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "Cold_PAuto", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [{"field": "tagId", "operator": "equals", "values": ["Cold_PAuto"]}]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": [],
    "havings": [],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "new_customer_interaction",
      "description": "Task automation gửi chương trình chăm sóc khách hàng và khuyến mãi để nurturing lead.",
      "category": "care",
      "subType": "Remind",
      "templateConfigData": {
        "title": "Gửi tin tương tác Automation Marketing",
        "content": "Hệ thống gửi tin chương trình chăm sóc khách hàng và khuyến mãi cho khách Cold để nurturing thành Hot."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 20
}
```

---

## VÍ DỤ 4 — Retail_HOT với time constraint (180 ngày)

**Input description**:
### Summary
Gán task ưu tiên chốt đơn cho khách có tag Retail_HOT trong 180 ngày qua.

### Điều kiện lọc (logic)
'customer_tagging_staging.tagId' = "Retail_HOT" AND customer_tagging_staging.createdAt trong 180 ngày gần đây

**Output LLM**:
```json
{
  "name": "assign_retail_hot_recent_interaction_task",
  "description": "Tạo task ưu tiên chốt đơn cho khách có tag Retail_HOT được gắn trong 180 ngày gần đây, có khả năng mua hàng cao.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "Retail_HOT", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [
          {"field": "tagId", "operator": "equals", "values": ["Retail_HOT"]},
          {"field": "createdAt", "operator": "gte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 180 DAY)"]}
        ]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": [],
    "havings": [],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "Interaction_Hot_Customer",
      "description": "Task chăm sóc khách có khả năng mua hàng cao, đã xem sản phẩm liên tục.",
      "category": "care",
      "subType": "Opportunity",
      "templateConfigData": {
        "title": "Ưu tiên chốt đơn ngay",
        "content": "Khách có khả năng mua hàng cao, xem sản phẩm liên tục trong 180 ngày gần đây. Ưu tiên liên hệ để chốt đơn."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "medium",
  "ruleComplexityScore": 35
}
```

---

## VÍ DỤ 5 — Interaction Hot_PAuto lead

**Input description**:
### Summary
Gán task Sales gọi ngay cho khách có tag Hot_PAuto để tránh bỏ lỡ cơ hội.

### Điều kiện lọc (logic)
customer_tagging_staging.tagId = Hot_PAuto

**Output LLM**:
```json
{
  "name": "assign_hot_lead_interaction_task",
  "description": "Tạo task cho đội Sale chăm sóc ngay khách hàng ở trạng thái Hot (tag Hot_PAuto) qua điện thoại hoặc nhắn tin để tránh bỏ lỡ cơ hội chốt đơn.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "Hot_PAuto", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [{"field": "tagId", "operator": "equals", "values": ["Hot_PAuto"]}]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": [],
    "havings": [],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "Interaction_Hot_Customer",
      "description": "Task interaction chăm sóc khách hàng ngay khi nhận được thông tin để tránh bỏ lỡ cơ hội chốt đơn.",
      "category": "care",
      "subType": "Opportunity",
      "templateConfigData": {
        "title": "Gọi ngay cho Hot Lead",
        "content": "Đội ngũ Sale chăm sóc khách hàng ngay khi nhận được thông tin qua điện thoại hoặc nhắn tin trực tiếp."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "low",
  "ruleComplexityScore": 20
}
```

---

## VÍ DỤ 6 — Multi-tag (operator `in`) + status enum (Nhóm 2, like)

**Input description**:
### Summary
Gán task follow-up cho khách có tag VIP_Gold hoặc VIP_Diamond, trạng thái tag đang active.

### Điều kiện lọc (logic)
customer_tagging_staging.tagId in ["VIP_Gold", "VIP_Diamond"] AND customer_tagging_staging.status active

**Output LLM**:
```json
{
  "name": "assign_vip_multi_tier_followup_task",
  "description": "Tạo task follow-up cho khách hàng thuộc 1 trong 2 hạng VIP_Gold hoặc VIP_Diamond, đang có trạng thái tag active, nhằm duy trì trải nghiệm chăm sóc nhất quán cho nhóm khách giá trị cao.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "vip_multi_tier", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [
          {"field": "tagId", "operator": "in", "values": ["VIP_Gold", "VIP_Diamond"]},
          {"field": "status", "operator": "like", "values": ["active"]}
        ]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": [],
    "havings": [],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "Interaction_Hot_Customer",
      "description": "Task follow-up cho nhóm khách hàng VIP nhiều hạng khác nhau.",
      "category": "care",
      "subType": "Opportunity",
      "templateConfigData": {
        "title": "Follow-up khách VIP đa hạng",
        "content": "Liên hệ khách hàng VIP (Gold hoặc Diamond) để duy trì trải nghiệm chăm sóc nhất quán, ưu tiên các ưu đãi đặc quyền phù hợp."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "medium",
  "ruleComplexityScore": 32
}
```

**Note**: `tagId in [...]` dùng khi thực sự có nhiều giá trị (khác Ví dụ 1/3/4/5 chỉ 1 giá trị nên dùng `equals`). `status like "active"` (Nhóm 2, không wildcard) — case-insensitive nhưng vẫn match chính xác toàn chuỗi. `like` KHÔNG có trong whitelist, PHẢI dùng `like`.

---

## VÍ DỤ 7 — Aggregation: khách được gắn tag Retail_HOT từ 3 lần trở lên trong 90 ngày (high complexity)

**Input description**:
### Summary
Gán task ưu tiên đặc biệt cho khách được gắn tag Retail_HOT từ 3 lần trở lên trong 90 ngày gần đây (tương tác lặp lại nhiều lần, mức độ quan tâm rất cao).

### Điều kiện lọc (logic)
customer_tagging_staging.tagId = "Retail_HOT" AND createdAt trong 90 ngày gần đây, GROUP BY customerId HAVING count >= 3

**Output LLM**:
```json
{
  "name": "assign_repeat_hot_customer_priority_task",
  "description": "Tạo task ưu tiên đặc biệt cho khách hàng được gắn tag Retail_HOT từ 3 lần trở lên trong 90 ngày gần đây — nhóm khách có mức độ quan tâm lặp lại rất cao, cần chăm sóc ngay để tránh bỏ lỡ cơ hội.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "Retail_HOT", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {
      "dtf_tag_count": {"model": "tag", "function": "count", "field": "*"}
    },
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [
          {"field": "tagId", "operator": "equals", "values": ["Retail_HOT"]},
          {"field": "createdAt", "operator": "gte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 90 DAY)"]}
        ]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": ["tag.customerId", "cuf.rootId", "tag.tagId", "tag.channelId"],
    "havings": [
      {"field": {"function": "count", "field": "*"}, "operator": "gte", "values": [3]}
    ],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "Interaction_Hot_Customer",
      "description": "Task ưu tiên đặc biệt cho khách hàng có tần suất tương tác Hot lặp lại nhiều lần.",
      "category": "care",
      "subType": "Opportunity",
      "templateConfigData": {
        "title": "Khách quan tâm lặp lại nhiều lần — ưu tiên chốt đơn",
        "content": "Khách hàng được gắn tag Retail_HOT từ 3 lần trở lên trong 90 ngày gần đây, thể hiện mức độ quan tâm rất cao. Liên hệ ưu tiên ngay để tránh bỏ lỡ cơ hội."
      },
      "priority": "high"
    }
  },
  "ruleComplexity": "high",
  "ruleComplexityScore": 78
}
```

## VÍ DỤ 8 — Warm lead + text search tên khách (preFilters trên cả 2 alias)

**Input description**:
### Summary
Gán task theo dõi cho khách tên chứa "Trần", đang có tag warm_lead_segment thuộc category customer_segmentation.

### Điều kiện lọc (logic)
customer_unified.name chứa "Trần" AND customer_tagging_staging.tagId = "warm_lead_segment" AND customer_tagging_staging.tagCategory = "customer_segmentation"

**Output LLM**:
```json
{
  "name": "assign_named_warm_segment_followup_task",
  "description": "Tạo task theo dõi cho khách hàng có tên chứa 'Trần', đang thuộc tag warm_lead_segment trong nhóm phân loại customer_segmentation, nhằm tiếp tục nurture trước khi trở thành cơ hội chốt đơn.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "warm_lead_segment", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [
          {"field": "tagId", "operator": "equals", "values": ["warm_lead_segment"]},
          {"field": "tagCategory", "operator": "like", "values": ["customer_segmentation"]}
        ]
      },
      "cuf": {
        "type": "and",
        "children": [
          {"field": "name", "operator": "like", "values": ["%Trần%"]}
        ]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": [],
    "havings": [],
    "orderBys": []
  },
  "configuration": {
    "taskTemplate": {
      "type": "new_customer_interaction",
      "description": "Task theo dõi khách hàng ở mức độ quan tâm trung bình, cần nurture thêm trước khi chuyển hóa thành cơ hội.",
      "category": "care",
      "subType": "Information",
      "templateConfigData": {
        "title": "Khách tiềm năng cần tiếp tục theo dõi",
        "content": "Gửi thông tin sản phẩm/ưu đãi phù hợp để duy trì sự quan tâm của khách hàng, chưa cần liên hệ trực tiếp gấp."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "medium",
  "ruleComplexityScore": 38
}
```

---

## VÍ DỤ 9 — Có yêu cầu sort tường minh (khách mới gắn tag gần đây nhất lên đầu)

**Input description**:
### Summary
Gán task liên hệ ngay cho khách có tag khach_hang_gia_tri_cao, ưu tiên xử lý khách được gắn tag gần đây nhất trước.

### Điều kiện lọc (logic)
customer_tagging_staging.tagId = "khach_hang_gia_tri_cao" AND status active, sắp xếp theo thời điểm gắn tag mới nhất trước

**Output LLM**:
```json
{
  "name": "assign_high_value_customer_recent_first_task",
  "description": "Tạo task liên hệ ngay cho khách hàng có tag khach_hang_gia_tri_cao, sắp xếp ưu tiên khách được gắn tag gần đây nhất lên đầu để đội Sales/CSKH xử lý theo đúng thứ tự thời gian.",
  "dslRule": {
    "systemFields": {"rootUserId": "_customer_id_", "taggingId": "khach_hang_gia_tri_cao", "labelId": ""},
    "targetModels": [
      {"model": "customer_tagging_staging_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "tag"},
      {"model": "customer_unified_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "cuf"}
    ],
    "rootModel": {"alias": "tag", "selectFields": ["tagId", "createdAt", "customerId", "channelId"]},
    "aliasFields": {},
    "selectFields": [
      {"modelAlias": "cuf", "field": "rootId", "alias": "_customer_id_"},
      {"modelAlias": "tag", "field": "channelId", "alias": "dtf_channel"},
      {"modelAlias": "tag", "field": "tagId", "alias": "dtf_tag"}
    ],
    "preFilters": {
      "tag": {
        "type": "and",
        "children": [
          {"field": "tagId", "operator": "equals", "values": ["khach_hang_gia_tri_cao"]},
          {"field": "status", "operator": "like", "values": ["active"]}
        ]
      }
    },
    "joins": [{"modelAlias": "cuf", "type": "inner", "on": "cuf.rootId = tag.customerId", "selectFields": ["curRootId as rootId"]}],
    "conditions": {},
    "groupBys": [],
    "havings": [],
    "orderBys": [{"alias": "tag.createdAt", "direction": "desc"}]
  },
  "configuration": {
    "taskTemplate": {
      "type": "Interaction_Hot_Customer",
      "description": "Task liên hệ khách hàng giá trị cao, ưu tiên xử lý theo thứ tự thời gian gắn tag gần nhất.",
      "category": "care",
      "subType": "Opportunity",
      "templateConfigData": {
        "title": "Liên hệ ngay — ưu tiên khách mới gắn tag",
        "content": "Đội ngũ Sales/CSKH liên hệ khách hàng theo thứ tự khách được gắn tag khach_hang_gia_tri_cao gần đây nhất trước, đảm bảo xử lý kịp thời."
      },
      "priority": "normal"
    }
  },
  "ruleComplexity": "medium",
  "ruleComplexityScore": 25
}
```