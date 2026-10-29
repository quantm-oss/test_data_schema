# purchase_history_item

## Description

Bảng TRANSACTION LINE ITEM: mỗi record = 1 sản phẩm trong 1 giao dịch (1 đơn hàng có thể có nhiều dòng).

Dùng khi solution cần:
- Filter theo sản phẩm đã mua (itemId, itemName), thương hiệu (brand, groupBrand), danh mục (ctgId, ctgName, ctgGroup)
- Aggregation theo sản phẩm (COUNT số lần mua, SUM quantity/totalCost của 1 sản phẩm/danh mục)
- Filter theo mã giảm giá (discountCode)

Tổng giá trị/số đơn của cả giao dịch → dùng `purchase_history` (1 record = 1 đơn), KHÔNG cộng dồn từ bảng này.

## Table Name Pattern

`purchase_history_item_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values / Stats | Note |
|---|---|---|---|
| id | String | | |
| orgId | String | | |
| version | UInt64 | | |
| customerId | String | | |
| customerPhone | String | | **channelIdField khi channelType=Phone** |
| customerEmail | String | | Có thể empty |
| transactionId | String nullable | | Mã giao dịch chứa dòng sản phẩm này |
| refTransactionId | String nullable | | |
| orderId | String nullable | | |
| receiptId | String nullable | | |
| refId | String nullable | | |
| purchaseCode | String nullable | | |
| itemId | LowCardinality(String) | | Mã sản phẩm |
| itemName | LowCardinality(Nullable(String)) | | Tên sản phẩm |
| itemImageUrl | String nullable | | |
| productLink | String nullable | | |
| unit | LowCardinality(Nullable(String)) | | Đơn vị tính |
| brand | LowCardinality(Nullable(String)) | | Thương hiệu |
| groupBrand | LowCardinality(Nullable(String)) | | Nhóm thương hiệu |
| ctgId | LowCardinality(Nullable(String)) | | Mã danh mục sản phẩm |
| ctgName | LowCardinality(Nullable(String)) | | Tên danh mục sản phẩm |
| ctgGroup | LowCardinality(Nullable(String)) | | Nhóm danh mục |
| storeId | LowCardinality(Nullable(String)) | | |
| quantity | UInt32 nullable | | Số lượng sản phẩm trong dòng |
| originalPrice | UInt64 nullable | | Đơn giá gốc |
| price | UInt64 nullable | | Đơn giá bán |
| discount | UInt64 nullable | | |
| discountValue | UInt64 nullable | | |
| totalCost | UInt64 nullable | | Thành tiền của dòng sản phẩm |
| totalProductCost | UInt64 nullable | | |
| discountCode | String nullable | | Mã giảm giá áp dụng |
| transDate | Date nullable | | **Date (không phải DateTime)** — KHÔNG dùng để filter thời gian; dùng `transCreatedAt` |
| transCreatedAt | DateTime | | **Field thời gian giao dịch dùng để filter** (UTC; mốc GMT+7 phải trừ 7 giờ) |
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
- channelType=Email → `channelIdField: "customerEmail"` (có thể empty)

## Data Quality Notes

- **Nhiều dòng cho 1 khách và 1 đơn**: đếm số khách → `groupBys` theo `customerPhone`; KHÔNG dùng COUNT dòng làm số đơn.
- **Filter thời gian luôn dùng `transCreatedAt`** (DateTime, UTC) thay cho `transDate` (Date). Mốc tuyệt đối theo GMT+7 → trừ 7 giờ để ra UTC.
- Chưa có thống kê enum cho `brand`, `ctgName`, `ctgGroup`, `itemName` → dùng `like` khi filter theo tên do user nhập.
- 43 fields total. Các Map `customField*` chưa có danh sách key cụ thể nên chưa dùng được key nào để filter
