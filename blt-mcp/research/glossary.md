# Glossary

> **Document Version:** 1.0
> **Date:** 2026-06-26
> **Purpose:** Define every important technical term used in the BLT-MCP project

---

## A

**ABAC (Attribute-Based Access Control)**
Access control model where access decisions are based on attributes of the user, resource, and environment. Recommended by OWASP MCP Top 10 for granular MCP tool authorization. [Source: OWASP MCP Top 10 MCP07]

**additionalProperties (JSON Schema)**
A JSON Schema keyword that controls whether properties beyond those explicitly defined are allowed. Setting `additionalProperties: false` is a security best practice for MCP tool schemas to prevent LLMs from inventing parameters. [Source: OWASP MCP Security Cheat Sheet]

**AGPL-3.0**
GNU Affero General Public License v3.0 — the license used by all OWASP BLT projects. Requires that modified versions of the software also be made available under the same license when run as a network service. [Source: BLT Repository]

**API Gateway**
A single entry point for multiple backend services. An MCP gateway pattern would route requests to different BLT services (BLT-API, BLT-Rewards) through a unified MCP interface. [Source: Obot Enterprise MCP Guide]

**API Key**
A static token used for authentication. BLT-API uses a shared static API key sent via the `X-BLT-API-Key` header. BLT-MCP uses a similar key in environment variables to authenticate with BLT-API. [Source: BLT-API PR #89]

**ASGI (Asynchronous Server Gateway Interface)**
The modern Python standard for async web servers. MCP Python SDK supports mounting to ASGI applications (FastAPI, Starlette) for HTTP transport. [Source: MCP Python SDK]

**Attack Tree**
A hierarchical diagram of potential attacks on a system. The BLT-MCP attack tree identifies 5 main branches: credential theft, tool exploitation, auth bypass, denial of service, and backend abuse. [Source: docs/10_SECURITY.md]

**Audit Logging**
The practice of recording all security-relevant events. For BLT-MCP, every tool call, resource read, and authentication attempt is logged with identity and timestamp. Required by OWASP MCP08:2025. [Source: OWASP MCP Top 10]

**Authorization Server**
In OAuth, the server that issues tokens after authenticating a user and obtaining authorization. The MCP specification (2025-06-18) requires a separate authorization server from the MCP resource server. [Source: MCP Spec 2025-06-18]

---

## B

**BACON**
A Bitcoin-based token system using the Runes protocol for incentivizing contributions within the BLT ecosystem. Supports both Bitcoin and Solana blockchains. [Source: BLT-Rewards Repository]

**BLT (Bug Logging Tool)**
An OWASP Foundation project — a gamified crowd-sourced QA testing and vulnerability disclosure platform. Users report bugs across websites, apps, and Git repositories. [Source: OWASP BLT Project Page]

**BLT-API**
The REST API for BLT built on Python/Cloudflare Workers with D1 database. The primary data source for BLT-MCP. [Source: BLT-API Repository]

**BLT-MCP**
The MCP server project described in this knowledge base. Provides AI agents with structured access to the BLT ecosystem through Resources, Tools, and Prompts. [Source: GSoC 2026 Ideas]

**BLT-Next**
The next-generation BLT architecture — migrating from Django monolith to static frontend on GitHub Pages with Cloudflare Python Workers. [Source: BLT-Next Repository]

**blt:// URI**
The URI scheme used by BLT-MCP for MCP resources. Examples: `blt://issues`, `blt://issues/{id}`, `blt://contributors`. [Source: BLT-MCP Repository]

**Bug Bounty**
A program where organizations offer rewards to individuals who discover and report vulnerabilities in their software. BLT supports company-sponsored bug bounties. [Source: OWASP BLT Project Page]

---

## C

**Capability Negotiation**
The MCP initialization process where client and server exchange information about which protocol features they support. Determines which capabilities (resources, tools, prompts, logging, etc.) are available during the session. [Source: MCP Lifecycle]

**Circuit Breaker**
A resilience pattern that prevents cascading failures by stopping requests to a failing service. Recommended for production MCP guardrails layer. [Source: Inspired by Frustration]

**CIMD (Client ID Metadata Document)**
An OAuth mechanism where the client's public metadata is published at a well-known URL. Supported by ChatGPT's MCP integration for client registration. [Source: OpenAI MCP Guide]

**Cloudflare D1**
Cloudflare's serverless SQLite database. Used by BLT-API and BLT-Next for edge data persistence with global replication. [Source: BLT-API Repository]

**Cloudflare Workers**
Cloudflare's serverless execution environment that runs code at the edge. BLT-API, BLT-Next, BLT-Rewards, and BLT-Pool all use Cloudflare Workers (Python runtime). [Source: BLT-API Repository]

**Control Plane**
In enterprise MCP architecture, a centralized layer between clients and servers that handles authentication, authorization, token management, filtering, and audit logging. [Source: Obot Enterprise MCP Guide]

**CVE (Common Vulnerabilities and Exposures)**
A dictionary of publicly known information security vulnerabilities. BLT stores CVE IDs and scores for issues, with enhanced integration planned. [Source: BLT Issue #5050]

---

## D

**D1** — *see Cloudflare D1*

**Deny-by-Default**
A security principle where access is denied unless explicitly granted. Applied to MCP tool authorization — unrecognized scopes or agents are blocked automatically. [Source: OWASP MCP Top 10 MCP07]

**Django**
Python web framework used by the main BLT monolith (v5.2). Being gradually replaced by edge-based architecture. [Source: BLT pyproject.toml]

---

## E

**Elicitation**
An MCP server feature where the server can request information from the user through the client. Part of the draft specification. [Source: MCP Spec Draft]

**enum**
A JSON Schema keyword that constrains a field to a fixed set of values. Using `enum` over free-form strings is an MCP best practice — it prevents the LLM from hallucinating invalid values. [Source: Inspired by Frustration]

---

## F

**FastMCP**
The high-level Python API for building MCP servers using decorators (`@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()`). Dramatically reduces boilerplate compared to the low-level Server API. [Source: MCP Python SDK]

**Federation**
The practice of connecting multiple identity providers. Future BLT-MCP improvement: OIDC integration with organizational IdP. [Source: OWASP MCP Top 10 MCP07]

---

## G

**Gamification**
The application of game-design elements (leaderboards, badges, points, tiers) to non-game contexts. BLT uses gamification to incentivize security researchers. [Source: BLT Main README]

**GSoC (Google Summer of Code)**
A global program focused on bringing student contributors into open source software development. BLT-MCP is an official OWASP GSoC 2026 project. [Source: GSoC 2026 Ideas]

**Guardrails**
The fourth layer of production MCP architecture. Includes rate limiting, circuit breakers, response caps, authentication, and audit logging. The layer most commonly skipped by MCP implementations. [Source: Inspired by Frustration]

---

## H

**Host (MCP)**
An LLM application that initiates MCP connections. Examples: Claude Desktop, ChatGPT, IDE extensions. [Source: MCP Spec 2025-11-25]

**HTMX**
A JavaScript library for adding dynamic behavior to HTML without writing JavaScript directly. Used by BLT-Next for progressive enhancement. [Source: BLT-Next Repository]

**Human-in-the-Loop (HITL)**
A security pattern requiring human approval before executing sensitive operations. Recommended by OWASP MCP Security Cheat Sheet for destructive or financial tool calls. [Source: OWASP MCP Security Cheat Sheet]

---

## I

**Idempotency**
The property that executing the same operation multiple times produces the same result. Important for MCP tools that may be retried by the LLM. [Source: MCP Best Practices]

**IdP (Identity Provider)**
A system that creates, maintains, and manages identity information. OIDC integration is a recommended future improvement for BLT-MCP. [Source: CSA Agentic MCP Security Guide]

**Initialization (MCP Lifecycle)**
The first phase of the MCP lifecycle where client and server exchange protocol versions, capabilities, and implementation information. Must be completed before any other messages are exchanged. [Source: MCP Lifecycle]

---

## J

**JIT (Just-in-Time) Access**
A security pattern where credentials are issued only when needed for a specific task, with time-limited validity. Recommended for enterprise MCP deployments. [Source: OWASP GenAI Security Blog]

**JSON-RPC 2.0**
The wire protocol used by MCP for all client-server messages. Defines request, response, notification, and error message types. [Source: MCP Spec 2025-11-25]

**JSON Schema 2020-12**
The JSON Schema version used by MCP for tool input schemas, output schemas, and elicitation schemas. Auto-generated from Pydantic models in the Python SDK. [Source: MCP Protocol Features]

**JWT (JSON Web Token)**
A compact, URL-safe token format used for authentication. Used by BLT-Next and potentially for BLT-MCP's OAuth tokens. Contains signed claims about the client identity and permissions. [Source: BLT-Next README]

---

## L

**LATEST_PROTOCOL_VERSION**
A constant in the MCP Python SDK (`mcp.shared.version`) that defines the protocol version the SDK advertises during initialization. [Source: MCP Protocol Features]

**Layered Tool Server**
An MCP architecture pattern where tools are grouped by layer (data, API, utility) for better organization. BLT-MCP uses this pattern: Resources (data) → Tools (actions) → Prompts (workflows). [Source: PADISO MCP Design Patterns]

**listChanged**
An optional MCP capability flag that enables servers to notify clients when their list of available resources, tools, or prompts changes. Enables dynamic capability discovery. [Source: MCP Protocol Features]

---

## M

**MCP (Model Context Protocol)**
An open protocol that enables seamless integration between LLM applications and external data sources and tools. Standardizes how AI models discover and interact with capabilities. [Source: MCP Spec 2025-11-25]

**MCP Client**
A connector within the host application that manages a session with one MCP server. Implements the client side of the protocol. [Source: MCP Spec 2025-11-25]

**MCP Server**
A service that provides context and capabilities to clients through the MCP protocol. Offers Resources, Tools, and/or Prompts. [Source: MCP Spec 2025-11-25]

**mcp-scan**
A tool for automatically detecting poisoned descriptions and cross-server shadowing in MCP servers. Recommended by OWASP MCP Security Cheat Sheet. [Source: OWASP MCP Security Cheat Sheet]

**mTLS (Mutual TLS)**
A security protocol where both the client and server authenticate each other using TLS certificates. Recommended by OWASP MCP Top 10 for high-security MCP deployments. [Source: OWASP MCP Top 10 MCP07]

---

## O

**OAuth 2.1**
The latest version of the OAuth authorization framework, consolidating and simplifying OAuth 2.0 with security best practices. Mandatory for HTTP-based MCP transport per the November 2025 specification. [Source: MCP Spec 2025-11-25; CSA Guide]

**OIDC (OpenID Connect)**
An identity layer on top of OAuth 2.0. Future BLT-MCP improvement for federated authentication with organizational IdPs. [Source: OWASP MCP Top 10 MCP07]

**OpenAPI**
A specification for describing REST APIs. BLT-API does not currently have a published OpenAPI specification; route discovery is available via `GET /v2/routes`. [Source: BLT-API PR #68]

**OWASP (Open Worldwide Application Security Project)**
A nonprofit foundation that works to improve software security. BLT-MCP is an OWASP project under the OWASP Foundation. [Source: OWASP Foundation]

---

## P

**PKCE (Proof Key for Code Exchange)**
An OAuth security extension that prevents authorization code interception attacks. Required for all public OAuth clients in MCP's HTTP transport. [Source: MCP Spec 2025-06-18; Descope MCP Security]

**Prompt (MCP)**
A user-controlled, reusable workflow template. BLT-MCP defines prompts like `triage_vulnerability`, `plan_remediation`, and `review_contribution`. [Source: MCP Protocol Features; GSoC 2026 Ideas]

**Prompt Chaining**
Executing multiple prompts in sequence to perform complex workflows. A key feature of MCP's prompt system for automation. [Source: MCP Blog — Prompts for Automation]

**Prompt Injection**
An attack where crafted input manipulates an LLM's behavior. Tool descriptions in MCP are a potential injection vector (OWASP MCP03:2025). [Source: OWASP MCP Top 10]

---

## R

**RBAC (Role-Based Access Control)**
Access control model where permissions are assigned to roles, and users are assigned to roles. Recommended for MCP tool authorization. [Source: OWASP MCP Top 10 MCP07]

**Resource (MCP)**
An application-controlled, read-only data source addressable by URI. BLT-MCP resources use the `blt://` URI scheme. [Source: MCP Protocol Features]

**Resource Server (OAuth)**
The server that hosts protected resources and accepts access tokens. In MCP, the MCP server acts as the OAuth Resource Server. [Source: MCP Spec 2025-06-18]

**Resource Template**
A parameterized URI pattern for MCP resources. Example: `blt://issues/{id}` where `{id}` is a parameter. [Source: MCP Blog — Prompts for Automation]

**Runes Protocol**
A protocol on the Bitcoin blockchain for creating fungible tokens. Used by BLT's BACON token system. [Source: BLT-Rewards Repository]

---

## S

**Sampling (MCP)**
A client-side feature enabling server-initiated LLM interactions. The server can request the client to generate text from an LLM. Used for recursive agentic behaviors. [Source: MCP Protocol Features]

**Scope Creep**
The gradual expansion of permissions beyond what was originally intended. OWASP MCP02:2025 identifies this as a key MCP security risk. Mitigated by tool-level scoping and periodic access reviews. [Source: OWASP MCP Top 10]

**SSE (Server-Sent Events)**
A one-way server-to-client streaming protocol. Earlier MCP transport option, now being deprecated in favor of Streamable HTTP. [Source: MCP Spec 2025-06-18]

**SSRF (Server-Side Request Forgery)**
An attack where a server is tricked into making requests to internal systems. A critical risk for MCP servers that make network requests (like BLT-MCP calling BLT-API). [Source: MCP Permissions Guide]

**stdio (Standard Input/Output)**
An MCP transport where the server runs as a subprocess and communicates via stdin/stdout. Used for local/desktop integration. Credentials come from environment variables. [Source: MCP Spec 2025-06-18]

**Streamable HTTP**
The modern MCP transport for remote servers. Uses HTTP POST with optional SSE streaming for responses. Supports OAuth 2.1 with PKCE. [Source: MCP Python SDK]

**SUPPORTED_PROTOCOL_VERSIONS**
A constant in the MCP Python SDK listing all protocol versions the SDK can work with. If the server responds with a version outside this list, the client raises a RuntimeError. [Source: MCP Protocol Features]

---

## T

**Token Bucket**
A rate-limiting algorithm that maintains a bucket of tokens that refill at a fixed rate. Each request consumes a token. Used for per-client rate limiting in BLT-MCP. [Source: docs/10_SECURITY.md]

**Tool (MCP)**
A model-controlled, executable function with side effects. The LLM decides when to invoke tools. BLT-MCP tools include `submit_issue`, `award_bacon`, `update_issue_status`, and `add_comment`. [Source: MCP Protocol Features; GSoC 2026 Ideas]

**Tool Poisoning (MCP03:2025)**
An attack where compromised tools or their outputs manipulate the LLM's behavior. Includes rug pulls, schema poisoning, and tool shadowing. [Source: OWASP MCP Top 10]

**Transport**
The communication mechanism between MCP client and server. Supported transports: stdio, Streamable HTTP, SSE (legacy). [Source: MCP Python SDK]

---

## U

**URI (Uniform Resource Identifier)**
Used in MCP to address resources. BLT-MCP uses the custom `blt://` scheme.

---

## V

**Version Negotiation**
The process during MCP initialization where the client and server agree on a mutually supported protocol version. If the server doesn't support the client's version, it responds with an alternative. [Source: MCP Lifecycle]

---

## W

**Webhook**
An HTTP callback that delivers events to a configured URL. Future BLT-MCP extension for event-driven workflows. [Source: MCP Blog — Prompts for Automation]

**Wrangler**
Cloudflare's CLI tool for managing Workers. Used by BLT-API for deployment and local development. [Source: BLT-API README]

---

## X

**X-BLT-API-Key**
The HTTP header used by BLT-API for authentication. BLT-MCP sets this header when making requests to BLT-API. [Source: BLT-API PR #89]

---

## Z

**Zero Trust**
A security model where no entity is trusted by default. Applied to MCP: "never trust, always verify" for every interaction. [Source: OWASP GenAI Security Blog]
