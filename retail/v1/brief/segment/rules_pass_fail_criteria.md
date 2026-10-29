## Tiêu chí PASS

`validate_the_conditions` CHỈ trả `matched: true` khi ĐỒNG THỜI thỏa:

1. `dataSources` resolve được ít nhất 1 giá trị hợp lệ theo `choose_data_source.md`.
2. TOÀN BỘ ý định cụ thể trong câu hỏi (objective, mỗi điều kiện lọc, mỗi điều kiện loại trừ, mốc thời gian nếu có) đều map được vào field có thật trong schema `unified_profile_model`.

## Tiêu chí FAIL — NGHIÊM NGẶT, KHÔNG giữ lại phần map được

Chỉ cần 1 ý định bất kỳ trong câu hỏi KHÔNG map được vào field thật trong schema → `matched: false` cho TOÀN BỘ câu hỏi, dù các phần khác đã map được rõ ràng. KHÔNG trả `briefSolution` (để `null`) khi `matched: false`.

## Bắt buộc khi trả matched: false

Phải điền đầy đủ:
- `resolved_summary`: mô tả ngắn gọn phần ý định ĐÃ hiểu đúng, dù cuối cùng vẫn fail toàn bộ — để bước sau biết đã đi đúng hướng phần nào.
- `unresolved_items`: mảng liệt kê MỌI cụm từ không map được, mỗi phần tử có `user_phrase` (cụm từ gốc trong câu hỏi) và `reason` (lý do cụ thể).

## Khi câu hỏi hoàn toàn chung chung (không có bất kỳ tiêu chí cụ thể nào)

Xử lý như 1 trường hợp `unresolved_items` với `user_phrase` = toàn bộ câu hỏi, `reason` = "Câu hỏi chưa cung cấp tiêu chí cụ thể (ngưỡng số, hạng thành viên, khoảng thời gian, segment/tag...) để xây dựng điều kiện lọc".