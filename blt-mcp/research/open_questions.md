# Open Questions

> **Document Version:** 1.0
> **Date:** 2026-06-26
> **Purpose:** Catalog unanswered questions requiring further investigation before implementation decisions

---

## 1. BLT-API Repository Access

**Question:** What is the correct URL for the BLT-API source repository?

**Context:** The repository `https://github.com/OWASP-BLT/BLT-API` is confirmed to exist via web search index (2 stars, 17 forks, Python, last push 2026-03-28, 9 contributors). However, **all GitHub access from this environment returns 404** — this includes `git clone`, raw content URLs, GitHub API, and web fetches. The issue affects all OWASP-BLT repos including BLT, BLT-Next, BLT-MCP — they all exist but are network-blocked.

**Known details from web search:**
- Cloudflare Workers Python app with D1 database (SQLite)
- wrangler.toml: D1 binding `blt_api`, migrations dir, Python compatibility
- Shared static API key auth (`X-BLT-API-Key`) added PR #89 (May 2026)
- `GET /v2/routes` endpoint for programmatic discovery (PR #68, March 2026)
- Stack: Python, Cloudflare Workers, D1, HTML/JS for homepage
- Endpoints: bugs, users, domains, orgs, projects, hunts, auth

**Impact:** Cannot inspect handler implementations or D1 schemas directly. Must rely on README, PR descriptions, and web search data.

**Resolution:** Environment-specific network restriction. Token-based access (PAT) or VPN may resolve.

**Status:** `Blocked (Environment Restriction)`

---

## 2. Repository Resources API

**Question:** What REST endpoints back the `blt://repos` and `blt://repos/{id}` resources?

**Context:** The existing BLT-MCP prototype and GSoC 2026 project description define `blt://repos` and `blt://repos/{id}` as MCP resources. However, no documented BLT-API endpoint maps to repository data. The BLT main application works with GitHub repositories, but the specific API contract is unclear.

**Possible sources:**
- BLT Django models may have a `Repository` model
- GitHub API integration may be handled client-side in the Django app
- BLT-API may proxy to GitHub's API

**Impact:** If no BLT-API endpoint exists, we need to either:
- Add an endpoint to BLT-API
- Interface directly with the Django backend
- Remove the resource from BLT-MCP scope

**Resolution:** Inspect the BLT Django models (specifically `website/models.py`) for Repository-related models.

**Status:** `Research Required`

---

## 3. Workflow Resources API

**Question:** What REST endpoints back the `blt://workflows` and `blt://workflows/{id}` resources?

**Context:** Same as repositories — `blt://workflows` is defined in the GSoC description and existing prototype, but no corresponding BLT-API endpoint is documented.

**Impact:** Similar to question 2 — may require BLT-API additions.

**Resolution:** Investigate BLT's workflow/automation features. Determine if workflows are a BLT concept or were created speculatively for the MCP interface.

**Status:** `Research Required`

---

## 4. BLT-Rewards API Contract

**Question:** What is the exact API contract for the BACON rewards system (needed by the `award_bacon` tool)?

**Context:** The `award_bacon` tool is listed in the GSoC 2026 project description and existing BLT-MCP prototype. BLT-Rewards is documented as a Cloudflare Workers-based system with Bitcoin Ordinals/Runes integration, but the API endpoints for awarding tokens are not publicly documented.

**Impact:** Without the BLT-Rewards API contract:
- `award_bacon` tool cannot be fully implemented
- Error handling and parameter validation are speculative
- Authentication requirements are unknown

**Resolution:** Access the BLT-Rewards source code (if public) or document the API through testing.

**Status:** `Research Required`

---

## 5. Comment Endpoint

**Question:** What REST endpoint handles adding comments to issues (needed by the `add_comment` tool)?

**Context:** `add_comment` is listed as an MCP tool, but no `POST /v2/bugs/{id}/comments` or similar endpoint is listed in the BLT-API README.

**Impact:** `add_comment` cannot be implemented without a backend endpoint.

**Resolution:** Check if comments are handled through the Django app API (not yet migrated to BLT-API).

**Status:** `Research Required`

---

## 6. MCP Python SDK v2 Migration Timing

**Question:** Should BLT-MCP target MCP Python SDK v1.x or v2?

**Context:** As of 2026-06-26:
- v1.x is stable but in maintenance mode
- v2 alpha is available, beta target 2026-06-30, stable target 2026-07-27

GSoC 2026 runs approximately June-September 2026. If v2 stable releases during the project timeline, migration mid-project could be disruptive.

**Decision factors:**
- If development starts before v2 stable: use v1.x with `mcp>=1.27,<2` pin
- If v2 stable is available: evaluate breaking changes before adopting
- Migration effort from v1.x to v2 is unknown

**Status:** `Decision Pending` — Monitor v2 release timeline.

---

## 7. MCP Spec Version Target

**Question:** Which MCP specification version should BLT-MCP implement?

**Context:**
- 2025-11-25: Current stable spec with auth framework
- 2025-06-18: Previous stable (auth server separation introduced)
- Draft: Includes MRTR, elicitation, new message patterns

**Decision factors:**
- MCP Python SDK v1.x targets 2025-11-25
- Draft features not fully implemented in any stable SDK

**Status:** `Decision Pending` — Likely 2025-11-25 for initial implementation, draft features as extensions.

---

## 8. BLT-API Rate Limits

**Question:** What are the rate limits for BLT-API?

**Context:** BLT-API mentions rate limiting in its README for `POST /users`, but global rate limits and per-endpoint limits are not documented.

**Impact:** BLT-MCP's rate limiter configuration depends on knowing upstream limits. Setting MCP limits too high will cause backend failures; too low will frustrate users.

**Resolution:** Requires testing or contacting maintainers.

**Status:** `Research Required`

---

## 9. BLT-API PATCH Support

**Question:** Does BLT-API support PATCH/PUT for updating existing resources?

**Context:** The `update_issue_status` tool needs to change the status of an existing bug. The documented BLT-API endpoints only show `GET /bugs/{id}` and `POST /bugs`. No PATCH/PUT endpoint is documented.

**Possible approaches:**
- BLT-API may support undocumented PATCH endpoints
- Status updates may go through a different mechanism
- May need to proxy directly to Django app
- BLT-MCP may need to contribute a PATCH endpoint to BLT-API

**Status:** `Research Required`

---

## 10. BLT-API D1 Schema Details

**Question:** What is the complete D1 database schema for BLT-API?

**Context:** The migration files (`0001_init.sql`, `0002_add_bugs.sql`, `0003_user_schema.sql`, `0004_org_schema.sql`) are referenced but we don't have access to their contents. wrangler.toml confirms D1 binding name `blt_api`, database ID `bb8bc336-a8d2-4aa4-a244-f492b15a97dd`, and migrations table `d1_migrations`.

**Known tables (from PR descriptions and code mentions):**
- `organization`, `organization_managers`, `organization_tags`, `organization_integrations` (migration 0004)
- Bug/issue-related tables (migration 0002)
- User-related tables (migration 0003)
- Initial schema with base tables (migration 0001)

**Impact:** Without schema details, resource response formatting and filtering parameters are based on observed API responses rather than authoritative schema.

**Resolution:** Access migration files (blocked) or reverse-engineer from API responses.

**Status:** `Blocked (Environment Restriction)`

---

## 11. Authentication for Future Multi-Tenant Support

**Question:** How should BLT-MCP handle multi-tenant authentication where different API keys have different scopes?

**Context:** BLT-API currently uses a shared static API key. For multi-tenant BLT-MCP deployments, different organizations need different access levels.

**Current constraints:**
- BLT-API has no per-user or per-org API key management
- No OAuth scope infrastructure exists in BLT-API

**Impact:** Multi-tenant support requires BLT-API changes beyond BLT-MCP scope.

**Status:** `Out of Scope for GSoC 2026` — Document as future improvement.

---

## 12. GSoC 2026 Mentors Confirmed

**Question:** Who are the confirmed BLT mentors for GSoC 2026?

**Answer (from owasp.org GSoC 2026 ideas page, accessed 2026-06-26):**
- Donnie Brown
- Ahmed ElSheikh
- Manikandan Chandran
- Rinkit Adhana
- Raj Gupta
- Vinamra Vaswani
- Carla Voorhees
- Jigyasu Rajput
- Rishab Kumar Jha
- Akshay Behl

All available on OWASP Slack in #project-blt channel.

**Status:** `Resolved`

---

## 13. Target MCP Client Platforms

**Question:** Which MCP client platforms should be prioritized for integration testing?

**Confirmed targets:**
- Claude Desktop (documented in existing BLT-MCP)
- Cline (documented in existing BLT-MCP)

**Potential targets:**
- ChatGPT (OpenAI has MCP support [OpenAI MCP Guide])
- VS Code extensions (multiple MCP extensions exist)
- Custom Python agents (via MCP Python SDK Client)

**Decision needed:** Which platforms to officially support and test against.

**Status:** `Decision Pending`

---

## 13. BLT-MCP Language Choice

**Question:** Should BLT-MCP be rewritten in Python (as planned in this research) or extended from the existing TypeScript prototype?

**Current arguments:**
- **Python:** Official MCP Python SDK, BLT-API also uses Python/Cloudflare Workers, FastMCP reduces boilerplate
- **TypeScript:** Existing prototype exists (7 stars, 23 forks, JavaScript, last push 2026-03-14, 7 contributors), wider MCP ecosystem

**Existing prototype details (from web search):**
- Language: JavaScript (single `src/index.ts` with Hono framework?)
- Resources: `blt://issues`, `blt://repos`, `blt://contributors`, `blt://workflows`, `blt://leaderboards`, `blt://rewards` (8 total with IDs)
- Tools: `submit_issue`, `award_bacon`, `update_issue_status`, `add_comment` (4 tools)
- Prompts: `triage_vulnerability`, `plan_remediation`, `review_contribution` (3 prompts)
- Auth: OAuth/API Key
- Transport: JSON-RPC 2.0 over stdio or HTTP/SSE
- Dependabot failures indicate maintenance challenges

**Status:** `Decision Pending` — See decision_log.md for analysis.

---

## 14. Existing BLT-MCP Prototype Source Layout

**Question:** What is the existing BLT-MCP TypeScript prototype's architecture?

**Answer (from web search of OWASP-BLT/BLT-MCP README, accessed 2026-06-26):**
- **Language:** JavaScript (TypeScript)
- **Build/run:** `node dist/index.js` (compiled output)
- **Entry:** `src/index.ts`
- **Framework:** Likely Hono (depends on npm, had Dependabot updates for `hono`)
- **3-layer MCP architecture:**
  - Resources: `blt://issues`, `blt://issues/{id}`, `blt://repos`, `blt://repos/{id}`, `blt://contributors`, `blt://contributors/{id}`, `blt://workflows`, `blt://workflows/{id}`, `blt://leaderboards`, `blt://rewards`
  - Tools: `submit_issue` (params: title, description, repo_id, severity, type), `award_bacon`, `update_issue_status`, `add_comment`
  - Prompts: `triage_vulnerability`, `plan_remediation`, `review_contribution`
- **Auth:** OAuth/API Key via env `BLT_API_BASE`, `BLT_API_KEY`
- **Transport:** JSON-RPC 2.0 over stdio or HTTP/SSE
- **Status:** Last push March 2026, Dependabot update failures (Apr 2026)
- **Contributors:** 7 (DonnieBLT, Copilot, snk-git-hub, Kunal241207, azizrebhi, dependabot, srinithivijayakumars139-wq)

**Status:** `Resolved` — Prototype is minimal TypeScript/Hono MCP server with 8 resources, 4 tools, 3 prompts.

---

## 15. BLT-API Programmatic Discovery Usage

**Question:** Should BLT-MCP use the `GET /v2/routes` endpoint for dynamic API discovery, or hardcode endpoint mappings?

**Pros of discovery:**
- Resilient to API changes (new endpoints auto-detected)
- Follows MCP's philosophy of dynamic capability discovery

**Cons of discovery:**
- Routes endpoint returns method+path only (no schemas)
- Additional latency per connection
- BLT-MCP needs semantic understanding that simple route lists don't provide

**Status:** `Decision Pending`

---

## 15. Backend Service for Missing APIs

**Question:** For BLT-MCP resources/tools without BLT-API endpoints (repos, workflows, comments, bacon), what interim strategy should be used?

**Options:**
1. Direct Django backend integration (deprecated architecture)
2. Contribute endpoints to BLT-API
3. Implement stubs until BLT-API adds support
4. Remove from BLT-MCP scope

**Status:** `Decision Pending`
