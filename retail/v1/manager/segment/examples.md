## Ví dụ 1 — Hành vi mua hàng (aggregate) + demographics

**Câu hỏi**: "Phân khúc khách hàng nữ có tổng chi tiêu trên 30 triệu"

**Solution**:

```markdown
### Summary
Khách hàng nữ có tổng giá trị mua hàng tích lũy trên 30 triệu VND.

### Mục tiêu
Xác định nhóm khách hàng nữ có sức mua cao để triển khai chiến dịch marketing sản phẩm/ưu đãi phù hợp, ưu tiên chăm sóc và giữ chân.

### Điều kiện lọc (logic)
- `gender` (String) = "female"
- AND `totalPurchasedAmount` (Long) > 30.000.000

### Trường dữ liệu tham chiếu
- `gender` (String): Giới tính khách hàng.
- `totalPurchasedAmount` (Long): Tổng giá trị đã mua, cộng dồn tất cả đơn hàng, đơn vị VND.

### Gợi ý tên segment
female_high_spender_30m
```

---

## Ví dụ 2 — Hạng thành viên + điểm loyalty (Map động)

**Câu hỏi**: "Phân nhóm khách hàng hạng Gold có điểm loyalty trên 1000"

**Solution**:

```markdown
### Summary
Khách hàng có hạng thành viên Gold và điểm loyalty tích lũy trên 1000 điểm.

### Mục tiêu
Nhận diện nhóm thành viên cao cấp có mức độ gắn kết loyalty cao để ưu tiên chăm sóc VIP, tặng quà tri ân hoặc mời tham gia chương trình đặc quyền.

### Điều kiện lọc (logic)
- `attributes.MembershipLevel` (String) = "Gold"
- AND `attributeInts.LoyaltyPoint` (Long) > 1000

### Trường dữ liệu tham chiếu
- `attributes.MembershipLevel` (String): Hạng thành viên (membership tier).
- `attributeInts.LoyaltyPoint` (Long): Số điểm loyalty hiện có của khách hàng.

### Gợi ý tên segment
gold_member_loyalty_1000
```

---

## Ví dụ 3 — Recency mua hàng (dùng field đếm ngày sẵn có)

**Câu hỏi**: "Khách hàng chưa mua hàng trong 90 ngày qua"

**Solution**:

```markdown
### Summary
Khách hàng có số ngày kể từ lần mua gần nhất vượt quá 90 ngày (khách hàng có dấu hiệu rời bỏ).

### Mục tiêu
Xác định nhóm khách hàng lâu ngày chưa quay lại mua sắm để triển khai chiến dịch win-back, khuyến khích quay lại mua hàng.

### Điều kiện lọc (logic)
- `lastPurchasedDays` (Long) > 90

### Trường dữ liệu tham chiếu
- `lastPurchasedDays` (Long): Số ngày kể từ lần mua gần nhất đến thời điểm hiện tại (recency).

### Gợi ý tên segment
lapsed_customers_90_days
```

---

## Ví dụ 4 — Segment/Tag đã gán (Array field)

**Câu hỏi**: "Khách hàng thuộc segment quan tâm sản phẩm Apple"

**Solution**:

```markdown
### Summary
Khách hàng hiện đang thuộc nhóm segment quan tâm sản phẩm Apple.

### Mục tiêu
Xác định nhóm khách hàng có mối quan tâm rõ ràng tới sản phẩm Apple để triển khai chiến dịch quảng bá dòng sản phẩm liên quan, cross-sell phụ kiện.

### Điều kiện lọc (logic)
- `userSegmentKeys` (Array<String>) chứa giá trị liên quan "apple" (dùng contains_any, khớp cả biến thể như "PRODUCT-INTEREST_apple-product-enthusiasts-new")

### Trường dữ liệu tham chiếu
- `userSegmentKeys` (Array<String>): Danh sách key của tất cả các segment mà khách hàng hiện đang thuộc về.

### Gợi ý tên segment
apple_interest_segment
```

---

## Ví dụ 5 — Mô hình RFM (Array<Map>)

**Câu hỏi**: "Khách hàng có điểm monetary theo mô hình RFM ở mức cao (trên 4)"

**Solution**:

```markdown
### Summary
Khách hàng có điểm Monetary (giá trị chi tiêu) theo mô hình eRFM đạt mức cao.

### Mục tiêu
Nhận diện nhóm khách hàng có giá trị chi tiêu cao theo mô hình phân tích RFM chuẩn hóa, phục vụ phân khúc VIP đa chiều hơn so với chỉ dựa vào tổng chi tiêu thô.

### Điều kiện lọc (logic)
- Trong `enrichmentModels` (Array<Map>), tồn tại phần tử có `model` (String) = "erfm" AND `erfm_monetary_score` (Long) > 4

### Trường dữ liệu tham chiếu
- `enrichmentModels` (Array<Map>): Danh sách kết quả các mô hình làm giàu dữ liệu, mỗi phần tử gồm `model` và các chỉ số eRFM.
- `enrichmentModels.model` (String): Tên mô hình, ví dụ "erfm".
- `enrichmentModels.erfm_monetary_score` (Long): Điểm số Monetary đã chuẩn hóa (thang điểm).

### Gợi ý tên segment
high_monetary_score_rfm
```

---

## Ví dụ 6 — Thời gian tương đối (epoch millis) + tương tác Zalo OA

**Câu hỏi**: "Khách hàng đã tương tác Zalo OA trong 30 ngày gần đây"

**Solution** (giả định THỜI GIAN HIỆN TẠI = 06/08/2026):

```markdown
### Summary
Khách hàng có thời điểm tương tác Zalo Official Account gần nhất nằm trong 30 ngày qua.

### Mục tiêu
Xác định nhóm khách hàng đang hoạt động tích cực trên kênh Zalo OA để triển khai chiến dịch chăm sóc/remarketing qua kênh này.

### Điều kiện lọc (logic)
- `attributeLongs.lastInteractOA` (Long, epoch millis) trong khoảng từ 07/07/2026 đến 06/08/2026 (GMT+7)

### Trường dữ liệu tham chiếu
- `attributeLongs.lastInteractOA` (Long, epoch millis): Thời điểm tương tác Zalo OA gần nhất.

### Gợi ý tên segment
zalo_oa_engaged_last_30_days
```

---

## Ví dụ 7 — Câu hỏi quá chung chung → matched: false

**Câu hỏi**: "Phân nhóm khách hàng thân thiết"

**Solution**:

```markdown
### Không thể tạo segment

Câu hỏi "khách hàng thân thiết" chưa cung cấp tiêu chí cụ thể để xác định mức độ "thân thiết" — không có ngưỡng số, hạng thành viên, hay khoảng thời gian rõ ràng để xây dựng điều kiện lọc.

### Chi tiết vấn đề

- Không có ngưỡng cụ thể cho tổng chi tiêu, số đơn hàng, hoặc điểm loyalty.
- Không có khoảng thời gian xác định mức độ hoạt động gần đây.
- "Thân thiết" là khái niệm trừu tượng, có thể ánh xạ theo nhiều field khác nhau (totalPurchasedCount, lastPurchasedDays, attributeInts.LoyaltyPoint...) nhưng cần user xác nhận tiêu chí cụ thể.

### Đề xuất cải thiện

Vui lòng bổ sung thông tin cụ thể:
1. Ngưỡng chi tiêu hoặc số đơn hàng cụ thể (VD: totalPurchasedAmount > bao nhiêu, hoặc totalPurchasedCount > bao nhiêu đơn)?
2. Hạng thành viên cụ thể nào (attributes.MembershipLevel = Gold/Silver/Diamond)?
3. Mức độ hoạt động gần đây cụ thể (lastPurchasedDays trong khoảng bao nhiêu ngày)?
```