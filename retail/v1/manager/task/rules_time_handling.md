Các field time trong schema Task, TẤT CẢ đều là kiểu `DateTime64` (native ClickHouse) — không có field String:

**Bảng `customer_unified`**:
- `createdAt`, `updatedAt` — kiểu `DateTime64(3, 'UTC')`

**Bảng `customer_tagging_staging`**:
- `createdAt`, `expiredAt`, `lastTouchedAt` — kiểu `DateTime64(3, 'UTC')`

## NGUYÊN TẮC STRICT — TUYỆT ĐỐI TUÂN THỦ

### 1. Format thời gian trong solution — STRICT dd/MM/yyyy

**BẮT BUỘC dùng format `dd/MM/yyyy` cho ngày, `HH:mm dd/MM/yyyy` khi cần giờ.**

- Đúng: `20/07/2026`, `00:00 20/06/2026`, `23:59 20/07/2026`
- Sai: `2026-07-20`, `20 tháng 7 2026`, `July 20, 2026`, `20/7/26`

### 2. Time tương đối → PHẢI convert sang tuyệt đối

**Câu hỏi user có phrase time tương đối → BẮT BUỘC compute và ghi khoảng thời gian tuyệt đối dựa trên `THỜI GIAN HIỆN TẠI` ở trên.**

Bảng convert:
| User nói | Compute → Ghi trong solution |
|---|---|
| "30 ngày gần đây" | "từ 20/06/2026 đến 20/07/2026" |
| "6 tháng gần nhất" | "từ 20/01/2026 đến 20/07/2026" |
| "tháng này", "tháng hiện tại" | "từ 01/07/2026 đến 31/07/2026" |
| "tuần này" | "từ 20/07/2026 đến 26/07/2026" |
| "24 giờ qua" | "từ 20/07/2026 12:00 đến 21/07/2026 12:00" |

**KHÔNG ghi "30 ngày gần đây" trong solution** — luôn convert sang tuyệt đối.

### 3. Note type explicit khi reference time field

Mỗi lần reference time field trong "Điều kiện lọc" và "Trường dữ liệu tham chiếu", PHẢI note type theo schema:

- `customer_unified.createdAt` (DateTime64(3, UTC))
- `customer_unified.updatedAt` (DateTime64(3, UTC))
- `customer_tagging_staging.createdAt` (DateTime64(3, UTC))
- `customer_tagging_staging.expiredAt` (DateTime64(3, UTC))
- `customer_tagging_staging.lastTouchedAt` (DateTime64(3, UTC))

Consistency: mọi time field trong 1 solution đều note type `DateTime64(3, UTC)` — không skip, KHÔNG ghi "String" hay "ISO8601" cho bất kỳ field nào trong 2 bảng này.

### 4. Timezone

- Mọi tính toán time dùng timezone **Việt Nam (GMT+7)**.
- Toàn bộ time field của cả 2 bảng lưu theo **UTC** trong ClickHouse → mental convert sang GMT+7 khi mô tả khoảng thời gian cho user.
- Ghi trong solution: khoảng thời gian mặc định GMT+7, note "(GMT+7)" khi cần rõ.

### 5. Không bịa time constraint

- Câu hỏi user KHÔNG mention thời gian → KHÔNG thêm time constraint vào solution.
- VD: "gán task cho khách có tag highvalue" → chỉ điều kiện tag, KHÔNG thêm "trong 30 ngày qua".

## Ví dụ áp dụng

### Ví dụ 1 — Task với time constraint

**User hỏi**: "Gán task chăm sóc khách vừa được gắn tag highvalue trong 7 ngày gần đây"

**Solution ĐÚNG**:
Điều kiện lọc (logic)
- `customer_tagging_staging.tagId` equals "highvalue"
- AND `customer_tagging_staging.createdAt` (DateTime64(3, UTC)) trong khoảng từ 14/07/2026 đến 20/07/2026 (GMT+7)

### Ví dụ 2 — Task không time constraint

**User hỏi**: "Gán task chăm sóc khách có tag highvalue"

**Solution ĐÚNG**:
Điều kiện lọc (logic)
- `customer_tagging_staging.tagId` equals "highvalue"
- AND `customer_tagging_staging.status` like "active"

(KHÔNG thêm time constraint vì user không mention)