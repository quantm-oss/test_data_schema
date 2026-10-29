# Manager Tier Prompts

Contains prompts for Manager AI tier — transforms user question into DSL Rule solution description (Markdown format).

Manager tier có 3 LLM steps: **rewrite** → **choose_schema** → **gen_solution**.

## Sub-directories by DSL Type

- **tag/**: Tag Rule flow
- **task/**: Task Rule flow
- **segment/**: Segment Assignment flow

## Pipeline

Manager AI input:
1. User question + conversation history
2. Tenant schema từ `dslSchemaUrl` (realtime fetch)
3. Business rules từ prompts files

Sau đó tạo solution markdown:
- Summary
- Mục tiêu
- Điều kiện lọc (logic)
- Trường dữ liệu tham chiếu
- Gợi ý tên tag

Solution này sau đó được human review, có thể edit, rồi gửi qua Master để convert thành DSL Rule JSON.