# Suggestion Tier Prompts

Contains prompts for Suggestion AI tier — retrieve similar DSL rules từ Chroma DB dựa vào user question.

Tenant: `bdf66a48c3ebfaec10d2feb3492099ea`

## Sub-directories by DSL Type

- **tag/**: Tag Rule flow (currently supported)
- **task/**: Task Rule flow
- **segment/**: Segment Assignment flow

## Pipeline

Suggestion AI:
1. **Rewrite** user question → clean Vietnamese with full context
2. **Embed** rewritten → vector
3. **Chroma search** top-K similar rules (collection scoped by orgId)
4. **Rerank** LLM scores each candidate
5. **Blend cosine + LLM score** với alpha, filter by threshold