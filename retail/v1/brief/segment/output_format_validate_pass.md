Khi `matched: true`, trả về đầy đủ object `briefSolution` theo cấu trúc sau (mọi field đều dựa trên schema đã cung cấp, KHÔNG đoán):

```json
{
  "matched": true,
  "resolved_summary": "Tóm tắt ngắn gọn toàn bộ yêu cầu đã hiểu",
  "unresolved_items": [],
  "briefSolution": {
    "objective": "Mục tiêu kinh doanh/nghiệp vụ của việc xác định tập khách hàng này",
    "dataSources": ["Bảng được chọn ra từ Danh sách nguồn dữ liệu khả dụng cho DSL Segment"],
    "conditions": [
      {"id": "<id>", "source": "<dataSources table>", "field": "<field thật trong schema>", "operator": "<toán tử>", "value": "<giá trị>"}
    ],
    "exclusions": [],
    "timeRange": null,
    "combinationLogic": "AND",
    "latestRecordPolicy": "LATEST_ONLY",
    "aggregation": null,
    "assumptions": ["Điều cần user xác nhận lại để đảm bảo chính xác, nếu có"]
  }
}
```

## Giải thích từng field

- `objective`: 1 câu ngắn gọn tiếng Việt, mô tả mục tiêu nghiệp vụ.
- `dataSources`: Bảng được chọn ra từ Danh sách nguồn dữ liệu khả dụng cho DSL Segment theo `choose_data_source.md` hiện tại.
- `conditions`/`exclusions`: mỗi phần tử có `id` tự đặt, `field` PHẢI có thật trong schema, `operator` (`>`, `<`, `=`, `!=`, `contains_any`, `contains_all`...), `value`.
- `timeRange`: `null` nếu câu hỏi không đề cập thời gian. Nếu có: `{"field": "<field thời gian thật>", "last": "<dạng tương đối, VD '30d', '6m'>", "appliesTo": ["<id của condition/exclusion áp dụng>"]}`.
- `combinationLogic`: `"AND"` hoặc `"OR"` — mặc định `"AND"` nếu câu hỏi không nói rõ.
- `latestRecordPolicy`: chọn 1 trong `LATEST_ONLY` (chỉ lấy bản ghi mới nhất mỗi khách hàng — MẶC ĐỊNH), `ALL_RECORDS` (áp điều kiện trên toàn bộ bản ghi), `FIRST_RECORD` (bản ghi đầu tiên/sớm nhất), `LATEST_PER_PERIOD` (mới nhất theo từng chu kỳ, dùng cho phân tích xu hướng). Nếu câu hỏi không đề cập, dùng `LATEST_ONLY`.
- `aggregation`: object mô tả phép tổng hợp nếu câu hỏi cần (VD đếm số lần, tổng giá trị...), `null` nếu không cần.
- `assumptions`: mảng string do bạn (LLM) tự đề xuất — liệt kê những điều user NÊN kiểm tra lại để đảm bảo độ chính xác (VD giả định về đơn vị, giả định về cách hiểu 1 cụm từ mơ hồ nhưng vẫn map được). Mảng rỗng `[]` nếu không có gì cần lưu ý.

Lưu ý: KHÔNG bao gồm field `deduplication` trong output — field này do hệ thống tự xử lý, không cần bạn sinh.

### Trường hợp đặc biệt — điều kiện thời gian đơn thuần (time-only)

Khi TOÀN BỘ ý định của câu hỏi chỉ là 1 mốc thời gian (VD "khách hàng đã mua hàng trong 45 ngày gần đây" — không có ngưỡng giá trị nào khác đi kèm field thời gian đó), tạo 1 `ConditionItem` với `operator` là `"exist"` (hoặc `"is_not_null"`) và `value: null` — mục đích của condition này CHỈ để có `id` cho `timeRange.appliesTo` tham chiếu tới, bản thân điều kiện lọc thời gian nằm trọn trong `timeRange`.

Ví dụ: câu hỏi "Khách hàng đã mua hàng trong 45 ngày gần đây":
```json
{
  "conditions": [
    {"id": "c1", "source": "<source>", "field": "lastPurchasedAt", "operator": "exist", "value": null}
  ],
  "exclusions": [],
  "timeRange": {"field": "lastPurchasedAt", "last": "45d", "appliesTo": ["c1"]}
}
```