# research.md

**Feature**: AI-Assisted Flexible Work & Life Balance Request System
**Created**: 2026-03-19

## Purpose

Resolve all NEEDS CLARIFICATION items from `plan.md` Technical Context and `spec.md` so Phase 1 design can proceed.

## Unknowns / Research Tasks

1. Approver routing policy (FR-004) — Question Q1: decide routing rules (Manager only; Manager+HR for caregiving/extended; Manager+HR+PeopleOps for policy-sensitive; or custom mapping). Impact: notification, escalation, and data access rules.
2. Authentication and Identity integration — SSO vs local accounts; HRIS integration method (API/SCIM/CSV). Impact: mapping employee_id, manager_id, and syncing org charts.
3. LLM / AI provider and data handling policy — which provider(s) are acceptable given privacy constraints; should prompts or PII be pseudonymized before external calls; on-prem vs hosted.
4. Data retention for request records, attachments, and audit logs — confirm retention periods per policy tag and any legal requirements.
5. Pilot scale and performance goals — number of pilot users, concurrency expectations, and response time SLOs for AI-assisted operations.
6. Policy tagging sources & format — canonical location of policy documents (URL or internal doc id) and mapping rules.
7. Attachment handling and PII scanning — allowed file types, virus scanning, and automatic redaction rules.
8. Fairness metrics and thresholds — confirm acceptable disparity thresholds and initial baseline data sources.
9. Notification channels — email, Slack/Teams, in-app notifications; preference handling per user.
10. Compliance with third-party processing — legal checklist for sending any user data to LLM vendors; contractual and technical mitigations.

## Recommended Actions

- Schedule a short working session with HR, Legal, Privacy, and Engineering to decide Q1 (Approver routing) and authentication approach.
- Run a quick vendor assessment for LLM providers focusing on data residency and contract terms.
- Draft a minimum viable data retention policy for pilot (e.g., 2 years for requests, 5 years for audit logs) for review by Privacy.

## Outputs expected from research

- Answers to Q1 and other clarifications recorded in `spec.md` as resolved items or updated requirement text.
- Decision on LLM provider(s) and a mitigation plan for any PII exposure.
- A short tech-choices doc with selected stack and justification to be inserted into `plan.md` Technical Context.

*** End of research.md
