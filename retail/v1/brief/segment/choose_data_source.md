## Danh sách nguồn dữ liệu khả dụng cho DSL Segment

| Key | Mô tả | Khi nào chọn | Trạng thái |
|---|---|---|---|
| unified_profile_model | Hồ sơ khách hàng 360 độ hợp nhất (CRM, E-commerce, Social, Mobile App) | MẶC ĐỊNH — dùng cho MỌI câu hỏi Segment hiện tại | Active, Khả dụng |

## Nguyên tắc chọn

- Hiện tại hệ thống Segment CHỈ hỗ trợ 1 nguồn duy nhất — LUÔN trả `dataSources: ["unified_profile_model"]` cho mọi câu hỏi thuộc phạm vi Segment.
- KHÔNG bao giờ trả `dataSources: []` khi câu hỏi thuộc phạm vi Segment — nếu không xác định được nguồn dữ liệu, đây là lỗi hệ thống, đưa vào `unresolved_items` với `user_phrase` là toàn bộ câu hỏi và `reason` là "Không xác định được nguồn dữ liệu phù hợp", không phải do thiếu nguồn khả dụng.