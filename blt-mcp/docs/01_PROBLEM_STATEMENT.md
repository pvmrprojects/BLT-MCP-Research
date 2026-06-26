# Problem Statement

> **Document Version:** 2.0
> **Status:** Draft
> **Date:** 2026-06-26
> **Author:** BLT-MCP Research & Planning

---

## 1. Introduction

This document defines the problem that the BLT-MCP project aims to solve. It exists to provide a shared understanding of why BLT-MCP is needed, what specific problems it addresses, and how it improves upon the current state of the OWASP BLT ecosystem.

### Motivation

The OWASP Bug Logging Tool (BLT) has grown from a single Django application into an ecosystem of 53+ repositories spanning REST APIs, static frontends, mobile applications, blockchain-based rewards systems, and automation bots [1]. This growth has created a powerful but fragmented platform — one where AI agents, the fastest-growing category of software consumers, cannot autonomously discover or interact with BLT's capabilities.

Simultaneously, the AI industry is converging on the Model Context Protocol (MCP) as the standard interface for AI-tool integration. Anthropic's Claude Desktop has native MCP support [2]. OpenAI's ChatGPT officially supports MCP servers as data sources [3]. Major IDEs and developer tools are adopting MCP for AI-powered workflows [4].

BLT currently has no MCP-compatible interface. This means BLT is invisible to the growing ecosystem of AI agents that discover, evaluate, and integrate services through MCP. Without an MCP server, BLT cannot participate in autonomous security workflows, AI-assisted bug triage, or protocol-driven tool integration.

BLT-MCP addresses this gap by implementing an MCP server that exposes BLT's capabilities through the protocol's three primitives — Resources (data), Tools (actions), and Prompts (workflows) — making BLT accessible to any MCP-compatible AI agent or client [5].

---

## 2. Current State of BLT

### Current Architecture

BLT is in active transition from a monolithic Django architecture to a modern edge-based architecture. The current landscape includes multiple coexisting systems:

```
┌──────────────────────────────────────────────────────────────────┐
│                     BLT Ecosystem (2026)                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────────┐         ┌──────────────────────────┐    │
│  │  BLT (Legacy)        │         │  BLT-API (Modern)        │    │
│  │  ───────────         │         │  ───────────────         │    │
│  │  • Django Monolith   │         │  • Cloudflare Workers    │    │
│  │  • PostgreSQL        │◀────────│  • Python 3.12+          │    │
│  │  • Session-based Auth│         │  • D1 (SQLite) DB        │    │
│  │  • Template views    │         │  • REST API (X-BLT-API-  │    │
│  │  • Legacy API views  │         │    Key auth)             │    │
│  └─────────────────────┘         │  • GET /v2/routes         │    │
│                                  └──────────┬───────────────┘    │
│                                             │                     │
│  ┌─────────────────────┐         ┌──────────┴───────────────┐    │
│  │  BLT-Next (Target)  │         │  BLT-Pages               │    │
│  │  ─────────────      │         │  ──────────              │    │
│  │  • GitHub Pages     │         │  • GitHub Pages          │    │
│  │  • Cloudflare       │         │  • Tailwind CSS          │    │
│  │    Python Workers   │         │  • BLT-API backend       │    │
│  │  • HTMX frontend    │         │  • Anonymous reporting   │    │
│  │  • JWT auth         │         └──────────────────────────┘    │
│  └─────────────────────┘                                         │
│                                                                   │
│  ┌─────────────────────┐         ┌──────────────────────────┐    │
│  │  BLT-Rewards        │         │  BLT-MCP (Target)        │    │
│  │  ───────────        │         │  ──────────────          │    │
│  │  • Bitcoin/Runes    │         │  • MCP Protocol Server   │    │
│  │  • BACON tokens     │         │  • Python/FastMCP        │    │
│  │  • Gamification     │         │  • stdio + HTTP transport│    │
│  └─────────────────────┘         └──────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

[1, 6, 7, 8, 9]

### Existing APIs

BLT exposes its data through three primary API surfaces:

**BLT-API (Cloudflare Workers REST API):**

| Endpoint Group | Methods | Auth Required | Data Store |
|---------------|---------|---------------|------------|
| Auth (`/auth/*`) | POST, GET | No (public) | D1 |
| Bugs (`/bugs/*`) | GET, POST | Yes | D1 |
| Users (`/users/*`) | GET, POST | Yes (partial) | D1 |
| Domains (`/domains/*`) | GET | Yes | D1 |
| Organizations (`/organizations/*`) | GET | Yes | D1 |
| Projects (`/projects/*`) | GET | Yes | D1 |
| Hunts (`/hunts/*`) | GET | Yes | D1 |
| Routes (`/routes`) | GET | No (public) | Router |
| Health (`/health`) | GET | No (public) | — |

[6]

**BLT (Django Legacy API):**

The main BLT application provides Django template views and some legacy REST endpoints. These are being progressively migrated to BLT-API as part of the BLT-Next initiative [7].

**BLT-Rewards API:**

The BACON token system operates as a separate service with its own API, documented only in its source code [9].

### Existing Workflows

Current BLT workflows are primarily human-driven through the web UI:

- **Bug Reporting**: User navigates to website → fills form → submits → moderator reviews → issue published
- **Bug Triage**: Human moderator reviews new issues → assigns severity → labels → notifies organization
- **Bug Bounty**: Organization creates bounty → researcher finds bugs → submits → verified → reward paid
- **Leaderboard**: Automated bot generates leaderboard every 6 hours via GitHub Actions [10]
- **BACON Rewards**: Manual or script-based BACON token distribution for verified contributions [9]

None of these workflows can be invoked programmatically by an AI agent.

### Authentication

BLT services use inconsistent authentication mechanisms:

| Service | Method | Mechanism | Session Model |
|---------|--------|-----------|---------------|
| BLT (Legacy) | Session-based | Django auth, CSRF tokens | Per-user session |
| BLT-API | Static API key | X-BLT-API-Key header | Shared key (no per-user scoping) |
| BLT-Next | JWT | Bearer token in Authorization header | Per-user token |
| BLT-Rewards | Unknown | Undocumented | Unknown |

[6, 11, 7]

The BLT-API authentication model is particularly limiting: a single shared static API key protects all protected endpoints equally, with no per-user or per-organization scoping [11]. This means an MCP server using BLT-API as a backend cannot distinguish between different users' permissions at the API level.

### Current Integrations

BLT currently integrates with:

- **GitHub** via GitHub Actions for leaderboard automation [10]
- **Slack** for notifications [12]
- **Email** via Mailgun for verification and notifications [6]
- **Bitcoin** via the Runes protocol for BACON token minting [9]

These integrations are point-to-point: each one is custom-built, and none can be reused by an AI agent without dedicated implementation.

### Current Developer Experience

Integrating with BLT today requires a developer to:

1. **Read documentation** across multiple sources (BLT README, BLT-API README, source code)
2. **Choose a backend** to integrate with (Django legacy API or BLT-API)
3. **Implement authentication** appropriate to the chosen backend
4. **Write HTTP client code** with proper error handling, pagination, and rate limiting
5. **Test against the live API** or set up a local development environment
6. **Repeat for each AI platform** (Claude Desktop, ChatGPT, custom agents)

This process takes 2-5 days per integration and produces platform-specific code that cannot be reused across AI clients [5].

---

## 3. Current State of AI Agents

### How AI Agents Currently Integrate with Software

AI agents integrate with external software through several established patterns:

```
┌─────────────────────────────────────────────────────────────────────┐
│              AI Agent Integration Patterns (Before MCP)              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐                                                    │
│  │  AI Agent     │───▶ Pattern 1: REST API Calls─────────────────▶│
│  │  (LLM)        │     Custom HTTP client code, manual auth         │
│  │               │                                                  │
│  │  • Claude     │───▶ Pattern 2: OpenAPI/Swagger───────────────▶│
│  │  • ChatGPT    │     Spec → generated client, schema validation   │
│  │  • Custom     │                                                  │
│  │  • IDE Agent  │───▶ Pattern 3: Custom SDK────────────────────▶│
│  └──────────────┘     Language-specific library, abstraction layer  │
│                       │                                              │
│                      ───▶ Pattern 4: Function Calling────────────▶│
│                         Platform-native tool definitions, schemas  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Pattern 1 — REST API Calls:** The developer writes custom code to construct HTTP requests, manage authentication, and parse responses. Each integration is unique and platform-specific.

**Pattern 2 — OpenAPI/Swagger:** The API provides an OpenAPI specification. Tools can auto-generate client libraries. However, AI agents cannot dynamically read and adapt to OpenAPI specs at runtime.

**Pattern 3 — Custom SDKs:** The service provides a language-specific library. This reduces boilerplate but locks the integration to specific languages and versions.

**Pattern 4 — Function Calling:** Platform-native tool definitions (e.g., OpenAI function calling). The developer manually defines tool schemas in JSON Schema format. These schemas are static, hardcoded, and drift from the API over time.

### Typical Workflow

A developer integrating BLT with an AI agent today typically follows this workflow:

1. **Select target AI platform** (Claude Desktop, ChatGPT, custom agent)
2. **Read BLT-API documentation** to understand available endpoints
3. **Write a tool definition** (JSON Schema) for each BLT action:
   - Tool name, description, parameter schema
   - HTTP method, URL, headers, body
   - Response parsing and error handling
4. **Register tools** with the AI platform's tool-use system
5. **Test** the integration with the specific platform
6. **Repeat** for every additional platform

Each step requires manual effort. The tool definitions are static — if BLT-API adds, removes, or changes an endpoint, the tool definitions must be updated manually.

### Current Limitations

#### REST APIs

REST APIs, while ubiquitous, have fundamental limitations when consumed by AI agents:

- **No capability discovery:** An agent cannot ask "what can you do?" and receive a structured answer. REST has no standardized mechanism for runtime capability advertisement [13].
- **No semantic tool descriptions:** REST endpoints describe HTTP operations (`GET /bugs/{id}`), not AI-agent actions ("Retrieve the full details of a specific bug report with all attachments and metadata"). The agent must infer semantics from endpoint names [14].
- **No input schemas at runtime:** REST responses include data, but request schemas are documented separately (README, OpenAPI spec). Agents cannot dynamically discover what parameters an endpoint expects.
- **Inconsistent error models:** REST APIs use HTTP status codes, but the semantics vary by service. A 400 could mean validation failure, rate limiting, or authentication error — an agent cannot reliably distinguish [15].
- **No standardized pagination:** Each API implements pagination differently. An agent must implement custom pagination logic per integration.

BLT-API partially addresses discoverability with its `GET /v2/routes` endpoint, but this returns only HTTP method + path pairs — not semantic capability descriptions, input schemas, or usage patterns that an LLM can reason about [16].

#### OpenAPI

OpenAPI (formerly Swagger) provides structured API descriptions, but has limitations for AI agents:

- **Static specifications:** An OpenAPI spec is a document, not a live protocol. The agent cannot re-fetch the spec to discover runtime changes.
- **No workflow abstraction:** OpenAPI describes individual endpoints, not multi-step workflows.
- **No prompt templates:** OpenAPI has no concept of reusable task templates.
- **Authentication coupling:** OpenAPI specs embed auth requirements, but the agent cannot negotiate credentials at runtime.
- **Schema size:** Large specs can exceed LLM context windows, forcing developers to subset the API.

#### Custom SDKs

Custom SDKs provide abstraction but introduce their own problems:

- **Language lock-in:** The SDK targets specific languages (Python, JavaScript, etc.). An agent built in language X cannot use SDK designed for language Y.
- **Version drift:** SDK versions lag behind API changes. The agent may use an outdated SDK with incorrect assumptions.
- **Maintenance burden:** The service provider must maintain SDKs across multiple languages, each with different release cycles.

#### Tool Calling

Platform-native tool calling (OpenAI function calling, Anthropic tool use) is the closest predecessor to MCP:

- **Static registration:** Tool definitions are registered at development time, not discovered at runtime.
- **Platform-specific format:** Each platform has its own tool definition format. A tool defined for OpenAI cannot be reused with Anthropic.
- **No workflow chaining:** Tool definitions describe individual functions, not coordinated multi-step workflows.
- **Documentation drift:** Tool descriptions in agent configs fall out of sync with API changes.

#### Context Limitations

Beyond protocol issues, AI agents face fundamental context window constraints:

- **Documentation size:** Comprehensive API documentation can exceed an agent's context window. An agent cannot hold BLT's full API surface in its active context.
- **Tool count limits:** Platforms limit the number of tools registered simultaneously (commonly 128-256). Large APIs may exceed this limit.
- **Response truncation:** Large responses (paginated lists, full bug reports) may exceed context limits, requiring the agent to implement its own truncation logic.

---

## 4. Core Problem

The core problem is a **protocol gap**: the OWASP BLT ecosystem exposes its capabilities through multiple, inconsistent, non-discoverable interfaces that AI agents cannot autonomously consume.

### Why Is BLT Difficult for AI Agents to Use?

1. **No single interface:** BLT's capabilities are spread across the Django legacy app, BLT-API, BLT-Next, BLT-Rewards, and BLT-Pages. An agent integrating with one has no access to the others [1].

2. **No standardized discovery:** An agent connecting to BLT has no way to ask "what can you do?" and receive a structured, machine-readable answer. The agent must know endpoint URLs, authentication details, and parameter schemas in advance [13].

3. **No workflow abstractions:** Common security workflows (triage a vulnerability, plan remediation, review a contribution) exist as human-driven processes in the web UI, not as programmable abstractions [5].

4. **Inconsistent authentication:** Different BLT services use different authentication mechanisms (session-based, static API key, JWT). An agent must implement all three and know which to use for each operation [6, 7, 11].

5. **No standardized error handling:** BLT-API returns HTTP errors with varying semantics. An agent must implement custom error handling per endpoint group.

### Why Are REST APIs Insufficient?

REST APIs were designed for human developers reading documentation and writing integration code, not for AI agents discovering capabilities at runtime. The specific insufficiencies are:

- **REST describes endpoints, not capabilities:** `GET /bugs/{id}` returns a bug's data, but does not describe that this capability can be used for "checking the current status of a reported vulnerability" — the semantic understanding an agent needs [14].

- **REST has no tool contract:** An MCP tool definition includes name, description, input schema, and expected behavior. A REST endpoint only has method + path + optional response schema.

- **REST has no workflow abstraction:** Complex operations require chaining multiple REST calls. There is no standardized way to express "triage a vulnerability" as a reusable sequence.

- **REST authentication is per-request, not per-session:** MCP's session-based model allows capability negotiation once at connection time. REST requires authentication on every request, with no session-level capability agreement.

BLT-API's `GET /v2/routes` endpoint [16] demonstrates the need for programmatic discovery, but returns only method + path pairs — not the semantic descriptions, parameter schemas, or usage patterns that an LLM needs to reason about tool selection.

### Why Do AI Systems Require Standard Interfaces?

AI systems operate differently from traditional software consumers:

- **No human-in-the-loop for integration:** An AI agent must autonomously discover, understand, and use capabilities without a developer reading documentation and writing glue code.
- **Runtime adaptivity:** An AI agent may encounter new services, updated APIs, or different deployment environments. It must adapt without recompilation or redeployment.
- **Cross-platform portability:** An agent's tool knowledge should transfer across platforms. A tool for BLT learned in Claude Desktop should work in ChatGPT without redefinition.

Standard protocols like MCP satisfy these requirements by providing:
- **Runtime capability discovery** (agents learn what's available at connection time)
- **Self-describing interfaces** (tools carry their own schemas and descriptions)
- **Cross-platform transport** (stdio for local, HTTP for remote)

### What Friction Exists Today?

Every integration point between an AI agent and BLT introduces friction:

| Integration Step | Friction | Consequence |
|-----------------|----------|-------------|
| Discover capabilities | Manual documentation review | Hours of human effort per integration |
| Understand authentication | Multi-service auth reverse engineering | Security risk from misconfigured auth |
| Write tool definitions | Manual JSON Schema authoring | Maintenance burden, documentation drift |
| Implement error handling | Per-endpoint custom logic | Inconsistent agent behavior |
| Handle pagination | Custom loop implementation | Incomplete data, excessive API calls |
| Test integration | Platform-specific testing | No portable test suite |
| Maintain over time | Manual updates for API changes | Integration rot, silent failures |

Total: an estimated 2-5 days of developer effort per AI platform per integration [5].

---

## 5. Root Cause Analysis

### Root Cause 1: No Standard Protocol for AI-Agent Integration

BLT was built before the emergence of protocol-based AI integration. Its interfaces (Django views, REST API) were designed for human developers, not AI agents. There is no MCP-compatible interface in the BLT ecosystem.

**Evidence:** BLT has been in development since 2012, predating MCP (announced 2024) by over a decade. Its existing interfaces reflect pre-AI-era design patterns [12].

### Root Cause 2: No Discoverable Capabilities

BLT services do not advertise their capabilities in a machine-readable format. An AI agent cannot query "what actions are available?" and receive a structured response with tool names, descriptions, parameter schemas, and expected behaviors.

**Evidence:** BLT-API's `GET /v2/routes` returns only `{ method: "GET", path: "/bugs" }` — insufficient semantic context for an LLM to reason about tool purpose and usage [16].

### Root Cause 3: No Standardized Prompts for Reusable Workflows

Security workflows (triage, remediation, review) are currently manual processes embedded in the web UI. There are no reusable, parameterized workflow templates that an AI agent can invoke with specific context.

**Evidence:** The GSoC 2026 project description explicitly identifies `triage_vulnerability`, `plan_remediation`, and `review_contribution` as prompts that BLT-MCP should provide, indicating these do not exist today [5].

### Root Cause 4: Authentication Fragmentation

Each BLT service implements its own authentication. An integrator must manage multiple credential types with no unified identity layer.

**Evidence:** BLT-API uses static API key (X-BLT-API-Key) [11], BLT main uses Django session auth [10], BLT-Next uses JWT [7]. These are fundamentally different authentication models.

### Root Cause 5: No Machine-Readable Capability Descriptions

BLT's capabilities are documented in human-readable README files, not in machine-parseable capability descriptors. An AI agent cannot programmatically determine what BLT can do.

**Evidence:** BLT-API's endpoints are documented in a Markdown table in the README [6]. No OpenAPI specification, no JSON Schema endpoint, no machine-readable capability registry exists.

### Root Cause 6: API Fragmentation Across Services

BLT's capabilities are distributed across multiple services with different interfaces, data formats, and base URLs. There is no unified entry point.

**Evidence:** The OWASP-BLT GitHub organization contains 53+ repositories, each potentially exposing its own API surface [1].

### Root Cause 7: Documentation Inconsistency

Documentation across BLT services uses different formats, levels of detail, and update cadences. There is no single source of truth for what BLT can do and how to integrate with it.

**Evidence:** BLT-API has README-based docs [6], BLT has its own README + GH Pages docs [10, 17], BLT-docs aggregates content from multiple sources [17].

### Root Cause 8: Manual Orchestration of Multi-Step Workflows

BLT workflows that span multiple actions (e.g., find a bug → assess severity → notify owner → track remediation) must be manually orchestrated by a developer or human operator. There is no programmable workflow abstraction.

**Evidence:** No webhook, no workflow API, no event-driven mechanism exists for chaining BLT operations into automated processes.

---

## 6. Stakeholder Pain Points

### Developers

- **Wasted time on integration boilerplate:** Each AI platform integration requires 200-500 lines of custom code for HTTP client setup, authentication, error handling, and pagination [5].
- **Documentation spelunking:** Finding the correct endpoint, parameters, and auth mechanism requires reading multiple README files and PR descriptions.
- **No reusable patterns:** Code written for Claude Desktop integration cannot be reused for ChatGPT or custom agents.

### Maintainers

- **Integration maintenance burden:** Every API change potentially breaks downstream AI agent integrations. Without a protocol layer, there is no version negotiation mechanism.
- **Documentation drift:** Tool descriptions in agent configs fall out of sync with API changes. Maintaining accuracy across all integrations is impossible.
- **Ecosystem fragmentation:** With 53+ repositories, maintaining consistent integration surfaces is a growing challenge [1].

### Researchers (Security Analysts)

- **Manual triage workload:** Vulnerability triage is currently a manual process through the web UI, consuming hours per issue.
- **No AI-assisted workflows:** Researchers cannot delegate repetitive triage tasks to AI agents because there is no protocol interface for autonomous action.
- **Cross-platform friction:** A researcher might use Claude Desktop for analysis, ChatGPT for report writing, and a custom IDE for code review — each platform requires a separate BLT integration.

### AI Agents

- **No autonomous discovery:** The agent cannot discover BLT's capabilities at runtime. It requires pre-configured tool definitions that are static and version-specific.
- **No standardized error handling:** The agent cannot reliably distinguish error types (validation failure vs. auth failure vs. rate limiting) without custom logic per endpoint.
- **No workflow templates:** Common tasks (triage, remediation planning) cannot be invoked as parameterized templates.

### Security Engineers

- **No audit trail for AI actions:** Without a protocol layer, AI agent actions are invisible in BLT's audit logs. There is no record of which agent performed which action.
- **No tool-level access control:** BLT-API's shared API key provides no per-tool or per-user authorization. An agent with the API key can access everything.
- **Credential exposure risk:** API keys embedded in agent configurations or environment variables are vulnerable to exposure through model output.

### Open Source Contributors

- **High barrier to entry:** Understanding BLT's full integration surface requires reading across multiple repositories and documentation sources.
- **Limited impact radius:** Contributors to one service cannot easily extend capabilities accessible through other services.
- **No playground for experimentation:** There is no sandboxed, documented interface for experimenting with BLT's capabilities programmatically.

### Organizations (Using BLT for Bug Bounties)

- **No automated security workflows:** Organizations cannot set up automated triage pipelines that use AI agents to pre-screen vulnerability reports.
- **Limited integration options:** Organizations using internal AI tools must build custom integrations rather than connecting through a standard protocol.
- **Scaling challenges:** As bug bounty programs grow, the lack of automated, AI-accessible interfaces becomes a bottleneck for vulnerability management.

---

## 7. Business Impact

### Productivity Loss

- Each AI platform integration requires 2-5 days of developer effort [5].
- Cross-platform integrations compound: supporting Claude Desktop + ChatGPT + a custom agent requires 6-15 days.
- Manual triage of vulnerability reports consumes analyst hours that could be reduced through AI-assisted workflows.

### Maintenance Cost

- Tool definitions in agent configs must be manually updated when BLT-API endpoints change.
- API changes in any of BLT's 53+ repositories can break downstream integrations [1].
- Without a protocol layer, there is no automated version negotiation — integrations break silently until a user reports the failure.

### Onboarding Complexity

- New contributors must understand BLT's architecture across multiple services before they can build integrations.
- The learning curve spans Django patterns, Cloudflare Workers, REST API design, and multiple authentication models.

### Contributor Friction

- Without standardized tool definitions, contributors cannot easily build new integrations.
- The effort-to-impact ratio is unfavorable: a contributor might spend days building an integration that only works for one AI platform.

### Automation Limitations

- Bug triage, remediation tracking, and contribution review cannot be automated because there is no programmable workflow interface.
- BLT's gamification features (BACON rewards, leaderboards) are not accessible to AI agents, limiting the potential for automated reward distribution.

### Scalability Issues

- As BLT grows more services, the fragmentation problem compounds. Each new service adds another integration surface.
- Without a protocol layer, the cost of maintaining integrations grows linearly (or worse) with the number of services.
- The GSoC 2026 initiative adds multiple new services (BLT-Preflight, BLT-NetGuardian, BLT-University, BLT-Rewards) — each will need its own AI integration without a unified protocol [5].

---

## 8. Technical Challenges

### Protocol Mismatch

BLT exposes REST APIs with HTTP semantics (GET/POST/status codes). MCP uses JSON-RPC 2.0 with request/response/notification semantics. Translating between these protocols requires:

- Mapping HTTP methods to JSON-RPC methods
- Converting HTTP status codes to MCP error codes
- Translating REST pagination (page/per_page) to MCP resource pagination
- Handling session management (present in MCP, absent in stateless REST)

### Authentication

| Challenge | Description | Impact |
|-----------|-------------|--------|
| Token management | BLT-API uses a single shared static key. MCP requires per-user authentication for HTTP transport [13]. | Cannot implement proper OAuth 2.1 without per-user credentials |
| Credential storage | API keys must be stored securely, not in environment variables accessible to the LLM [18]. | Requires credential vault or secrets manager |
| Auth negotiation | MCP's OAuth 2.1 flow requires an authorization server. BLT has none. | Must implement or integrate one |

### Tool Discovery

REST endpoints must be mapped to MCP tool definitions with appropriate names, descriptions, parameter schemas, and return schemas. Each mapping requires:

- Semantic name that conveys tool purpose to an LLM
- Description that clarifies behavior and edge cases
- JSON Schema input validation with `additionalProperties: false` [19]
- Return schema that agents can parse

### Resource Discovery

BLT data (issues, users, domains) must be exposed through MCP resource URIs (`blt://issues/{id}`). Challenges include:

- URI scheme design (which entities get their own URI pattern?)
- Pagination of resource lists (how to handle client-side iteration?)
- Content type selection (JSON, HTML, or domain-specific formats?)
- Resource subscription support (listChanged notification pattern)

### State Management

MCP sessions maintain state across the connection lifecycle. REST APIs are stateless. BLT-MCP must:

- Maintain session context (authenticated identity, negotiated capabilities)
- Cache frequently accessed data to reduce BLT-API calls
- Handle session expiry and reconnection

### Error Handling

| REST Error | MCP Error Code | Challenge |
|------------|---------------|-----------|
| 400 Bad Request | -32602 (Invalid Params) | Must parse BLT-API error responses |
| 401 Unauthorized | -32001 (Auth Error) | Must distinguish from 403 |
| 403 Forbidden | -32001 (Auth Error) | Must carry authorization context |
| 404 Not Found | -32602 (Invalid Params) | Cannot use JSON-RPC -32004 (Method Not Found) |
| 429 Rate Limited | -32005 (Rate Limited) | MCP has no standard rate limit error code |
| 500 Internal Error | -32603 (Internal Error) | Must avoid leaking internals |

### Security

BLT-MCP inherits security considerations from both BLT and MCP:

- **OWASP MCP01 (Token Mismanagement):** BLT-API keys must not be exposed to the LLM. The MCP server manages keys, not the AI model [20].
- **OWASP MCP03 (Tool Poisoning):** Tool descriptions are a prompt injection vector. Each description must be vetted for safety [21].
- **OWASP MCP07 (Insufficient Auth):** BLT-API's shared key provides no per-user scoping. MCP must implement its own authorization layer [22].
- **OWASP MCP08 (Lack of Audit):** All tool invocations must be logged with identity and timestamp [23].

### Versioning

- BLT-API evolves independently of BLT-MCP. BLT-MCP must handle API changes gracefully.
- MCP SDK v1.x vs v2 migration: v2 stable targets 2026-07-27, which may fall during GSoC development [24].
- MCP specification version negotiation: client-initiated, server must support the negotiated version [13].

### Compatibility

- BLT-MCP must work with multiple MCP client implementations (Claude Desktop, Cline, ChatGPT) that may implement different subsets of the specification.
- Each client may support different transports, auth mechanisms, and capability sets.
- The shared static API key model limits multi-tenant deployment scenarios.

---

## 9. Why MCP Solves This

### Resources (Data Access)

MCP Resources provide read-only access to data via addressable URIs. For BLT, this means:

- Issues, contributors, repositories, workflows, leaderboards, and rewards are exposed through `blt://` URIs [5]
- An agent reads `blt://issues/123` to get a specific bug's full details
- Resource templates like `blt://issues/{id}` provide parameterized access patterns
- Resources can support subscription notifications via `listChanged` events

This addresses the **no standardized read interface** problem. Instead of each agent implementing its own HTTP client for BLT-API endpoints, the MCP server translates `blt://` URIs into the appropriate REST calls automatically.

### Tools (Actions)

MCP Tools provide executable functions with typed input schemas. For BLT, this means:

- `submit_issue` — report a new bug with title, description, severity, type
- `award_bacon` — distribute BACON tokens to contributors
- `update_issue_status` — change issue status (open, in_progress, resolved, closed)
- `add_comment` — add comments to existing issues

[5]

Each tool includes:
- A **name** that clearly identifies its purpose
- A **description** that explains when and how to use it
- An **input schema** (JSON Schema 2020-12) that defines exactly what parameters are expected
- **Return semantics** that describe what the agent should expect

This addresses the **no standardized action interface** problem. Tool definitions are self-describing and available at runtime — no documentation lookup required.

### Prompts (Workflows)

MCP Prompts provide reusable, parameterized workflow templates. For BLT, this means:

- `triage_vulnerability` — guides the agent through severity assessment, duplicate checking, component identification, priority assignment, and stakeholder notification [5]
- `plan_remediation` — creates a remediation plan with steps, timeline, and resource requirements
- `review_contribution` — evaluates a contribution with quality metrics and BACON recommendations

Prompts can embed resource context. For example, `triage_vulnerability` can include the issue data from `blt://issues/{id}` directly in the prompt, giving the agent full context without additional fetches [25].

This addresses the **no reusable workflow abstraction** problem. Common security workflows become parameterized prompt templates that agents invoke as needed.

### JSON-RPC 2.0

MCP uses JSON-RPC 2.0 as its message format — a standardized, language-agnostic protocol for remote procedure calls. This provides:

- **Consistent message structure:** All messages follow `{ jsonrpc: "2.0", method, params, id }` format
- **Request/response semantics:** Standardized mapping of requests to responses
- **Notifications:** One-way messages that don't require responses (used for lifecycle events)
- **Error codes:** Standardized error categories that agents can reason about [13]

### Capability Discovery

MCP's initialization phase requires the client and server to exchange capability declarations:

```
Client:  { capabilities: { roots: {}, sampling: {}, experimental: {} } }
Server:  { capabilities: { prompts: {}, resources: {}, tools: {}, logging: {} } }
```

[26]

This means an agent connecting to BLT-MCP immediately learns:
- Which tools are available (with full schemas)
- Which resources can be read (with URI patterns)
- Which prompt templates can be invoked (with arguments)
- What protocol version is supported
- What logging capabilities exist

This is the fundamental shift from static, documented integration to dynamic, protocol-driven discovery.

### Standard Interfaces

MCP provides a protocol that works across all MCP-compatible clients:

- **Claude Desktop** [2]: Connect via stdio transport, configure in mcpServers JSON
- **ChatGPT** [3]: Connect via Streamable HTTP with OAuth 2.1
- **Cline** [5]: Connect via stdio transport with environment credentials
- **Custom agents** [24]: Connect via the MCP Python SDK Client

One server, multiple clients. No platform-specific code required.

### Reusable Integrations

An MCP server's capabilities are reusable across all MCP-compatible hosts:

- A tool definition written once serves Claude Desktop, ChatGPT, and any custom MCP client
- Resource URI patterns are consistent across clients
- Prompt templates work identically regardless of which client invokes them

### Multi-Client Support

MCP supports multiple concurrent clients through its transport options:

- **stdio:** One client per process (local desktop usage)
- **Streamable HTTP:** Multiple concurrent clients (cloud deployment) [24]
- **SSE (legacy):** One-to-many streaming (being deprecated)

BLT-MCP's dual transport support (stdio + Streamable HTTP) means it can serve both local developer tools and cloud-hosted AI platforms from the same codebase.

---

## 10. Expected Benefits

### Technical Benefits

| Benefit | Description | Source |
|---------|-------------|--------|
| Protocol-level capability discovery | Agents learn BLT capabilities at connection time, not from static docs | [13] |
| Self-describing tool schemas | JSON Schema 2020-12 for all tool inputs, available at runtime | [24] |
| Standardized error handling | Consistent MCP error codes across all BLT operations | [13] |
| Resource subscription support | Agents can subscribe to changes (listChanged) instead of polling | [13] |
| Dual transport architecture | stdio for local, HTTP for remote — same codebase, different config | [24] |

### Operational Benefits

| Benefit | Metric | Source |
|---------|--------|--------|
| Reduced integration time | 95%+ reduction (2-5 days → 15 minutes) | [5] |
| Eliminated documentation drift | Server is source of truth for tool definitions | [13] |
| Centralized audit logging | All tool invocations logged with identity and timestamp | [23] |
| Unified authentication | Single OAuth 2.1 flow for all BLT operations | [27] |
| Cross-platform portability | One configuration works with any MCP-compatible client | [13] |

### Community Benefits

- **Lower barrier to entry:** Contributors can integrate BLT with any MCP-compatible client using a single configuration
- **Ecosystem growth:** BLT becomes visible from the entire MCP ecosystem (Claude, ChatGPT, IDEs, custom agents)
- **Reusable knowledge:** Community members share MCP configurations, prompt templates, and integration examples
- **GSoC alignment:** BLT-MCP synergizes with other 2026 projects (BLT-Preflight, BLT-NetGuardian, BLT-University) [5]

### Research Benefits

- **Reproducible workflows:** Prompt templates encode security triage and remediation methodologies that researchers can study and improve
- **Automated analysis:** AI agents can process vulnerability reports at scale, enabling pattern analysis across large datasets
- **Auditable decisions:** Every automated triage decision is recorded, enabling root cause analysis and methodology refinement

### Performance Benefits

- **Reduced latency:** In-memory MCP transport for local agents eliminates HTTP overhead
- **Caching opportunities:** Session-level caching reduces repeated BLT-API calls
- **Batch operations:** MCP tools can implement batch operations that reduce round trips

### Developer Experience Benefits

- **Single configuration:** A JSON block in `mcpServers` replaces 200-500 lines of integration code [5]
- **Automatic capability updates:** Tool definitions update automatically when the server is restarted
- **Built-in error handling:** MCP's standardized error codes eliminate custom error handling logic
- **Local development:** stdio transport enables local testing without network or authentication setup

### AI Experience Benefits

- **Natural language interaction:** Agents reason about tool purpose through descriptions, not endpoint names
- **Guided workflows:** Prompt templates provide structure for complex multi-step tasks
- **Context-aware operations:** Resource embedding in prompts gives agents full context without additional fetches [25]
- **Autonomous capability discovery:** Agents adapt to new capabilities as the server evolves

---

## 11. Success Metrics

### Integration Metrics

| Metric | Current State | Target | Measurement Method |
|--------|---------------|--------|-------------------|
| Time to integrate new AI platform | 2-5 days | < 1 hour | Developer time tracking |
| Lines of integration code per platform | 200-500 lines | < 30 lines (config only) | Line count |
| Number of AI platforms supported | 0 (without custom code) | 3+ (Claude Desktop, ChatGPT, Cline) | Verification testing |
| Tool definition staleness | Drifts with API changes | Zero drift (server is source of truth) | Automated comparison |

### Capability Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Number of MCP resources | 8+ `blt://` URIs | Resource list enumeration |
| Number of MCP tools | 4+ tools | Tool list enumeration |
| Number of MCP prompts | 3+ prompt templates | Prompt list enumeration |
| Resource URI templates | Parameterized URIs for dynamic resources | Template pattern verification |
| Tool parameter schemas | JSON Schema 2020-12 with `additionalProperties: false` | Schema validation audit |

### Quality Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| MCP protocol compliance | 100% of required spec features | Compliance test suite |
| Test coverage | >80% for core server logic | Coverage report |
| Response latency (simple queries) | < 5 seconds | Performance benchmarking |
| Error message actionability | All errors include descriptive text | Manual review |
| Documentation coverage | All resources, tools, prompts documented | Documentation audit |

### Developer Experience Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Claude Desktop setup time | < 30 minutes | Step-by-step verification |
| ChatGPT integration time | < 1 hour | Step-by-step verification |
| Custom agent setup time | < 1 hour | Step-by-step verification |
| Sample configurations provided | 3+ (Claude Desktop, Cline, ChatGPT) | Documentation audit |

### Security Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| OWASP MCP Top 10 coverage | 5+ risks specifically mitigated | Security review |
| Tool schema validation | `additionalProperties: false` on all tools | Schema audit |
| Audit logging completeness | Every tool call logged with identity + timestamp | Log audit |
| Credential exposure | Zero secrets in source code | Secret scanning |
| Authentication enforcement | All protected operations require credentials | Auth bypass testing |

---

## 12. Risks

### Technical Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|------------|
| TR1 | **BLT-API endpoint gaps** for resources/tools not backed by documented REST endpoints (repos, workflows, comments, BACON) | High | High | Investigate alternative backends; contribute endpoints if in scope; document as known limitations. [6] |
| TR2 | **MCP Python SDK v2 breaking changes** during GSoC timeline (stable target 2026-07-27) | Medium | High | Pin to `mcp>=1.27,<2`. Document migration path. Monitor v2 release notes. [24] |
| TR3 | **BLT-API rate limiting** not aligned with MCP server rate limits | Medium | Medium | Make MCP rate limits configurable; start conservative; tune based on testing. |
| TR4 | **D1 database latency** for complex queries affecting MCP response times | Low | Medium | Implement caching layer in BLT-MCP; set appropriate timeout values. |
| TR5 | **Shared API key limitation** prevents OAuth 2.1 scoping at BLT-API level | Confirmed | Medium | MCP-level auth decouples from BLT-API; implement tool-level scoping in BLT-MCP itself. [11, 22] |

### Project Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|------------|
| PR1 | **GSoC timeline pressure** — 14 weeks for all deliverables (Resources, Tools, Prompts, Auth, Docs) | Medium | High | Prioritize core functionality; defer non-critical features to future scope. |
| PR2 | **Repository access issues** — GitHub blocked from development environment | Confirmed | Medium | Use web search and PR descriptions for source inspection; contact maintainers for access. |
| PR3 | **Scope creep** — extending BLT-MCP beyond MCP server into BLT backend modifications | Medium | Medium | Strictly enforce "out of scope" section; contribute backend changes separately if needed. |
| PR4 | **Mentor availability** — 10 mentors listed but may have limited availability | Low | Low | Document decisions for async review; use OWASP Slack for communication. |

### Security Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|------------|
| SR1 | **Tool description poisoning** — malicious or misleading tool definitions could manipulate AI behavior | Medium | High | Follow OWASP MCP Security Cheat Sheet: vet descriptions, use `additionalProperties: false`, pin tool definitions. [19, 21] |
| SR2 | **BLT-API key exposure** through LLM output (model leaks credential to user) | Medium | High | MCP server manages keys, not exposed to model; use short-lived scoped tokens. [20] |
| SR3 | **Insufficient audit logging** — AI agent actions leave no trace in BLT systems | Medium | Medium | Implement comprehensive audit logging at MCP layer per OWASP MCP08:2025. [23] |
| SR4 | **SSRF through BLT-API proxying** — agent may request arbitrary URLs | Low | Medium | Domain allowlist filtering; BLT-API URL validation (already implemented per PR #35). [28] |

### Adoption Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|------------|
| AR1 | **MCP ecosystem immaturity** — clients may implement different spec subsets | Medium | Medium | Test against Claude Desktop, Cline, and ChatGPT; document known compatibility. |
| AR2 | **Low community adoption** — BLT community may not use the MCP server | Low | Low | Lower integration barrier encourages adoption; target power users first. |
| AR3 | **Competing protocol emergence** — alternative to MCP may gain adoption | Low | High | Protocol-agnostic core design; modular interface could support future protocols. |

### Maintenance Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|------------|
| MR1 | **BLT-API breaking changes** post-GSoC break BLT-MCP | Medium | Medium | Version pinning; test suite catches regressions. |
| MR2 | **MCP specification evolution** — 2025-11-25 may be superseded | Low | Medium | Monitor spec changes; adopt new versions on stable release. |
| MR3 | **Dependency maintenance** — Python SDK, httpx, pydantic version updates | Low | Low | Automated dependency updates; CI/CD testing for regressions. |

---

## 13. Constraints

### Budget

- **Zero monetary budget:** GSoC provides a stipend to the contributor, but there is no project budget for infrastructure, commercial services, or paid tools.
- **Consequence:** All infrastructure must use free tiers (GitHub Actions, Cloudflare Workers free tier, open-source tools).
- **Mitigation:** Python, FastMCP, httpx, pytest are all open-source. Cloudflare Workers has a generous free tier sufficient for development and testing.

### Time

- **GSoC 2026 timeline:** Approximately 14 weeks of development (June-September 2026).
- **Estimated effort:** 350 hours for a Large GSoC project [5].
- **Consequence:** Scope must be carefully managed. Not all desired features can be delivered in 14 weeks.
- **Mitigation:** 9-sprint plan with clear priorities: Foundation → Resources → Tools → Prompts → Auth → Testing → Docs → Release.

### Technology

- **Language constraint:** Python 3.12+ (BLT-API uses Python, MCP Python SDK is the primary SDK) [6, 24].
- **SDK constraint:** MCP Python SDK v1.x (stable) with planned v2 migration consideration.
- **Backend constraint:** Must interface with BLT-API as currently deployed — no backend modifications allowed.
- **Transport constraint:** Must support both stdio and Streamable HTTP per GSoC description [5].

### Compatibility

- **Must work with:** Claude Desktop (primary target), Cline, ChatGPT (secondary target).
- **Must implement:** MCP specification 2025-11-25 (current stable).
- **Must support:** JSON-RPC 2.0, capability negotiation, lifecycle management.
- **Must handle:** BLT-API's shared static API key model and its limitations.

### Backward Compatibility

- **Existing BLT-MCP prototype:** TypeScript/JavaScript implementation with 8 resources, 4 tools, 3 prompts. The Python implementation should maintain equivalent capability coverage [29].
- **No breaking changes to BLT-API:** BLT-MCP is a consumer of BLT-API, not a modification to it.
- **No breaking changes to the BLT ecosystem:** BLT-MCP adds a new interface without modifying existing ones.

### Security

- **Must address:** OWASP MCP Top 10 risks MCP01 (Tokens), MCP03 (Tool Poisoning), MCP07 (Auth), MCP08 (Audit) [20].
- **Must follow:** OWASP MCP Security Cheat Sheet guidelines [19].
- **Must implement:** OAuth 2.1 with PKCE for HTTP transport per MCP spec [27].
- **Must not:** Expose secrets to the LLM, embed credentials in source code, allow unauthorized tool invocation.

### Open Source

- **License:** AGPL-3.0 (inherited from the existing BLT-MCP prototype and BLT-API) [29, 6].
- **Development:** Public repository under OWASP-BLT GitHub organization.
- **Governance:** OWASP Foundation open-source governance model.
- **Contribution:** Standard open-source contribution workflow with PR reviews and CI/CD.

### GSoC Requirements

- **Must be:** A Large project (350 hours) [5].
- **Must produce:** Production-grade, fully functional feature.
- **Must align:** With BLT's core security and performance goals.
- **Must follow:** Best practices including security testing, CI/CD integration, and documentation.
- **Evaluation:** Mid-term and final evaluations by mentors.

---

## 14. Assumptions

### Assumption 1: BLT-API Remains Available

**Statement:** BLT-API will remain the primary backend for BLT-MCP throughout GSoC development and beyond.

**Reasoning:** BLT-API is an active project (last push March 2026, recent PRs for API key auth and route discovery). The BLT-Next migration designates BLT-API as the modern backend. No deprecation signals exist [6, 7].

### Assumption 2: MCP Python SDK v1.x Is Production-Suitable

**Statement:** MCP Python SDK v1.x provides sufficient stability and features for production deployment.

**Reasoning:** v1.x is in maintenance mode with critical bug fixes and security patches. It implements the full 2025-11-25 specification. The SDK is used in production by multiple MCP servers [24].

### Assumption 3: MCP Specification 2025-11-25 Is Stable

**Statement:** The MCP specification (2025-11-25) will remain the stable version throughout GSoC development.

**Reasoning:** The specification follows semantic versioning. Draft features are explicitly marked as draft. No breaking changes are expected to the stable specification [13].

### Assumption 4: BLT-API Shared Key Is Sufficient

**Statement:** BLT-API's shared static API key model is sufficient for GSoC-scale deployment.

**Reasoning:** The key authenticates the BLT-MCP server to BLT-API. Multi-tenant authentication is handled at the MCP layer (OAuth 2.1 for HTTP, environment keys for stdio). The shared key limitation only affects multi-tenant BLT-API access, which is out of scope [11].

### Assumption 5: Existing TypeScript Prototype Is a Reference

**Statement:** The existing TypeScript BLT-MCP prototype serves as a functional reference but the Python implementation may differ in architecture.

**Reasoning:** The TypeScript prototype (7 stars, 23 forks, last push March 2026) is a minimal implementation with a single `src/index.ts` entry point. The Python implementation will use FastMCP decorators and follow BLT-API's Python conventions [29, 6].

### Assumption 6: BLT-Next Is the Target Architecture

**Statement:** The BLT-Next architecture direction (static frontend + Cloudflare Workers) represents the long-term BLT platform.

**Reasoning:** BLT-Next is a GSoC 2026 project actively migrating BLT from Django to edge architecture. BLT-API (also Cloudflare Workers) supports this direction. BLT-MCP aligns with this target [7, 6].

### Assumption 7: MCP Client Implementations Are Compatible

**Statement:** Major MCP client implementations (Claude Desktop, Cline, ChatGPT) implement a compatible subset of the MCP specification.

**Reasoning:** All three platforms claim MCP support. Claude Desktop and Cline use the same MCP SDKs under the hood. ChatGPT's MCP support follows the same specification [2, 3, 24].

### Assumption 8: BLT-API Availability for Testing

**Statement:** The live BLT-API at `https://api.owaspblt.org/v2` will be available for integration testing.

**Reasoning:** The API is deployed and serving production traffic. No maintenance windows or deprecation schedules are announced [6].

### Assumption 9: Mentor Availability

**Statement:** The 10 confirmed GSoC 2026 mentors will be available for guidance and code review.

**Reasoning:** Mentors are listed on the OWASP GSoC 2026 ideas page and have committed to the program. Mentorship is a GSoC requirement [5].

### Assumption 10: Network Access for Development

**Statement:** The development environment will have sufficient network access to reach GitHub, PyPI, and BLT-API.

**Reasoning:** Standard development dependencies require these services. If network access is restricted, alternative approaches (offline package installation, VPN) will be needed.

---

## 15. Alternatives Considered

### Alternative 1: REST-Only Integration

**Description:** Maintain the current approach of integrating BLT through its REST API directly, without an MCP layer.

**Pros:**
- No new infrastructure required
- No MCP protocol learning curve
- Direct access to all BLT-API endpoints

**Cons:**
- Each AI platform requires custom integration code
- No capability discovery — agents must know endpoint URLs in advance
- No tool-level semantics — REST endpoints don't describe AI-agent intent
- No workflow abstraction — multi-step processes must be manually scripted
- No standardized error handling — each platform handles errors differently
- Documentation drift between API changes and tool definitions

**Verdict:** Insufficient. REST-only integration was evaluated and found lacking for the reasons documented in Section 3. The GSoC project explicitly calls for an MCP interface [5].

### Alternative 2: GraphQL

**Description:** Implement a GraphQL API as a unified frontend for BLT services.

**Pros:**
- Strongly-typed schema available at runtime
- Single endpoint for all queries
- Client specifies exact data requirements
- Rich ecosystem of tooling

**Cons:**
- GraphQL queries are not AI-agent tool definitions — agents cannot discover available mutations as "tools"
- No standardized prompt/workflow abstraction
- No standardized error semantics across services
- Requires significant backend work (schemas, resolvers) beyond MCP's wrapper approach
- GraphQL's query flexibility can be a liability for AI agents (unbounded queries, N+1 problems)
- No protocol-level authentication framework

**Key Differentiator for MCP:** MCP's three-primitive model (Resources/Tools/Prompts) directly maps to the agent interaction patterns BLT needs. GraphQL provides flexible queries but no action or workflow abstraction.

**Verdict:** GraphQL was evaluated but rejected because it solves a different problem (flexible data fetching) than the one BLT-MCP addresses (AI-agent capability exposure).

### Alternative 3: Custom SDK

**Description:** Build a language-specific SDK (Python, JavaScript) that wraps BLT-API and provides a programmatic interface.

**Pros:**
- Language-native feel for developers
- Type hints and IDE autocompletion
- Can abstract away BLT-API details

**Cons:**
- Language-specific — does not help AI agents that don't use that language
- Requires maintenance across multiple languages
- No runtime capability discovery — SDK must be updated for API changes
- No standardized authentication — each SDK implements its own auth handling
- No workflow abstraction — multi-step processes are manual
- Every AI platform needs its own integration layer on top of the SDK

**Key Differentiator for MCP:** MCP is protocol-based, not SDK-based. Any MCP-compatible client (regardless of language or platform) can discover and use BLT-MCP's capabilities without a language-specific SDK.

**Verdict:** A custom SDK could complement BLT-MCP but does not replace it. MCP provides the protocol layer that SDKs cannot.

### Alternative 4: OpenAPI + Generated Clients

**Description:** Publish an OpenAPI specification for BLT-API and use code generators to create client libraries.

**Pros:**
- Standard API description format
- Auto-generated client libraries possible
- Documented API surface

**Cons:**
- OpenAPI specs are static documents, not live capabilities
- AI agents cannot dynamically discover and adapt to spec changes
- No workflow or prompt abstraction
- No standardized authentication negotiation
- Spec generation from BLT-API's Python code requires additional tooling
- Large specs exceed LLM context windows

**Key Differentiator for MCP:** MCP provides runtime capability discovery, which OpenAPI cannot. An agent connecting to an MCP server learns what's available at connection time. An agent reading an OpenAPI spec must do so before connecting.

**Verdict:** OpenAPI is useful for documentation but insufficient for AI-agent integration.

### Alternative 5: Plugin Architecture

**Description:** Build a custom plugin system where AI platforms implement BLT-specific plugins.

**Pros:**
- Platform-native look and feel
- Can leverage platform-specific features

**Cons:**
- Must implement separate plugin for each platform
- No standardization — each plugin is a custom integration
- High maintenance burden (N platforms × M API changes)
- No ecosystem effect — plugins don't benefit non-targeted platforms

**Key Differentiator for MCP:** MCP provides a single server that works with all MCP-compatible clients, eliminating the N × M integration matrix.

**Verdict:** Plugin architecture is the pre-MCP approach. MCP eliminates the need for platform-specific plugins.

### Alternative 6: MCP (Selected)

**Description:** Implement an MCP server that wraps BLT-API and exposes BLT capabilities through MCP's three primitives.

**Pros:**
- Standardized protocol adopted by Anthropic, OpenAI, and others
- Runtime capability discovery
- Three-primitive model (Resources/Tools/Prompts) maps to BLT domain
- Dual transport support (stdio + HTTP)
- OAuth 2.1 authentication framework
- Cross-platform compatibility (any MCP client works)
- Growing ecosystem adoption
- Official Python SDK with FastMCP high-level API [24]

**Cons:**
- Requires learning MCP protocol and SDK
- MCP ecosystem still maturing (client compatibility varies)
- MCP Python SDK v2 migration may be needed if stable releases during GSoC

**Verdict:** MCP is the clear winner. It directly addresses the fragmentation problem, provides the semantic primitives BLT needs, and aligns with industry direction.

### Comparison Summary

| Criteria | REST Only | GraphQL | Custom SDK | OpenAPI | Plugin | **MCP** |
|----------|-----------|---------|------------|---------|--------|---------|
| Capability discovery | No | Query-based | No | Static doc | No | **Runtime** |
| Tool semantics | No | No | Via code | No | Platform-specific | **Built-in** |
| Workflow abstraction | No | No | No | No | Platform-specific | **Prompts** |
| Cross-platform | No | No | Language-bound | Language-bound | No | **Yes** |
| Auth framework | Per-endpoint | Per-endpoint | Custom | Via spec | Custom | **OAuth 2.1** |
| Runtime adaptivity | No | No | No | No | No | **Yes** |
| Ecosystem adoption | Universal | High | Low | High | Low | **Growing** |
| Implementation effort | Low | High | Medium | Medium | Very High | **Medium** |

---

## 16. Conclusion

The OWASP BLT ecosystem has grown into a powerful but fragmented platform — 53+ repositories spanning multiple services, each with its own interface, authentication, and data model [1]. This fragmentation creates a fundamental barrier for AI agents, the fastest-growing category of software consumers. An AI agent cannot autonomously discover BLT's capabilities, invoke its actions, or execute its workflows without custom integration code that must be rewritten for every target platform.

The costs of this fragmentation are measurable: 2-5 days of developer effort per AI platform integration, documentation drift that breaks tool definitions silently, no audit trail for AI agent actions, and a growing maintenance burden as the ecosystem expands.

MCP solves this by providing a standardized protocol with three primitives that map directly to BLT's domain:
- **Resources** expose BLT data through `blt://` URIs
- **Tools** provide executable BLT actions with typed schemas
- **Prompts** deliver reusable security workflow templates

[5, 13]

Any MCP-compatible client — Claude Desktop, ChatGPT, Cline, or a custom agent — can discover and use BLT-MCP's capabilities without platform-specific code. One server, any client. Runtime discovery, no documentation lookups.

BLT-MCP is not a replacement for BLT-API. It is a protocol layer that wraps BLT's existing capabilities and makes them accessible to the growing ecosystem of MCP-native AI agents. It positions BLT as an AI-agent-first platform, aligns with the BLT-Next architecture direction, and provides a future-proof integration surface that decouples AI clients from backend implementation changes [7].

The existing TypeScript prototype demonstrates feasibility [29]. The MCP Python SDK provides a clean implementation path [24]. The GSoC 2026 framework provides the structure and mentorship [5]. What remains is the implementation: building a production-grade MCP server that makes BLT accessible to every AI agent, every IDE, and every chat interface — without requiring custom integration code.

BLT-MCP should exist because the future of software integration is protocol-driven, and BLT deserves to be part of that future.

---

## References

1. OWASP-BLT GitHub Organization. https://github.com/orgs/OWASP-BLT
2. Claude Desktop MCP Support. https://modelcontextprotocol.io/docs/develop/build-server
3. OpenAI — Building MCP Servers for ChatGPT Apps. https://developers.openai.com/api/docs/mcp
4. IBM — MCP Architecture Patterns for Multi-Agent AI Systems. https://developer.ibm.com/articles/mcp-architecture-patterns-ai-systems/
5. OWASP GSoC 2026 Ideas — BLT-MCP Description. https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
6. OWASP-BLT/BLT-API. https://github.com/OWASP-BLT/BLT-API
7. OWASP-BLT/BLT-Next. https://github.com/OWASP-BLT/BLT-Next
8. OWASP-BLT/BLT-Pages. https://github.com/OWASP-BLT/BLT-Pages
9. OWASP-BLT/BLT-Rewards. https://github.com/OWASP-BLT/BLT-Rewards
10. OWASP-BLT/BLT. https://github.com/OWASP-BLT/BLT
11. BLT-API PR #89 — Static API Key Authentication. https://github.com/OWASP-BLT/BLT-API/pull/89
12. OWASP BLT Official Project Page. https://owasp.org/www-project-bug-logging-tool/
13. MCP Specification (2025-11-25). https://modelcontextprotocol.io/specification/2025-11-25
14. Inspired by Frustration — Production MCP Server Architecture. https://inspiredbyfrustration.com/blog/mcp-server-architecture
15. PADISO — AI Agents in Production: MCP Server Design Patterns. https://www.padiso.co/blog/ai-agents-production-mcp-server-design-patterns/
16. BLT-API PR #68 — Routes Endpoint. https://github.com/OWASP-BLT/BLT-API/pull/68
17. OWASP-BLT/BLT-docs. https://github.com/OWASP-BLT/BLT-docs
18. Descope — MCP Server Security Best Practices. https://www.descope.com/blog/post/mcp-server-security-best-practices
19. OWASP MCP Security Cheat Sheet. https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
20. OWASP MCP Top 10. https://owasp.org/www-project-mcp-top-10/
21. OWASP MCP Top 10 — MCP03:2025 (Tool Poisoning). https://github.com/OWASP/www-project-mcp-top-10/
22. OWASP MCP Top 10 — MCP07:2025 (Insufficient Auth). https://github.com/OWASP/www-project-mcp-top-10/
23. OWASP MCP Top 10 — MCP08:2025 (Lack of Audit). https://github.com/OWASP/www-project-mcp-top-10/
24. MCP Python SDK Documentation. https://py.sdk.modelcontextprotocol.io/
25. MCP Blog — Prompts for Workflow Automation. https://blog.modelcontextprotocol.io/posts/2025-07-29-prompts-for-automation/
26. MCP Lifecycle Documentation. https://modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle
27. MCP Specification (2025-06-18). https://modelcontextprotocol.io/specification/2025-06-18
28. BLT-API PR #35 — URL Validation. https://github.com/OWASP-BLT/BLT-API/pull/35
29. OWASP-BLT/BLT-MCP. https://github.com/OWASP-BLT/BLT-MCP
30. CSA — Agentic MCP Security Best Practices Guide. https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/
31. MCP Protocol Features — Python SDK. https://py.sdk.modelcontextprotocol.io/protocol/
32. Working Software — MCP in Practice. https://www.workingsoftware.dev/mcp-in-practice-what-software-architects-need-to-know-about-the-model-context-protocol/
