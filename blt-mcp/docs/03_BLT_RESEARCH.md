# BLT Ecosystem Handbook

> **Document Version:** 1.0
> **Status:** Draft
> **Date:** 2026-06-26

---

## Architecture

### Current: Django Monolith

The OWASP Bug Logging Tool (BLT) was originally built as a Django monolithic application. The main repository contains the Django project, templates, models, views, and API endpoints all within a single codebase.

```
+----------------------------------------------------------+
|              BLT Django Monolith                          |
|                                                           |
|   +------------+  +-----------+  +-------------------+   |
|   | Templates  |  | Views     |  | REST API          |   |
|   | (Django)   |  | (Python)  |  | (Django Ninja)    |   |
|   +------------+  +-----------+  +-------------------+   |
|                                                           |
|   +------------+  +-----------+  +-------------------+   |
|   | Models     |  | Admin     |  | Celery Tasks      |   |
|   | (Django)   |  | (Django)  |  | (Background)      |   |
|   +------------+  +-----------+  +-------------------+   |
|                                                           |
|   +--------------------------------------------------+   |
|   |  PostgreSQL / Redis / Celery                      |   |
|   +--------------------------------------------------+   |
+----------------------------------------------------------+
```

**Key technologies:** Python 3.11+, Django 5.2+, Django REST Framework, PostgreSQL, Redis, Celery, Docker

[Source: BLT pyproject.toml, github.com/OWASP-BLT/BLT/blob/main/pyproject.toml; BLT README, github.com/OWASP-BLT/BLT]

### Target: Edge-Based Microservices

BLT is migrating toward a modular, edge-based architecture:

```
+------------------------------------------------------+
|                  BLT Ecosystem                        |
|                                                        |
|   +------------------+  +--------------------------+   |
|   | BLT-Next         |  | BLT-Pages                |   |
|   | (GitHub Pages    |  | (GitHub Pages            |   |
|   |  + Cloudflare    |  |  + Tailwind)             |   |
|   |  Workers)        |  |                          |   |
|   +------------------+  +--------------------------+   |
|                                                        |
|   +------------------+  +--------------------------+   |
|   | BLT-API          |  | BLT-MCP (this project)  |   |
|   | (Cloudflare      |  | (MCP Server Interface)  |   |
|   |  Workers + D1)   |  |                          |   |
|   +------------------+  +--------------------------+   |
|                                                        |
|   +------------------+  +--------------------------+   |
|   | BLT-Rewards      |  | BLT-NetGuardian         |   |
|   | (Cloudflare      |  | (Cloudflare Workers     |   |
|   |  Workers + Ord)  |  |  + D1)                  |   |
|   +------------------+  +--------------------------+   |
|                                                        |
|   +------------------+  +--------------------------+   |
|   | BLT-Pool         |  | BLT-Flutter              |   |
|   | (Cloudflare      |  | (Mobile App)             |   |
|   |  Workers + D1)   |  |                          |   |
|   +------------------+  +--------------------------+   |
|                                                        |
+------------------------------------------------------+
```

[Source: GSoC 2026 Ideas, owasp.org/www-community/initiatives/gsoc/gsoc2026ideas]

### Migration Status

Per the BLT MIGRATION_PLAN.md [Source: github.com/OWASP-BLT/BLT], the migration is structured as:

| Status | Component | Description |
|--------|-----------|-------------|
| ✅ Already Separate | BLT-Pages | Standalone issue reporting platform |
| ✅ Already Separate | BLT-MCP | MCP server (this project) |
| ✅ Already Separate | BLT-Rewards | BACON token system |
| 🟡 Mostly Done | BLT-API | Standalone REST API on Cloudflare Workers |
| 🟡 Mostly Done | BLT-Next | Static frontend + Workers backend |
| 🔵 Quick Migration | Queue system | Self-contained, minimal dependencies |
| 🔵 Quick Migration | Reminders | Minimal core coupling |
| 🔴 Complex Migration | Education platform | Deep model coupling |
| 🔴 Complex Migration | Core Auth | Shared auth infrastructure |

---

## Repositories

The OWASP-BLT GitHub organization contains 53+ public repositories [Source: github.com/orgs/OWASP-BLT]:

| Repository | Description | Language | Stars |
|------------|-------------|----------|-------|
| **BLT** | Main Django monolith | HTML/Python | 310 |
| **BLT-API** | REST API on Cloudflare Workers | Python | 2 |
| **BLT-MCP** | MCP server (this project) | JavaScript | 7 |
| **BLT-Next** | Next-gen architecture | HTML/Python | 15 |
| **BLT-Pages** | GitHub Pages bug reporting | HTML | 1 |
| **BLT-Flutter** | Mobile application | Dart | 26 |
| **BLT-Rewards** | BACON token system | Python | 10 |
| **BLT-Pool** | Contributor experience/GitHub App | Python | 4 |
| **BLT-NetGuardian** | Security scanning platform | Python | - |
| **BLT-Ideas** | GSoC brainstorming proposals | Python | 14 |
| **BLT-GSOC** | GSoC landing pages | HTML | - |
| **BLT-University** | Security education platform | - | - |
| **BLT-Preflight** | Pre-contribution advisories | - | - |
| **BLT-Hackathons** | Hackathon management | JavaScript | 9 |

---

## Services

### BLT (Main Django Application)

- **Purpose:** Core web application, user management, bug reporting UI, gamification
- **Tech Stack:** Django 5.2, Django REST Framework 3.16, PostgreSQL, Celery, Redis
- **Deployment:** Docker/Docker Compose
- **Authentication:** django-allauth (social auth), dj-rest-auth, session-based

[Source: pyproject.toml, github.com/OWASP-BLT/BLT]

### BLT-API

- **Purpose:** Edge-deployed REST API providing programmatic access to BLT data
- **Tech Stack:** Python on Cloudflare Workers, D1 (SQLite)
- **Deployment:** Cloudflare Workers with automatic D1 migrations
- **Authentication:** Shared static API key via `X-BLT-API-Key` header
- **Endpoints:** Bugs, Users, Domains, Organizations, Auth, Health
- **Base URL:** `https://api.owaspblt.org/v2`

[Source: BLT-API README, github.com/OWASP-BLT/BLT-API]

### BLT-MCP (Existing Prototype)

- **Purpose:** MCP server providing AI-agent access to BLT
- **Tech Stack:** TypeScript/Node.js
- **Transport:** stdio
- **Authentication:** BLT_API_KEY environment variable
- **Resources:** `blt://issues`, `blt://repos`, `blt://contributors`, `blt://leaderboards`, `blt://rewards`
- **Tools:** `submit_issue`, `award_bacon`, `update_issue_status`, `add_comment`

[Source: BLT-MCP README, github.com/OWASP-BLT/BLT-MCP]

### BLT-Next

- **Purpose:** Next-generation static frontend replacing Django templates
- **Tech Stack:** Static HTML + HTMX + Tailwind CSS (CDN), Cloudflare Python Workers
- **Database:** Cloudflare D1 (SQL)
- **Authentication:** JWT-based
- **Deployment:** GitHub Pages + Cloudflare Workers

[Source: BLT-Next README, github.com/OWASP-BLT/BLT-Next]

### BLT-Rewards (BACON)

- **Purpose:** Bitcoin-based token incentive system for contributors
- **Tech Stack:** Cloudflare Workers (Python), Bitcoin Ordinals/Runes protocol
- **Key Features:** Multi-chain support (Bitcoin & Solana), GitHub Actions integration
- **Components:** Ord server for Bitcoin inscriptions, API server for token management

[Source: BLT-Rewards/README, github.com/OWASP-BLT/BLT-Bacon]

### BLT-Pool

- **Purpose:** Contributor experience platform with mentor directory and GitHub App
- **Tech Stack:** Cloudflare Workers (Python), D1 database
- **Endpoints:** Mentor directory, GitHub webhooks, health checks
- **Integration:** GitHub App for issue assignment, leaderboard scoring, review signals

[Source: BLT-Pool/README, github.com/OWASP-BLT/BLT-GitHub-App]

### BLT-NetGuardian

- **Purpose:** Autonomous security scanning platform
- **Tech Stack:** Cloudflare Workers (Python), D1 database
- **Features:** Distributed scanning via volunteer CLI clients, Zero-Trust encrypted ingestion, vulnerability detection (XSS, SQLi, CSRF), Semgrep SAST
- **Deployment:** Frontend on GitHub Pages, Backend on Cloudflare Workers

[Source: BLT-NetGuardian, github.com/OWASP-BLT/BLT-NetGuardian]

---

## Authentication

### Current State

BLT's authentication is fragmented across services:

| Service | Method | Notes |
|---------|--------|-------|
| Main Django App | Session + django-allauth | Social auth (GitHub, Google), email/password |
| BLT-API | Static API Key (`X-BLT-API-Key`) | Shared key, public routes for /health |
| BLT-Next | JWT | JWT_SECRET env var |
| BLT-Pool | GitHub App Installation | OAuth via GitHub |
| BLT-MCP | API Key via environment | Delegates to BLT-API for backend calls |

[Source: BLT-API PR #89, github.com/OWASP-BLT/BLT-API/pull/89; BLT-Next README]

### Authentication Flow Diagram

```
User/Browser                    BLT-Next               BLT-API              Django App
    |                              |                      |                     |
    |-- Login ------------------->|                      |                     |
    |                              |-- POST /auth/signin->|                     |
    |                              |<-- JWT Token --------|                     |
    |<-- Session/Token ------------|                      |                     |
    |                              |                      |                     |
    |-- Report Bug --------------->|                      |                     |
    |                              |-- POST /bugs ------->|                     |
    |                              |   X-BLT-API-Key      |                     |
    |                              |                      |-- Sync ----------->|
    |<-- Confirmation -------------|<-- Response ---------|                     |
```

---

## Issue Lifecycle

The issue (bug report) lifecycle in BLT follows these stages:

```
Reported
    │
    v
Under Review
    │
    ├──> Verified (Confirmed by community)
    │         │
    │         v
    │    Acknowledged (Company confirms)
    │         │
    │         v
    │    In Progress (Fix underway)
    │         │
    │         v
    │    Fixed (Resolution deployed)
    │
    └──> Rejected (Not a valid bug)
              │
              v
         Won't Fix (Acknowledged but not addressed)
```

### Key States

| State | Description | Who Sets |
|-------|-------------|----------|
| Open | Initial state after reporting | System |
| Under Review | Being assessed by community | Moderator |
| Verified | Confirmed by community votes | Community |
| Acknowledged | Company confirms the issue | Organization manager |
| In Progress | Fix is being developed | Organization manager |
| Fixed | Resolution deployed | Organization manager |
| Rejected | Not a valid bug | Moderator |
| Won't Fix | Valid but not addressing | Organization manager |

[Source: OWASP BLT Issue Workflow, github.com/OWASP-BLT/BLT]

---

## Reward System

### BACON Tokens

BACON is a Bitcoin-based token system using the Runes protocol. Key characteristics:

- **Blockchain:** Bitcoin (via Runes protocol) + Solana
- **Purpose:** Incentivize security contributions
- **Minting:** Custom ord server infrastructure
- **Distribution:** Automated via GitHub Actions and BLT-Rewards API
- **Redemption:** Marketplace for swag and merchandise

[Source: BLT-Rewards, github.com/OWASP-BLT/BLT-Bacon]

### Gamification Elements

| Element | Description |
|---------|-------------|
| Points | Earned for reporting bugs and contributing |
| Badges | Achievement-based recognition for specific domains |
| Leaderboards | Top contributors ranked by activity and impact |
| Reputation Tiers | Progressive levels (Beginner → Expert) |
| Bug Bounties | Company-sponsored prize pools |

[Source: OWASP BLT Project Page, owasp.org/www-project-bug-logging-tool/]

---

## Database

### Main Django App (PostgreSQL)

The Django monolith uses PostgreSQL with key models:

```
User (django-allauth)
  ├── Bug/Issue
  │     ├── Screenshot
  │     ├── Comment
  │     ├── Vote/Verification
  │     └── CVE (cve_id, cve_score)
  ├── Domain
  │     └── Tag
  ├── Organization
  │     ├── Manager
  │     ├── Integration
  │     └── BugBounty
  ├── Contributor
  ├── BACON Reward
  └── Activity Log
```

[Source: BLT Issue #5050, github.com/OWASP-BLT/BLT/issues/5050]

### BLT-API (Cloudflare D1)

Uses SQLite-compatible D1 with migrations:

| Migration | Tables |
|-----------|--------|
| 0001_init.sql | Base tables |
| 0002_add_bugs.sql | Bug storage |
| 0003_user_schema.sql | User accounts |
| 0004_org_schema.sql | Organizations, managers, tags, integrations |

[Source: BLT-API, github.com/OWASP-BLT/BLT-API]

### BLT-Next (Cloudflare D1)

Unified schema defined in `schema.sql`:

```
users, bugs, domains, organizations, leaderboards, rewards
```

[Source: BLT-Next, github.com/OWASP-BLT/BLT-Next]

---

## Deployment

### Current: Docker Compose (Django)

```yaml
services:
  web:     # Django application
  db:      # PostgreSQL
  redis:   # Cache/Queue
  celery:  # Background tasks
```

[Source: BLT docker-compose.yml, github.com/OWASP-BLT/BLT]

### Target: Cloudflare Edge + GitHub Pages

| Service | Platform | URL |
|---------|----------|-----|
| BLT-Next Frontend | GitHub Pages | `https://owasp-blt.github.io/BLT-Next/` |
| BLT-API | Cloudflare Workers | `https://api.owaspblt.org` |
| BLT-Pages | GitHub Pages | `https://owasp-blt.github.io/BLT-Pages/` |
| BLT-Rewards | Cloudflare Workers | Cloudflare Workers |
| BLT-Pool | Cloudflare Workers | `https://pool.owaspblt.org` |
| BLT-GSOC | GitHub Pages | `https://owasp-blt.github.io/BLT-GSOC/` |
| BLT-NetGuardian | Cloudflare Workers | Cloudflare Workers |

### Environment Configuration

Key environment variables across services:

| Variable | Service | Purpose |
|----------|---------|---------|
| `DATABASE_URL` | Multiple | Database connection |
| `SECRET_KEY` | Django App | Django secret |
| `JWT_SECRET` | BLT-API, BLT-Next | JWT signing |
| `BLT_API_KEY` | BLT-API, BLT-MCP | API authentication |
| `BLT_API_BASE_URL` | BLT-API | Backend proxy target |
| `MAILGUN_API_KEY` | BLT-API | Email delivery |
| `OPENAI_API_KEY` | Django App | AI features |

---

## Technology Stack

### Core Technologies

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Backend (Legacy) | Python 3.11+, Django 5.2 | Main application framework |
| Backend (Modern) | Python 3.12+ on Cloudflare Workers | Edge API services |
| Backend (Data) | Cloudflare D1 (SQLite) | Edge database |
| Backend (Legacy DB) | PostgreSQL | Primary database |
| Backend (Legacy Cache) | Redis | Caching, task queue |
| Backend (Legacy Tasks) | Celery | Background task processing |
| Frontend (Legacy) | Django Templates | Server-rendered HTML |
| Frontend (Modern) | Static HTML + HTMX + Tailwind CSS | Edge-served static pages |
| Mobile | Flutter/Dart | Mobile application |
| Blockchain | Bitcoin Ordinals/Runes, Solana | BACON token rewards |

### Key Dependencies (Django App)

From pyproject.toml [Source: github.com/OWASP-BLT/BLT]:

| Package | Purpose |
|---------|---------|
| Django 5.2 | Web framework |
| djangorestframework 3.16 | REST API |
| django-allauth | Social authentication |
| dj-rest-auth | REST auth endpoints |
| django-import-export | Data import/export |
| whitenoise | Static file serving |
| django-storages (Google) | Cloud storage |
| django-redis | Redis cache backend |
| psycopg2-binary | PostgreSQL adapter |
| beautifulsoup4 | HTML parsing |
| user-agents | User agent parsing |

---

## Contribution Workflow

The BLT project follows a standard GitHub fork-and-PR workflow [Source: CONTRIBUTING.md, github.com/OWASP-BLT/BLT]:

```
1. Fork the repository
2. Create a feature branch (git checkout -b feature/amazing-feature)
3. Make changes following coding standards (Black, isort, ruff)
4. Run pre-commit before submitting
5. Commit with descriptive messages
6. Push to fork
7. Create Pull Request to main repository
8. Ensure CI passes
9. Respond to review feedback
```

### Coding Standards

- **Python:** Black formatter, isort imports, ruff linter
- **Pre-commit:** Must pass before submission
- **CI:** GitHub Actions workflows for testing and linting

### GitHub Bot Leaderboard

BLT has an automated leaderboard bot that runs on `pull_request_target` events:
- Scores contributions (open PRs, merged PRs, reviews, comments)
- Posts leaderboard comments on PRs
- Uses GitHub GraphQL API and REST API

---

## Known Limitations

1. **Django Monolith Coupling:** Deep model coupling makes extraction of individual services difficult. Migration plan identifies several "Complex Migration" components [Source: MIGRATION_PLAN.md].

2. **Fragmented Authentication:** Different services use different auth mechanisms (session, API key, JWT, GitHub App). No unified identity layer.

3. **Static API Key Security:** BLT-API uses a shared static API key. While practical for edge deployment, it lacks per-user scoping and rotation policies [BLT-API PR #89].

4. **No Standardized AI Interface:** Before BLT-MCP, there was no protocol-based interface for AI agent integration.

5. **CVE Integration:** Currently limited to basic CVE ID and score storage. Enhanced CVE model with full NVD integration is planned [Issue #5050].

6. **RCE Vulnerability in CI:** A past advisory (GHSA-wxm3-64fx-cmx9) identified unsafe `pull_request_target` workflows that could execute untrusted code. Mitigations are in place.

7. **Documentation Gaps:** BLT-docs is actively being built but many internal APIs lack formal OpenAPI specifications.

---

## Future Roadmap

### GSoC 2026 Projects

| Project | Description | Status |
|---------|-------------|--------|
| **BLT-Next** | Full migration from Django monolith to static/edge | Active |
| **BLT-MCP** | MCP server for AI agent integration | Active (this project) |
| **BLT-NetGuardian** | Distributed security scanning platform | Active |
| **BLT University** | Security education with interactive labs | Active |
| **BLT-Rewards** | BACON gamification enhancements | Active |
| **BLT-Preflight** | Pre-contribution security advisories | Active |

### Platform Evolution

1. **Complete API Extraction** — All Django views migrated to BLT-API endpoints
2. **Unified Authentication** — OAuth 2.1/OIDC across all services
3. **Event-Driven Architecture** — Webhooks/events for cross-service communication
4. **API Gateway** — Single entry point for all BLT services
5. **Enhanced CVE Integration** — Full NVD API integration with caching and search [Issue #5050]
6. **MCP Ecosystem** — BLT-MCP as the primary AI integration surface

---

## References

1. BLT Main Repository — https://github.com/OWASP-BLT/BLT
2. BLT pyproject.toml — https://github.com/OWASP-BLT/BLT/blob/main/pyproject.toml
3. BLT MIGRATION_PLAN.md — https://github.com/OWASP-BLT/BLT/blob/main/MIGRATION_PLAN.md
4. BLT Issue #5050 (CVE Integration) — https://github.com/OWASP-BLT/BLT/issues/5050
5. BLT Security Advisory (RCE) — https://github.com/OWASP-BLT/BLT/security/advisories/GHSA-wxm3-64fx-cmx9
6. CONTRIBUTING.md — https://github.com/OWASP-BLT/BLT/blob/main/CONTRIBUTING.md
7. BLT-API Repository — https://github.com/OWASP-BLT/BLT-API
8. BLT-API PR #89 (Auth) — https://github.com/OWASP-BLT/BLT-API/pull/89
9. BLT-Next Repository — https://github.com/OWASP-BLT/BLT-Next
10. BLT-MCP Repository — https://github.com/OWASP-BLT/BLT-MCP
11. BLT-Rewards Repository — https://github.com/OWASP-BLT/BLT-Bacon
12. BLT-Pool Repository — https://github.com/OWASP-BLT/BLT-GitHub-App
13. BLT-NetGuardian Repository — https://github.com/OWASP-BLT/BLT-NetGuardian
14. OWASP-BLT GitHub Organization — https://github.com/orgs/OWASP-BLT
15. OWASP BLT Project Page — https://owasp.org/www-project-bug-logging-tool/
16. GSoC 2026 Ideas — https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
