## Ví dụ 1 — PASS, 1 điều kiện đơn giản

**Câu hỏi**: "Khách hàng nữ có tổng chi tiêu trên 50 triệu"

**Output**:
```json
{
  "matched": true,
  "resolved_summary": "Khách hàng nữ, tổng chi tiêu trên 50 triệu VND",
  "unresolved_items": [],
  "briefSolution": {
    "objective": "Xác định nhóm khách hàng nữ có sức mua cao để triển khai chiến dịch marketing phù hợp",
    "dataSources": ["unified_profile_model"],
    "conditions": [
      {"id": "c1", "source": "unified_profile_model", "field": "gender", "operator": "=", "value": "female"},
      {"id": "c2", "source": "unified_profile_model", "field": "totalPurchasedAmount", "operator": ">", "value": 50000000}
    ],
    "exclusions": [],
    "timeRange": null,
    "combinationLogic": "AND",
    "latestRecordPolicy": "LATEST_ONLY",
    "aggregation": null,
    "assumptions": []
  }
}
```

## Ví dụ 2 — PASS, có mốc thời gian + loại trừ

**Câu hỏi**: "Khách hàng có số dư cao ở tài khoản chính, có giao dịch thẻ tín dụng trong 30 ngày qua, loại trừ khách đã đóng tài khoản"

**Output**:
```json
{
  "matched": true,
  "resolved_summary": "Khách hàng số dư cao, có giao dịch thẻ tín dụng 30 ngày qua, loại trừ tài khoản đã đóng",
  "unresolved_items": [],
  "briefSolution": {
    "objective": "Xác định khách hàng có số dư cao và đang hoạt động qua giao dịch thẻ tín dụng gần đây",
    "dataSources": ["unified_profile_model"],
    "conditions": [
      {"id": "c1", "source": "unified_profile_model", "field": "balance", "operator": ">", "value": 50000000},
      {"id": "c2", "source": "unified_profile_model", "field": "transaction_type", "operator": "=", "value": "CREDIT_CARD"}
    ],
    "exclusions": [
      {"id": "e1", "source": "unified_profile_model", "field": "status", "operator": "=", "value": "CLOSED"}
    ],
    "timeRange": {"field": "transaction_date", "last": "30d", "appliesTo": ["c2"]},
    "combinationLogic": "AND",
    "latestRecordPolicy": "LATEST_ONLY",
    "aggregation": null,
    "assumptions": ["Giả định ngưỡng số dư cao là trên 50 triệu VND do câu hỏi không nêu con số cụ thể — vui lòng xác nhận lại ngưỡng chính xác"]
  }
}
```

## Ví dụ 3 — FAIL, có 1 cụm không map được (KHÔNG có field nào khớp semantic)

**Câu hỏi**: "Khách hàng có điểm tín dụng ngân hàng trên 700, có số dư trên 50 triệu" (schema KHÔNG có field nào phản ánh "điểm tín dụng ngân hàng")

**Output**:
```json
{
  "matched": false,
  "resolved_summary": "Khách hàng có số dư trên 50 triệu",
  "unresolved_items": [
    {"user_phrase": "điểm tín dụng ngân hàng", "reason": "Không tìm thấy field/key nào trong schema unified_profile_model phản ánh khái niệm 'điểm tín dụng ngân hàng'"}
  ],
  "briefSolution": null
}
```

## Ví dụ 4 — PASS, field tồn tại nhưng giá trị/enum không rõ → RESOLVE bằng exist + assumptions (KHÔNG reject)

**Câu hỏi**: "Khách hàng thuộc nhóm VIP"

Schema có `attributes.vip` (String), mô tả: "Thông tin xác định trạng thái và hạng VIP của khách hàng" — khớp semantic nhưng không rõ giá trị cụ thể lưu trong đó.

**Output**:
```json
{
  "matched": true,
  "resolved_summary": "Khách hàng thuộc nhóm VIP (dựa trên trường attributes.vip có giá trị)",
  "unresolved_items": [],
  "briefSolution": {
    "objective": "Xác định nhóm khách hàng VIP để ưu tiên chăm sóc",
    "dataSources": ["unified_profile_model"],
    "conditions": [
      {"id": "c1", "source": "unified_profile_model", "field": "attributes.vip", "operator": "exist", "value": null}
    ],
    "exclusions": [],
    "timeRange": null,
    "combinationLogic": "AND",
    "latestRecordPolicy": "LATEST_ONLY",
    "aggregation": null,
    "assumptions": ["Giả định 'thuộc nhóm VIP' được hiểu là trường 'attributes.vip' tồn tại giá trị (không null) — vui lòng xác nhận lại nếu dữ liệu thực tế có giá trị khác (VD 'false') cần được loại trừ riêng"]
  }
}
```

## Ví dụ 5 — FAIL, câu hỏi hoàn toàn chung chung

**Câu hỏi**: "Phân nhóm khách hàng tiềm năng"

**Output**:
```json
{
  "matched": false,
  "resolved_summary": "",
  "unresolved_items": [
    {"user_phrase": "khách hàng tiềm năng", "reason": "Câu hỏi chưa cung cấp tiêu chí cụ thể (ngưỡng số, hạng thành viên, khoảng thời gian, segment/tag...) để xây dựng điều kiện lọc"}
  ],
  "briefSolution": null
}
```

## Ví dụ 6 — PASS, time-only condition (không có ngưỡng giá trị nào khác)

**Câu hỏi**: "Khách hàng đã mua hàng trong 45 ngày gần đây"

**Output**:
```json
{
  "matched": true,
  "resolved_summary": "Khách hàng có hoạt động mua hàng trong 45 ngày gần đây",
  "unresolved_items": [],
  "briefSolution": {
    "objective": "Xác định nhóm khách hàng có giao dịch mua hàng gần đây để triển khai chiến dịch chăm sóc, tri ân",
    "dataSources": ["unified_profile_model"],
    "conditions": [
      {"id": "c1", "source": "unified_profile_model", "field": "lastPurchasedAt", "operator": "exist", "value": null}
    ],
    "exclusions": [],
    "timeRange": {"field": "lastPurchasedAt", "last": "45d", "appliesTo": ["c1"]},
    "combinationLogic": "AND",
    "latestRecordPolicy": "LATEST_ONLY",
    "aggregation": null,
    "assumptions": []
  }
}
```