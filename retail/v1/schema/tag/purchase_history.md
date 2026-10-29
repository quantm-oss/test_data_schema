# purchase_history

## Description

Bảng TRANSACTION LOG chi tiết mọi đơn hàng.

Total 2010 records nhưng **CHẤT LƯỢNG DATA THẤP**: 99.5% records có `channel`, `storeCity`, `paymentMethod` empty. Chỉ ~10 records có data thực.

Data time range: **2017-11 đến 2025-09** (không có data recent ≤ 90 ngày).

Dùng khi solution cần:
- Aggregation theo customer (COUNT đơn, SUM chi tiêu)
- Filter theo store city, channel, payment method (chú ý sparse data)
- Filter theo transaction date

## Table Name Pattern

`purchase_history_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values / Stats | Note |
|---|---|---|---|
| id | String | | |
| orgId | String | | |
| version | UInt64 | | |
| customerId | String | | |
| customerName | String nullable | | |
| customerPhone | String | | **channelIdField khi channelType=Phone**. Example: `0987444504` |
| customerEmail | String | | Có thể empty |
| refId | String nullable | | |
| orderId | String nullable | | |
| receiptId | String nullable | | |
| purchaseCode | String nullable | | |
| storeId | String nullable | | |
| storeName | String nullable | | |
| storeCity | String nullable | Hà Nội(2), Hồ Chí Minh(1), Đà Nẵng(1), Hải Phòng(1), Cần Thơ(1), ""(2000) | Enum tiếng Việt có dấu. **99.5% records empty** |
| storeDistrict | String nullable | | |
| systemName | String nullable | | |
| channel | String nullable | offline(4), online(2), ""(2000), null(4) | **99.5% empty** |
| totalCost | UInt64 nullable | Min=0, Max=20.2M, Avg=~800K | 1998/2010 records có value > 0. **Field duy nhất dùng cho tổng giá/tổng chi tiêu** |
| totalProductCost | UInt64 nullable | | |
| vatCost | UInt64 nullable | | |
| totalCostWithVat | UInt64 nullable | | Tổng tiền đơn hàng đã gồm VAT. **KHÔNG dùng** cho câu hỏi về tổng giá/tổng chi tiêu → dùng `totalCost` |
| vatPercent | UInt8 nullable | | |
| discount | UInt64 nullable | | |
| discountPercent | UInt8 nullable | | |
| totalItems | String nullable | | |
| totalQuantity | UInt16 nullable | | |
| payment1 | UInt64 nullable | | Chi tiết method payment #1 |
| payment2 | UInt64 nullable | | |
| payment3 | UInt64 nullable | | |
| payment4 | UInt64 nullable | | |
| payment5 | UInt64 nullable | | |
| payment6 | UInt64 nullable | | |
| payment7 | UInt64 nullable | | |
| paymentMethod | String nullable | momo(5), cash(3), transfer(1), card(1), ""(2000) | Sparse |
| transDate | Date nullable | Range 2017-2025 | **Date (không phải DateTime)** — KHÔNG dùng để filter thời gian; dùng `transCreatedAt` |
| transCreatedAt | DateTime | | **Field thời gian giao dịch dùng để filter** (UTC; mốc GMT+7 phải trừ 7 giờ) |
| transUpdatedAt | DateTime nullable | | |
| createdAt | DateTime | | Audit |
| updatedAt | DateTime | | |
| createdBy | String nullable | | |
| updatedBy | String nullable | | |
| customFieldTexts | Map(String, Nullable(String)) | | Custom field dạng key-value (chuỗi) |
| customFieldLongs | Map(String, UInt64) | | Custom field dạng key-value (số nguyên 64-bit) |
| customFieldDoubles | Map(String, Float64) | | Custom field dạng key-value (số thực) |
| customFieldBooleans | Map(String, Bool) | | Custom field dạng key-value (boolean) |
| customFieldDates | Map(String, String) | | Custom field dạng key-value (ngày, lưu dạng chuỗi) |
| customFieldLists | Map(String, Array(Nullable(String))) | | Custom field dạng key-value (danh sách chuỗi) |

## channelIdField Guidance

Query direct:
- channelType=Phone → `channelIdField: "customerPhone"`
- channelType=Email → `channelIdField: "customerEmail"` (chú ý sparse)

## Data Quality Notes

- **Sparse data**: `channel`, `storeCity`, `paymentMethod` chỉ có ~6-10 records có data thực (còn lại empty `""`)
- **Không có data recent**: `transCreatedAt`/`transDate` mới nhất là 2025-09. Filter `gte_relative_day: [30]` sẽ trả 0 records.
- Solution "trong N ngày qua" với N nhỏ → LLM nên giữ, user sẽ hiểu data cũ
- **Filter thời gian luôn dùng `transCreatedAt`** (DateTime, UTC) thay cho `transDate` (Date, không có giờ nên không quy đổi múi giờ được). Mốc tuyệt đối theo GMT+7 → trừ 7 giờ để ra UTC (xem mục 6 system prompt Master Tag). Không dùng `transUpdatedAt` cho thời điểm mua.
- **Tổng giá / tổng chi tiêu / giá trị đơn hàng** → CHỈ dùng `totalCost`, KHÔNG dùng `totalCostWithVat`
- `totalItems` là **String** (không phải Number) — data quality issue
- 47 fields total (nhiều customField* Maps không list ở đây)