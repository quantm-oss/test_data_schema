## Cấu trúc message đề xuất (KHÔNG bao gồm câu nhắc xác nhận — hệ thống tự thêm sau)
```
Mục tiêu: <objective>

Điều kiện lọc:

<diễn giải condition 1 bằng ngôn ngữ business>
<diễn giải condition 2>
<diễn giải condition n>
<diễn giải condition ...>

Loại trừ (nếu có):

<diễn giải exclusion 1 bằng ngôn ngữ business>
<diễn giải exclusion 2>
<diễn giải exclusion n>
<diễn giải exclusion ...>


Khoảng thời gian: <diễn giải timeRange theo đúng nguyên tắc ở phần TIME HANDLING, nếu có>
```


## Ví dụ 1 — Không có timeRange

**briefSolution input**:
```json
{
  "objective": "Xác định nhóm khách hàng nữ có sức mua cao",
  "conditions": [
    {"id": "c1", "field": "gender", "operator": "=", "value": "female"},
    {"id": "c2", "field": "totalPurchasedAmount", "operator": ">", "value": 50000000}
  ],
  "exclusions": [],
  "timeRange": null,
  "assumptions": []
}
```

**message output**:
```
Mục tiêu: Xác định nhóm khách hàng nữ có sức mua cao

Điều kiện lọc:
- Khách hàng có giới tính nữ
- Tổng giá trị đã mua (cộng dồn tất cả đơn hàng) trên 50 triệu đồng
```


## Ví dụ 2 — Có timeRange (áp dụng đúng TIME HANDLING)

**briefSolution input**:
```json
{
  "objective": "Xác định khách hàng nữ có sức mua cao và đang hoạt động gần đây",
  "conditions": [
    {"id": "c1", "field": "gender", "operator": "=", "value": "female"},
    {"id": "c2", "field": "lastActivityAt", "operator": ">", "value": null}
  ],
  "exclusions": [],
  "timeRange": {"field": "lastActivityAt", "last": "30d", "appliesTo": ["c2"]},
  "assumptions": []
}
```

**message output**:
```
Mục tiêu: Xác định khách hàng nữ có sức mua cao và đang hoạt động gần đây

Điều kiện lọc:
- Khách hàng có giới tính nữ
- Khoảng thời gian: Hoạt động trong 30 ngày gần đây
```