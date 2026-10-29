# zma_log_cf

## Description

Ghi nhận toàn bộ hành vi và sự kiện của người dùng trong Zalo Mini App. Là nguồn dữ liệu thô phục vụ phân tích hành vi, đo lường tương tác và xây dựng chiến dịch cá nhân hóa.

Dùng khi solution nói về:
- Tương tác user với Zalo Mini App / Pango apps (PangoForm, PangoWheel, PangoConnect, ...)
- Filter theo appType, eventName, campaign UTM
- Tracking form submission, wheel spin, page visit
- Analysis behavior theo thời gian (hourKey, dayKey, eventAt)

## Table Name Pattern

`zma_log_cf_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values / Note |
|---|---|---|
| id | String | Mã định danh duy nhất bản ghi (primary key) |
| orgId | String | Mã định danh tổ chức. KHÔNG dùng để JOIN |
| enabledConnId | String nullable | ID của kết nối được kích hoạt |
| sessionId | String nullable | Mã định danh phiên làm việc hiện tại |
| cdpAppId | String nullable | Mã định danh ứng dụng trên nền tảng CDP |
| zaloAppId | String nullable | Zalo App ID |
| zaloUserAppId | String nullable | Mã định danh duy nhất user trong ứng dụng Zalo cụ thể (User App ID) |
| appType | String | **Enum**: PangoForm, PangoWheel, PangoConnect, PangoScan, Pango1Touch, PangoSell, PangoReward |
| zmaZone | String nullable | Vùng/zone trong hệ thống ZMA. Value: `Private` |
| devZaloAppId | String nullable | Zalo App ID môi trường dev |
| appTemplateType | String nullable | Mã template ứng dụng |
| zoaId | String nullable | Mã định danh Zalo Official Account |
| zoaUserId | String nullable | Mã định danh user trong Zalo Official Account |
| zoaUserDeviceId | String nullable | Mã thiết bị user Zalo OA |
| zoaUserName | String nullable | Tên user trong Zalo OA |
| zoaUserAvatar | String nullable | Link ảnh avatar user Zalo OA |
| userIdByFOA | Bool nullable | Có sử dụng ID user từ hệ thống follow Zalo OA không |
| phone | String | **channelIdField khi channelType=Phone**. Link với sms_interaction.phoneNumber, purchase_history.customerPhone |
| email | String nullable | Email user. Link với sms_interaction.email, purchase_history.customerEmail |
| eventTrigger | String nullable | Sự kiện trigger. Value: `UserEvent` |
| eventName | String | Tên sự kiện (form_submit, form_view, form_start, wheel_spin, wheel_view, page_visit, scan_qr, touch_call, touch_message, ...) |
| eventCtg | String nullable | Nhóm sự kiện (Form, Engagement, Marketing) |
| eventAt | DateTime64(3) | **Native DateTime** (KHÔNG phải epoch millis integer). Dùng cho filter time — so sánh trực tiếp dạng datetime, giống hệt cách xử lý `sms_interaction.eventAt` |
| eventData | String nullable | Dữ liệu chi tiết sự kiện (JSON string) |
| tid | String nullable | Transaction ID |
| cid | String nullable | Client ID / Visitor ID |
| hourKey | String nullable | Khóa theo giờ (0-23), phân tích hành vi theo khung giờ |
| dayKey | String nullable | Khóa theo ngày (1-31), phân tích hành vi theo ngày trong tháng |
| utmSource | String nullable | UTM source (zalo, facebook, google, web, direct) |
| utmMedium | String nullable | UTM medium (social, cpc, organic, ...) |
| utmCampaign | String nullable | UTM campaign — tên chiến dịch |
| utmKeyword | String nullable | UTM keyword |
| utmContent | String nullable | UTM content |
| pangoFormId | String nullable | ID form Pango Form |
| pangoFormName | String nullable | Tên form Pango Form |
| userAgent | String nullable | User-agent trình duyệt / ứng dụng |
| device | String nullable | Tên thiết bị |
| brwType | String nullable | Kiểu trình duyệt |
| brwName | String nullable | Tên trình duyệt |
| brwVersion | String nullable | Phiên bản trình duyệt |
| osName | String nullable | Tên hệ điều hành |
| osProducer | String nullable | Nhà phát hành OS |
| osVersion | String nullable | Phiên bản đầy đủ OS |
| osVersionMj | String nullable | Phiên bản chính OS |
| retry | Bool nullable | Bản ghi được retry hay không |
| retryReason | String nullable | Lý do retry (mặc định N/A) |
| screenView | String nullable | Tên màn hình / trang đang hiển thị |

## channelIdField Guidance

- channelType=Phone → `channelIdField: "phone"`
- channelType=Email → `channelIdField: "email"`
- channelType=ZaloUId → `channelIdField: "zaloUserAppId"`

## Enum Values chi tiết

### appType (7 values)
- **PangoForm**: Ứng dụng thu thập form (lead gen, đăng ký test drive)
- **PangoWheel**: Vòng quay may mắn / gamification
- **PangoConnect**: Kết nối Zalo Mini App
- **PangoScan**: QR code scanning
- **Pango1Touch**: Quick action (call/message showroom)
- **PangoSell**: (Chưa dùng trong dataset test)
- **PangoReward**: (Chưa dùng trong dataset test)

### eventName common values
- `form_submit`, `form_view`, `form_start` (PangoForm)
- `wheel_spin`, `wheel_view` (PangoWheel)
- `page_visit` (PangoConnect)
- `scan_qr` (PangoScan)
- `touch_call`, `touch_message` (Pango1Touch)

### eventCtg
- `Form`
- `Engagement`
- `Marketing`

### utmSource common values
- `zalo`, `facebook`, `google`, `web`, `direct`

## Data Quality Notes

- Nhiều fields nullable — có records với utmSource/utmCampaign empty
- **`eventAt` là native `DateTime64(3)`** — KHÔNG phải millisecond epoch integer. Xử lý giống hệt `sms_interaction.eventAt`, `purchase_history.transCreatedAt` (dataType `Timestamp`, so sánh trực tiếp dạng datetime, KHÔNG convert epoch millis).
- `phone` share với purchase_history/lead_info/sms_interaction (test cross-table)
- Không có field `transDate` — dùng `eventAt` cho filter thời gian