# Master Task Prompts

Per-tenant prompts cho Master Task tier — pipeline 7 steps ASYNC:
① load_prompt 
→ ② fetch_master_data (parallel 2 endpoints) 
→ ③ load_schema
→ ④ build_user_message 
→ ⑤ call_llm 
→ ⑥ transform 
→ ⑦ validate_operators

## Files

### Core (1)
- `system_prompt.md` — Template với 9 placeholders resolve từ manifest

### Fragments (9) — resolved bởi `_get_master_task_manifest()`
- `rules_dsl_structure.md` — Rules DSL JSON structure Task (targetModels, joins, conditions, groupBys, havings, orderBys)
- `rules_like.md` — Fuzzy string dùng like (mirror Master Tag)
- `rules_time_handling.md` — Time relative_day + between_relative_day
- `rules_rootmodel.md` — rootModel.selectFields đầy đủ
- `rules_complexity_scoring.md` — Scoring 0-100 (low/medium/high)
- `rules_task_template.md` — Rules configuration.taskTemplate mapping (subType + priority)
- `operators_whitelist.md` — 22 operators
- `examples_dsl.md` — 5 examples DSL Task từ production
- `output_schema.md` — Output JSON schema strict

## Runtime vars trong HỆ THỐNG (dynamic per request)
- `task_types` — CSV từ p_connect_task_type master-data
- `task_categories` — CSV từ p_connect_task_category master-data
- `datetime_vn` — Format "HH:MM thứ Weekday, DD/MM/YYYY (+7)"
- `schema_text` — Combined schema text từ prompts/{orgId}/schema/*.md

## LLM output structure (Structure C)
LLM chỉ gen: `name`, `description`, `dslRule` (core), `configuration.taskTemplate`, `ruleComplexity`, `ruleComplexityScore`.
BE transform sẽ inject: `information`, `taskTemplate` wrapper, DSL metadata constants, deduplication template.