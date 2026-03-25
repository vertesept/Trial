# Business Requirements Document (BRD)
## Portfolio Management v2 - AI-Assisted Rebuild

Date: 2026-03-19  
Prepared from repository source analysis

## 1. Purpose
This BRD defines business, product, UX, technical, and delivery requirements to rebuild the Portfolio Management v2 application with AI assistance while preserving existing behavior.

This document is based on the current repository implementation, with special focus on:
- Front-end design and UI behavior
- Front-end build/runtime configuration
- Full component and module inventory needed for a safe AI-assisted rebuild

## 2. Business Context
The application supports project portfolio governance across intake, tracking, review meetings, reporting, and admin configuration.

Business outcomes:
- Standardize intake and tracking of project demand
- Improve visibility into project status, schedule, and risk
- Support review cadence and executive reporting
- Enable admin users to maintain taxonomy/reference data without code changes

## 3. Goals and Success Criteria
Primary goals:
- Rebuild the front end and supporting API layer with no functional regressions
- Preserve all key user workflows and data fields
- Improve maintainability and security posture
- Provide deterministic AI implementation prompts/specs from this BRD

Success criteria:
- 100% parity for existing workflows and page routes
- No data loss for existing Azure Table entities
- Same or better performance for dashboard/report rendering
- Security improvements for admin auth and secret handling
- Automated tests for critical flows (intake, edit, reporting, admin CRUD)

## 4. In-Scope and Out-of-Scope
In scope:
- Multi-page portfolio management UI
- Data model and CRUD behavior for all tracked entities
- Azure Table integration pattern (direct or API mediated)
- Build/deploy configuration for web and API artifacts
- Migration path for current static JS app and React scaffold

Out of scope (unless approved):
- Net-new business capabilities not represented in current repository
- Mobile native apps
- Large data-warehouse/reporting redesign beyond existing reports

## 5. Stakeholders and Users
Stakeholder groups:
- PMO leadership
- Portfolio managers
- Business owners
- IT owners
- Executive review participants
- Platform/devops engineering

User personas:
- Intake Submitter: creates project submissions
- Portfolio Reviewer: analyzes dashboard and project details
- Meeting Coordinator: prepares review meeting exports
- Admin User: maintains projects and configuration reference data

## 6. Current-State Architecture Summary
There are two front-end tracks in the repository:

1) Active/legacy multi-page app (HTML + vanilla JS)
- Location: root `index.html`, `pages/*`, `js/*`, `css/*`
- Runtime: browser-side modules loaded via script tags
- Data access: direct Azure Table calls from browser (`js/api.js`)

2) Scaffold/transition app (React + shadcn UI + Tailwind)
- Location: `artifacts/portfolio-management/src/*`
- Current status: placeholder app shell and UI component library, not feature-complete

API/server track:
- Express API exists at `artifacts/api-server/src/*`
- Supports CRUD endpoints over Azure Tables
- Potential alternative to direct-browser Azure access

## 7. Front-End Design Requirements
### 7.1 Information Architecture and Navigation
Primary navigation in sidebar:
- Dashboard (`index.html`)
- Intake (`pages/intake.html`)
- Review Meeting (`pages/review-meeting.html`)
- Reports (`pages/reports.html`)
- Admin (`pages/admin.html`)

Requirement:
- Preserve this IA and route discoverability in rebuilt UX
- Maintain consistent cross-page navigation and active-state cues

### 7.2 Visual Language
Observed design tokens in CSS:
- Primary: `#2563eb`
- Success: `#16a34a`
- Warning: `#d97706`
- Danger: `#dc2626`
- Background/card/border typography token set in `css/styles.css`

Requirement:
- Define a formal design token system (color, spacing, radius, typography, elevation)
- Preserve semantic color mapping (status, risk, alerts)
- Preserve responsive behavior and print styles

### 7.3 Layout System
Current layout patterns:
- App shell with fixed sidebar + main content
- Card surfaces for content blocks
- Data tables with hover/sort/filter controls
- Form grids for intake/edit/admin

Requirement:
- Preserve desktop and mobile readability
- Keep table overflow handling on small screens
- Keep print-specific hiding rules for non-print controls

### 7.4 Interaction Patterns
Required interactions to preserve:
- Multi-select dropdown filters with search and clear
- Sortable table headers with direction indicators
- Modal confirmation dialogs
- Toast notifications
- Inline and form-level validation errors
- Gantt bar click-through to project detail

## 8. Front-End Configuration Requirements
### 8.1 Build and Serve
Monorepo/package tooling:
- `pnpm` workspace with catalog dependencies (`pnpm-workspace.yaml`)
- Root scripts enforce pnpm and run recursive typecheck/build

Web build config highlights:
- Vite multi-page input entries (`index.html` + `pages/*.html`)
- `PORT` env required at build/dev runtime
- `BASE_PATH` env supported
- Host set to `0.0.0.0`, `allowedHosts: true`

Requirement:
- Rebuild must preserve multi-page routing support or provide approved SPA route migration plan
- Environment requirements (`PORT`, `BASE_PATH`) must be explicit in deployment docs and CI

### 8.2 Security-Sensitive Config
Observed high-risk behavior:
- Root `vite.config.js` injects `AZURE_STORAGE_CONNECTION_STRING` into every HTML page via `window.__AZURE_CONN_STR__`
- Root `js/api.js` performs browser-side SharedKeyLite signing with AccountKey

Requirement:
- Do not expose account keys to browser in rebuild
- Move privileged Azure access to server/API layer using managed identity or secure envs
- Keep browser payloads credential-free

### 8.3 TypeScript and Package Governance
Observed config:
- Strict-ish TS settings in `tsconfig.base.json`
- Workspace references in root `tsconfig.json`
- Security control: `minimumReleaseAge: 1440` in `pnpm-workspace.yaml`

Requirement:
- Preserve workspace governance and supply-chain controls
- Enforce typed contracts between frontend and API

## 9. Functional Requirements by Page
### 9.1 Dashboard
Capabilities:
- Render project table with columns (ID, Name, Portfolio, Condition, Phase, Priority, BO, IT, Finish)
- Multi-dimensional filtering by portfolio/phase/condition/priority/owners
- Free-text search by name/ID
- Sorting by table headers
- Late/at-risk visual cues
- Link to project detail

### 9.2 Intake
Capabilities:
- Capture project metadata and ownership
- Required field validation
- Default lifecycle values on submit (`Submitted`, `Not Started`, `Green`, `Medium`)
- Create project entity and portfolio-details record

### 9.3 Project Detail
Capabilities:
- Read full project record and related labels
- View and edit summary, overview, people, schedule, budget, PMO notes
- Maintain status change log with before/after values
- Persist changes and refresh cache

### 9.4 Reports
Capabilities:
- Tabs: Late Projects, Workflow Status, Portfolio Summary, Owner Workload, Gantt
- Chart.js usage for chart views
- Workflow pipeline visual
- Gantt timeline with filters and today marker
- CSV export and print support

### 9.5 Review Meeting
Capabilities:
- List projects flagged for review meeting
- Print/PDF export behavior

### 9.6 Admin
Capabilities:
- Access-key gate and lock/unlock flow
- Tabs for projects and reference data maintenance
- CRUD for portfolios, owners, phases, stages, priorities, work types
- Project-level review-meeting toggle
- Deletion workflows with confirmation

## 10. Data and Domain Requirements
### 10.1 Core Entity Sets
Entity keys used in storage/API:
- `projects`
- `portfolios`
- `owners`
- `workflow-stages`
- `status-values`
- `review-sessions`
- `settings`
- `admin-users`
- `portfolio-details`
- `phases`
- `priorities`
- `work-types`

### 10.2 Project Domain Fields (minimum)
Required project attributes include:
- Identity: `id`, `name`, `description`
- Ownership: `portfolioId`, `businessOwnerId`, `itOwnerId`, `supportingEbMemberId`
- Lifecycle: `condition`, `phase`, `priority`, `workType`, `workflowStatus`
- Timeline: `startDate`, `finishDate`, `originalFinishDate`, `goLiveDate`, `originalGoLiveDate`
- Financials: `capitalBudget`, `expenseBudget`, `capitalSpend`, `expenseSpend`
- Narrative: `projectUpdate`, `businessJustification`, `submitterName`, `pmoNotes`
- Review controls: `reviewMeeting`, `reviewSessionIds`
- Audit metadata: `statusLog`, `createdAt`, `updatedAt`

### 10.3 Seed Data Requirements
Current app seeds defaults for:
- Portfolios, owners, workflow stages, phases, priorities, work types, status values, settings
- Sample projects
- Admin user (`admin` / hashed `admin123`)

Requirement:
- Rebuild must formalize seeding strategy by environment
- Production seed policy must avoid insecure test credentials

## 11. API and Integration Requirements
### 11.1 API Contract (required behavior)
CRUD surface expected by frontend:
- `list(entity)`
- `get(entity, id)`
- `create(entity, data)`
- `update(entity, id, data)`
- `remove(entity, id)`
- `status()`

### 11.2 Current Integration Modes
Mode A (root frontend):
- Direct Azure Table REST from browser

Mode B (artifact frontend):
- `/api/*` via Express server

Requirement:
- Target architecture must select one mode and deprecate the other
- Recommended: API-mediated access only (Mode B)

### 11.3 Azure Requirements
- Connection health endpoint equivalent to `/api/azure-status`
- Table lifecycle assumptions (table existence checks/creation) must be explicit
- JSON field serialization/deserialization rules must remain deterministic

## 12. Component Inventory for AI-Assisted Rebuild
### 12.1 Must-Rebuild Frontend Modules (legacy app)
- `js/api.js`
- `js/storage.js`
- `js/app.js`
- `js/projects.js`
- `js/reports.js`
- `js/charts.js`
- `js/admin.js`
- `js/ui.js`

### 12.2 Must-Rebuild Pages (legacy app)
- `index.html`
- `pages/intake.html`
- `pages/project-detail.html`
- `pages/reports.html`
- `pages/review-meeting.html`
- `pages/admin.html`

### 12.3 Style Assets
- `css/styles.css`
- `css/components.css`

### 12.4 React Scaffold Components to Account For
Core scaffold files:
- `artifacts/portfolio-management/src/App.tsx`
- `artifacts/portfolio-management/src/main.tsx`
- `artifacts/portfolio-management/src/index.css`
- `artifacts/portfolio-management/src/hooks/*`
- `artifacts/portfolio-management/src/lib/utils.ts`
- `artifacts/portfolio-management/src/pages/not-found.tsx`

shadcn/radix UI library inventory (present and reusable):
- Accordion, Alert, Alert Dialog, Aspect Ratio, Avatar, Badge, Breadcrumb
- Button/Button Group, Calendar, Card, Carousel, Chart, Checkbox
- Collapsible, Command, Context Menu, Dialog, Drawer, Dropdown Menu
- Empty, Field, Form, Hover Card, Input, Input Group, Input OTP
- Item, Kbd, Label, Menubar, Navigation Menu, Pagination, Popover
- Progress, Radio Group, Resizable, Scroll Area, Select, Separator
- Sheet, Sidebar, Skeleton, Slider, Sonner, Spinner, Switch, Table
- Tabs, Textarea, Toast, Toaster, Toggle, Toggle Group, Tooltip

Requirement:
- AI rebuild must explicitly decide whether to retain legacy MPA modules, migrate to React UI components, or run a phased hybrid
- Each module/page above must have parity acceptance tests before retirement

## 13. Non-Functional Requirements
Performance:
- Page load and filter/sort interactions should remain responsive for expected data volume
- Report and Gantt rendering should avoid full-page blocking

Reliability:
- Graceful error handling for API failures and partial data loads
- Deterministic retry/user feedback for failed saves

Security:
- Remove static admin key (`1234`) and client-side credential exposure
- Role-based authorization on admin endpoints
- Audit logging for admin changes

Accessibility:
- Maintain semantic labels, keyboard navigability, and focus management for modals/dropdowns

Observability:
- Structured API logs and client error telemetry for key failures

## 14. AI-Assisted Rebuild Delivery Requirements
### 14.1 AI Work Packet Inputs (required)
For each feature packet, provide AI with:
- Source file references from this BRD
- Required business rules and acceptance criteria
- Data contract schema (request/response)
- UX specs (states, errors, empty/loading)
- Security constraints

### 14.2 Recommended Rebuild Phases
Phase 1: Stabilize contracts
- Freeze entity schemas and API contracts
- Add integration tests around current CRUD behavior

Phase 2: Security hardening
- Remove browser-side Azure shared key usage
- Move to API-only data access

Phase 3: Front-end migration
- Migrate page-by-page into React routes/components (or preserve MPA intentionally)
- Reimplement filters, reports, admin flows with parity checks

Phase 4: Cleanup
- Remove duplicate tracks (`root` vs `artifacts`) after cutover
- Consolidate single source of truth for config and UI assets

## 15. Risks and Constraints
Key risks:
- Dual implementation tracks may drift and cause regression during migration
- Current secret exposure pattern is high risk
- Incomplete React scaffold theming (`red` placeholder tokens in `src/index.css`) can cause invalid UX if activated without completion
- Root `replit.md` references workspace packages not present in this repo tree, indicating documentation drift

Constraints:
- Existing Azure table data and names should be preserved unless migration is approved
- Build system assumes pnpm workspace and env-provided `PORT`

## 16. Acceptance Criteria
Business acceptance:
- All six primary UI sections functionally complete
- Portfolio stakeholders can run intake-to-report workflow end-to-end

Technical acceptance:
- Front-end build succeeds in CI using documented env vars
- API health and CRUD endpoints pass contract tests
- No secrets exposed in delivered browser bundle

Parity acceptance:
- Existing filters, sort behavior, exports, status logs, admin CRUD and review toggles are retained

## 17. Traceability Matrix (Source Files)
Primary configuration sources:
- `package.json`
- `pnpm-workspace.yaml`
- `tsconfig.base.json`
- `tsconfig.json`
- `vite.config.js`
- `artifacts/portfolio-management/vite.config.ts`
- `artifacts/api-server/src/routes/azure.ts`

Primary front-end behavior sources:
- `js/*.js`
- `pages/*.html`
- `css/*.css`

Scaffold/migration sources:
- `artifacts/portfolio-management/src/**/*`
- `artifacts/portfolio-management/components.json`

Deployment metadata sources:
- `artifacts/portfolio-management/.replit-artifact/artifact.toml`
- `artifacts/api-server/.replit-artifact/artifact.toml`

## 18. Open Decisions Required
1. Choose target frontend architecture: keep MPA, move to SPA, or phased hybrid.
2. Confirm data-access architecture: browser direct Azure vs API-only (recommended API-only).
3. Define authentication/authorization model replacing static admin key.
4. Confirm whether `artifacts/` or root app is canonical production source post-rebuild.
5. Approve test strategy and regression gates for AI-generated changes.
