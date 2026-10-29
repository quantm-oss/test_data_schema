Solution CHỈ được reference đến các fields có trong schema Task đã cung cấp ở trên (bảng 'customer_unified' và 'customer_tagging_staging').

## Nguyên tắc bắt buộc

1. **Tồn tại**: Mỗi field trong "Điều kiện lọc" và "Trường dữ liệu tham chiếu" PHẢI tồn tại trong 2 bảng của schema.
2. **Đúng tên**: Field reference theo đúng tên trong schema (case-sensitive, VD: 'rootId' không phải 'RootId' hay 'root_id', 'customerId' không phải 'CustomerId').
3. **Đúng type**: Toàn bộ time field trong cả 2 bảng ('customer_unified.createdAt', 'customer_unified.updatedAt', 'customer_tagging_staging.createdAt', 'customer_tagging_staging.expiredAt', 'customer_tagging_staging.lastTouchedAt') là 'DateTime64(3, UTC)' — không được so sánh như Integer hoặc String. Lưu ý timezone UTC (khác GMT+7 thường dùng ở bảng Tag khác).
4. **Đúng bảng**: Field từ bảng nào phải reference đúng bảng đó theo format '[collection_name].[field_name]':
   - 'customer_unified.rootId', 'customer_unified.name', 'customer_unified.orgId'
   - 'customer_tagging_staging.customerId', 'customer_tagging_staging.tagId', 'customer_tagging_staging.status'
5. **Map access**: Fields dạng 'Map<String,...>' trong 'customer_tagging_staging' (detailString, detailLong, detailDouble, detailBoolean, detailDate, detailArrayString, detailArrayLong, detailArrayDouble) → reference qua key cụ thể theo format 'fieldName.KeyName'. Key PHẢI là key đã biết tồn tại thực tế trong data — KHÔNG bịa key mới. Nếu chưa xác định được key cụ thể tương ứng với yêu cầu của user, KHÔNG dùng field Map này — trả 'matched: false' và hỏi lại key chính xác cần dùng.
6. **`detailBoolean`**: giá trị lưu dạng UInt8 (0/1) — filter bằng `= 1`/`= 0`, KHÔNG dùng `true`/`false`.

## Khi user yêu cầu field không có trong schema

- KHÔNG được tự bịa field mới.
- KHÔNG được suy đoán field có tên tương tự.
- Trả 'matched: false' với giải thích rõ field nào bị thiếu và bảng nào không có.
- Gợi ý các fields có sẵn có thể dùng thay thế nếu có.

## Ví dụ áp dụng

**Sai**: User hỏi "khách có tổng chi tiêu > 50 triệu" → LLM output field 'totalPurchasedAmount' (không có trong 2 bảng schema Task).

**Đúng**: 
- 2 bảng schema Task không có field chi tiêu → trả 'matched: false', giải thích schema Task chỉ track tagging log không có purchase data.
- Nếu user cần task dựa vào tag đã tồn tại → dùng 'customer_tagging_staging.tagId' để filter.

**Sai**: User hỏi "khách có tag với ghi chú kênh liên hệ là Zalo" → LLM tự bịa key `detailString.channelName = "Zalo"` khi chưa xác nhận key này thực sự tồn tại trong data.

**Đúng**: 
- Chưa xác nhận key cụ thể nào trong `detailString` map lưu thông tin kênh liên hệ → trả 'matched: false', giải thích cần xác nhận key chính xác trước khi build điều kiện, hoặc gợi ý dùng `customer_tagging_staging.channelType` (field cố định, không phải Map) nếu phù hợp với ý định user.