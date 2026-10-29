# Master Tier Prompts

Contains prompts for Master AI tier — transforms Manager solution output into executable DSL Rule JSON.

## Sub-directories by DSL Type

- **tag/**: Tag Rule flow (currently supported)
- **task/**: Task Rule flow (future, Round 2+)
- **segment/**: Segment Assignment flow (future, Round 2+)

## Purpose

Master AI reads:
1. Solution text (from Manager tier)
2. Tenant schema info (from `schema/` folder)
3. Master-data (tag category/subcategory/type via API)

Then produces DSL Rule JSON that:
- Converts cleanly to SQL via Pconnext UI converter
- Runs successfully against ClickHouse DB
- Returns semantically correct results

## Verification Status

Tag flow verified 5/5 realistic solutions (Test A-E) against tenant data.