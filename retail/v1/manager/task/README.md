# Manager DSL Task Prompts

Per-tenant prompts cho Manager Task tier — pipeline 5 steps:
① rewrite → ② fetch_schema → ③ fetch_task_tags → ④ gen_solution → ⑤ aggregate

## Files

### Core prompts (3)
- `system_prompt_rewrite.md` — Rewrite prompt Task-specific
- `system_prompt_gen_solution.md` — Template với placeholders resolve từ manifest + runtime vars

### Fragments (8) — resolved bởi `_get_manager_task_gen_solution_manifest()`

- `rules_no_hallucination.md` — Rules không bịa fields
- `rules_time_handling.md` — Rules xử lý thời gian
- `rules_reference_fields.md` — Rules chọn Trường tham chiếu
- `rules_solution_specific.md` — Rules tránh solution chung chung
- `rules_task_specific.md` — Task-specific: join 2 collections, chọn task tag
- `output_format_matched_true.md` — Format solution markdown khi matched=true
- `output_format_matched_false.md` — Format message khi matched=false
- `examples.md` — 3+ solution examples reference

## Runtime vars trong template

- `%(schema_json)s` — Task schema JSON từ HTTP fetch
- `%(task_tags)s` — CSV task tags từ master-data API
- `%(current_time)s` — Format "HH:MM thứ Weekday, DD/MM/YYYY (+7)"

## Key differences từ Manager Segment/Tag

- **Bắt buộc dùng cả 2 collections** (join `customer_unified.rootId` ↔ `customer_tagging_staging.customerId`)
- **Task tags** inject qua runtime var — LLM chọn từ danh sách available
- **Task name pattern**: `assign_[intent]_[target]_task`