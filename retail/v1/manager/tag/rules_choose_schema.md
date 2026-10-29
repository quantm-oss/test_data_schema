- Chỉ chọn bảng từ danh sách cho sẵn. Không bịa tên bảng mới.
- Ưu tiên bảng có trường định danh khách hàng (phone, customerId, refId).
- Nếu nhiều bảng có thể dùng, chọn bảng KHỚP NHẤT với hành vi/dữ liệu trong câu hỏi:
  - Câu hỏi về mua hàng, giao dịch, doanh thu → 'purchase_history'
  - Câu hỏi về sản phẩm/thương hiệu/danh mục đã mua, mã giảm giá → 'purchase_history_item'
  - Câu hỏi về danh tính, khách hàng -> 'customer_profile', 'customer_information'
  - Câu hỏi về lead automotive, classification, model interest → 'lead_info'
  - Câu hỏi về tương tác SMS (marketing campaign qua SMS) → 'sms_interaction'
  - Câu hỏi về tương tác Zalo Mini App, các sản phẩm Pango (PangoForm, PangoWheel, PangoConnect...) → 'zma_log_cf'
- Trong reason, nói cụ thể vì sao bảng được chọn (hoặc tại sao không bảng nào phù hợp).

Note:
- Nếu câu hỏi có liên quan tới lần mua hàng gần mua hàng đầu tiên hoặc gần nhất bao gồm:
  + Thời gian mua
  + Sản phẩm đã mua
  + Mua tại đâu
  + Mã giao dịch
  + Tổng giá trị đơn hàng
  + Thời điểm ghi nhận khách hàng
=> Xem bảng customer_profile, customer_information để tìm và lấy field phù hợp.

## Chọn nhiều bảng và JOIN

- Mặc định chọn 1 bảng. Chỉ chọn thêm bảng khi điều kiện lọc thực sự nằm ở bảng khác (VD: "khách nữ đã click SMS" → `customer_profile` + `sms_interaction`).
- Chỉ số mua hàng tổng hợp (tổng chi tiêu, số đơn, hạng thành viên, sản phẩm đã mua) đã có sẵn trong `customer_information` → KHÔNG chọn thêm `purchase_history` cho các chỉ số này.
- Các bảng nối với nhau bằng số điện thoại: `customer_profile.mainPhone`, `customer_information.phone`, `purchase_history.customerPhone`, `purchase_history_item.customerPhone`, `zma_log_cf.phone`, `sms_interaction.phoneNumber`, `lead_info.phone`.
- KHÔNG nối bằng `orgId`, `id`, `refId`, `customerId`.
