# customer_information

## Description

Bảng thông tin customer với **PURCHASE STATS đã tính sẵn** (denormalized). Ưu tiên dùng cho segmentation dựa trên purchase behavior — KHÔNG cần JOIN hoặc aggregation với `purchase_history`.

Total chỉ 14 records — dataset nhỏ.

Dùng khi solution nói về:
- VIP customer, member tier
- Tổng chi tiêu tích lũy
- Nearest single product
- Rating store

## Table Name Pattern

`customer_information_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values / Stats | Note |
|---|---|---|---|
| id | String | | |
| phone | String | | SĐT. **channelIdField khi channelType=Phone**. Example: `0987444504` |
| email | String | | Email. **channelIdField khi channelType=Email**. Có thể empty `""` |
| name | String | | Tên khách hàng |
| gender | String | male(11), female(3) | Enum. Chỉ 2 values, KHÔNG có mixed case (khác `customer_profile.gender`) |
| memberLevel | String | Kim Cương(6), Vàng(3), Bạc(3), Đồng(2) | Tier VIP tiếng Việt có dấu |
| source | String | Chiến dịch(5), Khác(4), Trực tiếp(3), Giới thiệu(2) | Enum tiếng Việt. **KHÁC với `contact_index.source`** |
| address | String | | |
| ward | String | | Phường/Xã |
| province | String | | Tỉnh/Thành |
| lastPurchasedStoreName | String | | |
| lastPurchasedAmount | Float64 | | Giá trị đơn hàng cuối |
| lastPurchasedAt | DateTime64(3) | | Timestamp đơn hàng cuối |
| ratingStore | Float64 | | Điểm đánh giá |
| totalOrder | UInt64 | | Tổng số đơn hàng |
| totalOrderAmount | Float64 | Min=1.2M, Max=175M, Avg=~40M | **Tổng chi tiêu tích lũy**. 14/14 records có value > 0 |
| listOrderItemName | Array(String) | | Danh sách sản phẩm đã mua |
| nearestSingleProduct | Array(String) | | Sản phẩm gần nhất |

## channelIdField Guidance

Query bảng này direct (không JOIN):

- channelType=Phone → `channelIdField: "phone"`
- channelType=Email → `channelIdField: "email"` (chú ý có records email empty)

## Data Quality Notes

- Dataset nhỏ (14 records), phù hợp segmentation testing
- KHÔNG có field `orgId` — multi-tenant qua table name suffix
- `memberLevel` values là tiếng Việt có dấu — filter phải EXACT match (`"Vàng"` không phải `"Vang"` hay `"vang"`)
- `source` values khác với `contact_index.source` (đừng nhầm lẫn)