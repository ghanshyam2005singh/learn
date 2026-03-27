# Codebase Guide — Alpha One Labs Learn

This document is for contributors who want to understand how the codebase is organised before writing code. Read this alongside [`CONTRIBUTING.md`](./CONTRIBUTING.md) before opening a PR.

---

## Table of Contents

- [Project Philosophy](#project-philosophy)
- [High-Level Architecture](#high-level-architecture)
- [Folder Structure](#folder-structure)
- [Frontend (`public/`)](#frontend-public)
- [Backend (`src/`)](#backend-src)
- [Database (`schema.sql`)](#database-schemasql)
- [Configuration & Secrets](#configuration--secrets)
- [Security Conventions](#security-conventions)
- [Out of Scope](#out-of-scope)
- [Adding a New Feature — Checklist](#adding-a-new-feature--checklist)

---

## Project Philosophy

`alphaonelabs/learn` is the **core education platform** — intentionally kept focused. Features that don't belong here live in their own repositories (see [Out of Scope](#out-of-scope)). Before adding anything, ask: *does this belong in the core learning experience?*

The stack is deliberately simple:

- No frontend framework — plain HTML, Tailwind CSS, vanilla JavaScript
- No ORM — raw parameterised SQL against Cloudflare D1
- No session state — stateless HMAC-signed JWTs

This keeps the codebase approachable for new contributors and fast to run at the edge.

---

## High-Level Architecture

```
Browser (public/*.html)
        │
        │  fetch() to /api/...
        ▼
Cloudflare Worker (src/worker.py)
        │
        │  delegates to service modules
        ├──► auth_service.py      — registration, login, token verification
        ├──► learning_service.py  — courses, sessions, enrollments
        ├──► user_service.py      — profiles, roles, dashboard data
        ├──► messaging_service.py — messages, threads
        ├──► study_groups_service.py — groups, membership
        └──► utils.py             — shared helpers
              │
              ▼
        Cloudflare D1 (SQLite — schema.sql)
```

The frontend is fully static — no server-side rendering. Pages call the Worker API over `fetch()` and render responses in the DOM.

---

## Folder Structure

```
learn/
├── public/                  # Frontend — static HTML pages and assets
│   ├── images/              # Static image assets
│   ├── index.html           # Landing / home page
│   ├── login.html           # Login and registration
│   ├── dashboard.html       # User dashboard
│   ├── course.html          # Course detail page
│   ├── teach.html           # Instructor / teaching interface
│   └── admin.html           # Admin interface
│
├── scripts/                 # Shell scripts for setup and deployment
│   └── upload-wrangler-secrets.sh
│
├── src/                     # Backend — Cloudflare Workers (Python)
│   └── worker.py            # Main entry point: on_fetch dispatcher + routing
│
├── .gitignore
├── .dev.vars                # Local secrets — NEVER commit this file
├── LICENSE
├── README.md
├── CONTRIBUTING.md
├── CODEBASE.md              # This file
├── schema.sql               # Cloudflare D1 database schema
└── wrangler.toml            # Cloudflare Workers configuration
```

---

## Frontend (`public/`)

All frontend code is **static** — plain HTML with Tailwind CSS utility classes and minimal vanilla JavaScript. There is no build step and no frontend framework.

Pages communicate with the backend exclusively through `fetch()` calls to `/api/...` routes on the Worker. The DOM is updated client-side based on API responses.

**Conventions:**
- One `.html` file per page — do not nest subdirectories inside `public/` without discussion
- No custom CSS files — Tailwind utility classes only
- No JavaScript frameworks — vanilla JS only
- Static assets (images, icons) go in `public/images/`
- Pages must be responsive — test at mobile widths before submitting a PR

---

## Backend (`src/`)

All business logic runs as a **Cloudflare Worker written in Python**. The entry point is `src/worker.py`, which receives every incoming request, parses the route, and delegates to the appropriate service module.

### `worker.py` — routing only

`worker.py` contains the `on_fetch` handler. Its only job is to match routes and call the right service. **Business logic does not belong here.**

### Service modules

As the project grows, `worker.py` is being split into focused service files. New features go into the relevant `_service.py` file:

| File | Responsibility |
|---|---|
| `auth_service.py` | Registration, login, token verification, password hashing |
| `learning_service.py` | Courses, activities, sessions, enrollments, tags |
| `user_service.py` | User profiles, dashboard data, role management |
| `messaging_service.py` | Messages, threads, communication between users |
| `study_groups_service.py` | Study group creation, membership, group sessions |
| `utils.py` | Shared helpers: encryption, HMAC blind indexes, ID generation, error responses |

If you are adding a new backend feature, it goes into the relevant `_service.py` — not directly into `worker.py`.

### Response format

All API responses follow a consistent JSON structure:

```python
# Success
return Response(json.dumps({"success": True, "data": result}), status=200)

# Error
return Response(json.dumps({"error": "Unauthorized"}), status=401)
```

---

## Database (`schema.sql`)

`schema.sql` is the single source of truth for the Cloudflare D1 database schema. All table definitions live here. If your feature requires a new table or column, the change must be included in your PR.

### Current tables

| Table | Purpose |
|---|---|
| `users` | User accounts, encrypted PII, HMAC indexes |
| `activities` | Learning activities / course content |
| `sessions` | Scheduled learning sessions |
| `enrollments` | User ↔ activity enrollment records |
| `tags` | Content tags |
| `activity_tags` | Many-to-many: activities ↔ tags |
| `session_attendance` | User attendance records for sessions |

### Database rules

- Always use **parameterized queries** — never string interpolation in SQL
- Sensitive fields (names, emails, etc.) must be stored **encrypted** — see Security Conventions
- Never store plaintext passwords

---

## Configuration & Secrets

### `wrangler.toml`

Cloudflare Workers configuration: Worker name, D1 database binding, assets directory, environment settings. Do not modify this file in a PR unless your change specifically requires a config update — and explain why in the PR description.

### `.dev.vars`

Local development secrets. This file is in `.gitignore` and **must never be committed**.

```
ENCRYPTION_KEY=your-dev-encryption-key
JWT_SECRET=your-dev-jwt-secret
```

For production, secrets are managed via Wrangler:

```bash
wrangler secret put ENCRYPTION_KEY
wrangler secret put JWT_SECRET
```

---

## Out of Scope

The following features do not belong in this repo and are maintained in separate repositories:

| Feature | Where it goes |
|---|---|
| AI research assistant | `scholarai` repo |
| Bot / automation tooling | `botlab` repo |
| AI-powered learning features | `learnpilot` repo |

If you are building something in this list, contribute to the relevant repo instead.

---

## Adding a New Feature — Checklist

Before opening a PR with a new feature:

- [ ] An issue exists and has been approved by a maintainer
- [ ] The feature belongs in this repo (see Out of Scope above)
- [ ] A new HTML page (if needed) is in `public/`
- [ ] Backend logic is in the correct `_service.py` file — not in `worker.py`
- [ ] Any new database tables or columns are added to `schema.sql`
- [ ] No PII is stored as plaintext
- [ ] No passwords are stored as plaintext or with fast hashes
- [ ] You have tested the change locally with `wrangler dev`

---

For the full contribution workflow, branch naming, commit conventions, and PR guidelines, see [`CONTRIBUTING.md`](./CONTRIBUTING.md).