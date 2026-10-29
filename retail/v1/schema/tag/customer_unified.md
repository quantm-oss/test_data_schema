# customer_unified

## Description

Bảng IDENTITY MERGE HISTORY. Track root user IDs qua merge operations.

**Ít khi query trực tiếp trong Master DSL** — chủ yếu dùng cho audit/traceability. Nếu solution không rõ ràng về merge history, KHÔNG chọn bảng này. Với DSL Task, bảng này dùng chung với `customer_tagging_staging` để join/xác định danh tính khách hàng.

## Table Name Pattern

`customer_unified_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Note |
|---|---|---|
| id | String | |
| orgId | LowCardinality(String) | |
| rootId | String | User ID gốc |
| curRootId | String | User ID hiện tại sau merge |
| source | LowCardinality(String) | VD: CRM |
| name | Nullable(String) | |
| partial | Bool | |
| merged | Bool | |
| mergedTo | Nullable(String) | |
| mergedSrc | Nullable(String) | |
| mergedLogId | Nullable(String) | |
| createdAt | DateTime64(3, 'UTC') | **Timezone UTC** |
| updatedAt | DateTime64(3, 'UTC') | **Timezone UTC** |

## Giá Trị Quan Sát Được (Tham Khảo — KHÔNG phải enum cố định)

Các giá trị dưới đây chỉ mang tính tham khảo phổ biến, không phải danh sách đóng hay cố định.

| Field | Giá trị phổ biến quan sát được | Tỷ lệ |
|---|---|---|
| `source` | `Customer`, `PurchaseHistory`, `ZaloProfile`, `CustomerMembership` | Chiếm đa số (>99%) |
| `source` (hiếm) | `SocialCRM`, `CRM`, `Import`, chuỗi rỗng | Số lượng nhỏ |
| `source` (data quality issue) | Một số ít giá trị dạng composite lạ (VD `"Phone:...:Customer"`, `"Email:...:SocialCRM | Phone:...:SocialCRM"`) | Rất hiếm (<10 records), có thể là lỗi ghi nhận, KHÔNG coi là format chuẩn cần filter theo |
| `merged` | Chủ yếu `false` (~99.83%), một phần nhỏ `true` (~0.17%) | |
| `partial` | 100% `false` trong sample hiện tại | |
| `name` | Chỉ ~0.016% NULL — hầu hết đều có giá trị | Khác với ấn tượng ban đầu từ sample nhỏ |

## Note

Không có field identifier direct (phone/email). Không phù hợp cho Master Tag Rule thông thường — chủ yếu dùng cho Task (join với `customer_tagging_staging` qua `rootId`/`id` ↔ `customerId`).

## Data Quality Notes

- `createdAt`/`updatedAt` lưu theo UTC (DateTime64(3, 'UTC')). Mốc tuyệt đối theo giờ GMT+7 phải quy đổi về UTC (xem quy tắc thời gian tuyệt đối).
- `merged=true` nghĩa là record đã được merge sang `mergedTo` — cân nhắc filter `merged=false` nếu solution cần "identity hiện hành, chưa bị merge".