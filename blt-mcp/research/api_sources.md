# API Sources

## Source 1: BLT-API Full Endpoint Reference

- **URL**: https://github.com/OWASP-BLT/BLT-API/blob/main/README.md
- **Title**: BLT-API README - REST API Endpoints
- **Author/Organization**: OWASP-BLT
- **Summary**: Complete reference for all BLT-API REST endpoints including bugs, users, domains, and organizations.
- **Important Technical Details**:

  **Bugs**:
  - `GET /bugs` — List paginated bugs (query params: page, per_page, status, domain, verified)
  - `GET /bugs/{id}` — Get single bug with screenshots and tags
  - `GET /bugs/search?q={query}` — Search bugs by URL or description
  - `POST /bugs` — Create new bug report

  **Users**:
  - `GET /users` — List paginated users
  - `POST /users` — Create user (rate-limited, validated, password-hashed)
  - `GET /users/{id}` — Get user
  - `GET /users/{id}/profile` — Profile with statistics
  - `GET /users/{id}/bugs` — Bugs by user
  - `GET /users/{id}/domains` — Domains by user
  - `GET /users/{id}/followers` — Followers
  - `GET /users/{id}/following` — Following

  **Domains**:
  - `GET /domains` — List paginated domains
  - `GET /domains/{id}` — Get domain
  - `GET /domains/{id}/tags` — Tags for domain

  **Organizations**:
  - `GET /organizations` — List organizations (with search, filtering by type/status)
  - `GET /organizations/{id}` — Get organization
  - `GET /organizations/{id}/domains` — Domains
  - `GET /organizations/{id}/bugs` — Bugs
  - `GET /organizations/{id}/managers` — Managers
  - `GET /organizations/{id}/tags` — Tags
  - `GET /organizations/{id}/integrations` — Integrations
  - `GET /organizations/{id}/stats` — Statistics

  **General**:
  - All available under `/v2` prefix
  - `GET /routes` — Programmatic API discovery (returns all registered routes as JSON)
  - `GET /health` — Health check
- **Why It Is Useful**: Primary data source that BLT-MCP will wrap. Every MCP resource and tool will correspond to one or more of these REST endpoints.

---

## Source 2: BLT-API Authentication

- **URL**: https://github.com/OWASP-BLT/BLT-API/pull/89
- **Title**: BLT-API Static API Key Authentication
- **Author/Organization**: OWASP-BLT / mdkaifansari04
- **Summary**: Pull request introducing shared static API key authentication for BLT-API. Requests include `X-BLT-API-Key` header.
- **Important Technical Details**:
  - Public routes (no auth required): `/`, `/v2`, `/health`, `/v2/health`, `OPTIONS`
  - Protected routes require `X-BLT-API-Key` header
  - Returns 401 for missing/invalid keys
  - Environment variable: `BLT_API_KEY`
  - Built-in public key works without env configuration
  - Case-insensitive header matching
- **Why It Is Useful**: BLT-MCP must handle authentication to BLT-API. This documents the auth mechanism to implement.

---

## Source 3: BLT-API URL Validation

- **URL**: https://github.com/OWASP-BLT/BLT-API/pull/35
- **Title**: BLT-API URL Protocol Validation on POST /bugs
- **Author/Organization**: OWASP-BLT / ojaswa072
- **Summary**: Adds URL protocol and domain validation to POST /bugs to prevent invalid or malicious URLs.
- **Important Technical Details**:
  - Validates URLs use http or https protocol (blocks javascript:, ftp:, etc.)
  - Validates URL contains a valid domain via netloc check
  - Returns 400 with clear error messages
- **Why It Is Useful**: Understanding BLT-API validation rules ensures BLT-MCP can provide meaningful error feedback to users.

---

## Source 4: BLT-API Organization Endpoints

- **URL**: https://github.com/OWASP-BLT/BLT-API/pull/41
- **Title**: BLT-API Organization API and Schema Migration
- **Author/Organization**: OWASP-BLT / mdkaifansari04
- **Summary**: Comprehensive organization data endpoints with SQL migration for schema (migration 0004_org_schema.sql).
- **Important Technical Details**:
  - New tables: organization, organization_managers, organization_tags, organization_integrations
  - Search across name, slug, and description
  - Filtering by organization type and status
  - Statistics: domain count, bug count, verified bugs, manager count
  - Pagination support
- **Why It Is Useful**: Organizations are a key BLT entity. Understanding the data model informs MCP resource/tool design.

---

## Source 5: BLT-MCP Existing Implementation — Resources (blt:// URIs)

- **URL**: https://github.com/OWASP-BLT/BLT-MCP
- **Title**: OWASP-BLT/BLT-MCP - Existing MCP Server
- **Author/Organization**: OWASP-BLT
- **Summary**: The existing BLT-MCP implementation (Node.js/TypeScript) that serves as the starting point for GSoC 2026 development.
- **Important Technical Details**:
  - Currently a JavaScript/TypeScript implementation
  - Resources via `blt://` URIs:
    - `blt://issues` — All issues
    - `blt://issues/{id}` — Specific issue
    - `blt://repos` — Tracked repositories
    - `blt://repos/{id}` — Specific repository
    - `blt://contributors` — All contributors
    - `blt://contributors/{id}` — Specific contributor
    - `blt://workflows` — All workflows
    - `blt://workflows/{id}` — Specific workflow
    - `blt://leaderboards` — Leaderboard rankings
    - `blt://rewards` — Rewards and bacon points
  - Tools: `submit_issue`, `award_bacon`, `update_issue_status`, `add_comment`
  - Environment config: `BLT_API_BASE`, `BLT_API_KEY`
  - stdio transport for MCP communication
  - Claude Desktop / Cline integration via mcpServers config
- **Why It Is Useful**: This is the existing codebase that the GSoC 2026 BLT-MCP project builds upon. Understanding its current state is critical before extending it.

---

## Source 6: BLT-MCP Prompts Layer

- **URL**: https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
- **Title**: GSoC 2026 Ideas - BLT-MCP Description
- **Author/Organization**: OWASP Foundation
- **Summary**: Official GSoC 2026 project description for BLT-MCP, detailing the three-layer architecture.
- **Important Technical Details**:
  - Resources: Read-only access via `blt://` URIs
  - Tools: Actions like `submit_issue`, `award_bacon`, `update_issue_status`, `add_comment`
  - Prompts: Reusable task templates — `triage_vulnerability`, `plan_remediation`, `review_contribution`
  - Authentication: OAuth 2.0 / API key
  - Transports: stdio and HTTP/SSE
  - JSON-RPC 2.0
  - Integration targets: Claude Desktop, custom AI agents, third-party tools
  - Goal: Unify fragmented REST/GraphQL endpoints into a single AI-agent-friendly interface
- **Why It Is Useful**: This is the project charter for BLT-MCP. All implementation should align with this vision.

---

## Source 7: BLT-API D1 Database Schema

- **URL**: https://github.com/OWASP-BLT/BLT-API
- **Title**: OWASP-BLT/BLT-API - D1 Database Integration
- **Author/Organization**: OWASP-BLT
- **Summary**: BLT-API uses Cloudflare D1 (SQLite-compatible) for domain and bug data persistence.
- **Important Technical Details**:
  - D1-integrated endpoints: `/domains`, `/domains/{id}/tags`, `/bugs`, `/bugs/{id}`
  - SQL migrations in `migrations/` directory
  - Test data seeds in `test_data.sql`
- **Why It Is Useful**: Understanding the underlying data store informs how BLT-MCP should handle data consistency and pagination.

---

## Source 8: BLT API OpenAPI/Programmatic Discovery

- **URL**: https://github.com/OWASP-BLT/BLT-API/pull/68
- **Title**: Add /routes endpoint for programmatic API discoverability
- **Author/Organization**: OWASP-BLT / azizrebhi
- **Summary**: Added GET /routes endpoint returning all registered API routes as structured JSON, built dynamically from the router's internal registry.
- **Important Technical Details**:
  - `Router.get_route_list()` returns `[{ method, path }, ...]`
  - Response: `{ success: true, data: [{ method, path }], count: N }`
  - Available at both `/routes` and `/v2/routes`
  - Self-including: GET /routes appears in its own output
- **Why It Is Useful**: BLT-MCP could use this endpoint for dynamic capability discovery rather than hardcoding endpoint URLs.
