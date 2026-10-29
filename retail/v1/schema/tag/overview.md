# Data Ecosystem Overview

## Business Context

Track lead journey (Web/FB/Zalo → classification → buying stage → purchase), customer VIP tier, transactions, và tương tác qua kênh SMS/Zalo Mini App.

## Channel Types Supported

Tenant hỗ trợ channel types:
- **Phone** (112K identifiers)
- **Email** (25K)
- **ZaloUId** (629)
- **FbPsid** (7)

## Tables Grouped by Purpose

### 1. Identity/Channel (CORE)

| Table | Purpose |
|---|---|
| **contact_index** | Channel identifier hub. Mỗi record = 1 identifier của contact. Field `value` = giá trị (SĐT/email/zalo), `type` = channel type |
| **contact_index_merge_log** | Audit log merge contacts (NOT for Master queries) |
| **customer_unified** | Track user root IDs qua merge operations |

### 2. Customer Master Data

| Table | Purpose |
|---|---|
| **customer_information** | Customer với PURCHASE STATS denormalized (totalOrder, totalOrderAmount, memberLevel). Bảng vàng cho segmentation |
| **customer_profile** | User master profile với demographics + main identifiers |

### 3. Business Transactions

| Table | Purpose |
|---|---|
| **lead_info** | Lead automotive data (classification, state, modelInterest, campaignType) |
| **purchase_history** | Transaction log chi tiết. Dùng cho aggregation COUNT/SUM |
| **purchase_history_item** | Line item của giao dịch: mỗi dòng = 1 sản phẩm trong 1 đơn (itemName, brand, ctgName, quantity, discountCode) |

### 4. Interaction Channels

| Table | Purpose |
|---|---|
| **zma_log_cf** | Zalo Mini App behavior logs. Tương tác với PangoForm/PangoWheel/PangoConnect/PangoScan/Pango1Touch |
| **sms_interaction** | SMS/OTT interaction logs. Track sent/clicked/failed campaigns |
| **zalo_user_profile** | Zalo user info + zaloOA |

### 5. System (NOT for Master queries)

| Table | Purpose |
|---|---|
| customer_tagging_staging | Destination cho tag execution results |
| customer_task_staging | Destination cho task results |
| dsl_result | Destination cho DSL execution (dùng làm `sourceModel`) |
| staff_directory | Staff info (5 fields, snake_case naming) |

## Table Selection Guide

Khi Master craft DSL, chọn bảng theo semantic solution:

- Solution nói **"VIP / member tier / tổng chi tiêu tích lũy"** → `customer_information` (đã denormalized, không cần aggregation)
- Solution nói **"lead / classification / model interest / campaign source"** → `lead_info`
- Solution cần **"aggregation theo thời gian, đếm đơn / sum chi tiêu trong period"** → `purchase_history`
- Solution nói **"đã mua sản phẩm X / thương hiệu / danh mục sản phẩm / mã giảm giá"** → `purchase_history_item`
- Solution nói **"Zalo user profile"** → `zalo_user_profile`
- Solution nói **"tương tác Zalo Mini App / Pango Form / Pango Wheel"** → `zma_log_cf`
- Solution nói **"SMS / campaign SMS / template SMS"** → `sms_interaction`

## Common JOIN Patterns

### Direct table query (đơn giản, ưu tiên)

Solution query field của 1 bảng → dùng field identifier của bảng đó làm `channelIdField`:
- `customer_information.phone`
- `lead_info.phone`
- `purchase_history.customerPhone`
- `purchase_history_item.customerPhone`
- `zalo_user_profile.phone`
- `zma_log_cf.phone`
- `sms_interaction.phoneNumber`

### JOIN nhiều bảng

Nối qua số điện thoại theo JOIN patterns trong "Quy tắc JOIN" (rules_table_selection).

## sourceModel cho taggingStrategy

BE sinh `sourceModel = "dsl_result_{orgId}"` (chú ý prefix `_staging`).

## Tables WITHOUT orgId Field

Các bảng KHÔNG có field `orgId` trong schema (multi-tenant qua table name suffix):
- customer_information
- lead_info
- staff_directory

Không được sinh filter `WHERE orgId = ...` cho các bảng này.

## Time Field Convention

Field thời gian dùng để filter theo từng bảng (tất cả đều là DateTime/DateTime64, lưu UTC → quy đổi GMT+7 sang UTC được):

| Table | Field thời gian dùng để filter | Type | Ghi chú |
|---|---|---|---|
| customer_profile | createdAt, updatedAt, firstPurchasedAt, lastPurchasedAt, firstSeenAt, lastActivityAt, lastTouchedAt, locationDetectedAt | DateTime64(3) | |
| customer_information | lastPurchasedAt | DateTime64(3) | |
| customer_unified | createdAt, updatedAt | DateTime64(3, 'UTC') | |
| lead_info | createdAt, updatedAt | DateTime64(3) | |
| purchase_history | **transCreatedAt** | DateTime | KHÔNG dùng `transDate` (Date, không quy đổi múi giờ được) và `transUpdatedAt` (thời điểm cập nhật bản ghi) |
| purchase_history_item | **transCreatedAt** | DateTime | KHÔNG dùng `transDate` (Date, không quy đổi múi giờ được) |
| zma_log_cf | eventAt | DateTime64(3) | |
| sms_interaction | eventAt, createdAt, updatedAt, actionAt | DateTime64(3) | |
| contact_index | createdAt, updatedAt | DateTime64(3) | |
| zalo_user_profile | createdAt | DateTime | |

**Field KHÔNG dùng để filter mốc thời gian tuyệt đối** (không quy đổi múi giờ được hoặc không phải thời gian):
- `purchase_history.transDate` (Date) → dùng `purchase_history.transCreatedAt`.
- `purchase_history_item.transDate` (Date) → dùng `purchase_history_item.transCreatedAt`.
- `customer_profile.birthday` (DateTime64): ngày sinh, dễ lệch ngày khi đổi múi giờ → lọc sinh nhật/ngày sinh bằng `birthdayDate` (ngày), `birthdayMonth` (tháng), `birthYear` (năm) — đây là số nguyên, không chịu ảnh hưởng múi giờ.
- `zalo_user_profile.dob` (UInt16, chỉ có năm) và `zma_log_cf.dayKey` (String, ngày 1-31): không phải mốc thời gian.
- `customer_profile.fieldUpdatedAt`, `lastSegmentAt` (Map UInt64): không dùng để filter thời gian.

**Chú ý**: `zma_log_cf.eventAt` và `sms_interaction.eventAt` là DateTime64(3).

**Múi giờ**: toàn bộ field thời gian trong ClickHouse lưu theo UTC, người dùng nói giờ GMT+7. Mốc tuyệt đối không có hậu tố phải trừ 7 giờ để ra UTC; khi có nhiều field thời gian, chọn field theo bảng trên (xem mục 6 của system prompt Master Tag).
