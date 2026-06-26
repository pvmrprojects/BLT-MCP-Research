# Decision Log

> **Document Version:** 1.0
> **Date:** 2026-06-26
> **Purpose:** Record every architectural assumption with justification and supporting sources
> **Status:** Preliminary decisions based on research phase; subject to revision during implementation

---

## Decision 1: Implementation Language

**Decision:** Build BLT-MCP in Python using the MCP Python SDK.

**Justification:**
- BLT-API (the primary backend) is also Python on Cloudflare Workers — shared language reduces cognitive overhead
- MCP Python SDK provides both FastMCP (high-level decorators) and low-level APIs
- Python's async ecosystem (httpx, asyncio) suits MCP's concurrent request patterns
- The existing TypeScript prototype (7 stars, 23 forks) has no recent commits since March 2026 and appears to be a prototype rather than production code
- BLT's GSoC 2026 page lists Python as a knowledge prerequisite

**Supporting Sources:**
- MCP Python SDK documentation: `py.sdk.modelcontextprotocol.io`
- BLT-API is Python: `github.com/OWASP-BLT/BLT-API`
- GSoC 2026 prerequisites list Python: `owasp.org/www-community/initiatives/gsoc/gsoc2026ideas`

**Risks:**
- MCP Python SDK v2 is in alpha — may need to pin to v1.x during GSoC
- TypeScript has a larger MCP server ecosystem for reference implementations

**Status:** `Confirmed`

---

## Decision 2: MCP SDK Version Targeting

**Decision:** Target MCP Python SDK v1.x (`mcp>=1.27,<2`) for initial implementation.

**Justification:**
- v1.x is stable and in maintenance mode — reliable for production use
- v2 is in alpha with beta target 2026-06-30 and stable target 2026-07-27
- GSoC development starts before v2 stable is guaranteed
- Migration path from v1.x to v2 is documented in the SDK's `docs/migration.md`

**Supporting Sources:**
- MCP Python SDK README: "v1.x is in maintenance mode and continues to receive critical bug fixes and security patches"
- v2 timeline: "beta on 2026-06-30 and a stable v2 on 2026-07-27"
- Recommendation: "add a `<2` upper bound to your version constraint"

**Risks:**
- If v2 releases during GSoC, may need to migrate mid-project
- Project documentation will need updating for v2 APIs

**Status:** `Confirmed` — will re-evaluate at v2 stable release.

---

## Decision 3: MCP Specification Version

**Decision:** Implement the 2025-11-25 specification version.

**Justification:**
- This is the current stable version of the MCP spec
- MCP Python SDK v1.x targets this version
- It includes the authorization framework (OAuth 2.1 for HTTP transport)
- Draft specification features (MRTR, elicitation) have no stable SDK support

**Supporting Sources:**
- MCP Spec 2025-11-25 is the current stable: `modelcontextprotocol.io/specification/2025-11-25`
- Draft spec features are not yet finalized: `modelcontextprotocol.io/specification/draft/basic`

**Status:** `Confirmed`

---

## Decision 4: Architecture Pattern

**Decision:** Use a combination of API Wrapper and Layered Tool Server patterns.

**Justification:**
- BLT-MCP wraps the existing BLT-API REST API (API Wrapper pattern)
- Organizes capabilities into three layers: Resources (data), Tools (actions), Prompts (workflows)
- This matches the GSoC 2026 project description's three-layer architecture
- The Layered Tool Server pattern provides clean separation of concerns
- Production MCP guidance recommends separating data access (Resources) from actions (Tools)

**Supporting Sources:**
- GSoC 2026 description: three-layer Resources/Tools/Prompts: `owasp.org/www-community/initiatives/gsoc/gsoc2026ideas`
- MCP Handbook architecture decision tree: "Wraps external API? → API Wrapper pattern": `github.com/ypollak2/mcp-handbook`
- PADISO: Layered Tool Server pattern: `www.padiso.co/blog/ai-agents-production-mcp-server-design-patterns/`
- Inspired by Frustration: "Resources = nouns, Tools = verbs": `inspiredbyfrustration.com/blog/mcp-server-architecture`

**Status:** `Confirmed`

---

## Decision 5: Transport Selection

**Decision:** Implement both stdio and Streamable HTTP transports.

**Justification:**
- stdio is required for local/desktop integration (Claude Desktop, Cline)
- Streamable HTTP is required for remote/cloud integration (ChatGPT, multi-client)
- The MCP Python SDK natively supports both
- Dual transport adds ~1 week of implementation effort but is required per GSoC spec

**Supporting Sources:**
- GSoC 2026: "JSON-RPC 2.0 over stdio or HTTP/SSE": `owasp.org/www-community/initiatives/gsoc/gsoc2026ideas`
- MCP Python SDK supports "stdio, SSE, and Streamable HTTP": `py.sdk.modelcontextprotocol.io`

**Risks:**
- HTTP transport adds significant security surface area (OAuth, CORS, rate limiting)
- SSE is being deprecated in favor of Streamable HTTP — will not implement SSE

**Status:** `Confirmed`

---

## Decision 6: Primary Backend Interface

**Decision:** Use BLT-API (Cloudflare Workers REST API) as the primary backend interface.

**Justification:**
- BLT-API is the modern, edge-deployed REST API for BLT
- It covers bugs, users, domains, and organizations — the primary BLT-MCP data domains
- It uses a simple API key authentication model that maps well to MCP's environment credential model
- It is the intended backend for all new BLT integrations (per BLT-Next and migration plan)

**Supporting Sources:**
- BLT-API README documents full REST coverage: `github.com/OWASP-BLT/BLT-API`
- BLT MIGRATION_PLAN.md designates BLT-API as the target for all API extraction
- BLT-Next uses BLT-API as its backend: `github.com/OWASP-BLT/BLT-Next`

**Risks:**
- Some BLT-MCP resources/tools may not have BLT-API equivalents (see open_questions.md)
- BLT-API uses a shared static API key — no fine-grained permissions

**Status:** `Confirmed` — with noted gaps for resources without BLT-API endpoints.

---

## Decision 7: Authentication Model

**Decision:** Implement a two-tier authentication model: environment credentials for stdio, OAuth 2.1 with PKCE for HTTP.

**Justification:**
- MCP Spec 2025-06-18 mandates OAuth 2.1 for HTTP transport
- OWASP MCP Security Cheat Sheet requires OAuth 2.0 with PKCE for remote servers
- CSA Guide confirms OAuth 2.1 with PKCE as mandatory for all remote MCP connections
- stdio transport should authenticate via environment variables per spec
- The MCP Python SDK has built-in OAuth support

**Supporting Sources:**
- MCP Spec 2025-06-18 authorization framework: `modelcontextprotocol.io/specification/2025-06-18`
- OWASP Cheat Sheet: "Use OAuth 2.0 with PKCE for remote server authorization"
- CSA Guide: "OAuth 2.1 with PKCE mandatory for all remote MCP connections": `labs.cloudsecurityalliance.org`
- MCP Spec 2025-11-25: OAuth 2.1 as the standard

**Risks:**
- OAuth 2.1 implementation requires an authorization server (separate component)
- For GSoC timeline, may need to implement a simplified auth flow initially

**Status:** `Confirmed` — OAuth for HTTP transport is not optional per spec.

---

## Decision 8: Tool Schema Security

**Decision:** Enforce `additionalProperties: false` on all MCP tool input schemas.

**Justification:**
- Prevents LLMs from inventing parameters not defined in the schema
- Required by OWASP MCP Security Cheat Sheet
- Identified as a top practice by production MCP practitioners
- Mitigates OWASP MCP03:2025 (Tool Poisoning) and MCP05:2025 (Command Injection)

**Supporting Sources:**
- OWASP Cheat Sheet: "Use strict JSON Schema with `additionalProperties: false`"
- Inspired by Frustration: "`additionalProperties: false` is almost always what you want"
- OWASP MCP Top 10 MCP03 (Tool Poisoning): `owasp.org/www-project-mcp-top-10/`

**Status:** `Confirmed`

---

## Decision 9: Guardrails Priority

**Decision:** Invest in Layer 4 guardrails (rate limiting, auth, audit logging, response caps) as a first-class architectural concern, not an afterthought.

**Justification:**
- "Most MCP servers nail layer 3 and punt on layer 4 — the exact layer that matters in production"
- Production MCP servers must handle "Claude hitting your server 60 times a minute at 2am"
- Guardrails are a core differentiator for production-readiness
- OWASP MCP08:2025 (Lack of Audit) and MCP07:2025 (Insufficient Auth) are addressed by guardrails

**Supporting Sources:**
- Inspired by Frustration: Layer 4 importance: `inspiredbyfrustration.com/blog/mcp-server-architecture`
- OWASP MCP Top 10 MCP07, MCP08

**Risks:**
- Adds implementation time (estimated 1-2 weeks)
- May be deprioritized under GSoC timeline pressure

**Status:** `Confirmed` — Documented as Sprint 5 and Sprint 6 in development plan.

---

## Decision 10: Testing Strategy

**Decision:** Use in-memory MCP transport for unit tests, integration tests against mocked BLT-API.

**Justification:**
- MCP Python SDK supports in-memory transport for testing without subprocesses or network
- Mocked BLT-API allows deterministic test scenarios
- Integration tests with real BLT-API for a subset of critical endpoints
- Protocol compliance tests verify correct MCP specification implementation

**Supporting Sources:**
- MCP Handbook: "Use in-memory transport for tests — no network, no subprocess": `github.com/ypollak2/mcp-handbook`
- MCP Python SDK: in-memory transport support

**Status:** `Confirmed`

---

## Decision 11: BLT-API Client Library Approach

**Decision:** Build a lightweight, purpose-built HTTP client for BLT-API rather than using an auto-generated client.

**Justification:**
- BLT-API doesn't have a published OpenAPI schema (uses `GET /routes` instead)
- The BLT-API endpoint surface is manageable (~30 endpoints)
- A purpose-built client allows clean error handling and response mapping specific to MCP needs
- No existing Python client library exists for BLT-API

**Risks:**
- Manual client requires maintenance when BLT-API adds endpoints
- Error handling must be manually implemented

**Status:** `Confirmed`

---

## Decision 12: BLT-MCP Prompt Design Approach

**Decision:** Design prompts as parameterized templates with embedded resource context, supporting chaining.

**Justification:**
- MCP's prompt system supports resource embedding (issue data included in prompt context)
- Prompt chaining enables complex security workflows (triage → plan → remediate)
- BLT-MCP prompts are task templates for security workflows, aligning with BLT's domain
- The GSoC project description specifies three prompts: `triage_vulnerability`, `plan_remediation`, `review_contribution`

**Supporting Sources:**
- MCP Blog — Prompts for Automation: prompt chains, resource embedding: `blog.modelcontextprotocol.io`
- GSoC 2026 Ideas: three prompt definitions: `owasp.org/www-community/initiatives/gsoc/gsoc2026ideas`

**Status:** `Confirmed`

---

## Decision 13: BLT-MCP Resource URI Scheme

**Decision:** Use `blt://` as the URI scheme for all BLT-MCP resources.

**Justification:**
- Already defined in the existing BLT-MCP prototype
- Consistent with MCP's URI-based resource addressing
- Allows clear separation from other MCP servers (file://, http://, etc.)
- Matches GSoC 2026 project description

**Supporting Sources:**
- BLT-MCP existing implementation: `github.com/OWASP-BLT/BLT-MCP`
- GSoC 2026 Ideas: "read-only access to issues... via `blt://` URIs"

**Status:** `Confirmed`

---

## Decision 14: Scope Management

**Decision:** Define MCP tool usage as "Complex Migration" in the Sprint Plan (Sprint 5) rather than deferring indefinitely.

**Justification:**
- Auth is a blocking requirement for HTTP transport per MCP spec
- Tool-level scoping is recommended by OWASP and CSA
- Without proper auth, BLT-MCP cannot be deployed for remote use
- Better to implement correctly from the start than add security retroactively

**Supporting Sources:**
- MCP Spec 2025-06-18: auth framework mandatory for HTTP
- CSA Guide: "Scopes at tool level, not server level"
- OWASP Cheat Sheet: "Use scoped, per-server credentials"

**Risks:**
- Adds complexity to the GSoC timeline
- May need simplified API key auth as interim solution

**Status:** `Confirmed`

---

## Decision 15: Error Strategy

**Decision:** Return descriptive, actionable error messages using standard MCP error codes, with backend error details included in the `data` field.

**Justification:**
- MCP defines standard JSON-RPC error codes (-32700 to -32099)
- LLMs need descriptive error text to correct their behavior
- "Failed to submit issue: BLT-API returned 401 (unauthorized)" is actionable
- Error details in `data` field preserve structured information

**Supporting Sources:**
- MCP Spec 2025-11-25: JSON-RPC error format
- MCP Best Practices: "Return meaningful errors"
- Inspired by Frustration: error handling as Layer 4 guardrail

**Status:** `Confirmed`

---

## Decision Log Summary

| # | Decision | Status | Sprint |
|---|----------|--------|--------|
| 1 | Python implementation language | Confirmed | Sprint 1 |
| 2 | MCP SDK v1.x targeting | Confirmed | Sprint 1 |
| 3 | MCP Spec 2025-11-25 | Confirmed | Sprint 1 |
| 4 | API Wrapper + Layered Tool Server pattern | Confirmed | All |
| 5 | stdio + Streamable HTTP transports | Confirmed | Sprint 1, 5 |
| 6 | BLT-API as primary backend | Confirmed | Sprint 1 |
| 7 | Two-tier auth (env + OAuth 2.1/PKCE) | Confirmed | Sprint 5 |
| 8 | `additionalProperties: false` enforcement | Confirmed | Sprint 3 |
| 9 | Guardrails as first-class concern | Confirmed | Sprint 5, 6 |
| 10 | In-memory transport for tests | Confirmed | Sprint 6 |
| 11 | Purpose-built BLT-API HTTP client | Confirmed | Sprint 1 |
| 12 | Prompts with resource embedding + chaining | Confirmed | Sprint 4 |
| 13 | `blt://` URI scheme | Confirmed | Sprint 2 |
| 14 | Tool-level scoping in Sprint 5 | Confirmed | Sprint 5 |
| 15 | Descriptive MCP error codes | Confirmed | Sprint 1 |
