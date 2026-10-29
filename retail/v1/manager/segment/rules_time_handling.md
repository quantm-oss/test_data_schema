`unified_profile_model` có time fields — TẤT CẢ đều cùng 1 type: `Long` (epoch milliseconds). KHÔNG có DateTime/Date/Timestamp riêng biệt như ClickHouse — đây là điểm khác biệt quan trọng so với Tag tier.

## Danh sách time fields trong unified_profile_model

- `createdAt` (Long, epoch millis): Thời điểm tạo record.
- `updatedAt` (Long, epoch millis): Thời điểm cập nhật record gần nhất.
- `birthday` (Long, epoch millis): Ngày sinh đầy đủ, = 0 nếu chưa có dữ liệu.
- `firstActivityAt` (Long, epoch millis): Thời điểm hoạt động đầu tiên.
- `lastActivityAt` (Long, epoch millis): Thời điểm hoạt động gần nhất.
- `lastTouchedAt` (Long, epoch millis): Thời điểm hồ sơ được cập nhật gần nhất (do job hệ thống).
- `firstPurchasedAt` (Long, epoch millis): Thời điểm đơn hàng đầu tiên, = 0 nếu chưa mua.
- `lastPurchasedAt` (Long, epoch millis): Thời điểm đơn hàng gần nhất.
- `lastPurchasedDays` (Long): Số NGÀY kể từ lần mua gần nhất tới hiện tại — đây là **số nguyên đếm ngày (recency)**, KHÔNG phải epoch millis, dùng so sánh trực tiếp (VD `<= 30`).
- `attributeLongs.MembershipIssuedAt` (Long, epoch millis): Thời điểm cấp thẻ/hạng membership.
- `attributeLongs.lastInteractOA` / `attributeLongs.LastInteractedOA` (Long, epoch millis): Thời điểm tương tác Zalo OA gần nhất.
- `lastSegmentAt.*` (Long, epoch millis): Thời điểm cập nhật gần nhất cho từng logic segment (key động, VD `lastSegmentAt.PRODUCT_INTEREST`).
- `fieldUpdatedAt.*` (Long, epoch millis): Thời điểm cập nhật gần nhất cho từng field cụ thể (key động).
- `enrichmentModels.updatedAt` / `enrichmentModels.expireAt` (Long, epoch millis): Thời điểm mô hình eRFM được tính/hết hạn.

## NGUYÊN TẮC STRICT — TUYỆT ĐỐI TUÂN THỦ

### 1. Note type — CHỈ 1 KIỂU DUY NHẤT

Mọi time field trong `unified_profile_model` note là `(Long, epoch millis)` — ngoại trừ `lastPurchasedDays` là `(Long)` số ngày đếm được (không phải millis).

**KHÔNG dùng `(DateTime)`, `(Date)`, `(Timestamp)`** cho bất kỳ field nào của Segment — các type đó chỉ áp dụng cho ClickHouse (Tag tier), KHÔNG tồn tại trong `unified_profile_model`.

### 2. Format thời gian trong solution — STRICT dd/MM/yyyy

**BẮT BUỘC dùng format `dd/MM/yyyy` cho ngày, `HH:mm dd/MM/yyyy` khi cần giờ** — dù field lưu trữ là epoch millis, solution vẫn viết mốc thời gian dạng người đọc được, không viết số millis thô.

- Đúng: `20/07/2026`, `00:00 20/06/2026`, `23:59 20/07/2026`
- Sai: `2026-07-20`, `1753027200000` (millis thô), `20 tháng 7 2026`, `July 20, 2026`

**KHÔNG mix format**. Dùng `dd/MM/yyyy` xuyên suốt solution.

### 3. Time tương đối → PHẢI convert sang tuyệt đối

**Câu hỏi user có phrase time tương đối → BẮT BUỘC compute và ghi khoảng thời gian tuyệt đối dựa trên 'THỜI GIAN HIỆN TẠI'.**

Bảng convert:
| User nói | Compute → Ghi trong solution |
|---|---|
| "30 ngày gần đây" | "từ {today-30} đến {today}" |
| "6 tháng gần nhất" | "từ {today-6months} đến {today}" |
| "tháng này", "tháng hiện tại" | "từ 01/{month}/{year} đến {last_day}/{month}/{year}" |
| "quý này" | "từ 01/{quarter_start_month}/{year} đến {quarter_end_day}/{quarter_end_month}/{year}" |
| "năm nay" | "từ 01/01/{year} đến 31/12/{year}" |
| "tuần này" | "từ {monday} đến {sunday}" |

**KHÔNG ghi "30 ngày gần đây" trong solution** — luôn convert sang ngày tuyệt đối.

**Ngoại lệ — `lastPurchasedDays`**: field này đã là recency dạng số ngày sẵn, KHÔNG cần convert sang khoảng ngày tuyệt đối. "Mua trong 30 ngày gần đây" có thể map trực tiếp `lastPurchasedDays <= 30` HOẶC `lastPurchasedAt` trong khoảng tuyệt đối — ưu tiên `lastPurchasedDays` nếu solution chỉ cần so sánh recency đơn giản, không cần khoảng ngày cụ thể.

### 4. Chọn đúng field theo nhu cầu nghiệp vụ

- Lọc theo **thời điểm hoạt động/tương tác** chung: dùng `lastActivityAt`/`firstActivityAt` (Long, epoch millis).
- Lọc theo **thời điểm mua hàng**: dùng `lastPurchasedAt`/`firstPurchasedAt` (Long, epoch millis), hoặc `lastPurchasedDays` (Long) nếu chỉ cần recency dạng số ngày.
- Lọc theo **thời điểm hồ sơ tạo/cập nhật**: dùng `createdAt`/`updatedAt` (Long, epoch millis).
- Lọc theo **thời điểm cấp hạng thành viên**: dùng `attributeLongs.MembershipIssuedAt` (Long, epoch millis).
- Lọc theo **thời điểm tương tác Zalo OA**: dùng `attributeLongs.lastInteractOA` (Long, epoch millis) — kiểm tra kỹ có thể tồn tại biến thể `LastInteractedOA`.
- Lọc theo **thời điểm cập nhật segment cụ thể**: dùng `lastSegmentAt.<KeyName>` (Long, epoch millis) tương ứng với segment key đang lọc.

### 5. Timezone

- Mọi tính toán time dùng timezone **Việt Nam (GMT+7)**.
- KHÔNG convert sang UTC hay timezone khác.
- Note "(GMT+7)" khi ghi khoảng thời gian trong solution.

### 6. Không bịa time constraint

- Câu hỏi user KHÔNG mention thời gian → KHÔNG thêm time constraint vào solution.
- VD: "khách hàng hạng Gold" → chỉ có điều kiện `attributes.MembershipLevel = "Gold"`, KHÔNG thêm "trong 30 ngày qua".

## Ví dụ áp dụng

### Ví dụ 1 — Mua hàng "30 ngày gần đây"

**User hỏi**: "Khách đã mua hàng trong 30 ngày gần đây"

**Solution ĐÚNG**:
```markdown
### Điều kiện lọc (logic)
- `lastPurchasedAt` (Long, epoch millis) trong khoảng từ 07/07/2026 đến 06/08/2026 (GMT+7)

### Trường dữ liệu tham chiếu
- `lastPurchasedAt` (Long, epoch millis): Thời điểm mua hàng gần nhất, dùng filter khoảng thời gian.
```

**Solution SAI**:
```
- purchase_history.transCreatedAt (DateTime) trong 30 ngày gần đây   ← field/prefix thuộc ClickHouse, KHÔNG tồn tại trong unified_profile_model
- lastPurchasedAt >= 2026-06-20                                       ← thiếu note type, sai format ngày
```

### Ví dụ 2 — Hạng thành viên cấp "tháng này"

**User hỏi**: "Khách được cấp thẻ thành viên trong tháng này"

**Solution ĐÚNG**:
```markdown
### Điều kiện lọc (logic)
- `attributeLongs.MembershipIssuedAt` (Long, epoch millis) trong khoảng từ 01/08/2026 đến 31/08/2026 (GMT+7)

### Trường dữ liệu tham chiếu
- `attributeLongs.MembershipIssuedAt` (Long, epoch millis): Thời điểm cấp thẻ/hạng membership.
```

### Ví dụ 3 — Hồ sơ tạo gần đây

**User hỏi**: "Hồ sơ khách hàng mới tạo trong 7 ngày qua"

**Solution ĐÚNG**:
```markdown
### Điều kiện lọc (logic)
- `createdAt` (Long, epoch millis) trong khoảng từ 30/07/2026 đến 06/08/2026 (GMT+7)

### Trường dữ liệu tham chiếu
- `createdAt` (Long, epoch millis): Thời điểm tạo record.
```

### Ví dụ 4 — Không có time constraint

**User hỏi**: "Khách hàng hạng Gold có điểm loyalty trên 1000"

**Solution ĐÚNG**:
```markdown
### Điều kiện lọc (logic)
- `attributes.MembershipLevel` (String) = "Gold"
- AND `attributeInts.LoyaltyPoint` (Long) >= 1000
```

(KHÔNG thêm time constraint vì user không mention thời gian)

### Ví dụ 5 — Recency dùng field đếm ngày sẵn có

**User hỏi**: "Khách hàng chưa mua hàng trong 90 ngày qua"

**Solution ĐÚNG**:
```markdown
### Điều kiện lọc (logic)
- `lastPurchasedDays` (Long) > 90

### Trường dữ liệu tham chiếu
- `lastPurchasedDays` (Long): Số ngày kể từ lần mua gần nhất đến hiện tại (recency).
```

(Ưu tiên `lastPurchasedDays` thay vì convert sang khoảng ngày tuyệt đối trên `lastPurchasedAt`, vì field này đã sẵn sàng cho phép so sánh recency trực tiếp)