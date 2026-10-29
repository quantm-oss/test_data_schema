LIỆT KÊ TẤT CẢ field có liên quan đến solution, KHÔNG được lược bớt. Phân loại:

- **IDENTIFY (BẮT BUỘC)**: field định danh khách hàng (phone, customerPhone, phoneNumber).
- **FILTER (BẮT BUỘC)**: liệt kê TỪNG field xuất hiện trong điều kiện lọc.
- **AGGREGATE (nếu solution dùng count/sum/avg/max/min)**: liệt kê field được tính toán.
- **BUSINESS (nếu liên quan)**: field hỗ trợ marketing (campaignName, formName, utmSource, templateName...).

Solution thực tế thường có 3-6 field tham chiếu. Nếu chỉ liệt kê 1-2 field, hãy xem lại điều kiện lọc có thiếu field nào không (vd lọc theo thời gian thì phải có field timestamp; lọc theo chiến dịch thì phải có campaignName/campaignId).

## Field kiểu mảng (Array) — chuẩn thống nhất

Áp dụng cho MỌI field có kiểu bắt đầu bằng `Array` (`Array(...)` hoặc `Array[...]`) trong schema, ở mọi bảng.

- Field kiểu text (String...): ghi điều kiện "chứa" như bình thường (`<field>` chứa `<text>`).
- Field kiểu mảng: KHÔNG so sánh theo phần tử (không ghi "có phần tử". Ghi theo VĂN BẢN: "mảng `<field>` (chuyển thành chuỗi `<field>Text`) chứa đoạn văn bản `<text>`".
- `<text>` là đúng giá trị người dùng nêu, không tự chuẩn hóa, không đổi hoa/thường, không thêm bớt. Với phần tử dạng "khóa: giá trị": câu hỏi `<field>.<khóa> = <giá trị>` hoặc `<field>.<khóa> LIKE '%<đoạn>%'` thì `<text>` = `<khóa>: <giá trị hoặc đoạn>` (khóa và giá trị liền kề, có dấu hai chấm và một dấu cách).
- Nhiều lựa chọn: mỗi lựa chọn là một điều kiện văn bản riêng, nối bằng HOẶC (hoặc VÀ nếu câu hỏi yêu cầu có đủ tất cả).
- Trong "Trường dữ liệu tham chiếu" chỉ liệt kê field mảng GỐC (`<field>`), KHÔNG liệt kê `<field>Text`.

Field mảng hiện có trong schema tenant: `lead_info.tags`, `lead_info.creativeClick`, `lead_info.featureClick`, `sms_interaction.tags`, `customer_profile.firstPurchasedProducts`, `customer_profile.lastPurchasedProducts`, `customer_profile.coreTagIds`, `customer_profile.coreLabelIds`, `customer_information.listOrderItemName`, `customer_information.nearestSingleProduct`.

Ví dụ: "khách đã xem tính năng an toàn trên website" → mảng `lead_info.featureClick` (chuyển thành chuỗi `featureClickText`) chứa đoạn văn bản `<tên tính năng user nêu>`.
