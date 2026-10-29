# WHITELIST OPERATORS (22 items)

Master AI chỉ được dùng operator trong danh sách sau. Bất kỳ operator nào ngoài whitelist sẽ bị BE validation reject.

['equals', 'not_equals', 'in', 'not_in', 'like', 'not_like',
'is_null', 'is_not_null', 'eq', 'neq', 'gt', 'gte', 'lt', 'lte',
'gte_relative_day', 'lte_relative_day', 'between_relative_day', not_between_relative_day
'contains_any', 'contains_all', 'exist', 'not_exist', 'relative_time']

## Lưu ý về fuzzy string matching

Cho MỌI fuzzy string matching, dùng `like` (case-insensitive) hoặc `not_like`.