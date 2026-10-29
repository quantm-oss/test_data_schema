# sms_interaction

## Description

Lưu trữ tương tác qua tin nhắn SMS/OTT giữa hệ thống và người dùng, bao gồm hành vi, sự kiện, dữ liệu chiến dịch.

Dùng khi solution nói về:
- Tương tác SMS marketing / notification
- Filter theo campaign name, template, service provider
- Analysis event (sent, clicked, failed, not_send)
- Tracking khách hàng nhận SMS trong period

## Table Name Pattern

`sms_interaction_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values / Note |
|---|---|---|
| id | String | Mã định danh duy nhất bản ghi (primary key) |
| orgId | String | Mã định danh tổ chức. KHÔNG dùng để JOIN |
| systemName | String nullable | Tên hệ thống phát sinh dữ liệu |
| internalCompanyId | String nullable | Mã công ty nội bộ |
| refId | String nullable | Mã tham chiếu tới đối tượng liên quan |
| phoneNumber | String | **channelIdField khi channelType=Phone**. Link với zma_log_cf.phone, purchase_history.customerPhone |
| email | String nullable | Email khách hàng |
| customerId | String nullable | Mã định danh khách hàng. KHÔNG dùng để JOIN |
| cdpCustomerId | String nullable | Mã định danh khách hàng trong CDP |
| serviceProvider | String nullable | Nhà cung cấp dịch vụ SMS. **Enum**: SmsPushNotification_VietguysBrandSMS, SmsPushNotification_IMediaSMS, SmsPushNotification_IMediaMMS, SmsPushNotification_VMGBrandSMS_CS, SmsPushNotification_BlueLinkSMS, SmsPushNotification_FPTBrandSMS, SmsPushNotification_VnPayBrandSMS, SmsPushNotification_IncomOmniSMS, SmsPushNotification_Sendinblue, SmsPushNotification_CustomSms, SmsPushNotification_ESMS |
| enabledConnId | String nullable | ID kết nối đang được sử dụng |
| gaTid | String nullable | Google Analytics Tracking ID |
| gaCid | String nullable | Google Analytics Client ID |
| o2oCid | String nullable | Client ID từ nền tảng O2O |
| zouid | String nullable | ID user Zalo OA |
| zoaId | String nullable | Zalo Official Account ID |
| fbPsid | String nullable | Facebook Page-scoped ID |
| fbPageId | String nullable | Facebook Page ID |
| trackId | String nullable | Mã tracking hành vi user |
| link | String nullable | Đường dẫn liên quan đến sự kiện |
| link2 | String nullable | Đường dẫn phụ / link fallback |
| eventAt | DateTime64(3) | **Native DateTime** (KHÔNG phải epoch millis integer). Dùng cho filter time — so sánh trực tiếp dạng datetime, không convert sang Long |
| event | String | **Enum**: `sent`, `failed`, `clicked`, `not_send` |
| messageId | String nullable | ID tin nhắn được gửi/nhận |
| templateId | String nullable | ID mẫu tin nhắn |
| templateName | String nullable | Tên mẫu tin nhắn (SMS_FlashSale_SUV, SMS_TestDrive_SUV, SMS_VIP_Welcome, ...) |
| campaignId | String nullable | ID chiến dịch |
| campaignName | String nullable | Tên chiến dịch (Flash Sale SUV Cuối Tuần, Test Drive SUV, Chăm sóc khách VIP, Loyalty Program, ...) |
| webhookId | String nullable | ID webhook đang xử lý |
| webhookType | String nullable | Loại webhook (send, click, error, skip) |
| cdpTemplateId | String nullable | ID mẫu trong CDP |
| cdpCampaignId | String nullable | ID chiến dịch trong CDP |
| cdpWebhookId | String nullable | ID webhook trong CDP |
| text | String nullable | Nội dung văn bản tin nhắn |
| actionAt | DateTime64(3) nullable | **Native DateTime nullable** (KHÔNG phải epoch millis integer) thời điểm hành động |
| reason | String nullable | Lý do thực hiện hành động (thường dùng với event failed/not_send) |
| tags | Array(String) | Tập hợp thẻ phân loại (VD: ['promo', 'flash_sale', 'suv']) |
| createdBy | String nullable | Người tạo bản ghi |
| createdAt | DateTime64(3) nullable | **Native DateTime nullable** thời gian tạo bản ghi |
| updatedBy | String nullable | Người cập nhật gần nhất |
| updatedAt | DateTime64(3) nullable | **Native DateTime nullable** thời gian cập nhật gần nhất |
| recordStatus | String nullable | Trạng thái bản ghi (active, ...) |
| timezone | Int32 nullable | Múi giờ GMT offset (VD: +7) |

## channelIdField Guidance

- channelType=Phone → `channelIdField: "phoneNumber"`
- channelType=Email → `channelIdField: "email"`
- channelType=ZaloUId → `channelIdField: "zouid"`
- channelType=FbPsid → `channelIdField: "fbPsid"`

## Enum Values chi tiết

### event (4 values)
- `sent`: Tin nhắn đã được gửi thành công
- `clicked`: User đã click vào link trong tin nhắn (engaged)
- `failed`: Gửi tin nhắn thất bại (thường do nhà mạng, số SĐT không hợp lệ)
- `not_send`: Không gửi (do DND list, opt-out, ...)

### serviceProvider (11 values docs)
Trong test data có: VietguysBrandSMS, IMediaSMS, IMediaMMS, VMGBrandSMS_CS, FPTBrandSMS, Sendinblue, BlueLinkSMS

Full docs enum (11): SmsPushNotification_VietguysBrandSMS, SmsPushNotification_IMediaSMS, SmsPushNotification_IMediaMMS, SmsPushNotification_VMGBrandSMS_CS, SmsPushNotification_BlueLinkSMS, SmsPushNotification_FPTBrandSMS, SmsPushNotification_VnPayBrandSMS, SmsPushNotification_IncomOmniSMS, SmsPushNotification_Sendinblue, SmsPushNotification_CustomSms, SmsPushNotification_ESMS

### campaignName common values (test data)
- Flash Sale campaigns: `Flash Sale SUV Cuối Tuần`, `Flash Sale Sedan Đặc Biệt`
- Test Drive: `Test Drive SUV Miễn Phí`, `Test Drive Sedan`
- VIP: `Chăm sóc khách VIP`, `Dịch vụ chăm sóc VIP`, `Nâng cấp thành viên VIP`
- Loyalty: `Loyalty Program - Tích Điểm`, `Đổi Điểm Loyalty`, `Loyalty Program - Chào Mừng`
- Others: `Mời tham quan showroom`, `Nhắc lịch bảo dưỡng`, `Thông báo chung`

## Data Quality Notes

- **`eventAt`, `actionAt`, `createdAt`, `updatedAt` là native `DateTime64(3)`** — KHÔNG phải millisecond epoch integer (khác `zma_log_cf.eventAt`, field đó thực sự là UInt64 millis). Khi generate DSL filter cho các fields này, xử lý như DateTime chuẩn (dataType `Timestamp` giống `purchase_history.transCreatedAt`), KHÔNG dùng logic convert epoch millis integer.
- `campaignName` tiếng Việt có dấu — filter phải exact match hoặc dùng `like`
- `event` distribution: sent(15), clicked(8), failed(4), not_send(3)
- `tags` là Array(String) — lọc theo chuẩn field mảng: `toString(<field>) AS <field>Text` rồi `like`
- `phoneNumber` share với zma_log_cf.phone, purchase_history.customerPhone, lead_info.phone (test cross-table)

## Sample Values (Time Fields)

Minh họa format thực tế của 3 time fields chính (ClickHouse hiển thị dạng `YYYY-MM-DD HH:MM:SS`):

| eventAt | actionAt | createdAt | updatedAt |
|---|---|---|---|
| 2026-07-09 17:00:00 | 2026-07-09 17:00:03 | 2026-07-09 17:00:00 | null |
| 2026-07-09 17:00:00 | 2026-07-09 17:00:03 | 2026-07-09 17:00:00 | null |
| 2026-07-09 22:30:00 | 2026-07-09 22:32:00 | 2026-07-09 22:30:00 | null |
| 2026-07-09 22:30:00 | 2026-07-09 22:32:00 | 2026-07-09 22:30:00 | null |
| 2026-07-08 16:15:00 | 2026-07-08 16:15:02 | 2026-07-08 16:15:00 | null |

**Quan sát**:
- `actionAt` luôn ≥ `eventAt` (hành động xảy ra sau hoặc cùng lúc sự kiện, lệch vài giây)
- `createdAt` thường trùng `eventAt` (bản ghi tạo ngay khi event xảy ra)
- `updatedAt` hiện tại **toàn bộ null** trong dataset — record chưa từng được update sau khi tạo. Nếu solution filter theo `updatedAt`, cần lường trước filter có thể trả 0 kết quả do dữ liệu hiện tại chưa có giá trị.