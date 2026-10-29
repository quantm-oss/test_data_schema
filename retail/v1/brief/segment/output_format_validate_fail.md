Khi có bất kỳ phần tử nào trong `unresolved_items` (theo `rules_pass_fail_criteria.md`), trả về:

```json
{
  "matched": false,
  "resolved_summary": "Tóm tắt phần đã hiểu đúng, dù vẫn fail toàn bộ",
  "unresolved_items": [
    {"user_phrase": "cụm từ gốc trong câu hỏi", "reason": "lý do cụ thể không map được"}
  ],
  "briefSolution": null
}
```

- `briefSolution` LUÔN là `null` khi `matched: false` — không trả bản nháp/partial.
- `unresolved_items` PHẢI liệt kê ĐẦY ĐỦ mọi cụm từ không map được, không chỉ 1 cụm đầu tiên.