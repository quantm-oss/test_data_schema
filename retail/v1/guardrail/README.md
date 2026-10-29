# Guardrail Tier Prompts (Tier 0)

Contains prompts for Guardrail layer — kiểm soát scope input câu hỏi của user trước khi đưa vào các tier sau (Suggestion, Manager).

Tenant: `bdf66a48c3ebfaec10d2feb3492099ea`

## Files

- **user_input.md**: System prompt cho evaluate scope câu hỏi user

## Purpose

Guardrail là **defense-in-depth** layer:
1. Reject câu hỏi ngoài phạm vi hệ thống (out_of_domain)
2. Reject prompt injection / jailbreak attempts
3. Reject câu hỏi mơ hồ (ambiguous)
4. Pass qua câu hỏi in-scope về DSL rules (tag/segment/task)

## Fail-safe

Nếu LLM guardrail fail → raise `GuardrailError` → HTTP 503. Không có fallback degraded (khác Suggestion).

## Note

Prompts tenant-specific để tune scope / phạm vi hỗ trợ riêng cho từng tenant business context.
Ví dụ: tenant retail có thể có scope khác tenant automotive.