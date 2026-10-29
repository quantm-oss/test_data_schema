## Data ecosystem

Tenant có 10 bảng queryable, chia nhóm:

1. **contact_index** — channel identifier hub. Dùng khi cần multi-channel resolution hoặc filter theo type/source ingest.
2. **customer_information** — customer với PURCHASE STATS đã denormalized (totalOrder, totalOrderAmount, memberLevel). Ưu tiên khi solution nói VIP, tier, tổng chi tiêu.
3. **customer_profile** — user master demographics + main identifiers.
4. **customer_unified** — identity merge history. ÍT khi query trực tiếp.
5. **lead_info** — lead automotive data (classification, state, modelInterest, campaignType, leadScore, tags).
6. **purchase_history** — transaction log chi tiết. Dùng cho aggregation COUNT/SUM.
7. **purchase_history_item** — line item của giao dịch (1 dòng = 1 sản phẩm trong 1 đơn). Dùng khi lọc theo sản phẩm, thương hiệu, danh mục, mã giảm giá.
8. **sms_interaction** — tương tác SMS/OTT marketing (event, campaignName, templateName, serviceProvider).
9. **zma_log_cf** — hành vi user trong Zalo Mini App / Pango apps (appType, eventName, utmSource).
10. **zalo_user_profile** — Zalo user info.

## Nguyên tắc chọn bảng theo semantic solution

- Solution nói **"VIP / member tier / tổng chi tiêu tích lũy"** → 'customer_information' (đã denormalized, không cần aggregation)
- Solution nói **"lead / classification / model interest / campaign source / lead score / trade-in"** → 'lead_info'
- Solution nói **"aggregation theo thời gian, đếm đơn / sum chi tiêu"** → 'purchase_history'
- Solution nói **"đã mua sản phẩm X / thương hiệu / danh mục sản phẩm / mã giảm giá"** → 'purchase_history_item'
- Solution nói **"tương tác SMS / nhận SMS / click SMS / chiến dịch SMS"** → 'sms_interaction'
- Solution nói **"tương tác Zalo Mini App / PangoForm / PangoWheel / PangoConnect / UTM tracking"** → 'zma_log_cf'
- Solution nói **"Zalo user profile"** → 'zalo_user_profile'
- Solution nói **"demographics / age / gender / birthday"** → 'customer_profile'
- Solution nói **"đơn hàng đầu tiên / lần đầu mua hàng / first purchase / vừa mới mua hàng lần đầu"** → 'customer_profile' / 'customer_information'

## Nguyên tắc channelIdField

- Query direct table → dùng field identifier của bảng đó:
  * 'customer_information.phone', 'customer_information.email'
  * 'lead_info.phone' (không có email)
  * 'purchase_history.customerPhone', 'purchase_history.customerEmail'
  * 'purchase_history_item.customerPhone', 'purchase_history_item.customerEmail'
  * 'sms_interaction.phoneNumber', 'sms_interaction.email', 'sms_interaction.zouid' (Zalo), 'sms_interaction.fbPsid'
  * 'zma_log_cf.phone', 'zma_log_cf.email', 'zma_log_cf.zaloUserAppId' (Zalo)
  * 'zalo_user_profile.phone', 'zalo_user_profile.userId'
  * 'customer_profile.mainPhone', 'customer_profile.mainEmail', 'customer_profile.mainZaloUserId', 'customer_profile.mainFacebookPsid'

- Query qua 'contact_index' → dùng field 'value' (universal identifier). Kết hợp filter 'type = 'Phone'|'Email'|'ZaloUId'|'FbPsid'' và 'status = 'active''.

## Quy tắc JOIN (khi Solution dùng từ 2 bảng trở lên)

- Chỉ JOIN các bảng mà Solution đã dùng. Solution chỉ có 1 bảng → không có `joins`.
- Nối bằng số điện thoại, CHỈ theo JOIN patterns dưới đây (alias chuẩn: `cp`, `ci`, `ph`, `phi`, `zma`, `sms`, `li`). KHÔNG nối bằng `orgId`, `id`, `refId`, `customerId`.

| Join | Điều kiện nối | Quan hệ |
|---|---|---|
| customer_profile ↔ customer_information | `cp.mainPhone = ci.phone` | 1-1 |
| customer_profile ↔ purchase_history | `cp.mainPhone = ph.customerPhone` | 1-N |
| customer_profile ↔ zma_log_cf | `cp.mainPhone = zma.phone` | 1-N |
| customer_profile ↔ sms_interaction | `cp.mainPhone = sms.phoneNumber` | 1-N |
| customer_profile ↔ lead_info | `cp.mainPhone = li.phone` | 1-N |
| customer_information ↔ purchase_history | `ci.phone = ph.customerPhone` | 1-N |
| customer_information ↔ zma_log_cf | `ci.phone = zma.phone` | 1-N |
| customer_information ↔ sms_interaction | `ci.phone = sms.phoneNumber` | 1-N |
| customer_information ↔ lead_info | `ci.phone = li.phone` | 1-N |
| customer_profile ↔ purchase_history_item | `cp.mainPhone = phi.customerPhone` | 1-N |
| customer_information ↔ purchase_history_item | `ci.phone = phi.customerPhone` | 1-N |
| purchase_history ↔ sms_interaction | `ph.customerPhone = sms.phoneNumber` | N-N |
| purchase_history ↔ zma_log_cf | `ph.customerPhone = zma.phone` | N-N |
| purchase_history ↔ lead_info | `ph.customerPhone = li.phone` | N-N |
| zma_log_cf ↔ sms_interaction | `zma.phone = sms.phoneNumber` | N-N |
| zma_log_cf ↔ lead_info | `zma.phone = li.phone` | N-N |
| sms_interaction ↔ lead_info | `sms.phoneNumber = li.phone` | N-N |

- Loại SĐT rỗng: thêm `{"field": "<field SĐT>", "operator": "not_equals", "values": [""]}` vào `joins[i].preFilter` của bảng join, để bản ghi SĐT rỗng không khớp chéo (`'' = ''`).
- `rootModel` = bảng chứa điều kiện lọc chính / đối tượng được gắn tag; `channelIdField` lấy từ `rootModel`.
- `joins[i].on` là CHUỖI SQL THÔ dạng `"<aliasBảngJoin>.<field SĐT> = <aliasRootModel>.<field SĐT>"`, VD `"sms.phoneNumber = cp.mainPhone"`.
- Mỗi phần tử `joins` CHỈ gồm các key `modelAlias`, `type`, `on`, `selectFields`, `preFilter` — `modelAlias` là alias của bảng đó trong `targetModels`. KHÔNG dùng key `model`/`alias` trong `joins`. VD: `{"modelAlias": "cp", "type": "inner", "on": "cp.mainPhone = ci.phone", "selectFields": ["mainPhone", "gender"], "preFilter": {"type": "and", "children": [{"field": "mainPhone", "operator": "not_equals", "values": [""]}, {"field": "gender", "operator": "in", "values": ["female", "Nữ"]}]}}`.
- Field dùng trong `on` PHẢI có trong `selectFields` của đúng bảng (`rootModel.selectFields` hoặc `joins[i].selectFields`).
- Điều kiện của `rootModel` đặt trong `preFilters` theo alias của `rootModel`. Điều kiện của bảng join đặt trong `joins[i].preFilter` của chính bảng đó (KHÔNG đặt trong `preFilters`).
- Khi JOIN bảng có nhiều dòng cho 1 khách (`purchase_history`, `purchase_history_item`, `zma_log_cf`, `sms_interaction`, `lead_info`) → thêm `groupBys` liệt kê các field output (gồm `channelIdField`) để mỗi khách chỉ còn 1 dòng. Cặp N-N (2 bảng nhiều dòng) nhân dòng mạnh → chỉ dùng khi điều kiện lọc thực sự nằm ở cả 2 bảng.
