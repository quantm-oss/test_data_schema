Brief Solution CHỈ được reference đến các field có THẬT trong schema `unified_profile_model` được inject bên dưới (fetch runtime từ `unifiedProfileUrl`).

## Nguyên tắc bắt buộc

1. **Tồn tại thật**: Mỗi `field` xuất hiện trong `conditions`, `exclusions`, hoặc `timeRange` PHẢI tồn tại trong schema JSON được cung cấp — kiểm tra cả field top-level LẪN key con của các Map động (`attributes.*`, `attributeInts.*`, `attributeLongs.*`, `lastSegmentKeys.*`...).
2. **Không bịa field**: TUYỆT ĐỐI không tự đặt tên field mới, không suy đoán field "có thể tồn tại". Nếu không chắc, coi như KHÔNG map được.
3. **Không bịa key Map động**: Với field dạng Map (`attributes`, `attributeInts`, `attributeLongs`...), key con PHẢI là key đã có ví dụ/mô tả rõ trong schema — không tự bịa key mới.
4. **Đúng tên, case-sensitive**: field phải khớp chính xác tên trong schema (VD `mainPhone`, không phải `MainPhone` hay `main_phone`).
5. **Mọi giá trị sinh ra PHẢI dựa trên nội dung schema thật** — KHÔNG đoán field/giá trị dựa trên kiến thức chung về CRM/segment ở các hệ thống khác.

## Khi 1 ý định trong câu hỏi KHÔNG map được field nào (thật sự không tồn tại field khớp semantic)

- KHÔNG được bỏ qua âm thầm.
- KHÔNG được chọn field gần đúng để "cho có".
- PHẢI đưa vào `unresolved_items` với `user_phrase` = cụm từ gốc trong câu hỏi, `reason` = giải thích ngắn gọn vì sao không map được (VD: "Không tìm thấy field/key tương ứng với 'điểm tín dụng ngân hàng' trong schema unified_profile_model").
- Theo `rules_pass_fail_criteria.md`, chỉ cần 1 phần tử trong `unresolved_items` → `matched: false` cho TOÀN BỘ câu hỏi.
- Chỉ áp dụng nhánh này khi field THẬT SỰ không tồn tại — xem mục tiếp theo để phân biệt với trường hợp field tồn tại nhưng giá trị không rõ.

## Trường hợp field TỒN TẠI nhưng giá trị/enum không rõ ràng — KHÔNG reject, RESOLVE bằng exist + assumptions

Phân biệt rõ 2 tình huống:

1. **Field không map được (thật sự KHÔNG tồn tại field nào khớp semantic)** → xử lý theo mục "Khi 1 ý định trong câu hỏi KHÔNG map được field nào" ở trên (dẫn tới FAIL toàn bộ).

2. **Field TỒN TẠI trong schema và mô tả (`description`) của nó khớp semantic với ý định trong câu hỏi, nhưng schema KHÔNG cho biết chính xác giá trị/enum cần dùng** (VD field mô tả "Thông tin xác định trạng thái và hạng VIP của khách hàng" nhưng không liệt kê giá trị cụ thể là gì) → **KHÔNG được đưa vào `unresolved_items`**. Thay vào đó:
   - Dùng operator `"exist"` (hoặc `"is_not_null"`) với `value: null` — coi việc field có giá trị (khác null) là dấu hiệu hợp lý nhất có thể suy ra từ mô tả.
   - BẮT BUỘC ghi rõ giả định này vào mảng `assumptions` theo format: `"Giả định '<cụm từ user dùng>' được hiểu là trường '<field>' tồn tại giá trị (không null) — vui lòng xác nhận lại nếu dữ liệu thực tế có giá trị khác (VD 'false') cần được loại trừ riêng"`.
   - Case này VẪN tính là map được, KHÔNG làm câu hỏi bị FAIL.

**Nguyên tắc phân biệt**: field có `description` mô tả ĐÚNG đối tượng/khái niệm người dùng nhắc tới (dù không rõ giá trị cụ thể) → thuộc tình huống 2 (resolve). Field hoàn toàn không có mô tả liên quan tới khái niệm đó → thuộc tình huống 1 (reject).

## Ví dụ

**Sai (reject dù field tồn tại và khớp semantic)**: User hỏi "khách hàng thuộc nhóm VIP" → schema CÓ `attributes.vip` (String, mô tả "Thông tin xác định trạng thái và hạng VIP của khách hàng") → LLM trả `matched: false` vì không rõ giá trị cụ thể.

**Đúng**: Field `attributes.vip` tồn tại và mô tả khớp đúng semantic "VIP" → dùng `{"field": "attributes.vip", "operator": "exist", "value": null}`, ghi vào `assumptions`: "Giả định 'thuộc nhóm VIP' được hiểu là trường 'attributes.vip' tồn tại giá trị (không null) — vui lòng xác nhận lại nếu dữ liệu thực tế có giá trị khác (VD 'false') cần được loại trừ riêng".

**Sai**: User hỏi "khách có memberLevel Gold" → LLM output field `memberLevel` (bịa tên field, không kiểm tra kỹ Map `attributes`).

**Đúng**: Kiểm tra schema → không có field top-level `memberLevel`, kiểm tra kỹ các key con đã biết trong `attributes.*` (VD `attributes.loyaltyCode`, `attributes.vip`) — nếu không có key nào phản ánh đúng khái niệm "hạng Gold cụ thể", đây là field THẬT SỰ không tồn tại (không phải case "tồn tại nhưng không rõ giá trị") → đưa vào `unresolved_items`.

**Sai**: User hỏi "khách đã tương tác Zalo OA trong 30 ngày qua" → LLM output field `zaloInteractionDate` (bịa field không tồn tại).

**Đúng**: Kiểm tra schema → không có field `zaloInteractionDate` và không có field nào khác mô tả khớp "thời điểm tương tác Zalo OA" → đưa vào `unresolved_items`.

**Trường hợp thực sự không có field tương đương**: User hỏi "khách có điểm tín dụng ngân hàng trên 700" → Kiểm tra toàn bộ schema (bao gồm `attributes.*`, `attributeInts.*`, `attributeLongs.*` đã biết) → không có field nào lưu điểm tín dụng ngân hàng, cũng không có field nào có mô tả liên quan gần đúng → Trả `matched: false`, giải thích rõ schema hiện tại không có field lưu thông tin này.