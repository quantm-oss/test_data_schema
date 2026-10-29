Solution phải CỤ THỂ, tránh mô tả quá chung chung không thực thi được.

## Nguyên tắc bắt buộc

### 1. Điều kiện lọc phải có ngưỡng cụ thể

- Sai: "khách hàng chi tiêu cao" (không có ngưỡng)
- Đúng: "khách hàng có 'customer_tagging_staging.confidence' >= 0.8" (ngưỡng cụ thể trên field cố định đã xác nhận tồn tại)

**Lưu ý về field Map động** (`detailString`, `detailLong`, `detailDouble`, `detailBoolean`, `detailDate`, `detailArrayString`, `detailArrayLong`, `detailArrayDouble`): Toàn bộ 8 Map này hiện RỖNG 100% trong dữ liệu thực tế — TUYỆT ĐỐI KHÔNG dùng để build điều kiện lọc. Nếu user cần ngưỡng trên 1 thuộc tính không có field cố định tương ứng, PHẢI trả `matched: false` và giải thích hệ thống hiện chưa có dữ liệu custom detail.

### 2. Task tag dùng để validate, không hiển thị trong output

- Task tag CHỌN giá trị cụ thể từ danh sách available để verify solution
- KHÔNG ghi vào solution output

### 3. Tên task gợi ý phải rõ nghĩa

Format: `assign_[intent]_[target]_task`

- Sai: `task1`, `customer_task`, `care_task`
- Đúng: `assign_highvalue_customer_care_task`, `assign_new_customer_welcome_task`, `assign_hot_lead_sales_call_task`

### 4. Summary + Mục tiêu phải mention

- **Target**: Nhóm khách hàng nào được gán task (VIP, Hot lead, new customer, có tag X)
- **Action**: Hành động gì (gọi điện, gửi SMS/Zalo, gán cho Sales/CSKH, follow-up)
- **Reason**: Lý do business (tăng conversion, duy trì mối quan hệ, tránh churn)

### 5. Không dùng phrase mơ hồ

- Sai: "chăm sóc phù hợp", "xử lý thích hợp", "theo dõi định kỳ", "hỗ trợ khi cần"
- Đúng: "gọi điện chăm sóc trong 24h", "gửi Zalo OA nhắc lịch bảo dưỡng", "gán task cho Sales team liên hệ ngay"

### 6. Yêu cầu "đếm số lần / số lượng bản ghi" LUÔN khả thi qua COUNT(*)

- User hỏi "gắn tag X từ N lần trở lên", "có nhiều hơn N bản ghi tag Y", "xuất hiện tag Z ít nhất N lần" → LUÔN matched=true, dùng aggregation `COUNT(*)` với `GROUP BY customerId` + `HAVING COUNT(*) >= N`.
- KHÔNG được từ chối yêu cầu này với lý do "không có field lưu sẵn số lần" — `COUNT(*)` đếm trực tiếp số bản ghi thỏa điều kiện filter (VD tagId, status, khoảng thời gian), không cần bất kỳ field cố định nào lưu sẵn con số đó.
- Chỉ trả `matched: false` nếu bản thân điều kiện filter đi kèm (VD tagId cụ thể) không xác định được — không phải vì thiếu "trường đếm".
- Ghi rõ trong "Điều kiện lọc (logic)": "Số lần khách hàng được gắn tag X trong khoảng thời gian Y >= N (COUNT bản ghi customer_tagging_staging thỏa điều kiện)".

## Khi câu hỏi user quá chung chung → matched=false

Nếu câu hỏi không đủ thông tin để tạo solution cụ thể:
- Không có ngưỡng, tag reference, thời gian → user chỉ nói "khách VIP"
- Không rõ action → user chỉ nói "làm gì đó cho khách này"
- Không rõ target → user chỉ nói "gán task"
- Cần dùng field Map động (`detailString`/`detailLong`/... ) nhưng không xác định được key cụ thể tồn tại thực tế

→ Trả `matched: false` với message giải thích thiếu thông tin gì và gợi ý user cung cấp thêm.