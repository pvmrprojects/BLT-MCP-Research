# MCP (Model Context Protocol) Sources

## Source 1: Official MCP Specification (2025-11-25)

- **URL**: https://modelcontextprotocol.io/specification/2025-11-25
- **Title**: Model Context Protocol Specification
- **Author/Organization**: Model Context Protocol (David Soria Parra, Justin Spahr-Summers) / Anthropic
- **Summary**: The authoritative specification defining the MCP protocol requirements, based on the TypeScript schema in schema.ts. Covers base protocol, server features (Resources, Prompts, Tools), client features (Sampling, Roots, Elicitation), lifecycle management, authorization, and security.
- **Important Technical Details**:
  - Uses JSON-RPC 2.0 as the message format
  - Stateful connections with capability negotiation
  - Three server primitives: Resources (data), Prompts (templates), Tools (functions)
  - Two client primitives: Sampling (LLM requests), Roots (filesystem boundaries)
  - Schema defined in TypeScript first, also available as JSON Schema
  - Security model emphasizes user consent and data privacy
- **Why It Is Useful**: This is the foundational reference for implementing any MCP server, including BLT-MCP. Every protocol message, lifecycle event, and capability must conform to this spec.

---

## Source 2: Official MCP Specification (2025-06-18)

- **URL**: https://modelcontextprotocol.io/specification/2025-06-18
- **Title**: Model Context Protocol Specification (June 2025)
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Previous stable version of the MCP specification. Introduced the Authorization framework for HTTP-based transports, formalizing MCP servers as OAuth Resource Servers with a separate authorization server.
- **Important Technical Details**:
  - Separated auth server from resource server roles (deprecating the integrated model)
  - Added authorization framework for HTTP-based transports
  - stdio transport should NOT use HTTP auth; instead retrieve credentials from environment
- **Why It Is Useful**: Understanding the evolution from 2025-06-18 to 2025-11-25 reveals important spec changes, especially around authorization architecture.

---

## Source 3: MCP Python SDK

- **URL**: https://py.sdk.modelcontextprotocol.io/
- **Title**: MCP Python SDK Documentation
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Official Python SDK implementing the full MCP specification. Provides both high-level (FastMCP) and low-level APIs for building MCP servers and clients.
- **Important Technical Details**:
  - Two API levels: FastMCP (high-level, decorator-based) and low-level server
  - Supports transports: stdio, SSE, Streamable HTTP
  - v1.x is stable; v2 in alpha (targeting beta 2026-06-30, stable 2026-07-27)
  - FastMCP example: `@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()` decorators
  - Automatic capability declaration based on registered handlers
  - Uses JSON Schema 2020-12 for tool input/output schemas
  - Supports ASGI mounting to existing servers
  - OAuth 2.1 with PKCE for remote server authorization
- **Why It Is Useful**: The Python SDK is the most relevant implementation toolkit for BLT-MCP if building in Python. FastMCP dramatically reduces boilerplate.

---

## Source 4: MCP Python SDK GitHub Repository

- **URL**: https://github.com/modelcontextprotocol/python-sdk
- **Title**: modelcontextprotocol/python-sdk
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Source code repository for the official Python SDK. 23K+ stars, MIT license. Includes examples, migration guides, and detailed README.
- **Important Technical Details**:
  - Repository structure: `src/mcp/` with server, client, shared modules
  - PyPI package: `mcp`
  - Key modules: `mcp.server.fastmcp`, `mcp.client`, `mcp.shared`
  - v2 alpha introduces significant API changes
- **Why It Is Useful**: Source code access allows understanding implementation details not covered in docs. Useful for debugging and customization.

---

## Source 5: MCP GitHub (Specification) Repository

- **URL**: https://github.com/modelcontextprotocol/modelcontextprotocol
- **Title**: modelcontextprotocol/modelcontextprotocol
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Central repository for the MCP specification, protocol schema, and official documentation. 8.4K+ stars, MIT license. Built with Mintlify.
- **Important Technical Details**:
  - Schema defined in TypeScript: `schema/2025-11-25/schema.ts`
  - Also available as JSON Schema: `schema/2025-11-25/schema.json`
  - Documentation lives in `docs/` directory as MDX files
  - 350+ contributors
  - Homepage: https://modelcontextprotocol.io
- **Why It Is Useful**: Source of truth for protocol schema. Essential for understanding exact message types and structures.

---

## Source 6: MCP Lifecycle Documentation

- **URL**: https://modelcontextprotocol.io/specification/2024-11-05/basic/lifecycle
- **Title**: MCP Lifecycle - Connection Initialization, Operation, Shutdown
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Defines the rigorous lifecycle for client-server connections: initialization (capability negotiation, protocol version agreement), operation (normal communication), and shutdown (graceful termination).
- **Important Technical Details**:
  - Three phases: Initialization, Operation, Shutdown
  - Client sends `initialize` request with protocol version, capabilities, implementation info
  - Server responds with its own capabilities and info
  - Client sends `initialized` notification to begin operations
  - Version negotiation: client sends preferred version, server can respond with alternative
  - Capabilities are negotiated per session (e.g., prompts, resources, tools, logging)
- **Why It Is Useful**: Every MCP server must implement this lifecycle. Essential for correct BLT-MCP initialization.

---

## Source 7: MCP Schema Reference

- **URL**: https://modelcontextprotocol.io/specification/2025-11-25/schema
- **Title**: MCP Schema Reference
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Auto-generated schema reference for all MCP protocol messages and structures.
- **Important Technical Details**:
  - `initialize` request/response types
  - `ServerCapabilities` structure
  - `Resource`, `Tool`, `Prompt` definitions
  - `Implementation` metadata (name, version, icons)
  - `_meta` field reserved for protocol metadata
- **Why It Is Useful**: Quick reference for correct message formatting when implementing BLT-MCP protocol handlers.

---

## Source 8: Build an MCP Server Tutorial

- **URL**: https://modelcontextprotocol.io/docs/develop/build-server
- **Title**: Build an MCP Server
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Step-by-step tutorial for building MCP servers. Covers tools, resources, prompts implementation with multiple language examples (Python, Java, Rust).
- **Important Technical Details**:
  - Weather server example with two tools: `get_alerts`, `get_forecast`
  - Claude for Desktop integration instructions
  - Multiple language SDK demonstrations
- **Why It Is Useful**: Practical starting point for BLT-MCP implementation patterns.

---

## Source 9: Build an MCP Client Tutorial

- **URL**: https://modelcontextprotocol.io/docs/develop/build-client
- **Title**: Build an MCP Client
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Tutorial for building MCP clients that connect to servers, using stdio transport and Claude API integration.
- **Important Technical Details**:
  - Client connects via `stdio_client` with `StdioServerParameters`
  - Session initialization: `ClientSession(stdio, write)`, `session.initialize()`
  - Tool discovery: `session.list_tools()`
  - Query processing with Claude API using available tools
- **Why It Is Useful**: Understanding client side helps design BLT-MCP server for optimal client consumption.

---

## Source 10: MCP Protocol Features

- **URL**: https://py.sdk.modelcontextprotocol.io/protocol/
- **Title**: MCP Protocol Features - Python SDK
- **Author/Organization**: Model Context Protocol / Anthropic
- **Summary**: Documentation of cross-cutting MCP protocol features including primitives, capability negotiation, and version negotiation.
- **Important Technical Details**:
  - Three primitives: Prompts (user-controlled), Resources (app-controlled), Tools (model-controlled)
  - Capabilities auto-declared by SDK when handlers are registered
  - `LATEST_PROTOCOL_VERSION` and `SUPPORTED_PROTOCOL_VERSIONS` in `mcp.shared.version`
  - JSON Schema 2020-12 for validation
  - Pydantic models auto-generate JSON schemas via `model_json_schema()`
- **Why It Is Useful**: Understanding the three-primitive model is essential for correctly designing BLT-MCP's interface.
