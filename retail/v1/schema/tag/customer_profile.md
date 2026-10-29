# customer_profile

## Description

Bảng USER MASTER profile với demographics + main identifiers cho cross-channel.

Dùng khi solution nói về:
- Demographics (gender, ageGroup, birthday)
- Multi-channel main identifiers (main phone/email/zalo/facebook)

## Table Name Pattern

`customer_profile_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values | Note |
|---|---|---|---|
| id | String | | |
| orgId | String | | |
| userId | String | | **= rootUserId trong `contact_index`**. Link identity |
| version | DateTime64(3) | | |
| name | String nullable | | |
| firstName | String nullable | | |
| lastName | String nullable | | |
| gender | String nullable | male, female, Male, Nữ, unknown | **INCONSISTENT case + language**. Filter cần cover cả 5 values. Xem `rules_enum_handling.md` |
| ageGroup | String nullable | "", N/A, [<1], 18-20, 18-24, 20-30, 25-34, 35-44 | Enum không đồng nhất (có value `[<1]` bracketed) |
| birthday | DateTime64(3) | | |
| birthdayDate | UInt32 | | |
| birthdayMonth | UInt8 | | |
| birthYear | UInt16 | | |
| mainPhone | String nullable | | Primary phone identifier |
| mainEmail | String nullable | | Primary email identifier |
| mainZaloUserId | String nullable | | Primary Zalo ID |
| mainFacebookPsid | String nullable | | Primary FB PSID |
| mainFacebookPageId | String nullable | | |
| avatarUrl | String nullable | | |
| locale | String nullable | | |
| mainZaloOAId | String nullable | | ID Zalo Official Account liên quan đến khách hàng. KHÔNG phải định danh khách hàng |
| address | String nullable | | Địa chỉ của khách hàng |
| city | String nullable | | Thành phố/tỉnh của khách hàng |
| district | String nullable | | Quận/huyện của khách hàng |
| countryCode | String nullable | | Mã quốc gia |
| customerType | String nullable | | Loại khách hàng |
| preferredChannel | String nullable | | Kênh tương tác ưu tiên |
| preferredCategory | String nullable | | Danh mục sản phẩm/nội dung ưu tiên |
| locationSource | String nullable | | Nguồn dữ liệu xác định vị trí |
| locationDetectedAt | DateTime64(3) | | Thời điểm xác định/cập nhật vị trí |
| firstPurchasedAt | DateTime64(3) | | Thời điểm mua hàng đầu tiên |
| firstPurchasedStoreId | String nullable | | Mã cửa hàng của lần mua đầu tiên |
| firstPurchasedAmount | Float64 | | Giá trị giao dịch mua đầu tiên |
| firstPurchasedProducts | Array(String) | | Danh sách sản phẩm của lần mua đầu tiên |
| lastPurchasedAt | DateTime64(3) | | Thời điểm mua hàng gần nhất |
| lastPurchasedStoreId | String nullable | | Mã cửa hàng của lần mua gần nhất |
| lastPurchasedAmount | Float64 | | Giá trị giao dịch mua gần nhất |
| lastPurchasedProducts | Array(String) | | Danh sách sản phẩm của lần mua gần nhất |
| firstSeenAt | DateTime64(3) | | Thời điểm hệ thống ghi nhận khách hàng lần đầu |
| lastActivityAt | DateTime64(3) | | Thời điểm hoạt động gần nhất của khách hàng |
| attributes | Map(String, String) | | Thuộc tính mở rộng dạng key-value (chuỗi) |
| attributeInts | Map(String, Int32) | | Thuộc tính mở rộng dạng key-value (số nguyên 32-bit) |
| attributeLongs | Map(String, Int64) | | Thuộc tính mở rộng dạng key-value (số nguyên 64-bit) |
| coreTagIds | Array(String) | | Danh sách ID tag cốt lõi đã gán cho khách hàng |
| coreLabelIds | Array(String) | | Danh sách ID label cốt lõi đã gán cho khách hàng |
| coreTagIdsAudit | Map(String, UInt64) | | Audit tag cốt lõi: tag ID → thời điểm/phiên bản |
| coreLabelIdsAudit | Map(String, UInt64) | | Audit label cốt lõi: label ID → thời điểm/phiên bản |
| lastUpdatedSource | String nullable | | Nguồn cập nhật dữ liệu gần nhất |
| firstUpdatedSource | String nullable | | Nguồn cập nhật dữ liệu đầu tiên |
| fieldSourceMap | Map(String, String) | | Tên field → nguồn dữ liệu đã cập nhật field đó |
| fieldUpdatedAt | Map(String, UInt64) | | Tên field → thời điểm cập nhật |
| lastSegmentKeys | Map(String, String) | | Khóa segment → thông tin segment gần nhất |
| lastSegmentPrioritys | Map(String, UInt8) | | Segment → mức độ ưu tiên gần nhất |
| lastSegmentAt | Map(String, UInt64) | | Segment → thời điểm gần nhất khách thuộc segment |
| systemCreated | String nullable | | Hệ thống tạo bản ghi |
| updatedBy | String nullable | | Người/hệ thống cập nhật gần nhất |
| updatedAt | DateTime64(3) | | Thời điểm cập nhật bản ghi gần nhất |
| createdAt | DateTime64(3) | | Thời điểm tạo bản ghi |
| createdBy | String nullable | | Người/hệ thống tạo bản ghi |
| lastTouchedAt | DateTime64(3) | | Thời điểm bản ghi được tác động gần nhất |

## channelIdField Guidance

- Định danh khách hàng qua `customer_profile` → `channelType: "UserId"` + `channelIdField: "userId"` (KHÔNG dùng cặp `Phone` + `mainPhone`; `userId` PHẢI có trong `rootModel.selectFields` và `selectFields` top-level). `mainPhone` vẫn dùng làm khóa JOIN/filter và cột trả về.
- channelType=Email → `channelIdField: "mainEmail"`
- channelType=ZaloUId → `channelIdField: "mainZaloUserId"`
- channelType=FbPsid → `channelIdField: "mainFacebookPsid"`

## Data Quality Notes

- `gender` có 5 values mixed case/language. Filter "khách hàng nam" phải dùng `in ["male", "Male"]`. Filter "khách hàng nữ" phải dùng `in ["female", "Nữ"]`.
- `ageGroup` không đồng nhất format
- `mainXxx` fields là nullable — customer có thể chỉ có 1 vài kênh