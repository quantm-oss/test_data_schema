Task rule có 3 đặc điểm bắt buộc phải tuân thủ.

## 1. BẮT BUỘC dùng cả 2 bảng — join pattern

Task luôn cần **join** giữa 2 bảng:

- `customer_unified` — cung cấp identifier khách hàng (`rootId`, tên, source)
- `customer_tagging_staging` — cung cấp thông tin và log tagging (`customerId`, `tagId`, `status`, channel info)

Join key: `customer_unified.rootId = customer_tagging_staging.customerId`

### Nguyên tắc

- Solution PHẢI reference đến cả 2 bảng trong "Trường dữ liệu tham chiếu".
- Điều kiện lọc có thể chủ yếu từ 1 bảng (VD chỉ filter `tagId`), nhưng identifier khách hàng luôn từ cả 2:
  - `customer_unified.rootId` — target khách hàng
  - `customer_tagging_staging.customerId` — cho phép match tagging log

### Ví dụ

**Solution ĐÚNG** — filter tag "highvalue", target rootId, dùng cả 2 bảng:

### Điều kiện lọc (logic)
- Khách hàng đang có tag "highvalue" (customer_tagging_staging.tagId equals "highvalue")
- AND status active (customer_tagging_staging.status like "active")

### Trường dữ liệu tham chiếu
- `customer_unified.rootId` (String): Định danh khách hàng (tham chiếu để gắn Task)
- `customer_tagging_staging.customerId` (String): Mã khách hàng
- `customer_tagging_staging.tagId` (String): ID của tag cần filter
- `customer_tagging_staging.status` (String): Trạng thái tag

**Solution SAI** — chỉ dùng 1 bảng:
### Trường dữ liệu tham chiếu
- `customer_tagging_staging.tagId` (String): ID của tag ← thiếu identifier khách hàng từ customer_unified

## 2. BẮT BUỘC user cung cấp tagId cụ thể

- Task rule filter theo tag → user PHẢI cung cấp tagId cụ thể (VD: "highvalue", "new_signup", "vip_repeat_customer_1Y_phone").

### Nguyên tắc

- User chỉ định tagId rõ ràng → tạo solution với `customer_tagging_staging.tagId equals "<tagId>"`.
- User KHÔNG chỉ định tagId cụ thể (chỉ nói chung chung "chăm sóc khách VIP", "gán task cho khách phù hợp") → trả `matched: false` với message giải thích cần cung cấp tagId.
- KHÔNG được LLM tự bịa tagId hay đoán từ context.

### Ví dụ

**Case ĐÚNG — user cung cấp tagId**:

User: "Gán task chăm sóc cho khách có tag highvalue"
→ Solution matched=true với tagId = "highvalue"

**Case matched=false — user không cung cấp tagId**:

User: "Gán task chăm sóc cho khách hàng quan trọng"
→ matched=false, message: "Cần cung cấp tagId cụ thể để filter khách hàng. Ví dụ: highvalue, vip_repeat_customer_1Y_phone..."

## 3. BẮT BUỘC chọn 1 task tag từ danh sách available

Task tags được cung cấp qua danh sách khả dụng ở trên. Solution PHẢI chọn 1 giá trị cụ thể.

### Nguyên tắc

- Chọn task tag khớp nhất với intent gán task (target + action).
- Task tag = giá trị cụ thể, PHẢI lấy từ danh sách task tags thực tế được inject vào prompt ở runtime — KHÔNG bịa tên tag ngoài danh sách đó.
- Nếu không có task tag nào phù hợp → trả `matched: false`.

### Mapping intent → task tag (chỉ là HINT minh họa, không phải danh sách cố định)

| Intent gán task | Task tag gợi ý (ví dụ minh họa) |
|---|---|
| Chăm sóc khách mới được import/register | `new_customer` |
| Chăm sóc theo tương tác gần đây | `new_customer_interaction` |
| Chăm sóc khách 1 tháng gần đây | `customer_1m` |
| Chăm sóc khách hàng general | `customer` |

(Bảng trên CHỈ là ví dụ minh họa cách đặt tên — LLM PHẢI dùng đúng giá trị có trong danh sách task tags thực tế được inject vào prompt ở runtime cho tenant hiện tại, KHÔNG mặc định các giá trị trong bảng này luôn tồn tại.)

## 4. Xử lý String

Phân loại field String theo 3 nhóm, mỗi nhóm 1 operator phù hợp:

**Nhóm 1 — ID hệ thống (exact match bắt buộc)**: `tagId`, `ruleId`, `customerId`, `requestId`, `dslResultId`
→ Dùng `equals`. KHÔNG dùng `like`/`contains` vì rủi ro match nhầm ID gần giống nhau (VD "vip" match nhầm cả "vip_gold").

**Nhóm 2 — Enum ít giá trị (rủi ro case-sensitivity)**: `status`, `channelType`, `identifyStatus`, `tagCategory`, `tagSubCategory`, `sourceType`
→ Dùng `like` với values KHÔNG có wildcard `%` (VD `values: ["active"]`) — match chính xác toàn chuỗi nhưng an toàn trước rủi ro sai case (đã phát hiện thực tế `status` lưu chữ thường `"active"`, không phải `"ACTIVE"`). KHÔNG dùng `equals` cho nhóm này để tránh fail do case không khớp.

**Nhóm 3 — Text tự do**: `name` (customer_unified), `note` (customer_tagging_staging)
→ Dùng `like` với wildcard `%keyword%` — fuzzy match chứa chuỗi con.

**Map access** (detailString, detailBoolean, detailLong, detailDouble, detailDate, detailArrayString, detailArrayLong, detailArrayDouble): TOÀN BỘ 8 Map này hiện RỖNG 100% trong dữ liệu thực tế — TUYỆT ĐỐI KHÔNG dùng.

### Ví dụ

**ĐÚNG**:
- `customer_tagging_staging.tagId` equals "highvalue"           ← Nhóm 1, ID hệ thống
- `customer_tagging_staging.status` like "active"               ← Nhóm 2, enum — an toàn trước rủi ro case
- `customer_tagging_staging.channelType` like "Phone"           ← Nhóm 2, enum
- `customer_unified.name` like "%Nguyễn%"                       ← Nhóm 3, text tự do

**SAI**:
- `customer_tagging_staging.tagId` like "%highvalue%"           ← Nhóm 1 không cần wildcard, rủi ro match nhầm ID khác
- `customer_tagging_staging.status` equals "ACTIVE"              ← SAI CASE, giá trị thật là chữ thường "active", equals sẽ không match được record nào
- `customer_unified.name` equals "Nguyễn Văn A"                  ← equals dễ miss variance của tên, Nhóm 3 phải dùng like + wildcard
- `customer_tagging_staging.detailString.channelName` like "%Zalo OA%"  ← field Map này rỗng 100%, không được dùng

## 5. Tên task gợi ý — pattern chuẩn

Format: `assign_[intent]_[target]_task`

Components:
- `assign_` — prefix cố định
- `[intent]` — mô tả ngắn gọn ý định (VD: `care`, `welcome`, `followup`, `nurturing`, `winback`)
- `[target]` — nhóm khách hàng target (VD: `highvalue_customer`, `new_customer`, `hot_lead`, `abandoned_cart`)
- `_task` — suffix cố định

### Ví dụ

- `assign_highvalue_customer_care_task` — chăm sóc khách giá trị cao
- `assign_new_customer_welcome_task` — welcome khách mới
- `assign_hot_lead_sales_call_task` — Sales gọi lead hot
- `assign_abandoned_cart_reminder_task` — nhắc giỏ hàng bỏ dở