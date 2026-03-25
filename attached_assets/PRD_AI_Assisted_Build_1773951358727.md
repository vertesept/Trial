# Product Requirements Document (PRD)
## Portfolio Management v2 - AI-Assisted Build

Date: 2026-03-19  
Source baseline: `BRD_AI_Assisted_Rebuild.md`

## 1. Product Vision
Deliver a secure, maintainable, and production-ready Portfolio Management platform that preserves current business workflows (dashboard, intake, review, reporting, admin) while enabling rapid feature delivery through AI-assisted implementation.

## 2. Product Objectives
1. Preserve end-to-end portfolio workflow parity with the existing application.
2. Eliminate client-side secret exposure and hardcoded admin access patterns.
3. Consolidate duplicated implementation tracks into one canonical product code path.
4. Establish AI-ready delivery controls so generated code is deterministic, testable, and auditable.

## 3. Users and Jobs-To-Be-Done
### 3.1 Intake Submitter
- Create new project requests with required ownership and business context.
- Receive immediate confirmation and project ID.

### 3.2 Portfolio Reviewer
- Monitor project health, schedule risk, and ownership accountability.
- Drill from aggregate view to detailed project context quickly.

### 3.3 Review Meeting Coordinator
- Prepare review-ready project lists with current updates.
- Export printable/PDF report for governance meetings.

### 3.4 Admin User
- Maintain reference data (portfolios, owners, phases, stages, priorities, work types).
- Manage project-level review-meeting flags.

## 4. Product Scope
### 4.1 In Scope for MVP Rebuild
- Dashboard page with full filters/sort/search and late/at-risk indicators.
- Intake page with required validation and default lifecycle values.
- Project detail page with edit mode and status change logging.
- Reports page with all report tabs and Gantt functionality.
- Review meeting page with print/PDF workflow.
- Admin area with secure auth and reference data CRUD.
- Azure-backed persistence through server-side API.

### 4.2 Out of Scope for MVP Rebuild
- New business domains beyond existing entities/workflows.
- Native mobile app.
- BI warehouse redesign.

## 5. Product Principles for AI-Assisted Delivery
1. Contract-first: API and data schemas are frozen before UI generation.
2. Small packets: AI work is executed in bounded feature packets with explicit acceptance tests.
3. No hidden assumptions: every generated artifact must cite source requirements and target files.
4. Secure-by-default: AI is never allowed to place credentials/secrets into browser code.
5. Parity before enhancement: regressions blocked until parity acceptance passes.

## 6. Functional Requirements
### FR-1 Dashboard
The system must:
- Display project table with parity columns and navigation to detail page.
- Support multi-select filtering by portfolio, phase, condition, priority, business owner, IT owner.
- Support search by ID/name.
- Support sortable columns and visual sort state.
- Highlight late and at-risk projects.

Acceptance:
- Given seeded data, applying any filter combination updates rows within 300 ms for normal dataset sizes.
- Clicking a project row/detail link opens `project-detail` for selected project ID.

### FR-2 Intake
The system must:
- Render required fields and ownership selectors.
- Enforce required validation before submission.
- Apply defaults: `workflowStatus=Submitted`, `phase=Not Started`, `condition=Green`, `priority=Medium`.
- Persist both project and intake detail records.

Acceptance:
- On successful submit, user sees confirmation and project ID.
- Invalid required fields show inline validation errors.

### FR-3 Project Detail and Editing
The system must:
- Render full project profile (summary, people, schedule, budget, narrative fields).
- Provide edit mode with save/cancel.
- Log changed fields with old/new values and timestamp metadata.

Acceptance:
- Editing any tracked field creates a status log entry.
- Cancel leaves persisted data unchanged.

### FR-4 Reports
The system must:
- Provide tabs for Late Projects, Workflow Status, Portfolio Summary, Owner Workload, Gantt.
- Provide CSV export and print function.
- Render charts and pipeline/Gantt visualizations.

Acceptance:
- Report tab changes update both visual and table output.
- Gantt filter changes re-render chart and preserve link-through to project detail.

### FR-5 Review Meeting
The system must:
- Render only projects flagged for review meeting.
- Include key fields (condition, phase, schedule, owner, latest update).
- Support print/PDF output.

Acceptance:
- If no projects are flagged, show empty-state message.

### FR-6 Admin
The system must:
- Require authenticated admin access.
- Support CRUD operations for projects and all reference data sets.
- Support project review-meeting toggle.
- Support delete operations with confirmation.

Acceptance:
- Unauthorized users cannot access admin functions.
- CRUD changes are persisted and reflected in UI immediately.

## 7. Non-Functional Requirements
### NFR-1 Security
- No Azure account keys in browser bundle.
- No hardcoded admin key.
- Role-based authorization for admin endpoints.
- Audit trail for admin mutations.

### NFR-2 Performance
- Dashboard/report interactions remain responsive under expected PMO dataset load.
- Initial page render target under 2s on standard enterprise network for cached assets.

### NFR-3 Reliability
- API/network failures display actionable user feedback.
- Writes are idempotent where applicable.

### NFR-4 Accessibility
- Keyboard-navigable filters, dialogs, and forms.
- Appropriate labels and ARIA semantics.

### NFR-5 Observability
- Structured server logs for CRUD and auth failures.
- Client telemetry for fatal UI errors.

## 8. Product Architecture Decision Requirements
The implementation must converge on one canonical architecture:
- Preferred: API-mediated access (`/api/*`) with Azure access only in backend.
- Legacy direct-browser Azure calls are deprecated and removed after cutover.

Canonical source decision required:
- Consolidate duplicated root and `artifacts/portfolio-management` front-end tracks into one codebase.

## 9. Data Model Requirements
Required entity sets:
- `projects`, `portfolios`, `owners`, `workflow-stages`, `status-values`, `review-sessions`, `settings`, `admin-users`, `portfolio-details`, `phases`, `priorities`, `work-types`

Project fields to preserve:
- Identity, ownership, lifecycle, schedule, financial, narrative, review controls, audit metadata as documented in BRD.

## 10. API Requirements
### 10.1 Functional API Surface
Required operations per entity:
- list
- get-by-id
- create
- update
- delete

System endpoints:
- health endpoint
- Azure connectivity/status endpoint (or equivalent diagnostics)

### 10.2 Contract Governance
- API contracts must be versioned and test-covered.
- Front-end types must be generated or validated against API schemas.

## 11. UX Requirements
1. Preserve current IA and page-level route discoverability.
2. Preserve key interaction patterns:
- Multi-select filter dropdowns
- Sortable headers
- Toasts and modal confirmation
- Inline validation
3. Preserve responsive behavior for mobile and print behavior for reports/meetings.
4. Replace placeholder theming tokens with production design tokens if React/Tailwind path is activated.

## 12. AI Build Execution Model
### 12.1 AI Work Packet Template
Every AI work packet must include:
- Objective
- In-scope files
- Out-of-scope files
- Requirements list (FR/NFR IDs)
- API/data contracts
- UX states (loading, empty, error, success)
- Acceptance tests
- Security checks

### 12.2 Required AI Guardrails
AI-generated code must:
- Never add secret material to client-side code.
- Avoid introducing new frameworks without explicit approval.
- Maintain existing naming conventions and entity keys.
- Include tests for all changed behavior.

### 12.3 Human Review Gates
Code cannot merge unless:
- PR includes mapping from changes to FR/NFR IDs.
- Tests pass in CI.
- Security checklist passes.
- Product parity checklist passes for affected flow.

## 13. Release Plan
### Release 1: Foundation and Security
- Freeze contracts and schemas.
- Move data access to API-only pattern.
- Implement secure admin auth baseline.

Exit criteria:
- No credential exposure in browser.
- Health and CRUD contract tests green.

### Release 2: Core Workflow Parity
- Rebuild dashboard, intake, project detail, reports, review meeting, admin.
- Complete parity tests vs existing behavior.

Exit criteria:
- All FR-1 through FR-6 accepted.

### Release 3: Consolidation and Hardening
- Remove deprecated duplicate paths.
- Improve telemetry and operational runbooks.
- Performance and accessibility remediation.

Exit criteria:
- Single canonical frontend implementation.
- NFR compliance sign-off.

## 14. Backlog (Epics and Stories)
### Epic A: Secure Data Access
- Story A1: Remove browser SharedKeyLite path and implement API-mediated repository calls.
- Story A2: Add server-side secret management and connection validation.
- Story A3: Add regression tests for all entity CRUD operations.

### Epic B: Admin Security
- Story B1: Replace static access-key auth with role-based authentication.
- Story B2: Add authorization middleware for admin routes.
- Story B3: Add audit log persistence for admin actions.

### Epic C: Workflow UI Parity
- Story C1: Dashboard parity.
- Story C2: Intake parity.
- Story C3: Project detail parity with status log.
- Story C4: Reports parity including Gantt.
- Story C5: Review meeting parity.

### Epic D: Product Consistency
- Story D1: Select and enforce canonical frontend architecture.
- Story D2: Consolidate duplicate root/artifacts frontend paths.
- Story D3: Standardize design tokens and component usage.

## 15. Metrics and KPIs
Adoption and usage:
- Weekly active admin users
- Weekly project updates submitted

Operational quality:
- API error rate
- UI fatal error rate
- Mean time to recover for failed deployments

Product effectiveness:
- Time to create intake submission
- Time to prepare review-meeting output
- Percent of projects with current status updates

## 16. Risks and Mitigations
Risk: Dual code paths cause inconsistent behavior.
- Mitigation: Architecture decision in Release 1; deprecate non-canonical path.

Risk: AI-generated regressions.
- Mitigation: Packetized scope, required test assertions, parity checklist.

Risk: Security regression from legacy patterns.
- Mitigation: Mandatory security gate and static scanning in CI.

## 17. Dependencies
- Azure Table Storage availability and permissions
- Environment variable provisioning in deployment pipeline
- CI support for pnpm workspace typecheck/build/test

## 18. Definition of Done
A feature is done only when:
1. All mapped FR/NFR requirements are implemented.
2. Unit/integration/e2e tests for changed behavior pass.
3. Security checklist passes with no critical findings.
4. Documentation is updated (PRD/BRD traceability and runbook notes).
5. Product owner acceptance is recorded.

## 19. AI Prompt Starters (Ready to Use)
### Prompt: Implement Feature Packet
"Implement [feature name] for Portfolio Management v2 using `PRD_AI_Assisted_Build.md` and `BRD_AI_Assisted_Rebuild.md`. Restrict edits to [file list]. Satisfy [FR IDs] and [NFR IDs]. Do not introduce new dependencies. Add tests covering [acceptance criteria]. Return a change summary mapped to requirement IDs."

### Prompt: Parity Verification
"Compare rebuilt [page/flow] against legacy behavior in [source files]. Produce a parity checklist and identify mismatches in filters, validation, persistence, and exports."

### Prompt: Security Review
"Review changed files for secret exposure, auth bypass, insecure client logic, and missing authorization checks. Report findings by severity with file references and remediation steps."

## 20. Source References
- `BRD_AI_Assisted_Rebuild.md`
- `index.html`, `pages/*.html`, `js/*.js`, `css/*.css`
- `artifacts/portfolio-management/src/**/*`
- `artifacts/api-server/src/**/*`
- `vite.config.js`, `artifacts/portfolio-management/vite.config.ts`
- `pnpm-workspace.yaml`, `package.json`, `tsconfig*.json`
