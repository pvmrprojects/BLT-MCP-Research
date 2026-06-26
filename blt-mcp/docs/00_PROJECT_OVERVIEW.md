# BLT-MCP: Project Overview

> **Document Version:** 2.0
> **Status:** Draft
> **Date:** 2026-06-26
> **Author:** BLT-MCP Research & Planning

---

## Executive Summary

BLT-MCP is a Google Summer of Code 2026 project under the OWASP Foundation that implements a Model Context Protocol (MCP) server for the OWASP Bug Logging Tool (BLT) ecosystem. The server enables AI agents and developers to log bugs, triage issues, query data, and manage security workflows from IDEs or chat interfaces through a standardized protocol [1].

The project implements the MCP standard — an open protocol for integrating LLM applications with external data sources and tools [2] — through three layers: **Resources** (read-only data via `blt://` URIs), **Tools** (executable actions), and **Prompts** (reusable workflow templates) [1]. This positions BLT as an AI-agent-first platform by replacing fragmented REST/GraphQL integration surfaces with a single, standardized, automatically-discoverable interface.

The project builds upon an existing Node.js/TypeScript prototype [3] and targets production-grade delivery with dual-transport support (stdio for local, Streamable HTTP for remote), OAuth 2.1 authentication, and comprehensive security controls aligned with OWASP MCP Security guidelines [4].

---

## Background

### What Is OWASP?

The Open Worldwide Application Security Project (OWASP) is a nonprofit foundation dedicated to improving software security. Founded in 2001, OWASP provides freely available tools, documentation, standards, and community resources including the OWASP Top 10, the OWASP Application Security Verification Standard (ASVS), and the OWASP Cheat Sheet Series. OWASP has been a Google Summer of Code mentoring organization since 2016 and hosts hundreds of security projects under its governance [5].

OWASP BLT is one of OWASP's production-grade projects, having achieved production status in May 2023 [6]. It operates under OWASP's open-source governance model with AGPL-3.0 licensing.

### What Is BLT?

OWASP BLT (Bug Logging Tool) is a gamified, community-driven platform for discovering and reporting bugs across websites, applications, and Git repositories [7]. Users can submit any type of issue — from design flaws to security vulnerabilities — following responsible disclosure ethics [6].

### Why BLT Exists

BLT was created to solve the problem of fragmented vulnerability reporting. Before BLT, security researchers and QA testers had no centralized platform to report bugs, track their fixes, receive recognition, or collaborate with organizations. BLT provides:

- A structured, responsible disclosure workflow for security vulnerabilities
- Gamified incentives (leaderboards, badges, BACON token rewards) to encourage community participation
- Sponsored bug bounty programs that allow organizations to incentivize specific testing
- A public record of security issues that improves overall web security [7]

### Current Capabilities

BLT offers the following capabilities as of version 2.1 (November 2025):

| Capability | Description |
|------------|-------------|
| QA Testing & Vulnerability Disclosure | Report bugs across websites, apps, and Git repositories |
| Bug Bounties | Sponsored bounty programs with prize pools |
| Gamification | Leaderboards, challenges, badges, BACON token rewards |
| AI-Powered Tools | AI-assisted coding, PR reviews, issue generation, similarity scanning |
| Comprehensive Dashboard | Track progress, statistics, and impact |
| Security Labs | Hands-on security education content |
| Hackathon Features | Structured hackathon event management |
| Dark Mode | UI accessibility option |

[7, 6]

BLT has grown to encompass over 53 public repositories under the OWASP-BLT GitHub organization, including services, APIs, dashboards, mobile applications, bots, and documentation [8].

### Current Architecture (High Level)

BLT is in transition from a monolithic Django architecture to a modern edge-based architecture:

```
┌─────────────────────────────────────────────────────┐
│                    BLT Ecosystem                     │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────┐  ┌──────────┐  ┌──────────────┐  │
│  │  BLT-Next     │  │BLT-Pages │  │ BLT-Flutter  │  │
│  │  (GitHub      │  │(GitHub   │  │ (Mobile App) │  │
│  │   Pages/HTMX) │  │ Pages)   │  │              │  │
│  └──────┬───────┘  └────┬─────┘  └──────┬───────┘  │
│         │               │               │           │
│  ┌──────┴───────────────┴───────────────┴───────┐  │
│  │              BLT-API (Cloudflare Workers)      │  │
│  │  Python REST API │ D1 Database │ Auth         │  │
│  └──────────────────┬───────────────────────────┘  │
│                     │                               │
│  ┌──────────────────┴───────────────────────────┐  │
│  │    BLT (Django Monolith — Legacy)             │  │
│  │    PostgreSQL │ Redis │ Celery                │  │
│  └──────────────────────────────────────────────┘  │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ BLT-Rewards  │  │ BLT-MCP      │                 │
│  │ (Bitcoin/    │  │ (This        │                 │
│  │  BACON)      │  │  Project)    │                 │
│  └──────────────┘  └──────────────┘                 │
└─────────────────────────────────────────────────────┘
```

**Key components:**

- **BLT (Legacy):** The original Django monolith with PostgreSQL, Redis, and Celery for background tasks. Hosts the main web application, user management, and legacy API endpoints [7].
- **BLT-API:** Modern REST API built with Python on Cloudflare Workers, using D1 (SQLite-compatible serverless database) for data persistence. Provides edge-deployed endpoints for bugs, users, domains, organizations, projects, and hunts [9].
- **BLT-Next:** Next-generation architecture migrating from Django to static frontend on GitHub Pages with Cloudflare Python Workers backend, using HTMX for dynamic interactions [10].
- **BLT-Pages:** Community-powered bug-reporting platform built on GitHub Pages with Tailwind CSS [11].
- **BLT-Rewards:** Bitcoin-based token (BACON) system using the Runes protocol for incentivizing contributions [12].
- **BLT-MCP (Existing Prototype):** Node.js/TypeScript MCP server with 8 resources, 4 tools, and 3 prompts [3].

---

## What Is MCP?

The Model Context Protocol (MCP) is an open protocol that standardizes how AI applications integrate with external data sources and tools. Created by Anthropic and released as an open specification, MCP addresses the fragmentation problem that emerged as LLM-based applications proliferated — each requiring custom integration code for every data source or service they needed to access [2].

### Why It Was Created

Before MCP, every AI application that needed to interact with external systems required bespoke integration code. AI-powered IDEs needed custom plugins to access project files, chat assistants needed per-service API wrappers, and autonomous agents required hand-crafted tool definitions for each system they interacted with. MCP provides a universal protocol analogous to USB for hardware peripherals — a single, standardized way for AI applications to discover and use capabilities from any MCP-compliant server [2].

### How AI Agents Use MCP

An AI agent using MCP follows this flow:

1. **Connection:** The agent (via an MCP client) connects to an MCP server, establishing a session with capability negotiation.
2. **Discovery:** The agent discovers the server's capabilities — available resources (data), tools (actions), and prompts (workflows).
3. **Interaction:** The agent reads resources to gather context, invokes tools to perform actions, and presents prompts to users for guided workflows.
4. **Lifecycle:** The connection follows a defined lifecycle: initialization (negotiation) → operation (normal communication) → shutdown [13, 14].

MCP uses JSON-RPC 2.0 as its message format, supports multiple transports (stdio for local processes, Streamable HTTP for remote servers), and provides an authorization framework based on OAuth 2.1 for HTTP transports [2, 15].

### Why MCP Is Becoming Important

MCP adoption is accelerating across the AI ecosystem:

- **Anthropic's Claude Desktop** has native MCP support, allowing users to connect MCP servers for file system access, database queries, and API interactions [16].
- **OpenAI's ChatGPT** officially supports MCP servers as data sources for apps, including deep research integration and company knowledge [17].
- **Major IDEs** are integrating MCP for AI-powered development workflows.
- **Enterprise adoption** is growing, with organizations deploying MCP as the standard integration layer for AI agents [18, 19].

Industry analysts project MCP becoming the dominant protocol for AI-tool integration, analogous to how REST/HTTP became the standard for web APIs [20].

---

## Why BLT Needs MCP

### Current Limitations

BLT's capabilities are distributed across multiple services with different interfaces:

| Service | Interface | Authentication | Documentation |
|---------|-----------|---------------|---------------|
| BLT (Django) | Django views, legacy REST | Session-based | Source code |
| BLT-API | Cloudflare REST API | Static API key (X-BLT-API-Key) | README |
| BLT-Rewards | Undocumented | Unknown | Source code |
| BLT-Next | Frontend only | JWT via API | Source code |

[7, 9, 12]

Each integration point requires custom client code, different authentication handling, and separate documentation lookups.

### REST API Limitations

While BLT-API provides modern REST endpoints [9], REST APIs present inherent challenges for AI agents:

- **No capability discovery:** AI agents must know endpoint URLs, parameters, and response formats in advance — they cannot discover available functionality at runtime.
- **Fragmented documentation:** Each service documents its own API surface; there is no unified reference.
- **Inconsistent patterns:** Authentication mechanisms, error formats, and pagination schemes differ between services.
- **No workflow abstraction:** Complex operations (like triaging a vulnerability and creating a remediation plan) require chaining multiple API calls manually.

### Why AI Agents Struggle Today

AI agents operate most effectively when they can:

1. **Discover capabilities** at runtime through a standardized interface
2. **Understand tool semantics** through well-defined input/output schemas
3. **Chain operations** into complex workflows without manual guidance
4. **Receive structured context** that fits naturally into their prompt window

Current BLT services do not support any of these patterns. Each AI integration requires hand-coded tool definitions, hard-coded endpoint URLs, and custom error handling — making BLT effectively invisible to autonomous AI agents.

### Benefits of Standardized Tool Access

An MCP interface provides:

- **Runtime capability discovery:** Agents query the server's capabilities automatically — no documentation lookup required [2].
- **Standardized tool schemas:** JSON Schema 2020-12 defines exactly what each tool expects and returns [21].
- **Unified authentication:** A single auth flow (OAuth 2.1 / API key) across all capabilities [15].
- **Workflow templates:** Reusable prompt templates capture common security workflows [22].
- **Protocol-level safety:** Standardized error codes, logging, and capability scoping [4].

---

## Project Vision

BLT-MCP envisions a future where every AI agent, IDE, and chat interface can interact with the complete BLT ecosystem as naturally as a human contributor. An agent should be able to:

- Discover that a vulnerability report needs triage and automatically assess its severity
- Query related issues, contributor history, and BACON reward balances in a single session
- Create a remediation plan, track its progress, and award BACON tokens upon completion
- Report a new bug with full context about the affected domain and existing related issues

In the long term, BLT-MCP aims to:

- Make BLT the reference implementation for an MCP-served vulnerability management platform
- Enable AI-driven security workflows that reduce manual triage and remediation effort by 80%
- Provide the protocol layer that connects all BLT services (rewards, learning, scanning, education) into a unified AI-accessible platform
- Become a model for how OWASP projects can expose their capabilities through MCP

---

## Goals

### Functional Goals

- Implement 10+ MCP resources providing read-only access to BLT data via `blt://` URIs [1]
- Implement 4+ MCP tools for executing BLT actions (submit_issue, award_bacon, update_issue_status, add_comment) [1]
- Implement 3+ MCP prompt templates for security workflows (triage_vulnerability, plan_remediation, review_contribution) [1]
- Support both stdio (local/desktop) and Streamable HTTP (remote/cloud) transports [23]
- Support OAuth 2.1 with PKCE for HTTP transport and API key for stdio transport [15, 4]

### Technical Goals

- Achieve full conformance with the MCP specification (2025-11-25) [2]
- Implement guardrails at Layer 4 of the MCP architecture model: rate limiting, authentication, audit logging, response size caps [24]
- Enforce `additionalProperties: false` on all tool input schemas per OWASP MCP Security guidelines [4]
- Achieve >80% test coverage with in-memory transport tests, integration tests, and protocol compliance tests [25]
- Implement structured audit logging capturing all tool invocations with identity and timestamp [26]

### Community Goals

- Provide comprehensive documentation for contributors, maintainers, and end users
- Publish integration examples for Claude Desktop, Cline, ChatGPT, and custom Python agents
- Establish contribution guidelines aligned with BLT project standards
- Achieve production-grade reliability suitable for OWASP project hosting

### Research Goals

- Evaluate the existing TypeScript prototype and determine optimal architecture for the Python implementation [3]
- Investigate the BLT-API endpoint coverage and identify gaps where BLT-MCP resources/tools lack backend support [9]
- Document the authentication integration pattern between MCP's OAuth 2.1 and BLT-API's static API key model [15]
- Explore prompt chaining patterns for security workflow automation [22]

---

## Scope

### In Scope

- MCP server implementation in Python using the official MCP Python SDK (FastMCP) [23]
- Resources layer: Read-only access to issues, contributors, repositories, workflows, leaderboards, and rewards via `blt://` URIs
- Tools layer: Executable actions for submitting issues, awarding BACON tokens, updating issue status, and adding comments
- Prompts layer: Reusable task templates for vulnerability triage, remediation planning, and contribution review
- Dual transport support: stdio (local) and Streamable HTTP (remote)
- OAuth 2.1 with PKCE authentication for HTTP transport; environment-based API key for stdio transport
- Error handling with descriptive, actionable error messages using standard MCP error codes
- Audit logging for all tool invocations
- Unit tests using in-memory MCP transport, integration tests against mocked BLT-API
- Comprehensive documentation: setup guide, API reference, integration examples, security architecture

### Out of Scope

- Modifying the BLT-API, BLT Django application, or any BLT backend service
- Implementing a graphical user interface or web dashboard
- Adding new BLT features not exposed through existing BLT-API endpoints
- Supporting alternative AI protocols (Anthropic Function Calling, OpenAI Plugins, etc.)
- GraphQL endpoint support or transformation
- Mobile application integration
- Performance benchmarking beyond standard acceptance criteria
- Multi-tenant authentication (shared API key is a BLT-API limitation)

### Future Scope

- Integration with BLT-Rewards for direct BACON token management [12]
- BLT-Preflight integration for pre-contribution security guidance
- BLT-NetGuardian integration for scanning result ingestion
- BLT University integration for learning path recommendations
- Multi-agent orchestration support [27]
- Prompt chaining and cross-server workflow automation [22]
- ChatGPT app integration as a data-only MCP server [17]
- Migration to MCP Python SDK v2 when stable (target 2026-07-27) [23]

---

## Expected Deliverables

| # | Deliverable | Description |
|---|-------------|-------------|
| 1 | **MCP Server Application** | Production-grade Python MCP server implementing the 2025-11-25 specification |
| 2 | **Resource Layer** | 8+ `blt://` resource URIs with URI templates for parameterized access |
| 3 | **Tool Layer** | 4+ MCP tools with typed JSON Schema input validation |
| 4 | **Prompt Layer** | 3+ prompt templates with embedded resource context |
| 5 | **stdio Transport** | Local process transport for Claude Desktop and IDE integration |
| 6 | **Streamable HTTP Transport** | Remote transport for cloud-based and multi-client deployments |
| 7 | **Authentication Module** | OAuth 2.1 with PKCE for HTTP; environment API key for stdio |
| 8 | **BLT-API Client Library** | Purpose-built HTTP client for BLT-API communication |
| 9 | **Guardrail System** | Rate limiting, audit logging, response caps, schema validation |
| 10 | **Test Suite** | Unit tests, integration tests, protocol compliance tests |
| 11 | **Documentation** | Setup guide, API reference, security architecture, integration examples |
| 12 | **Client Configuration Examples** | Claude Desktop, Cline, ChatGPT, custom Python agent configs |

---

## Success Criteria

### Functional Criteria

- Server initializes and negotiates capabilities correctly per the MCP specification lifecycle [13]
- All resource URIs return correct, up-to-date data from BLT-API
- All tools execute successfully against BLT-API endpoints with proper error handling
- All prompt templates generate correct, context-aware prompts with embedded resource data
- Both stdio and Streamable HTTP transports operate correctly
- Authentication rejects unauthorized requests and permits authorized ones
- Rate limiting prevents abuse while allowing legitimate usage

### Technical Criteria

- >80% code coverage for core server logic
- Tool input schemas enforce `additionalProperties: false` [4]
- No secrets or credentials in source code (all via environment variables)
- Audit logging captures: tool name, parameters (sanitized), user identity, timestamp, success/failure status
- Server responds to simple queries within 5 seconds
- Error messages return standard MCP error codes with descriptive text

### Protocol Compliance Criteria

- Server passes MCP protocol compliance checks [25]
- Correct implementation of initialize → initialized → operation lifecycle
- Capabilities correctly declared and negotiated per session
- JSON-RPC 2.0 message format strictly followed
- Resource list, tool list, and prompt list correctly returned

---

## Stakeholders

### Developers

Software engineers building and maintaining the BLT-MCP server. They need clear architecture documentation, well-defined contribution guidelines, and a codebase that follows Python best practices. The primary audience for technical documentation and inline code comments.

### Maintainers

OWASP BLT project maintainers who will review, merge, and deploy BLT-MCP code. They need confidence in the server's security posture, protocol compliance, and compatibility with the broader BLT ecosystem. The existing BLT maintainers include the GSoC 2026 mentor team [28].

### Researchers

Security researchers and vulnerability analysts who use BLT for bug disclosure workflows. They benefit from MCP-accelerated triage, automated remediation planning, and AI-assisted contribution review. BLT-MCP's prompts layer is designed specifically for this audience.

### AI Agents

Autonomous and semi-autonomous AI agents that consume BLT-MCP's resources, tools, and prompts programmatically. These are the primary consumers of the MCP protocol interface. The server must be designed for agent-driven discovery and usage without human guidance for routine operations.

### Security Engineers

Application security engineers integrating BLT into their security toolchain. They need reliable, auditable access to vulnerability data and the ability to automate security workflows. BLT-MCP's guardrails and audit logging are designed for this audience's compliance requirements.

### Contributors

Open-source contributors to the BLT ecosystem who want to extend BLT-MCP or integrate it with other BLT services. They need clear documentation, well-structured code, and examples of how to add resources, tools, and prompts.

---

## Technology Stack (Expected)

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| **MCP SDK** | Python `mcp` package (v1.x) | Official MCP Python SDK providing FastMCP high-level API and low-level server API. v1.x (>=1.27,<2) is stable with full spec support. v2 alpha available but not production-ready [23]. |
| **Runtime** | Python 3.12+ | BLT-API is also Python on Cloudflare Workers — shared language reduces cognitive overhead. Python's async ecosystem (asyncio, httpx) suits MCP's concurrent request patterns [9]. |
| **Client Library** | `httpx` | Modern async HTTP client for Python. Supports connection pooling, timeouts, and streaming — all required for BLT-API communication. |
| **Schema Validation** | Pydantic v2 / JSON Schema 2020-12 | MCP SDK uses Pydantic for automatic JSON Schema generation from Python type annotations. `additionalProperties: false` enforcement prevents hallucinated tool parameters [23, 24]. |
| **HTTP Server** | ASGI (uvicorn/fastapi) | MCP Python SDK supports ASGI mounting for Streamable HTTP transport. FastAPI is well-documented and widely adopted in the Python ecosystem [23]. |
| **Authentication** | OAuth 2.1 with PKCE | Mandatory per MCP specification for HTTP transport [15]. CSA and OWASP guidelines both require OAuth 2.1 with PKCE for remote MCP connections [29, 30]. |
| **Testing** | pytest + in-memory transport | MCP Python SDK supports in-memory transport for tests without subprocesses or network. pytest is the standard Python testing framework [23, 25]. |
| **Containerization** | Docker | Standardized deployment environment. CI/CD consistency. Aligned with BLT project conventions. |
| **CI/CD** | GitHub Actions | BLT ecosystem uses GitHub Actions for CI/CD. Automated testing, linting, and deployment [7]. |
| **Documentation** | Markdown | Consistent with BLT documentation conventions. Deployable to GitHub Pages via BLT-docs [31]. |

---

## High-Level Workflow

The following diagram illustrates the end-to-end flow from an AI client through BLT-MCP to the BLT ecosystem:

```
┌─────────────────────────────────────────────────────────────────────────┐
│  End-to-End BLT-MCP Request Flow                                         │
│                                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────────┐   │
│  │   User   │───▶│  MCP     │───▶│  BLT-    │───▶│  BLT Ecosystem   │   │
│  │  Intent  │    │  Client  │    │  MCP     │    │                  │   │
│  │          │    │          │    │  Server  │    │  ┌────────────┐  │   │
│  │ "Triage  │    │ Claude   │    │          │    │  │ BLT-API    │  │   │
│  │  issue   │    │ Desktop  │    │  ┌──────┐│    │  │ (D1 DB)    │  │   │
│  │  #1234"  │    │ ChatGPT  │    │  │Tool  ││───▶│  └────────────┘  │   │
│  └──────────┘    │ Cline    │    │  │Layer ││    │                  │   │
│       │          │ Custom   │    │  │      ││    │  ┌────────────┐  │   │
│       ▼          │ Agent    │    │  │Auth  ││    │  │ BLT-Next   │  │   │
│  Natural         └──────────┘    │  │Layer ││    │  │ (GitHub    │  │   │
│  Language:       │              │  │      ││    │  │  Pages)    │  │   │
│  "Find issue     │  JSON-RPC 2.0│  │Rate  ││    │  └────────────┘  │   │
│   #1234,         │  over stdio │  │Limit  ││    │                  │   │
│   assess         │  or HTTP    │  │  │      ││    │  ┌────────────┐  │   │
│   severity,      │              │  │Audit ││    │  │ BLT-Rewards│  │   │
│   and plan       │              │  │Logger││    │  │ (BACON)    │  │   │
│   remediation"   │              │  └──────┘│    │  └────────────┘  │   │
│                  │              └──────────┘    └──────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘

Flow steps:
1. User expresses intent in natural language to an MCP-capable AI client
2. Client connects to BLT-MCP via stdio (local) or Streamable HTTP (remote)
3. BLT-MCP authenticates the request and validates against rate limits
4. BLT-MCP translates the MCP request into BLT-API REST calls
5. BLT-API processes the request against its D1 database or proxy chain
6. BLT-MCP enriches the response with context, applies guardrails, and returns
7. Client presents the result to the user
```

### Detailed Request Flow

```
┌─────────┐     ┌─────────┐     ┌──────────┐     ┌─────────┐     ┌──────────┐
│  MCP    │     │  BLT-   │     │  Auth    │     │  BLT-   │     │  BLT     │
│  Client │     │  MCP    │     │  Guard   │     │  API    │     │  Backend │
│         │     │  Server │     │          │     │  Client │     │          │
└────┬────┘     └────┬────┘     └────┬─────┘     └────┬────┘     └────┬─────┘
     │                │                │                │                │
     │ 1. Initialize  │                │                │                │
     │────────────────▶                │                │                │
     │                │                │                │                │
     │ 2. Capabilities│                │                │                │
     │◀───────────────│                │                │                │
     │                │                │                │                │
     │ 3. List Tools  │                │                │                │
     │────────────────▶                │                │                │
     │ 4. Tool Schemas│                │                │                │
     │◀───────────────│                │                │                │
     │                │                │                │                │
     │ 5. Call Tool   │                │                │                │
     │────────────────▶                │                │                │
     │                │ 6. Auth Check  │                │                │
     │                │───────────────▶│                │                │
     │                │◀───────────────│                │                │
     │                │    7. OK      │                │                │
     │                │                │                │                │
     │                │ 8. Rate Check │                │                │
     │                │───────────────▶│                │                │
     │                │◀───────────────│                │                │
     │                │    9. OK      │                │                │
     │                │                │                │                │
     │                │ 10. API Call  │                │                │
     │                │────────────────────────────────▶                │
     │                │                │                │                │
     │                │                │                │ 11. Data Query │
     │                │                │                │───────────────▶│
     │                │                │                │◀───────────────│
     │                │                │                │   12. Results  │
     │                │                │                │                │
     │                │ 13. API Resp  │                │                │
     │                │◀────────────────────────────────│                │
     │                │                │                │                │
     │                │ 14. Audit Log │                │                │
     │                │───────────────▶                │                │
     │                │◀───────────────│                │                │
     │                │   15. Done    │                │                │
     │                │                │                │                │
     │ 16. Tool Result│                │                │                │
     │◀───────────────│                │                │                │
```

---

## Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|------------|
| R1 | **MCP Python SDK v2 breaking changes** during GSoC timeline | Medium | High | Pin to `mcp>=1.27,<2` until v2 stable (target 2026-07-27). Document migration path. [23] |
| R2 | **BLT-API endpoint gaps** for resources/tools not backed by documented REST endpoints (repos, workflows, comments, BACON) | High | High | Investigate alternative BLT services; contribute endpoints to BLT-API if within scope; document as known limitations if out of scope. [9] |
| R3 | **GitHub repository access issues** preventing source code inspection of BLT-API and existing BLT-MCP | Confirmed | Medium | Use web search, PR descriptions, and README documentation; contact maintainers for direct access. |
| R4 | **Shared static API key limitation** prevents multi-tenant authentication and fine-grained access control | Confirmed | Medium | MCP-level authentication decouples from BLT-API auth; tool-level scoping managed within BLT-MCP. Future: BLT-API per-user key support. [9, 32] |
| R5 | **MCP specification changes** during project duration | Low | High | Target stable spec version (2025-11-25). Monitor draft for breaking changes. [2] |
| R6 | **Rate limiting misconfiguration** — BLT-MCP rate limits not aligned with BLT-API upstream limits | Medium | Medium | Make BLT-MCP rate limits configurable; start conservative and tune based on testing. [9] |
| R7 | **Authentication complexity** — OAuth 2.1 implementation requires an authorization server component | Medium | Medium | Implement simplified auth flow initially (API key for both transports); add OAuth 2.1 in dedicated sprint. [15, 30] |
| R8 | **Prompt injection via tool descriptions** — malicious or misleading tool definitions could manipulate AI behavior | Medium | High | Follow OWASP MCP Security Cheat Sheet guidelines: use `additionalProperties: false`, audit tool description changes, pin tool definitions. [4, 33] |

---

## Assumptions

- **BLT-API remains the primary backend interface** for BLT-MCP throughout the project duration. If BLT-API changes significantly, BLT-MCP may need corresponding updates.
- **The MCP Python SDK v1.x is stable** and suitable for production deployment. Migration to v2 will be evaluated when v2 stable is released.
- **The existing TypeScript BLT-MCP prototype** serves as a functional reference but will be rewritten in Python rather than extended. Decision documented in decision_log.md.
- **BLT-API's shared static API key** is sufficient for GSoC-scale deployments. Multi-tenant authentication is deferred.
- **BLT-Next architecture direction** (static frontend + Cloudflare Workers) represents the long-term BLT platform. BLT-MCP aligns with this target architecture.
- **MCP specification 2025-11-25** remains the stable target throughout the GSoC development period. Draft features (MRTR, elicitation) are not required.
- **The live BLT-API endpoint at `https://api.owaspblt.org/v2`** is available for integration testing. Local development uses mocked endpoints.
- **BLT-MCP will be tested** primarily against Claude Desktop and Cline, with secondary support for ChatGPT and custom agents.
- **The OWASP BLT maintainer team** (10 confirmed GSoC 2026 mentors) is available for guidance and code review during the project [28].

---

## References

### MCP Specification & Documentation

1. OWASP GSoC 2026 Ideas — BLT-MCP Description. https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
2. Model Context Protocol Specification (2025-11-25). https://modelcontextprotocol.io/specification/2025-11-25
3. OWASP-BLT/BLT-MCP — Existing Prototype. https://github.com/OWASP-BLT/BLT-MCP
4. OWASP MCP Security Cheat Sheet. https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
5. OWASP Foundation. https://owasp.org
6. OWASP BLT Official Project Page. https://owasp.org/www-project-bug-logging-tool/
7. OWASP-BLT/BLT — Main Repository. https://github.com/OWASP-BLT/BLT
8. OWASP-BLT GitHub Organization. https://github.com/orgs/OWASP-BLT
9. OWASP-BLT/BLT-API — REST API. https://github.com/OWASP-BLT/BLT-API
10. OWASP-BLT/BLT-Next. https://github.com/OWASP-BLT/BLT-Next
11. OWASP-BLT/BLT-Pages. https://github.com/OWASP-BLT/BLT-Pages
12. OWASP-BLT/BLT-Rewards — BACON Token System. https://github.com/OWASP-BLT/BLT-Rewards
13. MCP Lifecycle Documentation. https://modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle
14. MCP Schema Reference. https://modelcontextprotocol.io/specification/2025-11-25/schema
15. Model Context Protocol Specification (2025-06-18). https://modelcontextprotocol.io/specification/2025-06-18
16. Build an MCP Server Tutorial. https://modelcontextprotocol.io/docs/develop/build-server
17. OpenAI — Building MCP Servers for ChatGPT Apps. https://developers.openai.com/api/docs/mcp
18. IBM — MCP Architecture Patterns for Multi-Agent AI Systems. https://developer.ibm.com/articles/mcp-architecture-patterns-ai-systems/
19. Obot — MCP Enterprise Architecture Reference Guide. https://obot.ai/blog/mcp-enterprise-architecture-reference-guide/
20. Working Software — MCP in Practice for Software Architects. https://www.workingsoftware.dev/mcp-in-practice-what-software-architects-need-to-know-about-the-model-context-protocol/
21. MCP Protocol Features — Python SDK. https://py.sdk.modelcontextprotocol.io/protocol/
22. MCP Blog — Prompts for Workflow Automation. https://blog.modelcontextprotocol.io/posts/2025-07-29-prompts-for-automation/
23. MCP Python SDK Documentation. https://py.sdk.modelcontextprotocol.io/
24. Inspired by Frustration — Production MCP Server Architecture. https://inspiredbyfrustration.com/blog/mcp-server-architecture
25. MCP Architecture Patterns Handbook. https://github.com/ypollak2/mcp-handbook
26. OWASP MCP Top 10 — MCP08:2025 (Lack of Audit). https://owasp.org/www-project-mcp-top-10/
27. PADISO — AI Agents in Production: MCP Server Design Patterns. https://www.padiso.co/blog/ai-agents-production-mcp-server-design-patterns/
28. OWASP GSoC 2026 Ideas Page — BLT Mentors. https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
29. CSA — Agentic MCP Security Best Practices Guide. https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/
30. Descope — MCP Server Security Best Practices. https://www.descope.com/blog/post/mcp-server-security-best-practices
31. OWASP-BLT/BLT-docs. https://github.com/OWASP-BLT/BLT-docs
32. OWASP MCP Top 10 — MCP07:2025 (Insufficient Auth). https://github.com/OWASP/www-project-mcp-top-10/
33. OWASP MCP Top 10 — MCP03:2025 (Tool Poisoning). https://github.com/OWASP/www-project-mcp-top-10/
34. MCP Python SDK GitHub Repository. https://github.com/modelcontextprotocol/python-sdk
35. MCP GitHub Specification Repository. https://github.com/modelcontextprotocol/modelcontextprotocol
36. MCP Example Remote Server. https://github.com/modelcontextprotocol/example-remote-server
37. OWASP GenAI Security — Secure MCP Server Development. https://genai.owasp.org/resource/a-practical-guide-for-secure-mcp-server-development/
38. OWASP GenAI — Guide for Securely Using Third-Party MCP Servers. https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/
39. MCP Server Permissions and Access Control Guide. https://readfa.com/blog/mcp-server-permissions/
40. Elastic Path — MCP Architecture Patterns. https://www.elasticpath.com/blog/mcp-magic-moments-guide-to-llm-patterns
41. OWASP BLT GSoC Landing Page. https://gsoc.owaspblt.org/
42. OWASP BLT Documentation. https://github.com/OWASP-BLT/BLT-docs
