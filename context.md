# Portfolio Management v3 — Context

## Overview

A portfolio and project management web application for tracking IT/business projects across portfolios. Supports intake, status tracking, reporting, review meetings, and admin management.

## Tech Stack

- **Language**: Vanilla JavaScript (ES2020+, no framework)
- **Markup**: HTML5
- **Styling**: Plain CSS (custom properties, flexbox, grid)
- **Storage**: Browser `localStorage` via a custom `Storage` module
- **Build/Dev Server**: Vite (serves static files only — no bundling of JS)
- **Hosting**: Served as static assets from the `public/` directory

## Architecture

This is a **multi-page application (MPA)** — each section is a standalone HTML file, not a single-page app.

```
artifacts/portfolio-management/
├── index.html                  # Dashboard (entry point)
├── public/
│   ├── css/
│   │   ├── styles.css          # Layout, typography, variables
│   │   └── components.css      # Reusable component styles (badges, cards, modals, etc.)
│   ├── js/
│   │   ├── storage.js          # localStorage abstraction + seed data
│   │   ├── app.js              # Auth utilities (login, session, isLate/isAtRisk helpers)
│   │   ├── ui.js               # UI helpers (badges, modals, toasts, sortable tables)
│   │   ├── projects.js         # Project CRUD operations
│   │   ├── reports.js          # Report generation logic
│   │   ├── charts.js           # Chart rendering utilities
│   │   └── admin.js            # Admin panel logic
│   └── pages/
│       ├── intake.html         # New project intake form
│       ├── project-detail.html # Individual project view/edit
│       ├── reports.html        # Portfolio reports
│       ├── review-meeting.html # Review meeting view
│       └── admin.html          # Admin portal (users, portfolios, owners)
```

## Pages

| Page | Path | Description |
|------|------|-------------|
| Dashboard | `/` | Project table with filters and search |
| Intake | `/pages/intake.html` | Submit a new project |
| Reports | `/pages/reports.html` | Portfolio-level reporting |
| Review Meeting | `/pages/review-meeting.html` | Meeting agenda and status review |
| Admin | `/pages/admin.html` | Manage users, portfolios, and owners |
| Project Detail | `/pages/project-detail.html?id=...` | View/edit a single project |

## Data Model

All data is persisted in `localStorage`. Keys are defined in `KEYS` (in `storage.js`):

- `PORTFOLIOS` — list of portfolio objects `{ id, name }`
- `OWNERS` — list of owner objects `{ id, fullName, email, role }`
- `PROJECTS` — list of project objects (see below)
- `ADMIN_USERS` — admin user accounts with hashed passwords

### Project Object

```js
{
  id: string,               // UUID
  name: string,
  portfolioId: string,
  condition: 'Red' | 'Amber' | 'Green',
  phase: string,            // e.g. 'In Progress', 'Not Started', 'Completed', 'On Hold'
  priority: 'Critical' | 'High' | 'Medium' | 'Low',
  businessOwnerId: string,
  itOwnerId: string,
  finishDate: string,       // ISO date
  workflowStatus: string,   // e.g. 'Under Review', 'Approved', 'Submitted', 'Pending Approval'
  updatedAt: string,        // ISO date
  notes: string,
}
```

## Key Conventions

- Scripts are loaded as plain `<script src="...">` tags (not ES modules)
- Global helpers (`Storage`, `Projects`, `UI`) are attached to the window scope
- `isLate(project)` and `isAtRisk(project)` helpers live in `app.js`
- Session-based admin auth uses `sessionStorage` with a SHA-256 password hash via `crypto.subtle`
- Seed data is injected on first load in `storage.js` if localStorage is empty
