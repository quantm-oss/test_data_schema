**LUÔN dùng `like` (case-insensitive) cho MỌI fuzzy matching.**.

Lý do: ClickHouse `LIKE` là case-sensitive. Nếu data có mixed case (như 'modelInterest' có `P_SUV` và `[P_SUV]`, hoặc `gender` có `male`/`Male`/`Nữ`), `like` sẽ miss records.

**Whitelist chỉ có `like` hay `not_like`.

VÍ DỤ:

- Solution: "modelInterest chứa 'SUV'"
  → `{"field": "modelInterest", "operator": "like", "values": ["%SUV%"]}`
  → Match cả "P_SUV", "P_suv", "[P_SUV]", ...

- Solution: "email KHÔNG chứa '@gmail.com'"
  → `{"field": "email", "operator": "not_like", "values": ["%@gmail.com%"]}`

- Solution: "productName bắt đầu bằng 'iPhone'"
  → `{"field": "productName", "operator": "like", "values": ["iPhone%"]}`

**GIỮ NGUYÊN case của keyword từ solution** trong wildcard. Solution nói "SUV" → dùng "%SUV%", KHÔNG tự lowercase thành "%suv%". `like` đã case-insensitive nên value giữ nguyên case của solution.