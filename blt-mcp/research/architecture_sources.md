# Architecture Sources

## Source 1: BLT-MCP GSoC 2026 Project Description

- **URL**: https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
- **Title**: BLT-MCP — Model Context Protocol Server for Complete BLT Interface
- **Author/Organization**: OWASP Foundation / OWASP-BLT
- **Summary**: Official GSoC 2026 project idea for BLT-MCP. Defines the three-layer MCP architecture (Resources, Tools, Prompts) for the BLT ecosystem.
- **Important Technical Details**:
  - **Resources**: Read-only access via `blt://` URIs (issues, repos, contributors, workflows, leaderboards, rewards)
  - **Tools**: Actions (`submit_issue`, `award_bacon`, `update_issue_status`, `add_comment`)
  - **Prompts**: Task templates (`triage_vulnerability`, `plan_remediation`, `review_contribution`)
  - Auth: JSON-RPC 2.0 with OAuth 2.0/API key
  - Transports: stdio and HTTP/SSE
  - Integration targets: Claude Desktop, custom AI agents, third-party tools
  - Goal: Position BLT as AI-agent-first platform, unify fragmented REST/GraphQL endpoints
  - Synergies: RAG bot capabilities, AI-guided recommendations, reputation scores, gamification data
- **Why It Is Useful**: This is the architectural blueprint for the entire BLT-MCP project. All design decisions should align with this vision.

---

## Source 2: Existing BLT-MCP Implementation

- **URL**: https://github.com/OWASP-BLT/BLT-MCP
- **Title**: OWASP-BLT/BLT-MCP
- **Author/Organization**: OWASP-BLT
- **Summary**: Current Node.js/TypeScript implementation of BLT-MCP. 7 stars, 23 forks, AGPL-3.0 license. Built for GSoC 2026.
- **Important Technical Details**:
  - Language: JavaScript (TypeScript source in `src/index.ts`)
  - Structure:
    ```
    blt-mcp/
    ├── src/index.ts          # Main server implementation
    ├── dist/                 # Compiled JS (generated)
    ├── package.json          # Dependencies
    ├── tsconfig.json         # TypeScript config
    ├── .env.example          # Env configuration
    └── mcp-config.json       # MCP client config example
    ```
  - Resources: `blt://issues`, `blt://issues/{id}`, `blt://repos`, `blt://repos/{id}`, `blt://contributors`, `blt://contributors/{id}`, `blt://workflows`, `blt://workflows/{id}`, `blt://leaderboards`, `blt://rewards`
  - Tools: `submit_issue`, `award_bacon`, `update_issue_status`, `add_comment`
  - Auth: BLT_API_KEY environment variable
  - Transport: stdio (primarily)
  - Config for Claude Desktop via `mcpServers` JSON
- **Why It Is Useful**: Starting point for development. Understanding the existing code structure is critical for extending it.

---

## Source 3: MCP Architecture Patterns Handbook

- **URL**: https://github.com/ypollak2/mcp-handbook
- **Title**: MCP Handbook - Architecture Patterns
- **Author/Organization**: Community (ypollak2)
- **Summary**: Comprehensive MCP handbook covering beginner to advanced topics. Includes architecture patterns decision tree, testing, security hardening, and production deployment.
- **Important Technical Details**:
  - Architecture decision tree:
    - Wraps external API? → API Wrapper pattern
    - Exposes database? → Database Explorer pattern
    - Accesses filesystem? → File Access pattern
    - Combines multiple sources? → Aggregator pattern
  - Three levels: Beginner (concepts, first server), Intermediate (patterns, testing, error handling), Advanced (security, production, performance)
  - Tool vs Resource vs Prompt decision guidance
  - Testing: in-memory transport, mocking, property-based testing
- **Why It Is Useful**: BLT-MCP fits the "API Wrapper" and "Aggregator" patterns. The handbook provides concrete implementation guidance.

---

## Source 4: IBM — MCP Architecture Patterns for Multi-Agent AI Systems

- **URL**: https://developer.ibm.com/articles/mcp-architecture-patterns-ai-systems/
- **Title**: Model Context Protocol architecture patterns for multi-agent AI systems
- **Author/Organization**: IBM Developer / Supal Chowdhury, Subrata Saha, Haitham Abdel Samad
- **Summary**: Explores client, server, and hybrid LLM placement patterns for scalable, modular multi-agent architectures using MCP.
- **Important Technical Details**:
  - Three placement patterns: client-side LLM, server-side LLM, hybrid
  - Multi-agent orchestration considerations
  - Real-world design trade-offs
- **Why It Is Useful**: Provides enterprise perspective on MCP architecture. BLT-MCP could support multi-agent scenarios with different LLM placement strategies.

---

## Source 5: Working Software — MCP in Practice for Software Architects

- **URL**: https://www.workingsoftware.dev/mcp-in-practice-what-software-architects-need-to-know-about-the-model-context-protocol/
- **Title**: MCP in practice: What software architects need to know about the Model Context Protocol
- **Author/Organization**: Working Software / Ole Wendland
- **Summary**: In-depth discussion of MCP architecture considerations including tool usage rates (80-90% Tools), latency trade-offs, and capability levels.
- **Important Technical Details**:
  - Tools used in 80-90% of MCP interactions
  - Three logical elements: Host (LLM), MCP Server (integration), Application (backend)
  - MCP Capability Levels:
    - Level 0: Wrap API in MCP server
    - Level 2: Add error descriptions and usage tips
    - Level 3: Provide real domain context
    - Level 4: Hypermedia-like autonomous capability discovery
  - Latency is a major issue — consider higher-level composite APIs
  - Trade-off between determinism and flexibility
- **Why It Is Useful**: Informs BLT-MCP design decisions about tool granularity, latency optimization, and capability level targeting.

---

## Source 6: Inspired by Frustration — How Production MCP Servers Are Built

- **URL**: https://inspiredbyfrustration.com/blog/mcp-server-architecture
- **Title**: MCP Server Architecture: How Production MCP Servers Are Actually Built
- **Author/Organization**: Inspired By Frustration
- **Summary**: Practical battle-tested guide on MCP server architecture focusing on the four layers: transport, JSON-RPC routing, tool definitions, and guardrails.
- **Important Technical Details**:
  - Four layers: Transport → JSON-RPC 2.0 Router → Tool Definitions → Guardrails
  - "Most MCP servers nail layer 3 and punt on layer 4 — the exact layer that matters in production"
  - Resources = nouns (data, readable), Tools = verbs (actions, with side effects)
  - Guardrails: rate limits, circuit breakers, response caps, auth
  - JSON-Schema best practices:
    - `additionalProperties: false` to prevent hallucinated fields
    - Prefer `enum` over free-form strings
    - Put units in field names (`timeout_ms` > `timeout`)
  - Consider REST+retrieval if only read access needed; MCP earns its overhead with multi-client, action-oriented, auditable use cases
- **Why It Is Useful**: Critical guidance for making BLT-MCP production-ready. Must invest in Layer 4 guardrails (rate limiting, circuit breakers, etc.).

---

## Source 7: PADISO — AI Agents in Production: MCP Server Design Patterns

- **URL**: https://www.padiso.co/blog/ai-agents-production-mcp-server-design-patterns/
- **Title**: AI Agents in Production: MCP Server Design Patterns
- **Author/Organization**: PADISO
- **Summary**: Production-ready MCP server design patterns covering monolithic, layered, stateful, and multi-agent orchestration patterns.
- **Important Technical Details**:
  - **Monolithic Tool Server**: All tools in one server (simple, good for small projects)
  - **Layered Tool Server**: Tools grouped by layer (data, API, utility) — better organization
  - **Stateful Tool Server**: Maintains session state across interactions
  - **Multi-Agent Orchestration**: Routing layer between clients and specialized servers
  - Transport decisions: HTTP (remote, cloud, multi-client), WebSocket (real-time, browser)
  - Observability: logging, error handling, resource management, caching, rate limiting
- **Why It Is Useful**: BLT-MCP likely fits the "Layered Tool Server" pattern (BLT API layer, business logic layer, utility layer). The patterns guide helps structure the implementation.

---

## Source 8: Obot — MCP Enterprise Architecture Reference Guide

- **URL**: https://obot.ai/blog/mcp-enterprise-architecture-reference-guide/
- **Title**: MCP Enterprise Architecture: The Complete Reference Guide
- **Author/Organization**: Obot / Bill Maxwell
- **Summary**: Enterprise MCP architecture emphasizing a control plane between clients and servers for identity, access, tokens, filtering, and audit logging.
- **Important Technical Details**:
  - Control plane handles: DCR, OAuth to IdP, token brokering, registry-based access, PII filtering, prompt injection detection, audit logging
  - MCP servers focus only on tools, resources, prompts
  - Registry-based access scoped to IdP groups
  - Tool-level access controls per registry
  - Agent-scoped composite servers for autonomous workloads
  - Offboarding: IdP group removal automatically ends MCP access
- **Why It Is Useful**: Enterprise architecture vision for MCP. BLT-MCP could benefit from a registry-based approach to tool access control, especially for multi-tenant BLT deployments.

---

## Source 9: MCP Example Remote Server (Official)

- **URL**: https://github.com/modelcontextprotocol/example-remote-server
- **Title**: modelcontextprotocol/example-remote-server
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Official example MCP server demonstrating all protocol features (tools, resources, prompts, sampling) with OAuth 2.0 separate auth server pattern.
- **Important Technical Details**:
  - Implements separate auth server pattern (recommended in spec)
  - Internal auth mode (same process, architecturally separate)
  - External auth mode (separate process — standard configuration)
  - 7 example tools, 100+ example resources with pagination/subscription
  - Supports Streamable HTTP (recommended) and SSE (legacy)
  - Implements OAuth 2.0 with PKCE
  - Multi-user session isolation with Redis-backed ownership
- **Why It Is Useful**: Official reference implementation. BLT-MCP should follow the same patterns, especially the separate auth server architecture.

---

## Source 10: Elastic Path — MCP Architecture Patterns: Routers, Tool Groups, Optimizers

- **URL**: https://www.elasticpath.com/blog/mcp-magic-moments-guide-to-llm-patterns
- **Title**: MCP Magic Moments: A Guide to LLM Patterns — Routers, Tool Groups, Optimizers
- **Author/Organization**: Elastic Path
- **Summary**: Four foundational MCP architecture patterns: Router, Tool Grouping, Optimizer, and Single Endpoint patterns for LLM-tool integration.
- **Important Technical Details**:
  - **Router Pattern**: Intelligent dispatcher analyzing requests and routing to appropriate handler
  - **Tool Grouping Pattern**: Logical drawers for tools — model selects group first, then specific tool
  - **Optimizer Pattern**: Pattern matching and grouping for common request patterns
  - **Single Endpoint Pattern**: Universal translator consolidating downstream API complexity
  - Patterns are compositional — production architecture may use multiple
  - Domain-Driven Design principles prevent infrastructure concerns leaking into business logic
- **Why It Is Useful**: BLT-MCP could benefit from Tool Grouping (organize BLT tools by domain — issues, rewards, users) and Single Endpoint (consolidate BLT-API complexity behind MCP interface).

---

## Source 11: OpenAI — Building MCP Servers for ChatGPT Apps

- **URL**: https://developers.openai.com/api/docs/mcp
- **Title**: Building MCP servers for ChatGPT Apps and API integrations
- **Author/Organization**: OpenAI
- **Summary**: Guide for building remote MCP servers that connect to ChatGPT as data-only apps, including deep research and company knowledge integration.
- **Important Technical Details**:
  - ChatGPT apps (formerly connectors) support MCP for data integration
  - Requires two read-only tools: `search` and `fetch`
  - Citation metadata generated when `url` is a non-empty string
  - OAuth with Client ID Metadata Documents (CIMD) supported
  - Supports public-client token exchange and private_key_jwt
- **Why It Is Useful**: BLT-MCP could potentially integrate with ChatGPT as a data source, providing BLT data to ChatGPT users. The `search`/`fetch` tool pattern is relevant.

---

## Source 12: MCP Blog — Prompts for Workflow Automation

- **URL**: https://blog.modelcontextprotocol.io/posts/2025-07-29-prompts-for-automation/
- **Title**: MCP Prompts: Building Workflow Automation
- **Author/Organization**: Model Context Protocol / Inna Harper
- **Summary**: Deep dive into MCP prompt system for building workflow automation, covering resource templates, prompt chains, and modular server architecture.
- **Important Technical Details**:
  - Static resources: specific content at unique URIs
  - Resource templates: URI patterns with parameters for dynamic content
  - Prompts are entry points to automation — can include embedded resources
  - Prompt Chains: execute multiple prompts in sequence
  - Cross-Server Workflows: coordinate multiple MCP servers
  - External Triggers: activate prompts via webhooks or schedules
- **Why It Is Useful**: BLT-MCP's Prompts layer (`triage_vulnerability`, `plan_remediation`, `review_contribution`) should follow these patterns. Prompt chaining enables complex security workflows.
