Khi filter với `equals` / `in` / `not_in` trên field có enum trong schema:

- **DÙNG ĐÚNG giá trị** trong danh sách enum (kể cả tiếng Việt có dấu, case exact như `"Vàng"` không phải `"vàng"`)
- **KHÔNG suy diễn** giá trị mới không có trong list (như `"Website"` khi enum chỉ có `"Web"`)

## Xử lý data quality issues

Nếu enum có DATA QUALITY ISSUES trong tenant, cover tất cả variants:

### Typo trong data

Ví dụ: 'lead_info.campaignType' có cả 'conversion' (đúng) và 'convertion' (typo). Solution nói "conversion campaign":

```json
{"field": "campaignType", "operator": "in", "values": ["conversion", "convertion"]}
```

### Mixed case + language

Ví dụ: 'customer_profile.gender' có 5 values 'male | female | Male | Nữ | unknown'.

Solution nói "khách hàng nữ":
```json
{"field": "gender", "operator": "in", "values": ["female", "Nữ"]}
```

Solution nói "khách hàng nam":
```json
{"field": "gender", "operator": "in", "values": ["male", "Male"]}
```

### Field có multiple formats

Ví dụ: 'lead_info.modelInterest' có 2 formats: 'P_SUV' (plain) và '[P_SUV]' (bracketed).

DÙNG `like` để match cả 2 format:
```json
{"field": "modelInterest", "operator": "like", "values": ["%SUV%"]}
```

## Enum tiếng Việt có dấu

Với 'customer_information.memberLevel', 'customer_information.source', 'purchase_history.storeCity' — enum values có dấu tiếng Việt. Filter phải EXACT match:

- `"Vàng"` (đúng), không phải `"Vang"` (bỏ dấu) hay `"vàng"` (lowercase)
- `"Hồ Chí Minh"` (đúng), không phải `"Ho Chi Minh"` hay `"TPHCM"`
- `"Chiến dịch"` (đúng), không phải `"Chien dich"`