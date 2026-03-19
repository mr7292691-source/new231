# data-model.md

**Feature**: AI-Assisted Flexible Work & Life Balance Request System
**Created**: 2026-03-19

## Entities

### Request
- `id` (UUID): primary key
- `employee_id` (string/UUID): foreign key to EmployeeProfile
- `request_type` (enum): [WFH, SHIFT_CHANGE, CAREGIVING, OTHER]
- `requested_schedule` (json): structured schedule details (days, start/end times, timezone)
- `duration` (object): { `type`: [temporary, permanent], `start_date`, `end_date` (nullable) }
- `attachments_meta` (json): list of {id, filename, content_type, storage_path, pii_flags}
- `ai_justification` (text): final editable justification produced by AI
- `ai_explainability` (json): {inputs_used, policy_references, redactions_applied}
- `policy_tags` (array of PolicyTag.tag_id)
- `status` (enum): [DRAFT, SUBMITTED, PENDING, APPROVED, DENIED, WITHDRAWN]
- `consent` (object): { `consented`: boolean, `consent_ts`: timestamp, `consent_snapshot_id` }
- `created_at` (timestamp)
- `updated_at` (timestamp)

Validation notes:
- `requested_schedule` must include timezone; `duration.end_date` required for temporary requests.
- `ai_justification` editable by employee before submission.

Retention:
- Default retention: 2 years (configurable by PolicyTag.retention_period). Audit snapshots retained separately per audit retention rules.

### EmployeeProfile
- `employee_id` (UUID)
- `name` (string)
- `email` (string)
- `manager_id` (UUID)
- `department` (string)
- `role` (string)
- `consent_flags` (json): user-level preferences
- `created_at`, `updated_at`

### Approver
- `approver_id` (UUID)
- `employee_id` (UUID) — link to EmployeeProfile if approver is internal
- `role` (enum): [MANAGER, HR, PEOPLEOPS]
- `scopes` (json): list of departments/units they can approve for

### PolicyTag
- `tag_id` (string)
- `name` (string)
- `authoritative_policy_reference` (string/URL)
- `retention_period_days` (int)
- `requires_hr_review` (boolean)

### OutcomeRecord
- `outcome_id` (UUID)
- `request_id` (UUID)
- `decision` (enum): [APPROVED, DENIED, MORE_INFO_REQUESTED, WITHDRAWN]
- `decision_reason` (text)
- `decision_actor_id` (UUID)
- `decision_timestamp` (timestamp)

### AuditLog
- `log_id` (UUID)
- `request_id` (UUID|null)
- `action` (string): e.g., `request.created`, `ai.suggested`, `approval.recorded`
- `actor_id` (UUID|null)
- `timestamp` (timestamp)
- `metadata` (json): includes pointers to stored snapshots and redaction markers

### AttachmentSnapshot (for stored attachments)
- `attachment_id` (UUID)
- `request_id` (UUID)
- `storage_path` (string)
- `content_hash` (string)
- `pii_flags` (json)
- `scanned_at` (timestamp)

## Relationships
- `Request.employee_id` -> `EmployeeProfile.employee_id`
- `OutcomeRecord.request_id` -> `Request.id`
- `AuditLog.request_id` -> `Request.id` (nullable for system events)
- `Request.policy_tags` -> `PolicyTag.tag_id`

## Indexing & Query Patterns
- Index `Request.status` + `created_at` for dashboards and time-to-decision queries
- Index `OutcomeRecord.decision_timestamp` and `request_id` for analytics
- Index `EmployeeProfile.manager_id` for routing queries

## Privacy & Retention Rules
- Audit snapshots containing model I/O are stored encrypted and access-controlled.
- Analytics datasets must be pseudonymized: remove `employee_id` and replace with hashed cohort id before aggregation.
- Default retention configuration stored in `PolicyTag.retention_period_days`; if unset, use 2 years for requests and 5 years for audit logs by default.

## Validation & Business Rules
- If `policy_tags` contains a tag with `requires_hr_review=true`, route to HR in addition to manager.
- If `duration.type` == `permanent`, require an explicit HR approval step (configurable).
- If `consent.consented == false`, block AI-assisted justification generation and record reason in `ai_explainability`.

*** End of data-model.md
