# contact_index

## Description

Bảng CORE identifier hub. Mỗi record là 1 channel identifier (Phone/Email/Zalo/FB) của contact. Cùng 1 contact có nhiều records với `type` khác nhau.

Total ~138K records, 99.9% status=active.

Dùng khi cần:
- Multi-channel identity resolution
- Filter theo channel type hoặc source ingest
- Join với các bảng business (purchase_history, customer_information) qua `value ↔ phone/email/customerPhone`

## Table Name Pattern

`contact_index_bdf66a48c3ebfaec10d2feb3492099ea`

## Fields

| Name | Type | Enum values | Note |
|---|---|---|---|
| id | String | | UUID |
| orgId | String | | tenant ID |
| contactKey | String | | composite key `{type}:{value}`. Example: `Phone:0987444504` |
| value | String | | **GIÁ TRỊ IDENTIFIER** — SĐT/email/zaloUid/fbPsid tuỳ `type`. Example: `0987444504`, `user@gmail.com`. **channelIdField khi query bảng này** |
| type | String | Phone(112K), Email(25K), ZaloUId(629), FbPsid(7) | Loại channel identifier |
| source | String | Customer, PurchaseHistory, ZaloProfile, CustomerMembership, SocialCRM, CRM, Import, "" | Nguồn ingest identifier |
| status | String | active, merged | **LUÔN filter `status='active'`** để tránh identifier đã merge |
| rootUserId | String | | User ID gốc trước merge |
| curRootUserId | String | | User ID hiện tại sau merge |
| name | String nullable | | Tên contact. Example: `"Trần Ngọc Thu Trang"` |
| groupValue | String nullable | | |
| confidence | String | "", "strong" | Confidence level |
| merged | Bool | | |
| mergedTo | String nullable | | |
| mergedSrc | String nullable | | |
| mergedLogId | String nullable | | |
| createdAt | DateTime64(3) | | UTC timestamp |
| updatedAt | DateTime64(3) | | UTC timestamp |
| updateReason | String nullable | | |

## channelIdField Guidance

Khi query bảng này với channelType tương ứng, dùng field `value` (không phải `phone`/`email`) làm `channelIdField`:

```json
{
  "channelType": "Phone",
  "channelIdField": "value",
  "targetModels": [{"model": "contact_index_bdf66a48c3ebfaec10d2feb3492099ea", "alias": "ci"}],
  ...
}
```

Kết hợp filter `type = 'Phone'` để chỉ lấy identifier loại Phone.

## Data Quality Notes

- 8 giá trị `source` khác nhau, chú ý không có `"Website"` — chỉ có `"Customer"`, `"PurchaseHistory"`, ...
- `status='merged'` records nên được filter loại bỏ (chỉ lấy `active`)