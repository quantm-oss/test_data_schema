## QUY TẮC CHỌN MÀU CHO SEGMENT

'information.color' là mã màu Hex đại diện cho segment, hiển thị trên UI.

## Format bắt buộc

- Format: `"#RRGGBB"` (7 ký tự, bắt đầu bằng `#`, theo sau là 6 ký tự hex uppercase hoặc lowercase)
- Ví dụ hợp lệ: `"#ADD8E6"`, `"#90EE90"`, `"#FFB6C1"`
- Ví dụ SAI: `"ADD8E6"` (thiếu `#`), `"#ADD"` (chỉ 3 ký tự), `"lightblue"` (tên màu)

## TUYỆT ĐỐI KHÔNG

- Dùng màu quá đậm/tối (VD `#000000`, `#FF0000`, `#0000FF` full saturation)
- Dùng màu neon/rực rỡ (VD `#00FF00`, `#FF00FF`)
- Dùng tên màu thay vì hex (VD `"lightblue"`)
- Dùng RGBA hoặc HSL format (VD `"rgba(173, 216, 230, 0.5)"`)