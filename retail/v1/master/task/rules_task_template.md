'configuration.taskTemplate' là block LLM gen chứa metadata Task template. BE sẽ transform block này thành 'information' + 'taskTemplate' trong response cuối.

## Cấu trúc LLM output

```json
"configuration": {
  "taskTemplate": {
    "type": "<chọn từ task_types khả dụng>",
    "description": "<mô tả tổng quát task type>",
    "category": "<chọn từ task_categories khả dụng>",
    "subType": "Information|Remind|Opportunity|Warning|Risk",
    "templateConfigData": {
      "title": "<tiêu đề ngắn gọn>",
      "content": "<nội dung chi tiết>"
    },
    "priority": "low|normal|high"
  }
}
```

## Nguyên tắc BẮT BUỘC

### 1. 'type' — Từ task_types khả dụng

- Chọn giá trị từ danh sách task_types được cung cấp trong HỆ THỐNG
- Match intent của solution (VD: solution về "hot lead interaction" → chọn 'Interaction_Hot_Customer' hoặc 'new_customer_interaction')
- Nếu không có type phù hợp → chọn type gần nhất về semantic

### 2. 'category' — Từ task_categories khả dụng

- Chọn giá trị từ danh sách task_categories được cung cấp
- Hiện tại thường chỉ có 'care' — nếu chỉ 1 giá trị → dùng nguyên
- Nếu có nhiều categories → chọn dựa vào context solution

### 3. 'subType' — Enum mapping từ description signal

Mapping intent trong description → subType. Phân biệt RÕ RÀNG 3 mức độ Hot/Warm/Cold — KHÔNG được map cả 3 vào cùng 1 giá trị:

| Signal trong description | subType | Ý nghĩa |
|---|---|---|
| "hot", "chốt đơn", "cơ hội bán", "chốt đơn ngay" | 'Opportunity' | Cơ hội chốt đơn ngay, ưu tiên cao nhất |
| "warm", "tiềm năng", "quan tâm", "AI đề xuất ưu tiên" | 'Information' | Tiềm năng, cần theo dõi/nurture tiếp — chưa phải cơ hội tức thời |
| "cold", "nurturing", "gửi tin tương tác", "khuyến mãi" | 'Remind' | Cần nurturing dài hạn, nhắc nhở/kích hoạt lại định kỳ — không phải cơ hội ngay |
| "cảnh báo", "risk cao", "churn", "sắp mất khách" | 'Warning' | Cảnh báo mức độ cao, cần hành động khẩn |
| "risk", "rủi ro chưa cao", "cần theo dõi" | 'Risk' | Rủi ro mức trung bình, theo dõi định kỳ |
| "nhắc lịch", "reminder", "bảo dưỡng", "định kỳ" | 'Remind' | Nhắc lịch định kỳ, không mang tính rủi ro |
| "thông báo", "info", "chăm sóc general", "theo dõi" | 'Information' | Thông báo/chăm sóc chung, không có urgency cụ thể |

**Default**: 'Information' nếu không match signal nào rõ ràng.

**Lưu ý phân biệt "cold" vs "reminder định kỳ"**: cả 2 cùng map `Remind`, nhưng ngữ cảnh khác nhau — "cold lead nurturing" là nhắc nhở để kích hoạt lại quan tâm đã nguội, còn "reminder bảo dưỡng" là nhắc lịch theo chu kỳ cam kết. Cả 2 đều phù hợp `Remind` vì bản chất đều là hành động "nhắc/kích hoạt định kỳ", không phải cơ hội tức thời (`Opportunity`) hay cảnh báo khẩn (`Warning`).

### 4. 'priority' — CỐ ĐỊNH 'normal'

**BUSINESS DECISION**: Priority mặc định = '"normal"' cho tất cả Task rules.

Không dùng '"low"' hay '"high"' unless description có explicit signal cực rõ (VD: "task khẩn cấp", "priority cao nhất") — nhưng default vẫn 'normal'.

### 5. 'description' — Mô tả tổng quát task type

- Ngắn gọn (< 200 chars)
- Mô tả **task type ở mức khái niệm**, không phụ thuộc ngữ cảnh cụ thể
- VD: "Task chăm sóc khách hàng có mức độ quan tâm cao nhằm tăng khả năng chuyển đổi"

### 6. 'templateConfigData.title' — Tiêu đề action-oriented

- Ngắn gọn (< 100 chars)
- Action-oriented — bắt đầu bằng verb hoặc noun phrase
- VD: "Khách đang quan tâm cao cần chăm sóc ngay", "Gọi ngay cho Hot Lead", "Ưu tiên chốt đơn ngay"

### 7. 'templateConfigData.content' — Nội dung hướng dẫn

- Chi tiết hơn title (< 1000 chars)
- Hướng dẫn cụ thể cho Sales/CSKH thực thi task
- VD: "Đội ngũ Sale chăm sóc khách hàng ngay khi nhận được thông tin qua điện thoại hoặc nhắn tin trực tiếp"

## Ví dụ hoàn chỉnh

### Case 1: Description "Chăm sóc khách Retail_HOT"

```json
"configuration": {
  "taskTemplate": {
    "type": "Interaction_Hot_Customer",
    "description": "Task chăm sóc khách hàng có mức độ quan tâm cao nhằm tăng khả năng chuyển đổi và chốt đơn nhanh chóng.",
    "category": "care",
    "subType": "Opportunity",
    "templateConfigData": {
      "title": "Khách đang quan tâm cao cần chăm sóc ngay",
      "content": "Đội ngũ Sales/CSKH liên hệ ngay với khách hàng qua điện thoại hoặc nhắn tin trực tiếp để tăng khả năng chốt đơn."
    },
    "priority": "normal"
  }
}
```

### Case 2: Description "Nurturing khách Warm quan tâm sản phẩm nhưng chưa sẵn sàng mua"

```json
"configuration": {
  "taskTemplate": {
    "type": "new_customer_interaction",
    "description": "Task theo dõi và nurture khách hàng có mức độ quan tâm trung bình để chuyển hóa dần thành cơ hội chốt đơn.",
    "category": "care",
    "subType": "Information",
    "templateConfigData": {
      "title": "Khách tiềm năng cần tiếp tục theo dõi",
      "content": "Gửi thông tin sản phẩm/ưu đãi phù hợp qua Zalo hoặc email để duy trì sự quan tâm, chưa cần liên hệ trực tiếp gấp."
    },
    "priority": "normal"
  }
}
```

### Case 3: Description "Reminder bảo dưỡng định kỳ"

```json
"configuration": {
  "taskTemplate": {
    "type": "new_customer_care",
    "description": "Task nhắc lịch bảo dưỡng định kỳ cho khách hàng.",
    "category": "care",
    "subType": "Remind",
    "templateConfigData": {
      "title": "Nhắc lịch bảo dưỡng định kỳ",
      "content": "Liên hệ khách hàng để nhắc lịch bảo dưỡng định kỳ theo chu kỳ đã cam kết."
    },
    "priority": "normal"
  }
}
```

### Case 4: Description "Cảnh báo khách churn 90 ngày không hoạt động"

```json
"configuration": {
  "taskTemplate": {
    "type": "new_customer_care",
    "description": "Task cảnh báo khách có nguy cơ churn để đội chăm sóc win-back.",
    "category": "care",
    "subType": "Warning",
    "templateConfigData": {
      "title": "Cảnh báo khách có nguy cơ rời bỏ",
      "content": "Khách hàng không có hoạt động trong 90 ngày qua, cần liên hệ để tìm hiểu lý do và win-back."
    },
    "priority": "normal"
  }
}
```