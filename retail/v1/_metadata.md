# Tenant Metadata

## Basic Info

- **orgId**: bdf66a48c3ebfaec10d2feb3492099ea
- **Business type**: Automotive dealership (car sales)

## Data Overview

- **Total tables**: 15 (10 queryable + 5 system/destination)
- **Queryable tables**: contact_index, customer_information, customer_profile, customer_unified, lead_info, purchase_history, purchase_history_item, zalo_user_profile, **zma_log_cf**, **sms_interaction**
- **System tables** (NOT for Master queries): contact_index_merge_log, customer_tagging_staging, customer_task_staging, dsl_result, staff_directory

## Channel Types Supported

Tenant hỗ trợ **4 channel types** (không có ZaloUAId):

- Phone
- Email
- ZaloUId
- FbPsid