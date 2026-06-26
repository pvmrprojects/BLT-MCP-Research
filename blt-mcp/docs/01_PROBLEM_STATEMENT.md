# Problem Statement

> **Document Version:** 1.0
> **Status:** Draft
> **Date:** 2026-06-26

---

## What Problems Exist Today?

### 1. Fragmented Ecosystem

The OWASP BLT ecosystem has grown to over 53 public repositories spanning multiple services, APIs, dashboards, mobile applications, and automation bots [OWASP-BLT GitHub Org, Source: github.com/orgs/OWASP-BLT]. This growth has created a fragmented integration surface:

- The **main Django application** serves the web UI and traditional API views
- **BLT-API** provides a separate REST API built on Cloudflare Workers with endpoints under `/v2` [BLT-API, Source: github.com/OWASP-BLT/BLT-API]
- **BLT-Next** migrates to an entirely different architecture (static frontend + Cloudflare Workers) [BLT-Next, Source: github.com/OWASP-BLT/BLT-Next]
- **BLT-Pages** offers yet another interface via GitHub Pages [BLT-Pages, Source: github.com/OWASP-BLT/BLT-Pages]
- **BLT-Rewards** operates a separate Bitcoin-based BACON token system [BLT-Rewards, Source: github.com/OWASP-BLT/BLT-Rewards]

Each of these surfaces has its own authentication mechanism, data formats, and interaction patterns. An AI agent or developer integrating with BLT must understand each surface independently — there is no unified discovery mechanism.

### 2. No AI-Native Integration Point

AI platforms are rapidly adopting protocol-based tool integration. OpenAI officially supports MCP for ChatGPT apps, where remote MCP servers connect models to new data sources and capabilities [OpenAI MCP Guide, Source: developers.openai.com/api/docs/mcp]. Claude Desktop has native MCP support built in. BLT currently has no MCP-compatible interface, meaning:

- AI agents cannot autonomously discover BLT's capabilities
- Each AI platform requires custom integration code
- BLT's security workflows cannot be invoked from AI-powered IDEs or chat interfaces

### 3. Manual Security Workflows

BLT's core value proposition — vulnerability disclosure and bug triage — currently requires human operators to:

- Navigate the web UI to view reported issues
- Manually assess and triage vulnerability reports
- Coordinate with reporters and organization managers
- Track remediation progress

These workflows cannot be delegated to AI agents because no standardized, protocol-based interface exists for autonomous interaction.

### 4. Inconsistent Authentication

BLT services use different authentication mechanisms:

- BLT-API uses a shared static API key via the `X-BLT-API-Key` header [BLT-API PR #89, Source: github.com/OWASP-BLT/BLT-API/pull/89]
- The main Django app uses session-based authentication
- BLT-Next uses JWT tokens

An integrator must implement multiple authentication strategies to access the full BLT feature set — and an AI agent has no standardized way to negotiate credentials.

---

## Why Are Current APIs Insufficient?

### REST API Limitations

BLT-API provides comprehensive REST endpoints for bugs, users, domains, and organizations [BLT-API README, Source: github.com/OWASP-BLT/BLT-API]:

| Endpoint Group | Methods | Count |
|---------------|---------|-------|
| Bugs | GET, POST | 4 |
| Users | GET, POST | 8 |
| Domains | GET | 3 |
| Organizations | GET | 9 |

While functional, these REST endpoints have fundamental limitations for AI agent integration:

**No capability discovery.** An AI agent cannot ask "what can you do?" and receive a structured answer. While BLT-API added a `GET /routes` endpoint for programmatic route discovery [BLT-API PR #68, Source: github.com/OWASP-BLT/BLT-API/pull/68], this only returns HTTP method + path pairs — not semantic capability descriptions, input schemas, or usage patterns that an LLM can reason about.

**No tool-level semantics.** REST endpoints describe HTTP operations, not AI-agent actions. An agent must infer that `POST /bugs` means "submit a bug report" from the endpoint name alone, with no structured tool description, parameter schema, or expected behavior contract.

**No prompt templates.** REST provides no mechanism for reusable workflow templates. Common BLT workflows (triage a vulnerability, plan a remediation, review a contribution) must be manually scripted each time.

**Pagination is agent-unfriendly.** While BLT-API supports pagination (`page`, `per_page` parameters) [BLT-API README], AI agents must implement pagination logic themselves, often resulting in incomplete data or excessive API calls.

### No Workflow Abstraction

REST APIs expose CRUD operations on resources. They do not expose higher-level workflow abstractions. Consider the difference:

- **REST**: `POST /bugs` with title, description, severity, repo_id
- **MCP Prompt**: `triage_vulnerability` which guides the agent through: assess severity, check for duplicates, identify affected components, assign priority, notify stakeholders

The prompt template encapsulates domain knowledge that would otherwise need to be embedded in every client implementation.

---

## How Do AI Agents Interact Today?

Without MCP, AI agents integrate with BLT through one of these patterns:

### Pattern 1: Custom HTTP Client Code

An agent or its developer writes custom Python/JavaScript code that:
1. Constructs HTTP requests to BLT-API endpoints
2. Manages authentication headers (`X-BLT-API-Key`)
3. Parses JSON responses
4. Handles pagination loops
5. Implements error handling and retries
6. Processes BLT-specific data formats

This code is **platform-specific** — an agent built for Claude Desktop cannot reuse the same logic in ChatGPT or a custom IDE extension. It must be rewritten or adapted for each host environment.

### Pattern 2: Tool-Use via API Wrappers

Some agents can use custom "tools" if the developer provides:
- A tool name and description
- A JSON Schema for parameters
- A function handler that makes the HTTP call

But these tool definitions are **static and hardcoded**. The agent cannot discover new tools when BLT-API adds endpoints. The tool descriptions may become stale. And tool schemas must be manually kept in sync with the underlying API — a maintenance burden across 53+ repositories.

### Pattern 3: Manual Human-in-the-Loop

The most common pattern today: a human operator uses the BLT web UI or dashboard to perform actions, while the AI agent only provides recommendations or analysis. This limits the value proposition of autonomous security workflows.

---

## What Limitations Exist?

### For AI Agents

| Limitation | Impact |
|------------|--------|
| No standardized discovery | Agent cannot dynamically learn what BLT capabilities exist |
| No semantic tool descriptions | Agent must infer purpose from REST endpoint names |
| No parameterized prompts | Common workflows cannot be invoked as reusable templates |
| Multiple auth mechanisms | Agent must implement different auth for each BLT service |
| No protocol-level error semantics | Agent cannot distinguish "bad request" from "auth expired" from "rate limited" |
| No subscription/notification | Agent must poll for updates rather than receiving push notifications |

### For Developers

| Limitation | Impact |
|------------|--------|
| Custom integration per AI platform | Code must be written for Claude, ChatGPT, IDEs, etc. separately |
| Documentation drift | Tool descriptions in agent configs fall out of sync with API changes |
| No standardized testing | Each integration must be tested against each AI platform independently |
| Security audit surface | Multiple custom integrations = multiple potential vulnerability vectors |
| Fragmented logging | No centralized audit trail for AI agent actions across services |

### For BLT Project Maintainers

| Limitation | Impact |
|------------|--------|
| API changes break integrations | Any REST endpoint modification breaks downstream agent tool definitions |
| No protocol versioning | No built-in mechanism to signal breaking changes to AI clients |
| Limited ecosystem reach | BLT is invisible to LLM platforms that discover capabilities through MCP |

---

## How Does MCP Solve Them?

MCP solves these problems through four key mechanisms, as defined by the official specification [MCP Specification, Source: modelcontextprotocol.io/specification/2025-11-25]:

### 1. Standardized Capability Discovery

MCP's initialization phase requires capability negotiation between client and server. The client sends capabilities (roots, sampling, experimental features), and the server responds with its capabilities (prompts, resources, tools, logging) [MCP Lifecycle, Source: modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle].

An AI agent connecting to BLT-MCP will immediately know:
- Which tools are available (with names, descriptions, and JSON schemas)
- Which resources can be read (with URI patterns and MIME types)
- Which prompt templates can be invoked (with arguments and descriptions)

No hardcoded configuration. No documentation drift. The server is the source of truth.

### 2. Three Semantic Primitives

MCP defines three primitives that map naturally to BLT's domain:

| MCP Primitive | BLT Mapping | Problem Solved |
|--------------|-------------|----------------|
| **Resources** (read-only data) | Issues, contributors, repos, leaderboards | No standardized read interface for BLT data |
| **Tools** (executable actions) | submit_issue, award_bacon, update_status | No standardized action interface for BLT operations |
| **Prompts** (workflow templates) | triage_vulnerability, plan_remediation | No reusable workflow abstraction |

[MCP Protocol Features, Source: py.sdk.modelcontextprotocol.io/protocol/]

This separation is critical. As noted by production MCP practitioners: "Resources = nouns (data, readable), Tools = verbs (actions, with side effects) ... if you model everything as a tool, the agent has to guess from descriptions which ones are safe to call speculatively" [Inspired by Frustration, Source: inspiredbyfrustration.com/blog/mcp-server-architecture].

### 3. Transport Flexibility

MCP supports multiple transports that serve different integration scenarios:

- **stdio**: For local/desktop integration (Claude Desktop, IDE extensions). No network exposure. Credentials come from environment.
- **Streamable HTTP**: For remote/cloud integration. Supports OAuth 2.1 with PKCE for secure authentication.

[MCP Python SDK, Source: py.sdk.modelcontextprotocol.io/]

This means BLT-MCP can serve both local developer tools (via stdio) and cloud-hosted AI platforms (via HTTP) with the same codebase, just different transport configurations.

### 4. Protocol-Level Authorization

The MCP specification (2025-06-18) introduced a formal authorization framework, separating the MCP server (OAuth Resource Server) from the authorization server [MCP Spec 2025-06-18, Source: modelcontextprotocol.io/specification/2025-06-18].

For BLT-MCP, this means:
- BLT-API keys are managed by the MCP server, not exposed to the AI model
- OAuth 2.1 with PKCE can be used for HTTP-based remote access [CSA Guide, Source: labs.cloudsecurityalliance.org]
- The MCP server enforces tool-level authorization, not the AI agent
- Audit logging captures every tool invocation with authenticated identity

This directly addresses OWASP MCP Top 10 risks including MCP01 (Token Mismanagement), MCP02 (Privilege Escalation), and MCP07 (Insufficient Authentication) [OWASP MCP Top 10, Source: owasp.org/www-project-mcp-top-10/].

### 5. Unified Interface

As described in the GSoC 2026 project description, BLT-MCP "unifies fragmented REST/GraphQL endpoints" into a single, agent-friendly interface where agents discover capabilities automatically without requiring custom API documentation [GSoC 2026 Ideas, Source: owasp.org/www-community/initiatives/gsoc/gsoc2026ideas].

This unification means:

| Before MCP | After MCP |
|------------|-----------|
| Agent reads BLT-API docs to find endpoints | Agent discovers capabilities at connection time |
| Developer writes custom tool definitions | Server provides tool definitions with schemas |
| Each AI platform needs separate integration | Any MCP-compatible client works immediately |
| Authentication varies by service | Authentication is handled once at the MCP layer |
| Workflows are manual or custom-coded | Workflows are reusable prompt templates |

---

## Who Benefits?

### AI Agents and LLM Platforms

- **Claude Desktop** users can interact with BLT through natural language — reporting bugs, querying leaderboards, triaging issues — without leaving the chat interface
- **ChatGPT** can integrate BLT as a data source using MCP's `search`/`fetch` tool pattern [OpenAI MCP Guide, Source: developers.openai.com/api/docs/mcp]
- **Custom AI agents** (security bots, automated triage systems, CI/CD pipelines) can interact with BLT through a standardized protocol
- **IDE extensions** (VS Code, JetBrains) can surface BLT workflows directly in the development environment

### Developers and Security Researchers

- Reduced integration effort: one MCP interface instead of multiple REST clients
- Automatic capability discovery: no documentation lookup needed
- Protocol-level error handling: consistent semantics across all operations
- Reusable prompt templates: common workflows are pre-built and versioned

### BLT Project and OWASP Community

- Protocol-native integration with the growing MCP ecosystem
- Future-proof architecture: decoupled from backend implementation changes
- Ecosystem growth: BLT becomes discoverable from any MCP-compatible tool
- Alignment with industry direction: OpenAI, Anthropic, Google, and others are adopting MCP

---

## What Are Measurable Improvements?

### Development Efficiency

| Metric | Before MCP | After MCP | Improvement |
|--------|-----------|-----------|-------------|
| Time to integrate BLT into new AI platform | 2-5 days (custom code, auth, testing) | 15 minutes (MCP client config) | 95%+ reduction |
| Lines of integration code per platform | 200-500 lines per platform | 10-20 lines (MCP config JSON) | 95%+ reduction |
| API discovery time | Hours reading docs, testing endpoints | Instant (server declares capabilities) | Near 100% elimination |

### Operational Improvements

| Metric | Before MCP | After MCP | Improvement |
|--------|-----------|-----------|-------------|
| Tool definition staleness | Drifts with every API change | Always current (server is source of truth) | 100% elimination |
| Authentication surface | Multiple mechanisms per service | Single OAuth/API key at MCP layer | 60-80% reduction |
| Audit coverage | Custom per integration | Built into MCP protocol | Universal coverage |

### Agent Capabilities

| Capability | Before MCP | After MCP |
|------------|-----------|-----------|
| Discover BLT features | Not possible autonomously | Automatic at connection |
| Execute workflows | Manual or custom-scripted | Invoke prompt templates |
| Subscribe to changes | Impossible (polling only) | Resource subscriptions (listChanged) |
| Cross-platform portability | None (custom per platform) | Full (any MCP client) |

### Security Posture (measured against OWASP MCP Top 10)

| Risk | Before MCP | After MCP |
|------|-----------|-----------|
| MCP01: Token Mismanagement | API keys embedded in agent configs | Keys managed by MCP server, not exposed to model |
| MCP02: Privilege Escalation | No tool-level access control | Tool-level authorization via OAuth scopes |
| MCP03: Tool Poisoning | No tool description validation | JSON Schema validation with additionalProperties: false |
| MCP07: Insufficient Auth | Multiple inconsistent auth mechanisms | Standardized OAuth 2.1 with PKCE |
| MCP08: Lack of Audit | No centralized audit trail | Built-in MCP logging capability |

[OWASP MCP Top 10, Source: owasp.org/www-project-mcp-top-10/; OWASP MCP Security Cheat Sheet, Source: cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html]

---

## References

1. OWASP-BLT GitHub Organization — https://github.com/orgs/OWASP-BLT
2. BLT-API Repository — https://github.com/OWASP-BLT/BLT-API
3. BLT-API PR #89 (Static API Key Auth) — https://github.com/OWASP-BLT/BLT-API/pull/89
4. BLT-API PR #68 (Routes Discovery) — https://github.com/OWASP-BLT/BLT-API/pull/68
5. BLT-Next Repository — https://github.com/OWASP-BLT/BLT-Next
6. BLT-Pages Repository — https://github.com/OWASP-BLT/BLT-Pages
7. BLT-Rewards Repository — https://github.com/OWASP-BLT/BLT-Rewards
8. MCP Specification (2025-11-25) — https://modelcontextprotocol.io/specification/2025-11-25
9. MCP Specification (2025-06-18) — https://modelcontextprotocol.io/specification/2025-06-18
10. MCP Lifecycle — https://modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle
11. MCP Protocol Features (Python SDK) — https://py.sdk.modelcontextprotocol.io/protocol/
12. MCP Python SDK — https://py.sdk.modelcontextprotocol.io/
13. GSoC 2026 Ideas — https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
14. OpenAI MCP Guide — https://developers.openai.com/api/docs/mcp
15. Inspired by Frustration — Production MCP Server Architecture — https://inspiredbyfrustration.com/blog/mcp-server-architecture
16. OWASP MCP Top 10 — https://owasp.org/www-project-mcp-top-10/
17. OWASP MCP Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
18. CSA Agentic MCP Security Guide — https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/
