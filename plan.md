# Implementation Plan: AI-Assisted Flexible Work & Life Balance Request System

**Branch**: `001-json-shortname-flexible` | **Date**: 2026-03-19 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-json-shortname-flexible/spec.md`

## Summary

Provide an AI-assisted system for employees to request flexible work arrangements
(WFH, shift changes, caregiving support). The AI converts informal reasoning into
editable professional justifications, suggests policy tags, and helps route the
request to approver(s). The system must enforce privacy, fairness, transparency,
and auditability while enabling HR analytics for outcome monitoring.

## Technical Context

**Language/Version**: NEEDS CLARIFICATION — recommended: Python 3.11 (backend), TypeScript/React (frontend)
**Primary Dependencies**: NEEDS CLARIFICATION — recommended: FastAPI or similar, React, PostgreSQL, Redis, and an LLM/AI provider SDK (subject to data handling rules)
**Storage**: NEEDS CLARIFICATION — recommended: PostgreSQL for transactional data; object storage (S3-compatible) for attachments; encrypted storage for PII
**Testing**: NEEDS CLARIFICATION — recommended: pytest (backend), Jest/Playwright (frontend) for contract and integration tests
**Target Platform**: Cloud-hosted web service (Linux containers) — confirm cloud provider and deployment model
**Project Type**: Web application (backend API + frontend UI)
**Performance Goals**: NEEDS CLARIFICATION — pilot scale: low concurrency (<=100 concurrent users); production goals TBD
**Constraints**: Privacy/legal constraints (GDPR/CCPA), no raw PII to third-party LLMs without approved data processing agreements; RBAC + audit logs required
**Scale/Scope**: NEEDS CLARIFICATION — initial pilot cohort size and expected concurrent users

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Gates derived from constitution:
- Privacy review (Data Protection Officer) — REQUIRED
- Fairness impact assessment (HR + Engineering) — REQUIRED
- Security review (InfoSec) — REQUIRED
- Legal review when policy interpretations are automated — REQUIRED

Status: GATES NOT SATISFIED — the plan includes required reviews and must record signoffs before Phase 1 work that affects approval logic or AI processing.

## Project Structure

### Documentation (this feature)

```text
specs/001-json-shortname-flexible/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
└── tasks.md
```

### Source Code (proposed)

```text
backend/
├── src/
│   ├── api/
│   ├── services/
│   └── models/
frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
tests/
├── contract/
├── integration/
└── unit/
```

**Structure Decision**: Web application split into `backend/` (API + business logic) and `frontend/` (React UI). This supports independent deployments and aligns with common HR integrations.

## Complexity Tracking

No constitution violations justified at this time; all mandatory reviews/gates are documented and must be scheduled prior to Phase 1 design work.

## Technical Plan

### Architecture Overview
- Frontend: React + TypeScript SPA that communicates with backend REST/GraphQL APIs. Hosted via CDN and static hosting.
- Backend: Python 3.11 service (FastAPI recommended) exposing REST endpoints for request intake, approvals, audit logs, and analytics.
- Data Storage: PostgreSQL for transactional data; S3-compatible object storage for attachments; Redis for short-term caching and job coordination.
- AI Adapter: Dedicated `ai_adapter` service that enforces consent and pseudonymization rules before calling external LLMs. All prompts and outputs are logged (audit snapshots) while minimizing PII exposure.
- Auth/Identity: Integrate with corporate SSO/HRIS (SCIM) for employee profiles and manager mapping. Support a fallback local auth option for pilot.
- Deployment: Containerized (Docker) services orchestrated by Kubernetes or managed container service; CI/CD pipelines to build, test, and deploy to staging and production.

### Key Interfaces & Contracts
- `/api/requests` POST/GET: intake and list requests; request body includes fields defined in `data-model.md`.
- `/api/approvals` POST: submit an approval decision; includes actor id, decision, and reason.
- `/api/policy-tags` GET: list canonical policy tags and references.
- AI Adapter internal interface: `ai_adapter.generate_justification(request_snapshot, consent=true)` -> `{justification, rationale, references}`.

### Data Protection & AI Handling
- Default policy: do not send raw PII to external LLM providers. Pseudonymize identifiers and redact sensitive fields unless an on-prem/approved vendor with DPA is confirmed.
- Maintain audit snapshots of model inputs and outputs in encrypted storage; only authorized roles may access full snapshots.
- Consent must be captured per request and included in the justification request to the AI Adapter.

### Observability & Auditing
- Emit structured logs for all lifecycle events (`request.created`, `ai.suggested`, `approval.recorded`, `request.withdrawn`).
- Store audit records in append-only store with retention rules. Support export for HR/legal review.
- Implement metrics (Prometheus) for system health, request throughput, decision latency, and fairness signals.

### Implementation Phases (tech-focused)
1. Foundation: repo scaffold, CI, infra IaC templates, DB migrations, auth integration stub, RBAC and audit service (see tasks T001-T011).
2. MVP (US1): request intake API, AI Adapter integration with pseudonymization pipeline, frontend request form, consent capture, audit logging (see tasks T012-T019).
3. Approvals (US2): routing engine, approver dashboard, decision API, notification service, conflict resolution (T020-T024).
4. Analytics (US3): outcome records, analytics pipeline, fairness audit jobs, HR dashboard (T025-T029).
5. Pilot & Compliance: run pilot with limited cohort, execute privacy/fairness/security reviews, iterate on routing and LLM handling.

### Testing & Validation Plan
- Unit tests for models and services; contract tests for API endpoints; integration tests for end-to-end flows.
- Fairness validation: implement synthetic tests and initial baseline audits during pilot; schedule quarterly audits.
- Privacy validation: DPO signoff on data flows and retention; automated scanners to detect PII in attachments before processing.

### Risks & Mitigations
- Risk: LLM vendor exposes PII — Mitigation: pseudonymization and DPA; prefer vendors supporting data residency.
- Risk: biased outcomes — Mitigation: fairness audits, human-in-the-loop approvals for sensitive cases, policy thresholds that trigger reviews.
- Risk: slow decision times — Mitigation: notifications, SLA tracking, escalation rules, and manager reminders.

### Agent Context & Next Steps
- Run `.specify/scripts/powershell/update-agent-context.ps1 -AgentType copilot` to update agent-specific context with new technology choices.
- Resolve outstanding NEEDS CLARIFICATION items in `research.md` (Q1: approver routing, auth integration, LLM vendor) before Phase 1 design work.


