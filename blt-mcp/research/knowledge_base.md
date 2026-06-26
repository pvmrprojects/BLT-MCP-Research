# Knowledge Base

> **Document Version:** 1.0
> **Status:** Consolidated from research phase
> **Date:** 2026-06-26
> **Purpose:** Cross-referenced index of all collected knowledge for BLT-MCP

---

## Table of Contents

1. [MCP Concepts](#mcp-concepts)
2. [BLT Concepts](#blt-concepts)
3. [API Concepts](#api-concepts)
4. [Security Concepts](#security-concepts)
5. [Architecture Concepts](#architecture-concepts)
6. [Deployment Concepts](#deployment-concepts)

---

## MCP Concepts

### Protocol Fundamentals

The Model Context Protocol (MCP) is an open protocol enabling integration between LLM applications and external data sources/tools. It uses JSON-RPC 2.0 messages over stateful connections with capability negotiation. The protocol defines three roles: Hosts (LLM apps), Clients (connectors), and Servers (service providers). [Source: MCP Spec 2025-11-25]

### Server Primitives

| Primitive | Control | Description | BLT-MCP Example |
|-----------|---------|-------------|-----------------|
| **Resources** | Application-controlled | Read-only contextual data addressable by URI | `blt://issues/{id}` |
| **Tools** | Model-controlled | Executable functions the LLM can invoke | `submit_issue()` |
| **Prompts** | User-controlled | Reusable workflow templates | `triage_vulnerability` |

[Source: MCP Protocol Features, py.sdk.modelcontextprotocol.io/protocol/]

### Client Primitives

| Primitive | Description | Relevance to BLT-MCP |
|-----------|-------------|----------------------|
| **Sampling** | Server asks client to generate LLM responses | Future: advanced triage prompts |
| **Roots** | Client tells server about filesystem/URI boundaries | Future: multi-tenant scoping |

### Lifecycle

Three mandatory phases:

1. **Initialization**: Client sends `initialize` with protocol version + capabilities. Server responds with its capabilities. Client sends `initialized` notification.
2. **Operation**: Normal message exchange (resource reads, tool calls, prompt gets).
3. **Shutdown**: Transport-level termination (no specific shutdown message defined).

Key point: Version negotiation allows the server to respond with a different protocol version if it doesn't support the client's preferred version.

[Source: MCP Lifecycle, modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle]

### Transports

| Transport | Use Case | Auth | Latency |
|-----------|----------|------|---------|
| stdio | Local/desktop (Claude Desktop, IDE) | Environment variables | Lowest |
| Streamable HTTP | Remote/cloud (ChatGPT, multi-client) | OAuth 2.1 with PKCE | Low |
| SSE (legacy) | One-way server→client streaming | OAuth 2.1 | Moderate |

Note: SSE is being deprecated in favor of Streamable HTTP. [Source: MCP Spec 2025-06-18]

### SDK Version Status

The MCP Python SDK is the primary implementation toolkit:

- **v1.x**: Stable, in maintenance mode. Pin with `mcp>=1.27,<2`.
- **v2 alpha**: Pre-release. Beta target: 2026-06-30. Stable target: 2026-07-27.

Key v2 changes: Simplified API, better typing, improved transport handling. [Source: MCP Python SDK README]

### Protocol Schema

- Source of truth: TypeScript schema at `schema/2025-11-25/schema.ts`
- Also available as JSON Schema at `schema/2025-11-25/schema.json`
- Uses JSON Schema 2020-12 for validation
- Tool input schemas auto-generated from Pydantic models via `model_json_schema()`

[Source: MCP Schema Reference]

### Cross-Reference: MCP vs REST vs GraphQL

| Dimension | REST | GraphQL | MCP |
|-----------|------|---------|-----|
| Discovery | Out-of-band (docs/OpenAPI) | Introspection query | In-band capability negotiation |
| Semantics | HTTP methods on resources | Query/mutation/subscription | Resources/Tools/Prompts |
| Auth model | Varies | Varies | OAuth 2.1 (HTTP) or env (stdio) |
| Agent fit | Poor | Moderate | Excellent (designed for LLMs) |

[Source: Inspired by Frustration; MCP Handbook]

---

## BLT Concepts

### Project Overview

OWASP BLT (Bug Logging Tool) is a gamified crowd-sourced QA testing and vulnerability disclosure platform. It allows users to report bugs across websites, applications, and Git repositories, with rewards and gamification elements. [Source: BLT Main README]

### Ecosystem Architecture (Current vs Target)

**Current:** Django monolith (Python 3.11+, Django 5.2, PostgreSQL, Redis, Celery)
**Target:** Edge-based microservices (Cloudflare Workers + D1 + GitHub Pages)

Migration status per BLT MIGRATION_PLAN.md:
- ✅ Already Separate: BLT-Pages, BLT-MCP, BLT-Rewards
- 🟡 Mostly Done: BLT-API, BLT-Next
- 🔵 Quick Migration: Queue system, Reminders
- 🔴 Complex Migration: Education platform, Core Auth

[Source: BLT MIGRATION_PLAN.md]

### Key Repositories (53+ total)

| Repository | Purpose | Tech Stack | BLT-MCP Relevance |
|------------|---------|------------|-------------------|
| OWASP-BLT/BLT | Main Django monolith | Django, PostgreSQL | Backend data source |
| OWASP-BLT/BLT-API | REST API on Cloudflare Workers | Python CF Workers, D1 | **Primary data source** |
| OWASP-BLT/BLT-MCP | MCP server prototype | TypeScript/Node.js | **This project** |
| OWASP-BLT/BLT-Next | Next-gen frontend | Static HTML + HTMX, CF Workers | Future alignment |
| OWASP-BLT/BLT-Pages | Bug reporting on GitHub Pages | HTML + Tailwind | Alternate UI |
| OWASP-BLT/BLT-Rewards | BACON token system | CF Workers, Bitcoin Ordinals | Tool: award_bacon |

### Issue Lifecycle

States: Open → Under Review → (Verified → Acknowledged → In Progress → Fixed) | (Rejected | Won't Fix)

Each state transition is triggered by community or organization managers. Verification is community-driven.

### Release History

| Version | Date | Key Features |
|---------|------|-------------|
| v2.1 | Nov 2025 | Dark mode, bounty payouts, security labs |
| v2.0 | Mar 2025 | Major redesign, BACON rewards, Slack integration |
| v1.5 | Jun 2024 | AI chatbot, website monitoring, crypto payments |
| v1.4 | Mar 2024 | New homepage, sidebar navigation |

### Technology Stack

- **Backend (legacy):** Python 3.11+, Django 5.2, Django REST Framework 3.16, Celery, Redis
- **Backend (modern):** Python 3.12+ on Cloudflare Workers, D1 (SQLite)
- **Frontend (legacy):** Django templates
- **Frontend (modern):** Static HTML + HTMX + Tailwind CSS
- **Mobile:** Flutter/Dart
- **Blockchain:** Bitcoin Ordinals/Runes, Solana (BACON)
- **Deployment:** Docker (legacy), Cloudflare Workers + GitHub Pages (modern)

### Known Limitations

1. Deep Django model coupling makes service extraction difficult (MIGRATION_PLAN.md marks Education and Core Auth as "Complex Migration")
2. Fragmented authentication across services (session, API key, JWT, GitHub App)
3. BLT-API uses shared static API key — no per-user scoping [BLT-API PR #89]
4. CVE integration is basic — enhanced model with full NVD API planned [Issue #5050]
5. Past RCE vulnerability in CI (GHSA-wxm3-64fx-cmx9) — mitigated

---

## API Concepts

### BLT-API Architecture

- **Runtime:** Python on Cloudflare Workers (edge-deployed)
- **Database:** Cloudflare D1 (SQLite-compatible) for bugs, domains; proxies to Django for other data
- **Versioning:** All endpoints under `/v2` prefix
- **Authentication:** Shared static API key via `X-BLT-API-Key` header
- **Discovery:** `GET /v2/routes` returns all registered routes

### Endpoint Groups

| Group | Endpoints | D1 Integrated? |
|-------|-----------|----------------|
| Health/Root | `GET /`, `GET /health` | No (public) |
| Routes | `GET /routes` | No (auth) |
| Bugs | `GET/POST /bugs`, `GET /bugs/{id}`, `GET /bugs/search` | Yes |
| Users | `GET/POST /users`, `GET /users/{id}/*` | No (proxied) |
| Auth | `POST /auth/signup`, `POST /auth/signin` | No |
| Domains | `GET /domains`, `GET /domains/{id}`, `GET /domains/{id}/tags` | Yes |
| Organizations | `GET /organizations/*` (9 sub-endpoints) | Yes |
| Hunts | `GET /hunts` | TBD |
| Projects | `GET /projects` | TBD |

### Key Validation Rules

- `POST /bugs`: URL must be http/https only, must have valid domain (netloc check). [Source: BLT-API PR #35]
- `POST /users`: Rate-limited, input validated, password hashed.

### Existing BLT-MCP Resources-to-API Mapping

| blt:// URI | REST Endpoint | Notes |
|------------|---------------|-------|
| `blt://issues` | `GET /v2/bugs` | Paginated |
| `blt://issues/{id}` | `GET /v2/bugs/{id}` | Includes screenshots, tags |
| `blt://contributors` | `GET /v2/users` | Paginated |
| `blt://contributors/{id}` | `GET /v2/users/{id}` | Includes profile stats |
| `blt://leaderboards` | `GET /v2/users?sort=reputation` | Derived, not a dedicated endpoint |
| `blt://rewards` | BACON API | Requires BLT-Rewards integration |
| `blt://repos` | TBD | **Research Required** — no clear mapping |
| `blt://workflows` | TBD | **Research Required** — no clear mapping |

### Cross-Reference: API Gaps

The following BLT-MCP resources/tools from the GSoC 2026 description lack documented REST endpoints:

- `blt://repos` and `blt://repos/{id}` — No explicit repository endpoint in BLT-API
- `blt://workflows` and `blt://workflows/{id}` — No workflow endpoint in BLT-API
- `award_bacon` tool — Requires BLT-Rewards API (separate service)
- `add_comment` tool — No dedicated comment endpoint in BLT-API

These may require BLT-API additions or direct integration with the Django backend.

---

## Security Concepts

### OWASP MCP Top 10 (v0.1)

| ID | Risk | BLT-MCP Mitigation |
|----|------|-------------------|
| MCP01 | Token Mismanagement | API key in env, never exposed to LLM |
| MCP02 | Privilege Escalation | Tool-level OAuth scopes, deny-by-default |
| MCP03 | Tool Poisoning | Static schemas, `additionalProperties: false`, mcp-scan |
| MCP04 | Supply Chain | Pinned deps, pip audit, Dependabot |
| MCP05 | Command Injection | Strict JSON Schema, type coercion |
| MCP06 | [Reserved] | — |
| MCP07 | Insufficient Auth | OAuth 2.1 PKCE (HTTP) or env (stdio) |
| MCP08 | Lack of Audit | Every tool call logged with identity |
| MCP09 | Shadow MCP Servers | Config management, official deployment |
| MCP10 | Context Injection | Filtered responses, no sensitive data |

[Source: OWASP MCP Top 10]

### Authentication Strategy by Transport

| Transport | Mechanism | Source |
|-----------|-----------|--------|
| stdio | Process isolation + env vars | MCP Spec 2025-06-18 |
| Streamable HTTP | OAuth 2.1 with PKCE | MCP Spec 2025-11-25; CSA Guide |

### Key Security Principles (from OWASP Cheat Sheet)

1. Grant minimum permissions per server
2. Use strict JSON Schema with `additionalProperties: false`
3. Require explicit user confirmation for destructive operations
4. Never auto-approve tool calls without showing full parameters
5. Never expose credentials to the LLM
6. Run local servers in sandboxed environments
7. Use OAuth 2.1 with PKCE for remote access

### Enterprise Security Maturity (CSA Framework)

| Level | Description | Target for BLT-MCP |
|-------|-------------|-------------------|
| 1 | Basic — ad hoc controls | Pre-GSoC (existing prototype) |
| 2 | Standardized — consistent across services | Sprint 5 (auth hardening) |
| 3 | **Advanced — tool-level scopes, audit, monitoring** | **GSoC target** |
| 4 | Dynamic — real-time risk adaptation | Future |

[Source: CSA Agentic MCP Security Guide]

---

## Architecture Concepts

### BLT-MCP Architecture Pattern

BLT-MCP follows a combination of:
- **API Wrapper pattern** (wraps BLT-API REST endpoints) [MCP Handbook]
- **Aggregator pattern** (combines BLT-API + BLT-Rewards + future services)
- **Layered Tool Server** [PADISO]: Data layer (resources), Action layer (tools), Workflow layer (prompts)

### Four-Layer Production Model (from Inspired by Frustration)

```
Layer 1: Transport (stdio | HTTP)
Layer 2: JSON-RPC 2.0 Router (handled by SDK)
Layer 3: Tool Definitions (schemas + handlers)
Layer 4: Guardrails (rate limits, circuit breakers, auth, caps) ← Most commonly skipped
```

Critical insight: "Most MCP servers nail layer 3 and punt on layer 4 — the exact layer that matters in production."

### MCP Capability Levels (from Working Software)

| Level | Description | BLT-MCP Target |
|-------|-------------|----------------|
| 0 | Wrap API in MCP server | Sprint 1-3 |
| 2 | Add error descriptions and usage tips | Sprint 3 |
| 3 | Provide real domain context (workflows) | Sprint 4 (Prompts) |
| 4 | Hypermedia-like autonomous capability discovery | Future |

### Request Flow Pipeline

```
Transport Adapter → Auth Middleware → Rate Limiter → JSON-RPC Router → Handler → BLT-API Client → Response Formatting → Audit Log
```

### Future Extension Points

1. **Multi-transport:** Add WebSocket for real-time subscriptions
2. **Plugin architecture:** Community-contributed resources, tools, prompts
3. **Multi-tenancy:** Per-organization API keys and scoped resources
4. **Event-driven:** Webhooks, resource subscriptions, async execution
5. **Caching:** In-memory cache with TTL, ETag support
6. **Federation/Gateway:** MCP gateway to multiple backend services

---

## Deployment Concepts

### Deployment Models

**Model 1: Local stdio** — For Claude Desktop and IDE extensions
- Server runs as subprocess
- Credentials from environment
- No network exposure

**Model 2: Remote HTTP** — For ChatGPT and cloud agents
- Server runs as HTTP service (ASGI)
- OAuth 2.1 with PKCE
- Deployable on Cloudflare Workers, Railway, Fly.io, or any ASGI-compatible host

### BLT Ecosystem Deployment

| Service | Platform | Type |
|---------|----------|------|
| BLT-Next frontend | GitHub Pages | Static |
| BLT-API | Cloudflare Workers | Edge Python |
| BLT-Pages | GitHub Pages | Static |
| BLT-Rewards | Cloudflare Workers | Edge Python |
| BLT-Pool | Cloudflare Workers | Edge Python |
| BLT-MCP (future) | Cloudflare Workers or ASGI host | Edge Python |
| Django App | Docker/VPS | Container |

### Configuration Management

All services use environment variables for configuration. Key variables:

| Variable | Service | Status |
|----------|---------|--------|
| `BLT_API_KEY` | BLT-API, BLT-MCP | Confirmed [BLT-API PR #89] |
| `BLT_API_BASE_URL` | BLT-API (backlink) | Confirmed [BLT-API README] |
| `JWT_SECRET` | BLT-API, BLT-Next | Confirmed |
| `DATABASE_URL` | BLT-Next, Django | Confirmed |

---

## References

### MCP Specification
1. MCP Specification (2025-11-25) — https://modelcontextprotocol.io/specification/2025-11-25
2. MCP Specification (2025-06-18) — https://modelcontextprotocol.io/specification/2025-06-18
3. MCP Lifecycle — https://modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle
4. MCP Schema Reference — https://modelcontextprotocol.io/specification/2025-11-25/schema
5. MCP Protocol Features — https://py.sdk.modelcontextprotocol.io/protocol/
6. MCP Specification Repository — https://github.com/modelcontextprotocol/modelcontextprotocol

### MCP SDK
7. MCP Python SDK Docs — https://py.sdk.modelcontextprotocol.io/
8. MCP Python SDK GitHub — https://github.com/modelcontextprotocol/python-sdk
9. MCP Example Remote Server — https://github.com/modelcontextprotocol/example-remote-server

### MCP Tutorials
10. Build an MCP Server — https://modelcontextprotocol.io/docs/develop/build-server
11. Build an MCP Client — https://modelcontextprotocol.io/docs/develop/build-client

### BLT Official
12. BLT Main Repository — https://github.com/OWASP-BLT/BLT
13. BLT Project Page — https://owasp.org/www-project-bug-logging-tool/
14. BLT Documentation — https://github.com/OWASP-BLT/BLT-docs
15. BLT Website — https://owaspblt.org
16. BLT Migration Plan — https://github.com/OWASP-BLT/BLT/blob/main/MIGRATION_PLAN.md
17. BLT Issues (CVE) — https://github.com/OWASP-BLT/BLT/issues/5050
18. BLT Security Advisory — https://github.com/OWASP-BLT/BLT/security/advisories/GHSA-wxm3-64fx-cmx9
19. OWASP-BLT GitHub Org — https://github.com/orgs/OWASP-BLT

### BLT Sub-Projects
20. BLT-API — https://github.com/OWASP-BLT/BLT-API
21. BLT-API PR #35 (URL validation) — https://github.com/OWASP-BLT/BLT-API/pull/35
22. BLT-API PR #41 (Organizations) — https://github.com/OWASP-BLT/BLT-API/pull/41
23. BLT-API PR #68 (Routes) — https://github.com/OWASP-BLT/BLT-API/pull/68
24. BLT-API PR #89 (Auth) — https://github.com/OWASP-BLT/BLT-API/pull/89
25. BLT-Next — https://github.com/OWASP-BLT/BLT-Next
26. BLT-Pages — https://github.com/OWASP-BLT/BLT-Pages
27. BLT-Rewards — https://github.com/OWASP-BLT/BLT-Bacon
28. BLT-MCP — https://github.com/OWASP-BLT/BLT-MCP
29. BLT-GSOC — https://github.com/OWASP-BLT/BLT-GSOC
30. BLT-Ideas — https://github.com/OWASP-BLT/BLT-Ideas
31. BLT-Pool — https://github.com/OWASP-BLT/BLT-GitHub-App
32. BLT-NetGuardian — https://github.com/OWASP-BLT/BLT-NetGuardian

### GSoC 2026
33. GSoC 2026 Ideas — https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
34. GSoC 2026 Overview — https://owasp.org/www-community/initiatives/gsoc/gsoc2026
35. BLT GSoC Landing — https://gsoc.owaspblt.org/

### Security
36. OWASP MCP Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
37. OWASP MCP Server Development Guide — https://genai.owasp.org/resource/a-practical-guide-for-secure-mcp-server-development/
38. OWASP MCP Top 10 — https://owasp.org/www-project-mcp-top-10/
39. OWASP Third-Party MCP Servers Guide — https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/
40. MCP Permissions Guide — https://readfa.com/blog/mcp-server-permissions/
41. Descope MCP Security — https://www.descope.com/blog/post/mcp-server-security-best-practices
42. CSA Agentic MCP Security — https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/
43. OWASP GenAI Security Blog — https://genai.owasp.org/2025/04/22/securing-ais-new-frontier-the-power-of-open-collaboration-on-mcp-security/

### Architecture & Community
44. MCP Architecture Handbook — https://github.com/ypollak2/mcp-handbook
45. IBM MCP Architecture — https://developer.ibm.com/articles/mcp-architecture-patterns-ai-systems/
46. Working Software MCP — https://www.workingsoftware.dev/mcp-in-practice-what-software-architects-need-to-know-about-the-model-context-protocol/
47. Inspired by Frustration — Production MCP — https://inspiredbyfrustration.com/blog/mcp-server-architecture
48. PADISO MCP Design Patterns — https://www.padiso.co/blog/ai-agents-production-mcp-server-design-patterns/
49. Obot MCP Enterprise — https://obot.ai/blog/mcp-enterprise-architecture-reference-guide/
50. Elastic Path MCP Patterns — https://www.elasticpath.com/blog/mcp-magic-moments-guide-to-llm-patterns
51. MCP Blog — Prompts for Automation — https://blog.modelcontextprotocol.io/posts/2025-07-29-prompts-for-automation/
52. OpenAI MCP Guide — https://developers.openai.com/api/docs/mcp
