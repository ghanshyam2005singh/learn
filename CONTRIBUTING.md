# Contributing to Alpha One Labs — Learn

Thank you for your interest in contributing! Please read this document fully before opening an issue or PR. It keeps the codebase clean, consistent, and maintainable for everyone.

> **New here?** Start with [`CODEBASE.md`](./CODEBASE.md) to understand the project structure before diving in.

---

## Table of Contents

- [The Golden Rule](#the-golden-rule)
- [What This Repo Is](#what-this-repo-is)
- [What This Repo Is NOT](#what-this-repo-is-not)
- [Step-by-Step Contribution Workflow](#step-by-step-contribution-workflow)
- [Branch Naming](#branch-naming)
- [Commit Messages](#commit-messages)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Code Style](#code-style)
- [File & Folder Conventions](#file--folder-conventions)
- [What Gets Closed](#what-gets-closed)
- [Other Repos in the Ecosystem](#other-repos-in-the-ecosystem)

---

## The Golden Rule

**Open an issue first. Wait for maintainer approval. Then start coding.**

No exceptions. PRs that arrive without a linked, approved issue will be closed — regardless of code quality. This protects your time and ours.

---

## What This Repo Is

`alphaonelabs/learn` is the **core education platform** built on:

- **Backend:** Python, Cloudflare Workers
- **Database:** Cloudflare D1 (SQLite)
- **Frontend:** Static HTML + Tailwind CSS + minimal vanilla JavaScript

This repo handles: authentication, courses, sessions, user profiles, study groups, and communication.

---

## What This Repo Is NOT

The following features do not belong here and are maintained in separate repositories:

| Feature | Repo |
|---|---|
| AI research assistant | `scholarai` |
| Bot / automation tooling | `botlab` |
| AI-powered learning features | `learnpilot` |

> If you are unsure whether your feature belongs here, ask in an issue or on Discord before writing any code.

---

## Step-by-Step Contribution Workflow

### 1. Find or create an issue

- Browse [open issues](../../issues) before starting — someone may already be working on it.
- If no issue exists for what you want to work on, **open one first** using a clear title and description of the problem or feature.
- For GSoC applicants: focus on clearly scoped issues — do not raise PRs just to increase PR count.

### 2. Wait for maintainer approval

- A maintainer will review your issue and either approve it, ask questions, or close it if it is out of scope.
- **Do not start writing code until a maintainer has confirmed the issue is valid and assigned (or approved) to you.**
- This step saves you from building something that won't be merged.

### 3. Fork and branch

Once approved:

```bash
git clone https://github.com/YOUR_USERNAME/learn.git
cd learn
git checkout -b feat/your-feature-name
```

Follow the [Branch Naming](#branch-naming) conventions below.

### 4. Set up your environment

Install Wrangler:

```bash
npm install -g wrangler
wrangler login
```

Create your local secrets file (never commit this):

```bash
# .dev.vars
ENCRYPTION_KEY=your-dev-encryption-key
JWT_SECRET=your-dev-jwt-secret
```

Set up the database:

```bash
wrangler d1 create education_db
# Add the generated database_id to wrangler.toml
wrangler d1 execute education_db --file=schema.sql
```

### 5. Make your changes

- Keep changes focused — one feature or fix per branch.
- Follow the [Code Style](#code-style) guidelines.
- Test your changes locally with `wrangler dev` before opening a PR.

### 6. Sync with main before opening a PR

Always rebase or merge the latest `main` into your branch before submitting:

```bash
git fetch origin
git rebase origin/main
```

Resolve any conflicts locally. Do not open a PR with merge conflicts.

### 7. Open a Pull Request

Use the [PR Guidelines](#pull-request-guidelines) below. Your PR must:

- Link to the approved issue
- Include a clear description of what changed and how to test it
- Pass the pre-PR checklist

### 8. Respond to review feedback

- Maintainers may request changes — address them promptly.
- Push additional commits to the same branch; do not open a new PR.
- Once approved, a maintainer will merge your PR.

---

## Branch Naming

Use the format: `type/short-description`

| Type | When to use |
|---|---|
| `feat/` | Adding a new feature |
| `fix/` | Fixing a bug |
| `refactor/` | Restructuring code without changing behaviour |
| `docs/` | Documentation only changes |
| `test/` | Adding or updating tests |
| `chore/` | Config, tooling, dependency updates |

**Examples:**

```
feat/study-groups-api
fix/login-token-expiry
refactor/worker-modular-routes
docs/contributing-guide
```

---

## Commit Messages

Follow this format:

```
type: short description (max 72 chars)

Optional longer explanation if needed.
```

**Examples:**

```
feat: add PBKDF2 password hashing with per-user salt
fix: resolve D1 query error on session enrollment
refactor: split worker.py into modular service files
docs: update README setup instructions
```

- Use present tense ("add" not "added")
- Keep the first line under 72 characters
- Reference the issue number: `fix: token expiry bug (#42)`

---

## Pull Request Guidelines

### Pre-PR checklist

Before opening a PR, confirm all of the following:

- [ ] An approved issue exists and is linked in this PR
- [ ] Your branch is up to date with `main` (rebased, no conflicts)
- [ ] Your code follows the style guide below
- [ ] You have tested your changes locally with `wrangler dev`
- [ ] You have not introduced plaintext storage of PII or passwords
- [ ] You have not added unnecessary dependencies
- [ ] No `console.log` / `print` debugging left in your code
- [ ] No commented-out dead code

### PR title format

Same as commit messages: `type: short description`

```
feat: add user profile API endpoint
fix: correct HMAC blind index on email lookup
refactor: modularise auth routes into auth_service.py
```

### PR description

Every PR must include:

```
## What does this PR do?
[Describe what you changed and why]

## What was the state before this change?
[Describe the previous behaviour or the gap that existed]

## What is the state after this change?
[Describe the new behaviour and what it enables]

## How to test it?
[Steps to reproduce / verify the change works correctly]

## Screenshots (if applicable)

## Related issue
Closes #[issue number]
```

### PR size

- Keep PRs **small and focused** — one feature or fix per PR.
- If your change touches more than 3–4 files for unrelated reasons, split it into multiple PRs.
- Draft PRs are welcome for early feedback — label them clearly.

### After your PR is submitted

- Watch for review comments — respond and push fixes to the same branch.
- Do not force-push after a review has started unless asked to.
- If a maintainer requests significant changes, discuss in the PR before reworking.

---

## Code Style

### Python (`src/`)

- Follow [PEP 8](https://peps.python.org/pep-0008/)
- Use `snake_case` for function and variable names
- Each service file handles one domain only (auth, learning, users, etc.)
- All database queries must use **parameterized statements** — no string interpolation in SQL
- Never store plaintext PII, passwords, or tokens in D1

```python
# Good
cursor.execute("SELECT * FROM users WHERE email_index = ?", [email_hmac])

# Bad — SQL injection risk
cursor.execute(f"SELECT * FROM users WHERE email_index = '{email_hmac}'")
```

- All new Worker routes must follow the existing pattern in `worker.py`
- Return consistent JSON responses:

```python
# Success
return Response(json.dumps({"success": True, "data": result}), status=200)

# Error
return Response(json.dumps({"error": "Unauthorized"}), status=401)
```

### HTML / CSS (`public/`)

- Use **Tailwind CSS utility classes** — do not write custom CSS files
- Keep JavaScript minimal and inline only when necessary — no JS frameworks
- Pages must be **responsive** — test on mobile widths
- Do not add new external CDN dependencies without prior discussion

### General

- No commented-out dead code in PRs
- No `console.log` / `print` debugging left in submitted code
- Environment variables go in `.dev.vars` locally and Wrangler secrets in production — never hardcoded

---

## File & Folder Conventions

| Path | Purpose |
|---|---|
| `src/worker.py` | Main `on_fetch` dispatcher and routing only |
| `src/*_service.py` | Domain-specific logic (one file per domain) |
| `src/utils.py` | Shared helpers: encryption, ID generation, error responses |
| `public/*.html` | One file per page — static HTML connected to Worker APIs |
| `public/images/` | Static image assets only |
| `schema.sql` | All D1 table definitions live here |
| `scripts/` | Shell scripts for setup/deployment only |
| `.dev.vars` | Local secrets — **never commit this file** |

> `worker.py` should only route requests to the correct service file. Business logic does not belong in the dispatcher.

---

## What Gets Closed

PRs will be closed without merging if they:

- Have no linked, approved issue
- Add features that belong in a separate repo (see [What This Repo Is NOT](#what-this-repo-is-not))
- Mix multiple unrelated changes in one PR
- Introduce plaintext storage of sensitive data
- Add unnecessary files, folders, or dependencies not discussed in an issue
- Arrive with merge conflicts against `main`

This is not personal — it is how the repo stays maintainable. You are always welcome to open a new issue and try again with the feedback addressed.

---

## Other Repos in the Ecosystem

If your contribution does not belong in `learn`, it may belong in one of these:

- **`scholarai`** — AI-powered research assistant
- **`botlab`** — automation and bot tooling
- **`learnpilot`** — AI learning features

Check the Alpha One Labs GitHub organisation for the full list.

---

If you have questions, open an issue or ask on the community Discord.