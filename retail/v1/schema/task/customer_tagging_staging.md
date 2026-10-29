# customer_tagging_staging

## Description

Bảng lưu trữ chi tiết về việc gắn tag và áp dụng rule cho khách hàng. Đây là audit trail cho tất cả các hoạt động tagging (DSL Task).

Dùng khi solution nói về:
- Lịch sử/log gắn tag cho khách hàng
- Trạng thái tag hiện tại (VD: active)
- Kênh gắn tag (Phone, email, zalo...)
- Độ tin cậy (confidence) của tag/rule
- Chi tiết mở rộng theo tag (dạng key-value động: text, số, boolean, ngày, danh sách)
- Request/batch xử lý tagging

## Table Name Pattern

`customer_tagging_staging_<orgId>`

## Fields

| Name | Type | Note |
|---|---|---|
| id | UUID | Khóa định danh duy nhất (primary key) |
| orgId | LowCardinality(String) | Mã tổ chức. Liên kết với `customer_unified.orgId` |
| dslResultId | String | ID kết quả từ DSL (query/engine) |
| tagId | String | ID của tag |
| ruleId | String | ID của rule |
| status | LowCardinality(String) | Trạng thái hiện tại. Data thực tế quan sát: `active` |
| customerId | String | Mã khách hàng liên quan. Liên kết với `customer_unified.id` |
| channelType | LowCardinality(String) | Loại kênh. Data thực tế quan sát: `Phone` |
| channelId | String | ID của kênh |
| channelGroupId | LowCardinality(Nullable(String)) | Nhóm kênh (nếu có), nullable |
| identifyStatus | LowCardinality(String) | Trạng thái định danh khách hàng |
| tagCategory | LowCardinality(String) | Nhóm phân loại tag |
| tagSubCategory | LowCardinality(Nullable(String)) | Phân loại phụ của tag, nullable |
| sourceType | LowCardinality(String) | Kiểu nguồn dữ liệu |
| sourceIds | Array(String) | Danh sách ID nguồn dữ liệu |
| detailString | Map(String, String) | Thông tin chi tiết dạng text key-value động |
| detailLong | Map(String, Int64) | Thông tin chi tiết dạng số nguyên động |
| detailDouble | Map(String, Float64) | Thông tin chi tiết dạng số thực động |
| detailBoolean | Map(String, UInt8) | Thông tin chi tiết dạng boolean động — **giá trị 0/1**, KHÔNG dùng true/false |
| detailDate | Map(String, String) | Thông tin chi tiết dạng ngày (lưu String), key động |
| detailArrayString | Map(String, Array(String)) | Thông tin chi tiết dạng danh sách text, key động |
| detailArrayLong | Map(String, Array(Int64)) | Thông tin chi tiết dạng danh sách số nguyên, key động |
| detailArrayDouble | Map(String, Array(Float64)) | Thông tin chi tiết dạng danh sách số thực, key động |
| confidence | Float64 | Mức độ tin cậy (0.0 - 1.0). Data thực tế quan sát: `0.0` (chưa có giá trị thực sự khác 0 trong sample) |
| note | Nullable(String) | Ghi chú bổ sung, nullable |
| version | String | Phiên bản dữ liệu |
| createdAt | DateTime64(3, 'UTC') | Thời điểm tạo. **Native DateTime64, KHÔNG phải String ISO8601** |
| expiredAt | DateTime64(3, 'UTC') | Thời điểm hết hạn. **Native DateTime64, KHÔNG phải String ISO8601** |
| createdBy | String | Người/hệ thống tạo |
| extendDueDateCount | Int32 | Số lần gia hạn |
| lastTouchedAt | DateTime64(3, 'UTC') | Lần cuối cùng được cập nhật/đụng tới. **Native DateTime64, KHÔNG phải String ISO8601** |
| systemUpdateReason | Nullable(String) | Lý do/kết quả cập nhật từ hệ thống. **Tên field đúng là `systemUpdateReason`, KHÔNG phải `systemUpdateRes`** |
| requestId | String | Mã request gốc |
| requestBatch | Int64 | Batch request ID |
| tagRuleUserKey | String | Khóa user từ rule gắn tag |
| tagRuleSrcUserKey | String | Khóa user nguồn ban đầu từ rule |

## Giá Trị Quan Sát Được (Tham Khảo — KHÔNG phải enum cố định)

| Field | Giá trị tham khảo | Ghi chú |
|---|---|---|
| `status` | `active` | |
| `channelType` | `Phone` | |
| `tagCategory` | `customer_segmentation` | |
| `tagSubCategory` | `value_based` | |
| `identifyStatus` | `identified` | |
| `sourceType` | `dsl_result_<orgId>` | Có vẻ là giá trị hệ thống tự sinh khi tag tạo qua DSL engine, không phải category do user chọn |
| `channelGroupId` | Toàn bộ NULL trong sample | |
| `tagId` / `ruleId` hiện có | `khach_hang_gia_tri_cao` | khi user cung cấp tagId khác, KHÔNG tự động coi là sai chỉ vì chưa từng thấy trong danh sách này |

## Indexes (tham khảo, không ảnh hưởng DSL generation)

- Set index: `tagId`, `ruleId`, `status`, `tagRuleUserKey`, `tagRuleSrcUserKey`
- Tokenbf (full-text-like) index: `sourceIds`, `customerId`, `channelId`
- Minmax index: `createdAt`, `expiredAt`, `lastTouchedAt`
- Partition: `toYYYYMM(createdAt)`, Order by: `(tagId, customerId, createdAt)`

## Quan hệ với bảng khác

- `orgId` liên kết `customer_unified.orgId`
- `customerId` liên kết `customer_unified.id`

## Data Quality Notes

- **`createdAt`, `expiredAt`, `lastTouchedAt` là native `DateTime64(3, 'UTC')`** — xử lý DSL giống các time field DateTime64 khác trong hệ thống (so sánh trực tiếp dạng datetime), KHÔNG convert epoch millis, KHÔNG coi là String.
- **Lưu ý timezone UTC**: Khác với các bảng Tag khác (thường xử lý theo GMT+7/Vietnam trong solution), field thời gian ở bảng này lưu trữ theo **UTC** — cần quy đổi khi hiển thị/so sánh với mốc giờ Vietnam.
- Toàn bộ 8 field `Map(String, ...)` (`detailString`, `detailLong`, `detailDouble`, `detailBoolean`, `detailDate`, `detailArrayString`, `detailArrayLong`, `detailArrayDouble`) là key động — key tùy theo nghiệp vụ cụ thể của từng tag/rule, không có danh sách key cố định. Khi solution cần reference field trong các Map này, PHẢI có key cụ thể do nghiệp vụ xác định (kiểm tra qua data thực tế trước), không suy đoán.
- `detailBoolean` dùng `UInt8` (0/1), giống pattern các field boolean khác trong hệ thống (VD `lead_info.pricePageVisited`) — filter bằng `= 1`/`= 0`, không dùng `true`/`false`.
- **`systemUpdateReason`** (không phải `systemUpdateRes`) — tên field đã được sửa đúng theo ClickHouse thật.
- Trong 176 records mẫu: `status` chỉ có giá trị `active` (100%), `channelType` chỉ có giá trị `Phone` (100%), `confidence` toàn bộ = `0.0`, không có record nào rỗng ở `tagId`/`ruleId`/`customerId`/`status`. Đây là data hiện có tại thời điểm kiểm tra — có thể mở rộng thêm giá trị khác trong tương lai (VD `status` có thể có `inactive`/`expired` theo mô tả nghiệp vụ, dù chưa xuất hiện trong sample).