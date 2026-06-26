# MCP Handbook

> **Document Version:** 1.0
> **Status:** Draft
> **Date:** 2026-06-26

---

## Table of Contents

1. [Architecture](#architecture)
2. [JSON-RPC](#json-rpc)
3. [Resources](#resources)
4. [Tools](#tools)
5. [Prompts](#prompts)
6. [Roots](#roots)
7. [Sampling](#sampling)
8. [Transport](#transport)
9. [Authentication](#authentication)
10. [Lifecycle](#lifecycle)
11. [SDK](#sdk)
12. [Best Practices](#best-practices)
13. [Examples](#examples)
14. [Security](#security)
15. [Comparison with REST](#comparison-with-rest)
16. [Comparison with GraphQL](#comparison-with-graphql)
17. [Common Mistakes](#common-mistakes)
18. [Future Roadmap](#future-roadmap)

---

## Architecture

The Model Context Protocol (MCP) is an open protocol that enables seamless integration between LLM applications and external data sources and tools [MCP Specification, Source: modelcontextprotocol.io/specification/2025-11-25]. It provides a standardized way to connect LLMs with the context they need.

### High-Level Architecture

```
+------------------+       +------------------+       +------------------+
|                  |       |                  |       |                  |
|   MCP Host       |       |   MCP Client     |       |   MCP Server     |
|  (LLM App)       |------>|  (Connector)     |------>|  (Service)       |
|                  |       |                  |       |                  |
|  - Claude Desktop|       |  - SDK Instance  |       |  - Resources     |
|  - IDE Extension |       |  - Session Mgmt  |       |  - Tools         |
|  - ChatGPT       |       |  - Transport     |       |  - Prompts       |
|  - Custom Agent  |       |  - Auth          |       |  - Auth          |
|                  |       |                  |       |                  |
+------------------+       +------------------+       +------------------+
                                                             |
                                                             v
                                                    +------------------+
                                                    |                  |
                                                    |   Application    |
                                                    |  (Backend)       |
                                                    |                  |
                                                    |  - BLT-API       |
                                                    |  - Database      |
                                                    |  - External Svc  |
                                                    |                  |
                                                    +------------------+
```

### Three Roles

| Role | Description | Examples |
|------|-------------|----------|
| **Host** | LLM application that initiates connections | Claude Desktop, VS Code extension, ChatGPT |
| **Client** | Connector within the host that manages one server session | MCP Client SDK instance |
| **Server** | Service that provides context and capabilities | BLT-MCP, database server, API wrapper |

[MCP Specification, Source: modelcontextprotocol.io/specification/2025-11-25]

### Three Primitives

MCP servers offer three core features to clients:

```
+---------------------------------------------------+
|                  MCP Server                        |
|                                                    |
|   +-------------+   +----------+   +----------+   |
|   | Resources   |   |  Tools   |   | Prompts  |   |
|   | (read-only) |   | (actions)|   |(templates)|   |
|   |             |   |          |   |          |   |
|   | blt://issues|   | submit   |   | triage   |   |
|   | blt://repos |   | award    |   | plan     |   |
|   | blt://users |   | update   |   | review   |   |
|   +-------------+   +----------+   +----------+   |
|                                                    |
+---------------------------------------------------+
```

| Primitive | Control | Description | Example Use |
|-----------|---------|-------------|-------------|
| **Resources** | Application-controlled | Contextual data managed by the client application | File contents, API responses, DB rows |
| **Tools** | Model-controlled | Functions exposed to the LLM to take actions | API calls, data updates, side effects |
| **Prompts** | User-controlled | Interactive templates invoked by user choice | Slash commands, menu options, workflows |

[MCP Protocol Features, Source: py.sdk.modelcontextprotocol.io/protocol/]

---

## JSON-RPC

MCP uses JSON-RPC 2.0 as its wire protocol. All messages between clients and servers MUST follow the JSON-RPC 2.0 specification [MCP Specification, Source: modelcontextprotocol.io/specification/2025-11-25].

### Message Types

```
+------------------+          +------------------+
|     Client       |          |     Server       |
|                  |          |                  |
|  Request --------+--------->+------- Response  |
|                  |          |                  |
|  Notification ---+--------->+  (no response)   |
|                  |          |                  |
|  <----- Notification ------+|  (server->client)|
+------------------+          +------------------+
```

### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "submit_issue",
    "arguments": {
      "title": "Login page XSS vulnerability",
      "description": "Found XSS in login form",
      "severity": "high"
    }
  }
}
```

### Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Issue created successfully (ID: 1234)"
      }
    ]
  }
}
```

### Error Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32603,
    "message": "Internal error",
    "data": "Failed to submit issue: BLT-API returned 401"
  }
}
```

### Standard Error Codes

| Code | Meaning |
|------|---------|
| -32700 | Parse error |
| -32600 | Invalid request |
| -32601 | Method not found |
| -32602 | Invalid params |
| -32603 | Internal error |
| -32000 to -32099 | Server error (non-standard) |

### Notification

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized",
  "params": {}
}
```

Notifications have no `id` field — the server does not respond.

### Key Methods

| Method | Direction | Description |
|--------|-----------|-------------|
| `initialize` | Client → Server | Start session, negotiate capabilities |
| `initialized` | Client → Server | Signal ready for operation (notification) |
| `resources/list` | Client → Server | List available resources |
| `resources/read` | Client → Server | Read a specific resource |
| `tools/list` | Client → Server | List available tools |
| `tools/call` | Client → Server | Execute a tool |
| `prompts/list` | Client → Server | List available prompts |
| `prompts/get` | Client → Server | Get a specific prompt |
| `logging/setLevel` | Client → Server | Configure logging |
| `completion/complete` | Client → Server | Get argument completions |

---

## Resources

Resources are application-controlled, read-only data that the client can access. They are analogous to files or API responses — structured data that the LLM can use as context.

### URI Scheme

Resources are addressed by URI. The URI scheme is server-defined:

```
scheme://host/path
```

For BLT-MCP, this would be `blt://issues`, `blt://contributors`, etc.

### Resource Types

```
+-------------------------------------+
|          Resources                  |
|                                     |
|   +-----------+  +---------------+  |
|   |  Static   |  |  Template     |  |
|   |           |  |               |  |
|   | Fixed URI |  | URI pattern   |  |
|   | Known at  |  | with params   |  |
|   | connect   |  |               |  |
|   +-----------+  +---------------+  |
|                                     |
|   +-----------+  +---------------+  |
|   |  List     |  |  Subscribe    |  |
|   |           |  |               |  |
|   | Paginated |  | Real-time     |  |
|   | filtering |  | change notify |  |
|   +-----------+  +---------------+  |
+-------------------------------------+
```

### Static Resources

Represent specific content with unique URIs:

```json
{
  "uri": "blt://leaderboards",
  "name": "Leaderboard Rankings",
  "description": "Current BLT leaderboard with top contributors",
  "mimeType": "application/json"
}
```

### Resource Templates

URI patterns with parameters for dynamic content:

```
blt://issues/{id}
blt://contributors/{id}
blt://repos/{id}
```

Registration:

```json
{
  "uriTemplate": "blt://issues/{id}",
  "name": "Specific Issue",
  "description": "Details of a specific issue by ID",
  "mimeType": "application/json"
}
```

### Resource Lifecycle

```
Client                    Server
  |                         |
  |-- resources/list ------>|  Client asks: what resources exist?
  |                         |-- Returns list of resources + templates
  |<-- resource list -------|
  |                         |
  |-- resources/read ------->|  Client asks: give me this resource
  |   uri: blt://issues/42  |
  |                         |-- Server fetches from backend
  |<-- resource content ----|
  |                         |
  |-- subscribe ----------->|  Client asks: notify me on changes
  |                         |-- Server registers subscription
  |<-- notifications ------>|  Server pushes changes when they occur
```

### Best Practices for Resources

1. **Resources = nouns.** Data that the agent might want to prefetch. As practitioners note: "anything idempotent and cheap that the agent might want to prefetch goes on the resource surface" [Inspired by Frustration, Source: inspiredbyfrustration.com/blog/mcp-server-architecture].

2. **Use templates for parameterized access.** Instead of defining 1000 separate resources, define one template.

3. **Support `listChanged`.** When implemented, clients can subscribe to changes, eliminating polling.

4. **MIME types matter.** Declare the correct MIME type for each resource so clients can handle appropriately.

---

## Tools

Tools are model-controlled functions that the LLM can invoke to perform actions. They are the verbs of the MCP protocol — operations with side effects, computation, or external interactions.

### Tool Definition

Each tool has a name, description, and input schema:

```json
{
  "name": "submit_issue",
  "description": "Submit a new bug report to BLT",
  "inputSchema": {
    "type": "object",
    "properties": {
      "title": {
        "type": "string",
        "description": "Issue title"
      },
      "description": {
        "type": "string",
        "description": "Detailed description"
      },
      "severity": {
        "type": "string",
        "enum": ["low", "medium", "high", "critical"]
      },
      "repo_id": {
        "type": "string",
        "description": "Repository ID (optional)"
      }
    },
    "required": ["title", "description"],
    "additionalProperties": false
  }
}
```

### Tool Categories

```
+-----------------------------------------------------+
|                      Tools                          |
|                                                      |
|   +------------------+  +------------------------+  |
|   |  Read-only       |  |  Write/Side Effects   |  |
|   |                  |  |                        |  |
|   | - search_issues  |  | - submit_issue        |  |
|   | - get_user_stats |  | - award_bacon         |  |
|   | - list_orgs      |  | - update_issue_status |  |
|   +------------------+  | - add_comment         |  |
|                          +------------------------+  |
|                                                      |
|   +------------------+  +------------------------+  |
|   |  Composite       |  |  Utility               |  |
|   |                  |  |                        |  |
|   | - triage_issue   |  | - ping                |  |
|   | (calls multiple  |  | - format_report       |  |
|   |  backend APIs)   |  |                        |  |
|   +------------------+  +------------------------+  |
+-----------------------------------------------------+
```

### Tool Execution Flow

```
LLM/AI Agent               MCP Server              Backend (BLT-API)
    |                         |                         |
    |-- tools/list ---------->|                         |
    |                         |-- Return tool defs      |
    |<-- tool definitions ----|                         |
    |                         |                         |
    |-- tools/call ---------->|                         |
    |   name: submit_issue    |                         |
    |   args: {...}           |                         |
    |                         |-- POST /bugs ---------->|
    |                         |   X-BLT-API-Key: ...    |
    |                         |                         |-- Create bug
    |                         |<-- 201 Created ---------|
    |<-- result --------------|                         |
    |   "Issue #1234         |                         |
    |    created"            |                         |
```

### Best Practices for Tools

1. **Tools = verbs.** Actions with side effects or computation. The model decides when to invoke them.

2. **Design idempotent tools when possible.** If the same tool is called twice with identical params, the result should be the same (or safely repeatable).

3. **Use strict JSON Schema.** Set `additionalProperties: false` to prevent the model from inventing fields [OWASP MCP Security Cheat Sheet, Source: cheatsheetseries.owasp.org].

4. **Prefer `enum` over free-form strings.** The model picks from the list instead of hallucinating values [Inspired by Frustration, Source: inspiredbyfrustration.com/blog/mcp-server-architecture].

5. **Put units in field names.** `timeout_ms` beats `timeout` because descriptions get summarized.

6. **Meaningful error messages.** Return actionable error text, not just error codes.

7. **Guard against prompt injection.** Tool descriptions and parameters are a potential injection vector [OWASP MCP Top 10 MCP03, Source: owasp.org/www-project-mcp-top-10/].

---

## Prompts

Prompts are user-controlled, reusable templates that guide the user or model through specific tasks. They are invoked by user choice — like slash commands or menu options.

### Prompt Structure

```json
{
  "name": "triage_vulnerability",
  "description": "Guide the user through triaging a new vulnerability report",
  "arguments": [
    {
      "name": "issue_id",
      "description": "The ID of the issue to triage",
      "required": true
    }
  ]
}
```

### Prompt Categories for BLT-MCP

```
+---------------------------------------------------+
|                    Prompts                         |
|                                                    |
|   +------------------+  +----------------------+  |
|   |  Triage          |  |  Remediation         |  |
|   |                  |  |                      |  |
|   | triage_vuln      |  | plan_remediation     |  |
|   | assess_severity  |  | suggest_fix          |  |
|   | check_dup        |  | assign_owner         |  |
|   +------------------+  +----------------------+  |
|                                                    |
|   +------------------+  +----------------------+  |
|   |  Review          |  |  Report              |  |
|   |                  |  |                      |  |
|   | review_contrib   |  | generate_vuln_report |  |
|   | audit_workflow   |  | export_findings      |  |
|   +------------------+  +----------------------+  |
+---------------------------------------------------+
```

### Prompt with Embedded Resources

Prompts can include resource content for context:

```json
{
  "name": "triage_vulnerability",
  "description": "Triage a vulnerability issue",
  "arguments": [
    { "name": "issue_id", "required": true }
  ],
  "resource": {
    "uri": "blt://issues/{issue_id}",
    "description": "The issue details to triage"
  }
}
```

This embeds the issue data directly into the prompt context so the AI model can work with it without a separate `resources/read` call.

### Prompt Chains

Prompts can be chained for complex workflows:

```
triage_vulnerability
  └─> assess_severity
        └─> plan_remediation
              └─> assign_owner
                    └─> notify_stakeholders
```

[MCP Blog — Prompts for Automation, Source: blog.modelcontextprotocol.io]

### Best Practices for Prompts

1. **Prompts = templates.** They capture domain knowledge about how to perform a task. "Prompts are the entry points to your automation" [MCP Blog].

2. **Include resources.** Give the model context by embedding relevant resource data.

3. **Support argument completion.** Use the `completion/complete` method to help the model fill in arguments.

4. **Plan for chaining.** Design prompts that naturally compose into workflows.

---

## Roots

Roots are a **client-side** feature — the client provides the server with information about directories or URI boundaries it should operate within.

### Purpose

Roots define the scope of the server's operations. When a client connects, it can tell the server:

"What filesystem boundaries are relevant"
"Which URIs are in scope"
"What namespaces to use"

```
Client                              Server
  |                                   |
  |-- initialize ------------------->|
  |   capabilities: {                |
  |     roots: {                     |
  |       listChanged: true          |
  |     }                            |
  |   }                              |
  |                                   |
  |-- roots/list ------------------>|
  |   (server asks for roots)        |
  |                                   |
  |<-- roots ------------------------|
  |   [                              |
  |     { uri: "blt://" },           |
  |     { uri: "file:///projects" }  |
  |   ]                              |
```

### When to Use Roots

- **Filesystem servers**: Tell the server which directories are accessible
- **BLT-MCP**: Define which BLT namespaces are in scope (e.g., only certain organizations)
- **Multi-tenant setups**: Restrict server operations to specific tenants

---

## Sampling

Sampling is a **client-side** feature that enables server-initiated LLM interactions. The server can request the client to generate a response from an LLM.

### Why Sampling Matters

Sampling enables recursive agentic behaviors:

```
Server detects ambiguous issue description
  └─> Server sends sampling request to client
        └─> Client asks LLM to clarify
              └─> LLM response sent back to server
                    └─> Server continues processing
```

### Sampling Request

```json
{
  "method": "sampling/createMessage",
  "params": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "This issue description is ambiguous. Can you clarify?"
        }
      }
    ],
    "maxTokens": 500,
    "systemPrompt": "You are a security triage assistant."
  }
}
```

### When to Use Sampling

- **Complex triage**: Server needs LLM reasoning to classify an issue
- **Natural language processing**: Server needs to extract structured data from text
- **Decision support**: Server needs AI guidance for routing or prioritization

Sampling is optional — not all MCP servers implement it. BLT-MCP may leverage sampling for advanced triage prompts.

---

## Transport

MCP supports multiple transport mechanisms. The choice of transport affects security, latency, deployment model, and client compatibility.

### Transport Comparison

```
            +---------------------------+
            |     MCP Transports        |
            +---------------------------+
            |                           |
            v                           v
    +-----------------+       +-------------------+
    |     stdio       |       |  Streamable HTTP  |
    |                 |       |                   |
    |  Local          |       |  Remote           |
    |  Subprocess     |       |  HTTP POST        |
    |  No network     |       |  SSE streaming    |
    |  Environment    |       |  OAuth 2.1        |
    |  credentials    |       |  CORS             |
    +-----------------+       +-------------------+
                                       |
                                       v
                               +-------------------+
                               |     SSE (legacy)  |
                               |                   |
                               |  One-way server   |
                               |  to client stream |
                               |  Deprecated       |
                               +-------------------+
```

### stdio

```
┌─────────────┐   stdin/stdout    ┌──────────────┐
│             │──────────────────>│              │
│   Client    │                   │  MCP Server  │
│  (Process)  │<──────────────────│  (Process)   │
└─────────────┘                   └──────────────┘
```

**Best for:** Local/desktop integration (Claude Desktop, IDE extensions)

**Characteristics:**
- Server runs as a subprocess
- Messages over stdin/stdout
- No network exposure
- Credentials from environment variables (not HTTP auth)
- Simple, secure, low latency

**Configuration example:**

```json
{
  "mcpServers": {
    "blt": {
      "command": "python",
      "args": ["-m", "blt_mcp.server"],
      "env": {
        "BLT_API_BASE": "https://blt.owasp.org/api",
        "BLT_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

[BLT-MCP README, Source: github.com/OWASP-BLT/BLT-MCP]

### Streamable HTTP

```
┌─────────────┐   POST /mcp       ┌──────────────┐
│             │──────────────────>│              │
│   Client    │   SSE stream      │  MCP Server  │
│  (Browser)  │<──────────────────│  (HTTP)      │
│             │                   │              │
└─────────────┘                   └──────────────┘
```

**Best for:** Remote/cloud integration (ChatGPT, multi-client scenarios)

**Characteristics:**
- Server runs as HTTP service
- Clients make POST requests
- Responses can stream via SSE
- Supports OAuth 2.1 with PKCE
- CORS configuration for browser clients
- ASGI mounting for existing servers

**Key considerations:**
- "If the only requirement is 'let Claude read our docs,' a REST endpoint + retrieval is simpler. MCP earns its overhead when multiple AI clients hit the same surface, the agent needs to take actions, and per-tool audit logs matter" [Inspired by Frustration].

---

## Authentication

MCP's authentication model depends on the transport.

### stdio Authentication

For stdio transport, authentication is handled through the environment, not the protocol:

- Credentials are passed as environment variables to the subprocess
- The server uses these credentials to authenticate with downstream services (e.g., BLT-API)
- No protocol-level auth — the OS process boundary provides isolation

```
Server Process Environment:
  BLT_API_KEY=sk-xxxxx
  BLT_API_BASE=https://blt.owasp.org/api
```

### HTTP Authentication (OAuth 2.1)

For Streamable HTTP transport, the MCP specification (2025-06-18) mandates an authorization framework [MCP Spec 2025-06-18, Source: modelcontextprotocol.io/specification/2025-06-18]:

```
+--------+                               +---------------+
|        |-- (A) Authorization Request ->|   Resource    |
|        |                               |   Owner (RO)  |
|        |<-- (B) Authorization Grant ---|               |
|        |                               +---------------+
|        |                               +---------------+
| MCP    |-- (C) Auth Request ---------->| Authorization |
| Client |                               |   Server (AS) |
|        |<-- (D) Access Token ----------|               |
|        |                               +---------------+
|        |                               +---------------+
|        |-- (E) Tool Call + Token ----->|   MCP Server  |
|        |                               | (Resource Svr)|
|        |<-- (F) Response --------------|               |
+--------+                               +---------------+
```

**Key requirements:**
- MCP servers are OAuth Resource Servers (separate from the Authorization Server)
- OAuth 2.1 with PKCE is mandatory for all public clients
- Use RFC 8414 (OAuth 2.0 Authorization Server Metadata) for server discovery
- Scopes should be defined at the tool level, not server level

[Descope — MCP Server Security, Source: www.descope.com/blog/post/mcp-server-security-best-practices; CSA Guide, Source: labs.cloudsecurityalliance.org]

### BLT-API Key Authentication

The existing BLT-MCP prototype and BLT-API use shared static API keys:

```
X-BLT-API-Key: sk_xxxxx
```

[BLT-API PR #89, Source: github.com/OWASP-BLT/BLT-API/pull/89]

For BLT-MCP, the API key is configured in the MCP server's environment and used to authenticate with the BLT-API backend — it should never be exposed to the AI model.

### Security Best Practices

From the OWASP MCP Security Cheat Sheet [Source: cheatsheetseries.owasp.org]:

- **Do:** Use scoped, per-server credentials — never share tokens across servers
- **Do:** Prefer ephemeral, short-lived tokens over long-lived keys
- **Do:** Require explicit user confirmation for destructive operations
- **Don't:** Auto-approve tool calls without showing full parameters
- **Don't:** Trust tool descriptions blindly (they are a prompt injection vector)
- **Don't:** Store secrets in environment variables accessible to the LLM

---

## Lifecycle

MCP defines a rigorous lifecycle for client-server connections [MCP Lifecycle, Source: modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle].

### Lifecycle Phases

```
Phase 1: Initialization
+------------------+                   +------------------+
|                  |-- initialize ---->|                  |
|     Client       |   (version,       |     Server       |
|                  |    capabilities)  |                  |
|                  |<-- response ------|                  |
|                  |   (version,       |                  |
|                  |    capabilities)  |                  |
|                  |                   |                  |
|                  |-- initialized --->|                  |
|                  |   (notification)  |                  |
+------------------+                   +------------------+
         |                                      |
         v                                      v
Phase 2: Operation
+------------------+                   +------------------+
|                  |-- resources/list ->|                  |
|     Client       |-- tools/call ----->|     Server       |
|                  |-- prompts/get ---->|                  |
|                  |<-- responses ------|                  |
+------------------+                   +------------------+
         |                                      |
         v                                      v
Phase 3: Shutdown
+------------------+                   +------------------+
|     Client       |--- close -------->|     Server       |
|                  |  (transport-level)|                  |
+------------------+                   +------------------+
```

### Initialization in Detail

**Step 1: Client sends `initialize` request**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-11-25",
    "capabilities": {
      "roots": { "listChanged": true },
      "sampling": {}
    },
    "clientInfo": {
      "name": "claude-desktop",
      "version": "1.0.0"
    }
  }
}
```

**Step 2: Server responds with its capabilities**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-11-25",
    "capabilities": {
      "resources": { "subscribe": true, "listChanged": true },
      "tools": { "listChanged": true },
      "prompts": { "listChanged": true },
      "logging": {}
    },
    "serverInfo": {
      "name": "blt-mcp",
      "version": "1.0.0"
    }
  }
}
```

**Step 3: Client sends `initialized` notification**

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized",
  "params": {}
}
```

### Version Negotiation

During initialization:
- Client sends its preferred protocol version
- If the server supports it, it responds with the same version
- If not, the server responds with a version it does support
- If the client does not support the server's version, it must disconnect

### Capability Negotiation

| Capability | Side | Description |
|-----------|------|-------------|
| `resources` | Server | Offers readable resources with optional subscribe/listChanged |
| `tools` | Server | Exposes callable tools with optional listChanged |
| `prompts` | Server | Offers prompt templates with optional listChanged |
| `logging` | Server | Emits structured log messages |
| `completions` | Server | Provides argument completion suggestions |
| `roots` | Client | Provides filesystem roots to server |
| `sampling` | Client | Supports LLM sampling requests |

---

## SDK

The MCP Python SDK is the official Python implementation of the Model Context Protocol [MCP Python SDK, Source: py.sdk.modelcontextprotocol.io].

### Version Status

| Version | Status | Release Target |
|---------|--------|----------------|
| v1.x | Stable (maintenance) | Current |
| v2.0 alpha | Pre-release | Beta: 2026-06-30, Stable: 2026-07-27 |

> v1.x is in maintenance mode. Projects should pin `mcp>=1.27,<2` before v2 stable release [MCP Python SDK README, Source: github.com/modelcontextprotocol/python-sdk].

### SDK Architecture

```
+---------------------------------------------+
|              MCP Python SDK                  |
|                                              |
|   +-----------------+  +-----------------+  |
|   |   High-Level    |  |   Low-Level     |  |
|   |   (FastMCP)     |  |   (Server)      |  |
|   |                 |  |                 |  |
|   | @mcp.tool()     |  | server = Server |  |
|   | @mcp.resource() |  | handler = ...   |  |
|   | @mcp.prompt()   |  |                 |  |
|   +-----------------+  +-----------------+  |
|                                              |
|   +-----------------+  +-----------------+  |
|   |   Shared        |  |   Transports    |  |
|   |                 |  |                 |  |
|   | Version consts  |  | stdio           |  |
|   | JSON Schema     |  | SSE             |  |
|   | Pydantic models |  | Streamable HTTP |  |
|   +-----------------+  +-----------------+  |
+---------------------------------------------+
```

### FastMCP (High-Level API)

The FastMCP API uses decorators for concise server definition:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("BLT-MCP", json_response=True)

@mcp.tool()
def submit_issue(title: str, description: str, severity: str = "medium") -> str:
    """Submit a new bug report to BLT"""
    # Implementation calls BLT-API
    return f"Issue created: {result.id}"

@mcp.resource("blt://issues/{issue_id}")
def get_issue(issue_id: str) -> str:
    """Get issue details by ID"""
    return json.dumps(fetch_issue(issue_id))

@mcp.prompt()
def triage_vulnerability(issue_id: str) -> str:
    """Triage a vulnerability report"""
    return f"Review issue {issue_id} and assess its severity, impact, and priority."
```

[Source: py.sdk.modelcontextprotocol.io]

### Key SDK Features

- **Automatic capability declaration**: Capabilities are auto-declared when handlers are registered
- **JSON Schema generation**: Pydantic models auto-generate JSON schemas via `model_json_schema()`
- **ASGI mounting**: Servers can mount to existing ASGI applications (FastAPI, Starlette)
- **In-memory transport**: Enables testing without network
- **OAuth 2.1 support**: Built-in support for authorization flows

---

## Best Practices

### Architecture Best Practices

1. **Follow the four-layer model**: Transport → JSON-RPC Router → Tool Definitions → Guardrails [Inspired by Frustration].

2. **Use the right architecture pattern**:
   - Wrapping an external API? → API Wrapper pattern
   - Exposing a database? → Database Explorer pattern
   - Combining multiple sources? → Aggregator pattern

   [MCP Architecture Patterns Handbook, Source: github.com/ypollak2/mcp-handbook]

3. **Separate concerns**: "A control plane that handles auth, a registry system that handles access, audit logging that handles visibility, and filters that handle data protection" [Obot Enterprise Architecture, Source: obot.ai].

### Design Best Practices

4. **Resources = nouns, Tools = verbs.** Don't conflate them. "If you model everything as a tool, the agent has to guess from descriptions which ones are safe to call speculatively" [Inspired by Frustration].

5. **Design for idempotency.** Tool calls may be retried. Make them safe to execute multiple times.

6. **Keep tool scope narrow.** One tool = one action. Composite workflows belong in Prompts.

7. **Provide rich descriptions.** The LLM uses tool/resource/prompt descriptions to decide what to invoke. Bad descriptions = bad agent decisions.

### Implementation Best Practices

8. **Use strict JSON Schema.** `additionalProperties: false` prevents hallucinated parameters.

9. **Prefer enums over strings.** `enum: ["low", "medium", "high"]` gives the model clear choices.

10. **Put units in field names.** `timeout_ms` beats `timeout` (descriptions get summarized away).

11. **Validate inputs at the MCP layer.** Don't trust the model — validate before forwarding to backend.

12. **Return meaningful errors.** "Failed to submit issue: BLT-API returned 401 (unauthorized)" is better than "Error -32603".

### Testing Best Practices

13. **Use in-memory transport for tests.** No network, no subprocess — fast and deterministic.

14. **Test capability negotiation.** Verify the server correctly declares and negotiates capabilities.

15. **Test tool schemas.** Verify JSON Schema generation is correct and strict.

16. **Test error paths.** Network failures, auth failures, validation errors — all must return proper MCP error responses.

---

## Examples

### Example 1: Simple Tool Server (Python)

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("BLT-MCP")

@mcp.tool()
def search_bugs(query: str, status: str = "open") -> str:
    """Search BLT bugs by query string, optionally filtered by status"""
    import httpx
    response = httpx.get(
        f"https://blt.owasp.org/api/v2/bugs/search",
        params={"q": query, "status": status},
        headers={"X-BLT-API-Key": "..."}
    )
    return response.text

@mcp.resource("blt://stats")
def get_stats() -> str:
    """Get overall BLT statistics"""
    return json.dumps({
        "total_bugs": 15000,
        "open_bugs": 3200,
        "contributors": 850
    })

@mcp.prompt()
def plan_remediation(issue_id: str) -> str:
    """Create a remediation plan for a verified issue"""
    return f"Create a step-by-step remediation plan for issue {issue_id}"
```

### Example 2: Client Connection (Python)

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def connect_to_blt():
    server_params = StdioServerParameters(
        command="python",
        args=["-m", "blt_mcp.server"],
        env={"BLT_API_KEY": "sk-xxxxx"}
    )

    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # List tools
            tools = await session.list_tools()
            print([t.name for t in tools])

            # Call tool
            result = await session.call_tool(
                "search_bugs",
                {"query": "XSS", "status": "open"}
            )
```

### Example 3: Claude Desktop Configuration

```json
{
  "mcpServers": {
    "blt": {
      "command": "python",
      "args": ["-m", "blt_mcp.server"],
      "env": {
        "BLT_API_BASE": "https://blt.owasp.org/api",
        "BLT_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

[Source: github.com/OWASP-BLT/BLT-MCP]

### Example 4: Resource with Template

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("BLT-MCP")

@mcp.resource("blt://issues/{issue_id}")
def get_issue(issue_id: str) -> str:
    """Get a specific issue by ID"""
    response = httpx.get(
        f"https://blt.owasp.org/api/v2/bugs/{issue_id}",
        headers={"X-BLT-API-Key": "..."}
    )
    return response.text

@mcp.resource("blt://leaderboards")
def get_leaderboard() -> str:
    """Get the current leaderboard"""
    return json.dumps(leaderboard_data)
```

---

## Security

Security is a first-class concern for MCP. As noted by OWASP: "MCP servers operate with delegated user permissions — the critical connection point between AI assistants and external tools, APIs, and data sources. Unlike traditional APIs, MCP servers operate with delegated user permissions, dynamic tool-based architectures, and chained tool calls, increasing the potential impact of a single vulnerability" [OWASP GenAI Security Guide, Source: genai.owasp.org].

### Threat Categories

Based on the OWASP MCP Top 10 [Source: owasp.org/www-project-mcp-top-10/]:

```
+------------------------------------------------------+
|                  MCP Threat Landscape                 |
|                                                      |
|  MCP01: Token Mismanagement & Secret Exposure        |
|  MCP02: Privilege Escalation via Scope Creep         |
|  MCP03: Tool Poisoning                               |
|  MCP04: Software Supply Chain Attacks                |
|  MCP05: Command Injection & Execution                |
|  MCP07: Insufficient Authentication & Authorization  |
|  MCP08: Lack of Audit and Telemetry                  |
|  MCP09: Shadow MCP Servers                           |
|  MCP10: Context Injection & Over-Sharing             |
+------------------------------------------------------+
```

### Security Controls Matrix

| Threat | Control | Implementation |
|--------|---------|----------------|
| MCP01: Token Exposure | Never expose credentials to the LLM | Credentials in server env, not tool params |
| MCP02: Scope Creep | Tool-level authorization | OAuth scopes per tool, not per server |
| MCP03: Tool Poisoning | Validate tool descriptions | Pin tool definitions, scan with mcp-scan |
| MCP05: Injection | Strict JSON Schema input validation | `additionalProperties: false`, enum types |
| MCP07: Insufficient Auth | OAuth 2.1 with PKCE | Mandatory for HTTP transport |
| MCP08: Audit Gaps | Comprehensive logging | Log every tool call with identity + timestamp |

### Defense in Depth

"Security requires defense in depth: no single control is sufficient" [CSA Guide, Source: labs.cloudsecurityalliance.org].

```
Layer 1: Authentication
  - OAuth 2.1 with PKCE (HTTP)
  - Environment credentials (stdio)
  - mTLS for mutual authentication

Layer 2: Authorization
  - Tool-level scopes
  - RBAC/ABAC for multi-tenant
  - Per-request permission evaluation

Layer 3: Input Validation
  - JSON Schema with additionalProperties: false
  - Domain allowlists for URLs
  - Enum constraints on categorical fields

Layer 4: Sandboxing
  - Container isolation
  - Network restrictions
  - Filesystem containment

Layer 5: Monitoring
  - Audit logging
  - Behavioral anomaly detection
  - Tool description change alerts
```

### BLT-MCP Security Requirements

From the OWASP MCP Security Cheat Sheet [Source: cheatsheetseries.owasp.org]:

| Requirement | Rationale |
|-------------|-----------|
| Tool schemas use `additionalProperties: false` | Prevents LLM from inventing parameters |
| API keys stored in environment, not code | Prevents credential leakage |
| No credentials exposed to LLM in tool results | Prevents prompt injection from extracting secrets |
| Destructive operations require confirmation | Prevents accidental or malicious data loss |
| Audit logging enabled | Tracks all AI agent actions for incident response |

---

## Comparison with REST

| Aspect | REST | MCP |
|--------|------|-----|
| **Discovery** | Out-of-band (documentation, OpenAPI) | In-band (capability negotiation at connect) |
| **Semantics** | HTTP methods on resources | Resources (data), Tools (actions), Prompts (templates) |
| **Wire Protocol** | HTTP 1.1/2 | JSON-RPC 2.0 |
| **Auth Model** | Varies (Bearer, API key, OAuth) | OAuth 2.1 (HTTP) or environment (stdio) |
| **State** | Stateless by design | Stateful session with lifecycle |
| **Streaming** | SSE or WebSocket out-of-band | Built-in via Streamable HTTP |
| **Schema** | OpenAPI (separate document) | Built-in JSON Schema in every tool definition |
| **Agent Fit** | Poor — no semantic tool descriptions | Excellent — designed for LLM consumption |
| **When to use** | Human-facing CRUD APIs | AI-agent-facing capability exposure |
| **Latency** | Lower (mature caching) | Higher (LLM reasoning between calls) |

**Key insight**: "MCP earns its overhead when multiple AI clients hit the same surface, the agent needs to take actions (not just read), and per-tool audit logs matter" [Inspired by Frustration, Source: inspiredbyfrustration.com/blog/mcp-server-architecture].

---

## Comparison with GraphQL

| Aspect | GraphQL | MCP |
|--------|---------|-----|
| **Query Language** | Custom query syntax (SDL) | JSON-RPC 2.0 with method names |
| **Data Fetching** | Client specifies exact fields | Server defines resource URIs |
| **Mutations** | Single mutation endpoint with type system | Tools with JSON Schema definitions |
| **Subscriptions** | WebSocket-based | SSE or Streamable HTTP |
| **Schema** | SDL schema definition | Inline JSON Schema per tool/resource |
| **Discovery** | Introspection query | Capability negotiation at connect |
| **Auth** | Custom (often Bearer token) | OAuth 2.1 with PKCE (spec-mandated) |
| **Agent Fit** | Moderate — requires query generation, no tool semantics | Excellent — designed for LLM agent interaction |
| **Caching** | Excellent (field-level) | Limited (resource-level) |
| **N+1 Problem** | Well-known issue | Not applicable (tool-oriented) |

**Key insight**: GraphQL is optimized for flexible, client-driven data fetching. MCP is optimized for LLM-driven tool use and workflow execution. They serve different purposes — an application could expose both a GraphQL API for human developers and an MCP server for AI agents.

---

## Common Mistakes

### 1. Modeling Everything as Tools

> "If you model everything as a tool, the agent has to guess from descriptions which ones are safe to call speculatively, and it will guess wrong on the expensive ones."

**Fix**: Put idempotent, cheap data access on the resource surface. Reserve tools for actions with side effects.

### 2. Weak JSON Schema

> "The schema in a tool definition isn't just documentation — it's the contract the model generates against."

**Fix**: Always set `additionalProperties: false`, use `enum` constraints, put units in field names.

[Inspired by Frustration, Source: inspiredbyfrustration.com/blog/mcp-server-architecture]

### 3. Ignoring Guardrails

> "Most MCP servers nail layer 3 (tool definitions) and punt on layer 4 (guardrails) — which is exactly the layer that matters when Claude hits your server 60 times a minute at 2am."

**Fix**: Implement rate limiting, circuit breakers, response caps, auth, and audit logging.

### 4. Descriptive but Unspecific Tool Names

Bad: `process_data`
Good: `submit_bug_report`

Bad: `get_info`
Good: `get_contributor_statistics`

### 5. Exposing Credentials to the LLM

Never return API keys, tokens, or secrets in tool results. The LLM may inadvertently include them in responses or store them in context.

### 6. Forgetting Lifecycle Correctness

Every MCP server must implement the initialize → initialized → operation lifecycle correctly. Skipping this breaks protocol compliance.

### 7. Prompt Injection via Tool Descriptions

Tool descriptions are a potential injection vector [OWASP MCP Top 10, Source: owasp.org/www-project-mcp-top-10/]. An attacker who can modify tool descriptions can manipulate the LLM's behavior. Pin tool definitions and use `mcp-scan` for detection.

---

## Future Roadmap

### MCP Python SDK v2

The v2 SDK is in alpha with beta targeting 2026-06-30 and stable release on 2026-07-27 [MCP Python SDK README, Source: github.com/modelcontextprotocol/python-sdk].

Expected improvements:
- Simplified API
- Better typing support
- Improved transport handling
- Enhanced auth support

### Protocol Evolution

Based on the draft specification [MCP Spec Draft, Source: modelcontextprotocol.io/specification/draft/basic]:

- **Versioning and Compatibility**: Formalized version negotiation and backward compatibility
- **Message Patterns**: Multi-round-trip requests (MRTR) for complex interactions
- **Elicitation**: Server-initiated user information requests with various field types

### Broader Ecosystem

Major AI platforms are adopting MCP:

- **OpenAI**: MCP support for ChatGPT apps, deep research, and company knowledge [OpenAI MCP Guide, Source: developers.openai.com/api/docs/mcp]
- **Claude Desktop**: Native MCP client with server configuration
- **Google**: MCP support in Gemini and IDX
- **Microsoft**: VS Code MCP extensions and GitHub Copilot integration

### BLT-MCP Roadmap

Aligning with the GSoC 2026 timeline:

1. Foundation: MCP server skeleton + BLT-API client (Weeks 1-3)
2. Resources layer: all `blt://` URIs (Weeks 4-6)
3. Tools layer: actions with validation (Weeks 7-9)
4. Prompts layer: workflow templates (Weeks 10-11)
5. HTTP transport + security hardening (Week 12)
6. Documentation + final polish (Weeks 13-14)

---

## References

1. MCP Specification (2025-11-25) — https://modelcontextprotocol.io/specification/2025-11-25
2. MCP Specification (2025-06-18) — https://modelcontextprotocol.io/specification/2025-06-18
3. MCP Lifecycle — https://modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle
4. MCP Protocol Features (Python SDK) — https://py.sdk.modelcontextprotocol.io/protocol/
5. MCP Python SDK — https://py.sdk.modelcontextprotocol.io/
6. MCP Python SDK README — https://github.com/modelcontextprotocol/python-sdk/blob/main/README.md
7. Build an MCP Server Tutorial — https://modelcontextprotocol.io/docs/develop/build-server
8. Build an MCP Client Tutorial — https://modelcontextprotocol.io/docs/develop/build-client
9. OWASP MCP Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
10. OWASP GenAI Security — MCP Server Development Guide — https://genai.owasp.org/resource/a-practical-guide-for-secure-mcp-server-development/
11. OWASP MCP Top 10 — https://owasp.org/www-project-mcp-top-10/
12. CSA Agentic MCP Security Best Practices — https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/
13. Descope — MCP Server Security — https://www.descope.com/blog/post/mcp-server-security-best-practices
14. Inspired by Frustration — Production MCP Architecture — https://inspiredbyfrustration.com/blog/mcp-server-architecture
15. MCP Architecture Patterns Handbook — https://github.com/ypollak2/mcp-handbook
16. MCP Blog — Prompts for Automation — https://blog.modelcontextprotocol.io/posts/2025-07-29-prompts-for-automation/
17. IBM — MCP Architecture Patterns — https://developer.ibm.com/articles/mcp-architecture-patterns-ai-systems/
18. PADISO — AI Agents in Production — https://www.padiso.co/blog/ai-agents-production-mcp-server-design-patterns/
19. Obot — MCP Enterprise Architecture — https://obot.ai/blog/mcp-enterprise-architecture-reference-guide/
20. Elastic Path — MCP Patterns — https://www.elasticpath.com/blog/mcp-magic-moments-guide-to-llm-patterns
21. OpenAI — MCP for ChatGPT — https://developers.openai.com/api/docs/mcp
22. Working Software — MCP in Practice — https://www.workingsoftware.dev/mcp-in-practice-what-software-architects-need-to-know-about-the-model-context-protocol/
23. BLT-MCP Repository — https://github.com/OWASP-BLT/BLT-MCP
24. BLT-API PR #89 (Auth) — https://github.com/OWASP-BLT/BLT-API/pull/89
25. MCP Example Remote Server — https://github.com/modelcontextprotocol/example-remote-server
