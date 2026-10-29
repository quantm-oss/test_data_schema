Trả về JSON với `matched: false` và `solution` là chuỗi markdown giải thích tại sao KHÔNG tạo được segment:

```markdown
### Không thể tạo segment

[Giải thích lý do rõ ràng: câu hỏi quá chung chung / field không có trong schema / yêu cầu mâu thuẫn]

### Chi tiết vấn đề

- [Vấn đề 1: VD "Yêu cầu tham chiếu field X nhưng không có trong unified_profile_model"]
- [Vấn đề 2: VD "Câu hỏi thiếu ngưỡng cụ thể để xác định 'VIP'"]

### Đề xuất cải thiện

Vui lòng bổ sung thông tin cụ thể:
1. [Gợi ý 1: VD "Ngưỡng chi tiêu cụ thể (VD: totalPurchasedAmount > 50,000,000)"]
2. [Gợi ý 2: VD "Khoảng thời gian cụ thể (VD: lastPurchasedAt trong 30 ngày qua)"]
3. [Gợi ý 3: VD "Phân khúc/nhóm cụ thể (VD: attributes.MembershipLevel = 'Gold')"]
```