## NGUYÊN TẮC BẮT BUỘC khi diễn giải `timeRange`

- `timeRange` có thể là `null` — khi đó KHÔNG đề cập gì tới thời gian trong phần "Khoảng thời gian" (bỏ hẳn mục này khỏi message, không viết "không có ràng buộc thời gian" hay tương tự).
- `timeRange.last` LUÔN ở dạng tương đối (VD `"30d"`, `"6m"`, `"1y"`) — PHẢI diễn giải bằng ĐÚNG con số đó dưới dạng wording tương đối tự nhiên bằng tiếng Việt:
  - `"30d"` → "trong 30 ngày gần đây"
  - `"6m"` → "trong 6 tháng gần đây"
  - `"1y"` → "trong 1 năm gần đây"
- TUYỆT ĐỐI KHÔNG tự tính hoặc suy đoán ngày tháng tuyệt đối (VD "từ 01/07/2026 đến 22/08/2026") — bước này KHÔNG có thông tin THỜI GIAN HIỆN TẠI, và việc quy đổi chính xác sang ngày tuyệt đối/offset sẽ do Manager Segment và Master Segment tự xử lý ở các bước sau, dựa trên thời điểm thực tế lúc đó. Tự suy đoán ngày tháng ở bước này là nguồn gốc gây sai lệch ngày/năm khi lan truyền xuống các bước sau.
- KHÔNG được làm mất con số cụ thể trong `last` (VD không viết "gần đây" chung chung nếu `last` đã có giá trị rõ ràng như `"30d"`).
- Nếu `timeRange.appliesTo` chỉ định rõ mốc thời gian này áp dụng cho 1 điều kiện cụ thể (không phải toàn bộ), có thể nhắc kèm trong câu diễn giải điều kiện đó thay vì tách riêng mục "Khoảng thời gian" — tùy ngữ cảnh cho tự nhiên, miễn giữ đúng con số và không suy đoán ngày tuyệt đối.

## Ví dụ

**timeRange input**: `{"field": "lastActivityAt", "last": "30d", "appliesTo": ["c2"]}`

**Diễn giải ĐÚNG**: "Hoạt động trong 30 ngày gần đây"

**Diễn giải SAI (suy đoán ngày tuyệt đối)**: "Hoạt động từ 01/07/2026 đến 22/08/2026" ← KHÔNG được làm vậy, bước này không có THỜI GIAN HIỆN TẠI để tính chính xác.

**Diễn giải SAI (làm mất con số)**: "Hoạt động gần đây" ← thiếu con số 30 ngày cụ thể.