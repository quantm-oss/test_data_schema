# lead_info

## Description

Bảng LEAD DATA cho automotive (car dealership). Track lead từ Web/FB, buying journey, model interest, lead scoring, và hành vi tương tác marketing.

Total 200 records.

Dùng khi solution nói về:
- Lead classification (Hot/Warm/Cold)
- Buying stage (Ready-to-Buy, High Intent, Consideration, Interest)
- Model xe quan tâm (SUV, Sedan, MPV, ...)
- Campaign source (Web, FB, search, lead_form, conversion)
- Lead scoring (leadScore, finalScore) — xếp hạng độ ưu tiên lead
- Hành vi truy cập trang (price page, test drive page)
- Nhu cầu trade-in (đổi xe cũ lấy mới)
- Tag/label gán tự động qua DSL (tags)
- Tương tác marketing creative/feature (creativeClick, featureClick)

## Table Name Pattern

`lead_info_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values | Note |
|---|---|---|---|
| refId | String | | |
| phone | String | | SĐT lead. **channelIdField khi channelType=Phone**. Example: `0989730346` |
| createdAt | DateTime64(3) | | Range: mid 2025 |
| updatedAt | DateTime64(3) | | |
| classification | String | Hot(50), Warm(50), Cold(30), ""(70) | **70 records có empty value** — filter cần chú ý |
| state | String | High Intent(45), Interest(40), Ready-to-Buy(35), Consideration(10), ""(70) | Giai đoạn buying journey |
| address | String | | |
| addressDetail | String | | |
| modelInterest | String | P_ElectricCity(35), [P_SUV](30), P_SUV(25), P_MPV(25), P_Crossover(25), P_Sedan(20), [P_Crossover](20), [P_MPV](10), [P_Electric](10) | **2 formats**: plain và bracketed. Dùng `like` để match cả 2 |
| planToPurchased | String | | |
| source | String | Web(115), FB(85) | **CHỈ 2 values** — không có Website/Facebook |
| campaignType | String | lead_form(80), search(70), conversion(35), convertion(10 typo), ""(5) | **TYPO**: `convertion` là data quality issue trong data values. Xem `rules_enum_handling.md` |
| keyWord | String | | Keyword search |
| modelView | String | | Mẫu xe đã xem |
| pricePageStage | String | | Hành vi user trên trang xem giá bán |
| testdricePageStage | String | | Hành vi user trên trang đăng ký lái thử. **⚠️ TYPO trong TÊN FIELD** — ClickHouse column thực tế là `testdricePageStage` (thiếu chữ "v" so với "testdrive"). PHẢI dùng đúng chính tả này khi generate DSL, KHÔNG được tự sửa thành `testdrivePageStage` |
| reason | String | | |
| barrier | String | | |
| tradeCar | String | | Xe trade-in |
| tradeInfo | String | | |
| leadScore | Int64 | | Điểm đánh giá lead, tương tự eRFM nhưng chuyên biệt cho ngành Ô tô |
| visitTimes | Int64 | | Số lần khách hàng truy cập trang để xem dòng xe |
| visitRange | Int64 | | Số ngày ghi nhận hành vi truy cập xem xe |
| finalScore | Int64 | | Tổng điểm đánh giá cuối cùng của lead |
| pricePageVisited | UInt8 | 0, 1 | Đã truy cập trang xem giá bán chưa. **0 = false, 1 = true** — filter bằng số nguyên (`= 1`), KHÔNG dùng `true`/`false` |
| tradeIn | UInt8 | 0, 1 | Có nhu cầu đổi xe cũ lấy mới không. **0 = false, 1 = true** |
| testdrivePageVisited | UInt8 | 0, 1 | Đã truy cập trang đăng ký lái thử chưa. **0 = false, 1 = true**. Lưu ý: field này chính tả ĐÚNG (`testdrive`), khác với `testdricePageStage` ở trên (chính tả SAI) — đây là 2 fields riêng biệt, không nhầm lẫn |
| tags | Array(String) | | Danh sách tag được DSL gán dựa trên hành vi và đặc điểm của khách hàng. Lọc theo chuẩn field mảng: `toString(<field>) AS <field>Text` rồi `like` |
| creativeClick | Array(String) | | Danh sách nội dung/tính năng marketing mà user đã click. Lọc theo chuẩn field mảng: `toString(<field>) AS <field>Text` rồi `like` |
| featureClick | Array(String) | | Danh sách tính năng dòng xe khách hàng đã xem/quan tâm trên website. Lọc theo chuẩn field mảng: `toString(<field>) AS <field>Text` rồi `like` |

## channelIdField Guidance

Query direct:
- channelType=Phone → `channelIdField: "phone"`

Không có field email trong bảng này.

## Data Quality Notes

- **classification/state có 70 records empty** — chiếm 35% dataset. Filter `NOT NULL` hoặc `!= ""` nếu cần loại bỏ.
- **campaignType có typo trong DATA VALUE** `convertion` (10 records). Nếu solution muốn "conversion campaigns", dùng `in ["conversion", "convertion"]` để cover cả 2.
- **modelInterest có 2 formats**: `P_SUV` (plain) và `[P_SUV]` (bracketed). Cùng semantic. Dùng `like '%SUV%'` để match cả 2.
- **testdricePageStage có typo trong TÊN FIELD** (không phải data value) — ClickHouse column name thực sự là `testdricePageStage`, thiếu chữ "v". Đây KHÔNG phải lỗi tài liệu — PHẢI dùng chính xác tên này khi generate DSL, nếu không SQL sẽ fail vì cột không tồn tại.
- **3 fields UInt8 boolean-like** (`pricePageVisited`, `tradeIn`, `testdrivePageVisited`) dùng giá trị `0`/`1`, KHÔNG dùng `true`/`false` khi generate `values` trong DSL condition. Ví dụ đúng: `{"field": "tradeIn", "operator": "equals", "values": [1]}`.
- KHÔNG có field `orgId` — multi-tenant qua table name suffix.
- 1 phone có thể có multiple records (duplicates), acceptable behavior.