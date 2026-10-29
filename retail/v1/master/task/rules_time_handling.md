Task DSL Rule dùng các operator time relative sau (KHÔNG có 'relative_time' như Segment).

## BẮT BUỘC — Cách ghi điều kiện thời gian vào DSL (database UTC, người dùng GMT+7)

Các ký hiệu `gte_relative_day: [N]`, `lte_relative_day: [N]`, `between_relative_day: [A, B]` trong file này CHỈ dùng để xác định số ngày N / A / B. Khi ghi vào DSL, KHÔNG dùng operator `*_relative_day`, mà chuyển sang `gte`/`lte` với giá trị là biểu thức mốc 00:00 giờ Việt Nam (KHÔNG bọc dấu nháy, giữ nguyên văn, chỉ thay số ngày):

`MỐC(N)` = `toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL N DAY)`

| Xác định được | Ghi vào DSL |
|---|---|
| `gte_relative_day: [N]` | `{"field": "<field>", "operator": "gte", "values": ["MỐC(N)"]}` |
| `lte_relative_day: [N]` | `{"field": "<field>", "operator": "lte", "values": ["MỐC(N)"]}` |
| `between_relative_day: [A, B]` | 2 điều kiện cùng nằm trong một nhóm `and`: `gte` `MỐC(B)` và `lte` `MỐC(A)` |

VD "trong 7 ngày qua": `{"field": "createdAt", "operator": "gte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 7 DAY)"]}`.

**QUY TẮC CHẶT khi ghi `MỐC(N)` (kiểm tra từng điều kiện thời gian trước khi trả DSL):**
1. Operator lấy ĐÚNG theo bảng trên, KHÔNG tự đổi: "N ngày gần đây"/"hôm nay" → `gte`; "trước N ngày" → `lte`; KHÔNG thay `lte` bằng `lt`, `gte` bằng `gt` (và ngược lại). `lt`/`gt` CHỈ dùng cho trường hợp "ngoài khoảng A-B ngày".
2. Khoảng "A đến B ngày" / "trong khoảng A-B ngày qua" / "cách đây A-B ngày" (kể cả câu phủ định như "không tương tác trong khoảng 7-30 ngày qua") → BẮT BUỘC đúng 2 điều kiện trên CÙNG 1 field, đặt cạnh nhau trong CÙNG 1 nhóm `"type": "and"`: `gte` với `MỐC(<số LỚN hơn>)` và `lte` với `MỐC(<số NHỎ hơn>)`. KHÔNG được thiếu 1 trong 2 điều kiện, KHÔNG gộp thành 1 điều kiện.
3. Số ngày LỚN hơn (xa hiện tại hơn) luôn đi với `gte`; số NHỎ hơn (gần hiện tại hơn) luôn đi với `lte`. VD 7-30 ngày → `gte MỐC(30)` + `lte MỐC(7)`; KHÔNG viết `gte MỐC(7)` + `lte MỐC(30)` (khoảng rỗng).
4. `values` có ĐÚNG 1 phần tử là chuỗi biểu thức chép NGUYÊN VĂN mẫu `toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL N DAY)`, chỉ thay N bằng số nguyên ≥ 0: KHÔNG bọc thêm dấu nháy đơn ngoài biểu thức, KHÔNG đổi khoảng trắng/hoa thường, KHÔNG đổi `DAY` thành `WEEK`/`MONTH` (tự quy đổi: 1 tuần = 7, 1 tháng = 30 ngày), KHÔNG thêm `+`/`-` khác, KHÔNG tự tính ra ngày cụ thể.
5. "Hôm nay" → `gte MỐC(0)`; KHÔNG dùng cặp `gte MỐC(0)` + `lte MỐC(0)` (khoảng rỗng).

Ví dụ đúng — khoảng 7-30 ngày:
```json
{"type": "and", "children": [
  {"field": "eventAt", "operator": "gte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 30 DAY)"]},
  {"field": "eventAt", "operator": "lte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 7 DAY)"]}
]}
```
Ví dụ đúng — trước 90 ngày: `{"field": "createdAt", "operator": "lte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 90 DAY)"]}` (SAI: `"operator": "lt"`).

## Operators time

- 'gte_relative_day: [N]' → field >= today - N days (trong N ngày gần đây tính đến trước)
- 'lte_relative_day: [N]' → field <= today - N days (trước N ngày)
- 'between_relative_day: [A, B]' → field trong khoảng [today - B, today - A] (từ A ngày trước đến B ngày trước, B > A)

## Time fields cho Task

Toàn bộ time field trong cả 2 bảng đều là **`DateTime64(3, 'UTC')`** (native ClickHouse), KHÔNG có field String:

- 'customer_tagging_staging.createdAt' — Thời điểm gắn tag
- 'customer_tagging_staging.expiredAt' — Thời điểm hết hạn tag
- 'customer_tagging_staging.lastTouchedAt' — Lần cuối tương tác
- 'customer_unified.createdAt', 'customer_unified.updatedAt' — Timestamp khách hàng

## NGUYÊN TẮC ƯU TIÊN SỐ 1 — Luôn dùng số ngày N tường minh nếu có, KHÔNG tự tính lại từ ngày tuyệt đối

Description từ Manager Task thường chứa **CẢ 2**: câu chữ gốc có số ngày tường minh (VD "trong 100 ngày gần đây") **VÀ** cặp ngày tuyệt đối tương ứng (VD "từ 13/05/2026 đến 20/08/2026").

**BẮT BUỘC**: Nếu description có con số N tường minh trong câu chữ (dạng "N ngày qua", "N ngày gần đây", "trong N ngày") → **DÙNG TRỰC TIẾP N làm offset**, ví dụ `gte_relative_day: [N]`.

**Chỉ khi description KHÔNG có con số N tường minh nào** (chỉ có cặp ngày tuyệt đối, không có câu chữ dạng "N ngày") → mới áp dụng THUẬT TOÁN TÍNH OFFSET ở mục dưới đây.

### Ví dụ áp dụng nguyên tắc ưu tiên

**Description**: "Gán task ... trong khoảng từ 13/05/2026 đến 20/08/2026 (100 ngày gần đây)"
- Description CÓ con số tường minh "100 ngày" → dùng trực tiếp `gte_relative_day: [100]`
- **SAI**: tự trừ 20/08/2026 - 13/05/2026 rồi ra offset 99 hoặc 101 do tính nhầm — KHÔNG được làm vậy khi đã có sẵn con số 100.

**Description**: "Gán task ... trong khoảng từ 25/10/2025 đến 20/08/2026" (KHÔNG có số ngày tường minh trong câu chữ)
- Không có con số N sẵn → áp dụng thuật toán tính offset ở mục dưới.

## Nguyên tắc mapping description → operator

Description Manager Task đã convert time về **absolute dd/MM/yyyy** (và thường kèm số ngày tường minh trong Summary/Mục tiêu — LUÔN ưu tiên số này theo nguyên tắc trên). Master Task convert **absolute → relative_day** cho DSL:

| Description có | DSL operator | Values |
|---|---|---|
| "trong 7 ngày qua" hoặc "từ 21/07/2026 đến 28/07/2026" (7 ngày) | 'gte_relative_day' | '[7]' |
| "trong 30 ngày qua" hoặc "từ 30/06/2026 đến 30/07/2026" | 'gte_relative_day' | '[30]' |
| "180 ngày gần đây" | 'gte_relative_day' | '[180]' |
| "cách đây 30-60 ngày" | 'between_relative_day' | '[30, 60]' |
| "trước 90 ngày qua" | 'lte_relative_day' | '[90]' |

**Bảng trên chỉ là ví dụ minh họa** — KHÔNG phải danh sách đóng. Với cặp ngày tuyệt đối khác (không khớp ví dụ) và KHÔNG có số N tường minh, áp dụng thuật toán dưới đây.

## THUẬT TOÁN TÍNH OFFSET — chỉ dùng khi description KHÔNG có số N tường minh

### Bước 1 — Tính offset

`offset = số ngày dương lịch giữa THỜI GIAN HIỆN TẠI và ngày trong description` (làm tròn xuống, không cộng/trừ thêm 1).

- Ngày = hôm nay → offset = `0`
- Ngày = N ngày trước hôm nay → offset = `N`

TUYỆT ĐỐI KHÔNG lấy offset từ số liệu khác trong context (`windowTime.value`, `dueInDays`, số trong bảng ví dụ...) — chỉ tính từ chênh lệch ngày thực tế.

### Bước 2 — Chọn operator theo offset của end date

- offset end date = `0` (= hôm nay) → `gte_relative_day: [offset_start]`, KHÔNG dùng `between_relative_day`.
- offset end date > `0` (mốc quá khứ cụ thể) → `between_relative_day: [offset_start, offset_end]`.

### Ví dụ (THỜI GIAN HIỆN TẠI = 20/08/2026)

**Case A**: "từ 25/10/2025 đến 20/08/2026" → offset_end = 0 → `gte_relative_day: [300]`.
Sai (đã từng xảy ra): `between_relative_day: [305, 30]`.

**Case B**: "từ 01/01/2026 đến 31/03/2026" (hiện tại đã qua 31/03) → offset_end = 142 (> 0) → `between_relative_day: [231, 142]`.

## Ví dụ preFilter với time

```json
"preFilters": {
  "tag": {
    "type": "and",
    "children": [
      {"field": "tagId", "operator": "in", "values": ["Retail_HOT"]},
      {"field": "createdAt", "operator": "gte", "values": ["toStartOfDay(now('Asia/Ho_Chi_Minh') - INTERVAL 180 DAY)"]}
    ]
  }
}
```

## Không bịa time constraint

- Description KHÔNG mention time → KHÔNG thêm time filter vào DSL
- Description KHÔNG có ngày cụ thể → KHÔNG generate absolute date filter

## Timezone

Mọi tính toán time = GMT+7 (Việt Nam). Database lưu UTC nên mốc thời gian PHẢI ghi bằng biểu thức `MỐC(N)` ở mục "BẮT BUỘC" trên đầu (đã quy đổi 00:00 giờ Việt Nam).