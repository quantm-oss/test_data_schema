# Manager Tag Flow Prompts

Prompts cho Manager Tag flow (3 LLM steps: rewrite → choose_schema → gen_solution).

## Files

### System prompts (main templates)

- **system_prompt_rewrite.md**: Standalone prompt cho step rewrite (không placeholder)
- **system_prompt_choose_schema.md**: Template cho step choose_schema (2 placeholders)
- **system_prompt_gen_solution.md**: Template cho step gen_solution (nhiều placeholders)

### Component files (inject vào templates qua placeholders)

- **rules_choose_schema.md**: Rules cho step choose_schema
- **rules_no_hallucination.md**: Rules chống hallucinate schema/field
- **rules_reference_fields.md**: Nguyên tắc chọn fields đầy đủ ở "Trường dữ liệu tham chiếu"
- **rules_solution_specific.md**: Nguyên tắc vàng — tránh solution chung chung
- **output_format_matched_true.md**: Format output khi tạo được solution
- **output_format_matched_false.md**: Format output khi không tạo được solution
- **examples.md**: 5 examples solutions (1 per table)

## Placeholder Mapping

`system_prompt_choose_schema.md` uses:
- `%(rules_choose_schema)s` → content of `rules_choose_schema.md`
- `%(table_list)s` → dynamic, injected từ code (schema URL fetch result)

`system_prompt_gen_solution.md` uses:
- `%(table_detail)s` → dynamic, injected từ code (schema URL detail của bảng chọn)
- `%(current_time)s` → dynamic, injected từ code (thời gian hiện tại VN)
- `%(rules_no_hallucination)s` → content of `rules_no_hallucination.md`
- `%(rules_reference_fields)s` → content of `rules_reference_fields.md`
- `%(rules_solution_specific)s` → content of `rules_solution_specific.md`
- `%(output_format_matched_true)s` → content of `output_format_matched_true.md`
- `%(output_format_matched_false)s` → content of `output_format_matched_false.md`
- `%(examples)s` → content of `examples.md`

## Update Policy

- Tune rules riêng (VD `rules_no_hallucination.md`) không ảnh hưởng các files khác
- Thêm example mới: append vào `examples.md`
- Change output format: chỉ edit `output_format_*.md`