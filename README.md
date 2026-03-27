# AlphaOneLabs — Learn

> An open-source education platform for learners and educators to connect, collaborate and grow.

Alpha One Labs is built to go beyond traditional online education — providing courses, study groups, peer connections and a collaborative learning environment in one place.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [Set Up Instructions](#set-up-instructions)
- [Contributing](#contributing)
- [Community](#community)

---

## Project Overview

Alpha One Labs is an education platform that lets educators create and manage courses while students learn, collaborate and engage with peers. Core features include authentication, courses, sessions, user profiles, study groups and peer communication.

Want to understand the codebase before diving in? Start with **[`CODEBASE.md`](./CODEBASE.md)** for a full walkthrough of the project architecture.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML + Tailwind CSS + Vanilla JavaScript |
| Backend | Python (Cloudflare Workers) |
| Database | Cloudflare D1 (SQLite) |

---

## Set Up Instructions

### Prerequisites

Make sure you have installed:
- [Node.js](https://nodejs.org/)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/)

Install Wrangler globally:

```bash
npm install -g wrangler
```

### Clone the Repository

```bash
git clone https://github.com/alphaonelabs/learn.git
cd learn
```

### Login to Cloudflare (one-time)

```bash
wrangler login
```

### Setup Database (D1)

Create the database:

```bash
wrangler d1 create education_db
```

Add the generated `database_id` to your `wrangler.toml`:

```toml
[[d1_databases]]
binding = "DB"
database_name = "education_db"
database_id = "YOUR_DATABASE_ID"
```

Apply the schema:

```bash
wrangler d1 execute education_db --file=schema.sql
```

### Setup Environment Variables

**For local development** — create a `.dev.vars` file in the project root:

```
ENCRYPTION_KEY=your-dev-encryption-key
JWT_SECRET=your-dev-jwt-secret
```

> ⚠️ Never commit `.dev.vars`. It is listed in `.gitignore`.

**For production** — use Wrangler secrets:

```bash
wrangler secret put ENCRYPTION_KEY
wrangler secret put JWT_SECRET
```

### Run the Backend

```bash
wrangler dev
# Runs at http://127.0.0.1:8787
```

### Run the Frontend

Open directly:

```bash
public/index.html
```

Or use a local server:

```bash
npx serve public
# Runs at http://localhost:3000
```

---

## Contributing

We welcome contributions of all kinds — bug fixes, new features, documentation improvements, and more.

**Before writing any code, please read [`CONTRIBUTING.md`](./CONTRIBUTING.md) in full.** It covers:

- How to open an issue and get it approved before starting work
- Branch naming and commit message conventions
- PR guidelines including what to include before and after changes
- Code style for Python and HTML/CSS
- What gets closed without merging

> **The short version:** open an issue first → wait for maintainer approval → then start coding.

---

## Community

Have questions or want to discuss ideas before opening an issue? Join us on the [Alpha One Labs Slack](https://join.slack.com/t/alphaonelabs/shared_invite/zt-7dvtocfr-1dYWOL0XZwEEPUeWXxrB1A).

---

*For a full breakdown of the folder structure, architecture decisions, and security conventions, see [`CODEBASE.md`](./CODEBASE.md).*