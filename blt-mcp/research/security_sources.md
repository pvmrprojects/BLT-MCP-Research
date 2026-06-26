# Security Sources

## Source 1: OWASP MCP Security Cheat Sheet

- **URL**: https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
- **Title**: MCP Security Cheat Sheet
- **Author/Organization**: OWASP Cheat Sheet Series
- **Summary**: Comprehensive security guidelines for MCP deployments covering least privilege, tool schema validation, sandboxing, authentication, and monitoring.
- **Important Technical Details**:
  - **Do**: Grant each MCP server minimum permissions, use scoped per-server credentials, request narrow OAuth scopes, prefer ephemeral short-lived tokens
  - **Do**: Inspect tool descriptions, parameter names, types, and return schemas before approval
  - **Do**: Pin tool definitions using cryptographic hashes
  - **Do**: Use strict JSON Schema with `additionalProperties: false`
  - **Do**: Sandbox local MCP servers in containers, restrict filesystem/network access
  - **Do**: Require explicit user confirmation for destructive/financial operations
  - **Do**: Use OAuth 2.0 with PKCE for remote server authorization
  - **Don't**: Auto-approve tool calls without showing full parameters
  - **Don't**: Trust tool descriptions blindly (they are a prompt injection vector)
  - **Don't**: Share tokens across MCP servers
  - **Don't**: Run MCP servers with full host access
  - **Tool recommendation**: `mcp-scan` for automated detection of poisoned descriptions
- **Why It Is Useful**: Directly applicable to BLT-MCP design. Must ensure tool descriptions are safe, authentication is robust, and user consent is respected.

---

## Source 2: OWASP GenAI Security — Practical Guide for Secure MCP Server Development

- **URL**: https://genai.owasp.org/resource/a-practical-guide-for-secure-mcp-server-development/
- **Title**: A Practical Guide for Secure MCP Server Development
- **Author/Organization**: OWASP GenAI Security Project
- **Summary**: Actionable guidance for securing MCP servers covering architecture, authentication, validation, session isolation, and hardened deployment.
- **Important Technical Details**:
  - MCP servers operate with delegated user permissions — amplifies impact of single vulnerability
  - Key areas: secure architecture, strong authN/authZ, strict validation, session isolation, hardened deployment
  - Audience: software architects, platform engineers, development teams
  - Published: February 16, 2026
  - Part of broader GenAI Security resource library
- **Why It Is Useful**: Provides formal OWASP-endorsed methodology for securing MCP servers. Should be used as a compliance checklist for BLT-MCP.

---

## Source 3: OWASP MCP Top 10

- **URL**: https://owasp.org/www-project-mcp-top-10/
- **Title**: OWASP MCP Top 10 (v0.1)
- **Author/Organization**: OWASP Foundation
- **Summary**: The top 10 security concerns for MCP-enabled systems, spanning model misbinding, context spoofing, prompt-state manipulation, and covert channel abuse.
- **Important Technical Details**:
  | ID | Category | Description |
  |---|---------|-------------|
  | MCP01:2025 | Token Mismanagement & Secret Exposure | Hard-coded credentials, long-lived tokens, secrets in model memory/logs |
  | MCP02:2025 | Privilege Escalation via Scope Creep | Permissions expanding over time, granting excessive capabilities |
  | MCP03:2025 | Tool Poisoning | Compromised tools/plugins inject malicious context to manipulate model behavior |
  | MCP04:2025 | Software Supply Chain Attacks | Dependency tampering in MCP server supply chain |
  | MCP05:2025 | Command Injection & Execution | Injection attacks through tool parameters |
  | MCP06:2025 | [Reserved] | |
  | MCP07:2025 | Insufficient Authentication & Authorization | Weak/missing identity validation in multi-agent ecosystems |
  | MCP08:2025 | Lack of Audit and Telemetry | Limited observability impeding incident response |
  | MCP09:2025 | Shadow MCP Servers | Unapproved/unsupervised MCP deployments outside governance |
  | MCP10:2025 | Context Injection & Over-Sharing | Sensitive info leaking across shared context windows |
- **Why It Is Useful**: BLT-MCP must specifically address MCP01 (token management for BLT API keys), MCP03 (tool description safety), MCP07 (auth), and MCP08 (audit logging).

---

## Source 4: OWASP GenAI Security — Guide for Securely Using Third-Party MCP Servers

- **URL**: https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/
- **Title**: CheatSheet - A Practical Guide for Securely Using Third-Party MCP Servers 1.0
- **Author/Organization**: OWASP GenAI Security Project
- **Summary**: Framework for safely deploying and managing external MCP servers, covering tool poisoning, prompt injection, memory poisoning, and tool interference.
- **Important Technical Details**:
  - Risk categories: tool poisoning, prompt injection, memory poisoning, tool interference
  - Mitigations: authentication, authorization, client sandboxing, secure server discovery, governance workflows
  - Emphasizes least-privilege access and human-in-the-loop oversight
  - Published: November 4, 2025
- **Why It Is Useful**: BLT-MCP should follow these guidelines when integrating with third-party AI clients.

---

## Source 5: MCP Server Permissions and Access Control Guide

- **URL**: https://readfa.com/blog/mcp-server-permissions/
- **Title**: MCP Server Permissions and Access Control: A Security Guide
- **Author/Organization**: readfa.com
- **Summary**: Practical guide for scoping MCP server permissions across filesystem, network, and tool-level access control patterns.
- **Important Technical Details**:
  - Permissions emerge from three layers: tool definitions, implementation code, transport-level controls
  - SSRF critical risk for network-requesting MCP servers
  - Permission patterns: role-based, consent-gated, credential scoping
  - Pre-publication security scanner recommended
  - Permission checklist: minimum tools, path containment, symlink resolution, sensitive file blocking, domain allowlists, SSRF protection, env isolation, credential scoping, consent requirements, audit logging
- **Why It Is Useful**: Provides actionable permission design patterns directly applicable to BLT-MCP tool definitions.

---

## Source 6: OWASP MCP Top 10 — Tool Poisoning Details

- **URL**: https://github.com/OWASP/www-project-mcp-top-10/blob/master/2025/MCP03-2025%E2%80%93Tool-Poisoning.md
- **Title**: MCP03:2025 - Tool Poisoning
- **Author/Organization**: OWASP
- **Summary**: Detailed analysis of tool poisoning risks including rug pulls, schema poisoning, and tool shadowing.
- **Important Technical Details**:
  - Sub-techniques: rug pulls (malicious updates to trusted tools), schema poisoning (corrupting interface definitions), tool shadowing (fake/duplicate tools)
  - Impact: manipulated model behavior, unauthorized data access
- **Why It Is Useful**: BLT-MCP tool descriptions must be carefully vetted to prevent tool poisoning attacks through prompt injection.

---

## Source 7: OWASP MCP Top 10 — Insufficient Authentication & Authorization

- **URL**: https://github.com/OWASP/www-project-mcp-top-10/blob/master/2025/MCP07-2025%E2%80%93Insufficient-Authentication%26Authorization.md
- **Title**: MCP07:2025 - Insufficient Authentication & Authorization
- **Author/Organization**: OWASP
- **Summary**: Comprehensive breakdown of authN/authZ failures in MCP ecosystems with remediation guidance.
- **Important Technical Details**:
  - Symptoms: no mutual auth between agents and tools, shared/static/long-lived tokens, no RBAC/ABAC, no identity correlation in access logs
  - Remediations: mTLS, short-lived scoped tokens (JWT/OAuth2), token binding, RBAC/ABAC, per-request permission evaluation, deny-by-default, IAM/OIDC integration, centralized PDP, comprehensive audit logging
- **Why It Is Useful**: BLT-MCP must implement proper authentication (not just proxy BLT-API keys) and authorization for tool-level access control.

---

## Source 8: Descope — MCP Server Security Best Practices

- **URL**: https://www.descope.com/blog/post/mcp-server-security-best-practices
- **Title**: MCP Server Security Best Practices to Prevent Risk
- **Author/Organization**: Descope / Rishi Bhargava
- **Summary**: Enterprise-focused MCP security guide covering OAuth 2.1, PKCE, scope-based access control, progressive scoping, and credential management.
- **Important Technical Details**:
  - MCP servers are OAuth Resource Servers (per June 2025 spec)
  - Authorization function belongs to a dedicated auth server
  - OAuth 2.1 with PKCE is mandatory for HTTP-based MCP
  - Progressive scoping: agents request only needed scopes for current task
  - Secrets must be stored in credential vaults, never in environment variables accessible to LLM
  - Centralize policy enforcement in dedicated auth layer
- **Why It Is Useful**: Enterprise-grade security guidance. BLT-MCP should adopt OAuth 2.1 with PKCE for remote access and implement progressive scoping.

---

## Source 9: CSA — Agentic MCP Security Best Practices Guide

- **URL**: https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/
- **Title**: Agentic MCP Security Best Practices Guide
- **Author/Organization**: Cloud Security Alliance (CSA)
- **Summary**: Enterprise security framework for MCP covering six threat categories, four-level security maturity model, and cross-reference to OWASP ASI, CSA AICM, and MITRE ATLAS.
- **Important Technical Details**:
  - Defense in depth required — no single control is sufficient
  - OAuth 2.1 with PKCE mandatory for all remote MCP connections (per Nov 2025 spec)
  - OAuth 2.0 Authorization Server Metadata (RFC 8414) for server discovery
  - Scopes at tool level, not server level
  - Tool description changes must trigger alert and require re-approval
  - MCP as critical infrastructure — same rigor as API gateway or identity provider
- **Why It Is Useful**: Provides maturity model for BLT-MCP security. Target Level 3+ for production readiness.

---

## Source 10: OWASP GenAI — Securing AI's New Frontier

- **URL**: https://genai.owasp.org/2025/04/22/securing-ais-new-frontier-the-power-of-open-collaboration-on-mcp-security/
- **Title**: Securing AI's New Frontier: The Power of Open Collaboration on MCP Security
- **Author/Organization**: OWASP GenAI Security Project / John Sotiropoulos
- **Summary**: Overview of MCP security challenges with recommendations for network segmentation, zero trust, JIT access, and continuous validation.
- **Important Technical Details**:
  - Network segmentation: isolate MCP servers in dedicated security zones
  - Zero trust: "never trust, always verify" for every MCP interaction
  - JIT access: time-limited credentials for specific tasks
  - Continuous validation: re-validate authorization for every request
  - Behavioral anomaly detection for suspicious patterns
- **Why It Is Useful**: Zero trust architecture principles should guide BLT-MCP's security design.
