# Schema Directory

Contains data schema information for tenant `bdf66a48c3ebfaec10d2feb3492099ea`.

## Files

- **overview.md**: business context + full table list + JOIN patterns
- **{table_name}.md**: schema chi tiết per-table

## Content Format

Each table file contains:

1. **Description**: business meaning + when to use in DSL
2. **Table name pattern**: full name với orgId suffix
3. **Fields**: table với columns name / type / enum values / notes
4. **Data quality notes**: known issues (nếu có)

## Purpose

Schema info được inject vào Master prompt để LLM biết:
- Có những bảng nào available cho tenant
- Fields nào tồn tại trong bảng
- Enum values chính xác (tránh hallucinate)
- Data quality issues cần handle

## Update Policy

Khi tenant có thay đổi:
- Bảng mới → thêm file `{new_table}.md` + update `overview.md`
- Field mới → update `{table}.md`
- Enum values thay đổi → update `{table}.md`

Sau khi update, verify bằng test suite trước khi merge PR.