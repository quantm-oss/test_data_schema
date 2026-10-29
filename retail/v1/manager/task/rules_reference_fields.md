Phần "Trường dữ liệu tham chiếu" trong solution phải CHỌN ĐẦY ĐỦ các fields cần thiết để thực thi rule task.

## Nguyên tắc bắt buộc

### Category 1 — IDENTIFY (BẮT BUỘC)

Fields định danh khách hàng để biết gán task cho ai. Task rule LUÔN cần cả 2:

- 'customer_unified.rootId' (String) — Định danh khách hàng chính
- 'customer_tagging_staging.customerId' (String) — Mã khách hàng

### Category 2 — JOIN

Fields làm khóa liên kết giữa 2 bảng. Task rule luôn có JOIN vì buộc dùng cả 2 bảng:

- Join key: 'customer_unified.rootId = customer_tagging_staging.customerId'
- Trong Solution, chỉ cần list 2 fields IDENTIFY ở trên là đã đủ (JOIN implicit).

### Category 3 — FILTER (nếu có điều kiện lọc)

Mọi field xuất hiện trong "Điều kiện lọc (logic)" PHẢI có trong "Trường dữ liệu tham chiếu":

- 'customer_tagging_staging.tagId' — nếu filter theo tag
- 'customer_tagging_staging.status' — nếu filter theo trạng thái
- 'customer_tagging_staging.channelType', 'channelId' — nếu filter theo kênh
- 'customer_tagging_staging.tagCategory', 'tagSubCategory' — nếu filter theo category
- 'customer_tagging_staging.createdAt', 'expiredAt' — nếu filter theo time (type: DateTime64(3, UTC))
- KHÔNG dùng field Map (`detailString`, `detailLong`, `detailDouble`, `detailBoolean`, `detailDate`, `detailArrayString`, `detailArrayLong`, `detailArrayDouble`) — toàn bộ 8 Map hiện đang rỗng 100% trong dữ liệu thực tế (đã xác nhận qua kiểm tra). Nếu user cần custom detail không có field cố định tương ứng, trả `matched: false`.

### Category 4 — SELECT (nếu có value hiển thị)

Fields để BE lấy thông tin hiển thị cho user cuối. Optional cho Task:

- 'customer_unified.name' — Tên khách hàng (nullable)
- 'customer_unified.source' — Nguồn dữ liệu gốc

**Bỏ qua category này nếu solution không cần hiển thị thông tin.**

### Category 5 — AGGREGATE (nếu có tính toán)

Fields dùng cho aggregation (count, sum, avg).

- **Đếm số lần gắn tag / số bản ghi tagging cho 1 khách hàng**: dùng `COUNT(*)` (đếm số dòng trong `customer_tagging_staging` theo `customerId`) — KHÔNG cần field cố định nào cả, `COUNT(*)` luôn khả dụng để đếm số bản ghi thỏa điều kiện filter (VD "gắn tag X từ N lần trở lên" → `GROUP BY customerId HAVING COUNT(*) >= N`).
- Có thể dùng: 'customer_tagging_staging.confidence' (Double) nếu aggregate confidence
- Có thể dùng: 'customer_tagging_staging.extendDueDateCount' (Int32) nếu count extensions

**Bỏ qua category này nếu solution không có aggregation.**

### Category 6 — BUSINESS (nếu có context marketing)

Fields hỗ trợ context business cho Marketing/CSKH:

- 'customer_tagging_staging.tagCategory', 'tagSubCategory' — Phân loại tag
- 'customer_tagging_staging.channelType', 'channelGroupId' — Kênh business (channelGroupId nullable)
- 'customer_tagging_staging.sourceType', 'sourceIds' — Nguồn dữ liệu business

**Bỏ qua category này nếu solution không cần thêm business context.**

## Format

Mỗi field ghi trên 1 dòng với format:
- [collection_name].[field_name] ([Kiểu dữ liệu]): [Mô tả ngắn gọn ý nghĩa business]

Time field ('customer_unified.createdAt/updatedAt', 'customer_tagging_staging.createdAt/expiredAt/lastTouchedAt') ghi type là `DateTime64(3, UTC)` — KHÔNG ghi String hay ISO8601.

**KHÔNG cần label category trong output** (IDENTIFY/JOIN/FILTER/SELECT/AGGREGATE/BUSINESS chỉ là hướng dẫn nội bộ để chọn fields đủ). Solution chỉ list fields theo thứ tự tự nhiên.

## Ví dụ áp dụng

### Ví dụ 1 — Task filter theo tag (chỉ IDENTIFY + FILTER)

**Điều kiện lọc**: khách có tag "highvalue" active

**Trường dữ liệu tham chiếu ĐÚNG**:
- 'customer_unified.rootId' (String): Định danh khách hàng (tham chiếu để gắn Task) ← IDENTIFY
- 'customer_tagging_staging.customerId' (String): Mã khách hàng ← IDENTIFY
- 'customer_tagging_staging.tagId' (String): ID của tag cần filter ← FILTER
- 'customer_tagging_staging.status' (String): Trạng thái tag (chỉ active) ← FILTER

### Ví dụ 2 — Task filter theo channel + time + business context

**Điều kiện lọc**: khách được gắn tag qua kênh Phone trong 7 ngày qua

**Trường dữ liệu tham chiếu ĐÚNG**:
- 'customer_unified.rootId' (String): Định danh khách hàng ← IDENTIFY
- 'customer_tagging_staging.customerId' (String): Mã khách hàng trong log tagging ← IDENTIFY
- 'customer_tagging_staging.channelType' (String): Loại kênh cần filter (Phone) ← FILTER
- 'customer_tagging_staging.channelId' (String): ID kênh cụ thể ← FILTER + BUSINESS
- 'customer_tagging_staging.createdAt' (DateTime64(3, UTC)): Thời điểm gắn tag ← FILTER
- 'customer_tagging_staging.tagCategory' (String): Phân loại tag để phân biệt loại chăm sóc ← BUSINESS

### Ví dụ 3 — Task với SELECT (hiển thị tên khách)

**Điều kiện lọc**: khách có tag "highvalue" để đội Sales biết tên gọi

**Trường dữ liệu tham chiếu ĐÚNG**:
- 'customer_unified.rootId' (String): Định danh khách hàng ← IDENTIFY
- 'customer_unified.name' (String, nullable): Tên khách hàng để đội Sales biết gọi ai ← SELECT
- 'customer_tagging_staging.customerId' (String): Mã khách hàng ← IDENTIFY
- 'customer_tagging_staging.tagId' (String): ID của tag ← FILTER
- 'customer_tagging_staging.status' (String): Trạng thái tag ← FILTER

## Field kiểu mảng (Array) — chuẩn thống nhất

Áp dụng cho MỌI field có kiểu bắt đầu bằng `Array` (`Array(...)` hoặc `Array[...]`) trong schema, ở mọi bảng.

- Field kiểu text (String...): ghi điều kiện "chứa" như bình thường (`<field>` chứa `<text>`).
- Field kiểu mảng: KHÔNG so sánh theo phần tử (không ghi "có phần tử". Ghi theo VĂN BẢN: "mảng `<field>` (chuyển thành chuỗi `<field>Text`) chứa đoạn văn bản `<text>`".
- `<text>` là đúng giá trị người dùng nêu, không tự chuẩn hóa, không đổi hoa/thường, không thêm bớt. Với phần tử dạng "khóa: giá trị": câu hỏi `<field>.<khóa> = <giá trị>` hoặc `<field>.<khóa> LIKE '%<đoạn>%'` thì `<text>` = `<khóa>: <giá trị hoặc đoạn>` (khóa và giá trị liền kề, có dấu hai chấm và một dấu cách).
- Nhiều lựa chọn: mỗi lựa chọn là một điều kiện văn bản riêng, nối bằng HOẶC (hoặc VÀ nếu câu hỏi yêu cầu có đủ tất cả).
- Trong "Trường dữ liệu tham chiếu" chỉ liệt kê field mảng GỐC (`<field>`), KHÔNG liệt kê `<field>Text`.

Field mảng hiện có trong schema Task của tenant: `customer_tagging_staging.sourceIds`. Các field `detailArrayString`/`detailArrayLong`/`detailArrayDouble` là `Map(...)` (không phải Array) và đang rỗng — vẫn KHÔNG dùng (xem Category 3).
