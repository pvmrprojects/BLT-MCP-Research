# BLT-MCP Security Architecture

> **Document Version:** 1.0
> **Status:** Draft
> **Date:** 2026-06-26

---

## Threat Model

### Trust Boundaries

```
+---------------------+       +-------------------------+       +------------------+
|                     |       |                         |       |                  |
|  Untrusted Zone     |       |  Semi-Trusted Zone      |       |  Trusted Zone    |
|                     |       |                         |       |                  |
|  - External AI      |------>|  - BLT-MCP Server       |------>|  - BLT-API       |
|    Agents           |       |  - JSON-RPC Router      |       |  - D1 Database   |
|  - Public Internet  |       |  - Auth Middleware      |       |  - Backend Svc   |
|  - Unauthenticated  |       |  - Rate Limiter         |       |  - Env Secrets   |
|    Requests         |       |  - Audit Logger         |       |                  |
|                     |       |                         |       |                  |
+---------------------+       +-------------------------+       +------------------+
```

### Threat Categories (OWASP MCP Top 10)

| ID | Threat | Risk Level | BLT-MCP Relevance |
|----|--------|------------|-------------------|
| MCP01 | Token Mismanagement & Secret Exposure | **High** | BLT-API key stored in environment must never leak to LLM |
| MCP02 | Privilege Escalation via Scope Creep | **Medium** | Tool-level authorization prevents agents from overreaching |
| MCP03 | Tool Poisoning | **High** | Tool descriptions and schemas must be validated to prevent prompt injection |
| MCP04 | Software Supply Chain Attacks | **Medium** | Dependency pinning and vulnerability scanning required |
| MCP05 | Command Injection & Execution | **Medium** | Strict JSON Schema validation with additionalProperties: false |
| MCP07 | Insufficient Authentication & Authorization | **High** | OAuth 2.1 with PKCE for HTTP transport; API key for stdio |
| MCP08 | Lack of Audit and Telemetry | **Medium** | Every tool call logged with identity and timestamp |
| MCP09 | Shadow MCP Servers | **Low** | Official deployment only; configuration management |
| MCP10 | Context Injection & Over-Sharing | **Medium** | Resource responses filtered; no sensitive data leakage |

[Source: OWASP MCP Top 10, owasp.org/www-project-mcp-top-10/]

### Attack Tree

```
Goal: Compromise BLT-MCP to access/modify BLT data
===================================================
    |
    |-- 1. Steal API Credentials
    |       |-- 1.1 Prompt injection in tool descriptions
    |       |       +-- Mitigation: Validate tool descriptions, pin with hashes
    |       |-- 1.2 Exfiltrate env vars via tool output
    |       |       +-- Mitigation: Never return env values in tool results
    |       |-- 1.3 LLM context leakage
    |               +-- Mitigation: Short-lived tokens, credential vaults
    |
    |-- 2. Exploit Tool Definitions
    |       |-- 2.1 Schema poisoning (invented fields)
    |       |       +-- Mitigation: additionalProperties: false, strict enums
    |       |-- 2.2 Tool shadowing
    |       |       +-- Mitigation: Pin tool definitions, detect changes
    |       |-- 2.3 Parameter injection
    |               +-- Mitigation: Server-side validation, type coercion
    |
    |-- 3. Bypass Authentication
    |       |-- 3.1 stdio: No auth needed (process boundary)
    |       |       +-- Mitigation: Acceptable risk (OS isolation)
    |       |-- 3.2 HTTP: Steal/forge tokens
    |       |       +-- Mitigation: OAuth 2.1 with PKCE, short-lived tokens
    |       |-- 3.3 HTTP: Replay attacks
    |               +-- Mitigation: Token binding, nonce validation
    |
    |-- 4. Denial of Service
    |       |-- 4.1 Rate limit exhaustion
    |       |       +-- Mitigation: Token bucket per client
    |       |-- 4.2 Large resource reads
    |               +-- Mitigation: Response size caps, pagination
    |
    |-- 5. Abuse BLT-API Backend
            |-- 5.1 SSRF via BLT-API calls
            |       +-- Mitigation: BLT-API has domain allowlist
            |-- 5.2 Mass data exfiltration
                    +-- Mitigation: Rate limits, pagination constraints
```

---

## Authentication

### Strategy by Transport

| Transport | Auth Mechanism | Rationale |
|-----------|---------------|-----------|
| **stdio** | Process isolation + Environment variables | The MCP server runs as a subprocess of the client. No network authentication needed. Credentials are passed via environment variables, never exposed to the LLM. [MCP Spec 2025-06-18] |
| **Streamable HTTP** | OAuth 2.1 with PKCE | Required by MCP specification for HTTP-based transports. MCP server acts as OAuth Resource Server. [Descope MCP Security Guide] |

### stdio Authentication

```python
# Environment variables set by client process
os.environ["BLT_API_KEY"] = "sk-xxxxx"
os.environ["BLT_API_BASE"] = "https://api.owaspblt.org/v2"

# Server reads from environment to authenticate with BLT-API
client = BLTAPIClient(
    base_url=os.environ["BLT_API_BASE"],
    api_key=os.environ["BLT_API_KEY"]
)
```

**Security properties:**
- OS process boundary provides isolation
- No network exposure of credentials
- No protocol-level auth tokens needed
- Credentials never reach the LLM context

### OAuth 2.1 for HTTP Transport

```
Authorization Flow:
====================

1. Client discovers auth server metadata
   GET /.well-known/oauth-authorization-server
   Response: { authorization_endpoint, token_endpoint, ... }

2. Client initiates Authorization Request
   GET /authorize?
       response_type=code&
       client_id=blt-mcp-client&
       redirect_uri=...&
       code_challenge=SHA256(code_verifier)&
       code_challenge_method=S256&
       scope=tools:read+tools:write

3. User authenticates and consents
   Redirect with authorization code

4. Client exchanges code for token
   POST /token
   Body: grant_type=authorization_code&
         code=AUTHORIZATION_CODE&
         code_verifier=VERIFIER&
         redirect_uri=...
   Response: { access_token, token_type, expires_in, refresh_token }

5. Client makes MCP requests with token
   Authorization: Bearer eyJhbGci...
```

[Source: CSA Agentic MCP Security Guide, labs.cloudsecurityalliance.org]

---

## OAuth

### OAuth Roles

| Role | BLT-MCP Mapping |
|------|-----------------|
| **Resource Server** | BLT-MCP server (exposes tools/resources/prompts) |
| **Authorization Server** | Separate auth service (internal or external provider) |
| **Client** | AI agent application (Claude Desktop, ChatGPT, custom agent) |
| **Resource Owner** | BLT user or organization |

### Scope Design

Scopes are defined at the tool level, not the server level (per CSA recommendation):

| Scope | Tools Covered |
|-------|---------------|
| `tools:read` | search_bugs, list_issues, get_user_profile |
| `tools:write` | submit_issue, add_comment, update_issue_status |
| `tools:admin` | award_bacon, manage_organizations |
| `resources:read` | All blt:// URI reads |

### Token Considerations

| Property | Requirement |
|----------|-------------|
| Token format | JWT (signed, with standard claims) |
| Claims | `iss`, `sub`, `aud`, `exp`, `iat`, `scope`, `client_id` |
| Lifetime | 1 hour (access token), 24 hours (refresh token) |
| Binding | Token bound to client_id, sentinel for agent identity |
| Storage | Credential vault, never in LLM context |

---

## JWT

### Token Structure

```json
{
  "iss": "https://auth.blt.owasp.org",
  "sub": "client:claude-desktop-user-abc123",
  "aud": "https://mcp.blt.owasp.org",
  "exp": 1719398400,
  "iat": 1719394800,
  "scope": "tools:read tools:write resources:read",
  "client_id": "blt-mcp-client",
  "tool_permissions": ["submit_issue", "search_bugs"],
  "resource_restrictions": {
    "max_results": 100,
    "allowed_uris": ["blt://issues/*", "blt://users/*"]
  }
}
```

### Validation

```python
async def validate_token(token: str) -> TokenClaims:
    """Validate JWT and return claims."""
    try:
        payload = jwt.decode(
            token,
            key=JWKS_CLIENT.get_signing_key_from_jwt(token),
            audience="https://mcp.blt.owasp.org",
            algorithms=["RS256"],
            options={
                "verify_exp": True,
                "verify_iat": True,
                "require": ["iss", "sub", "aud", "exp", "scope"]
            }
        )
        return TokenClaims(**payload)
    except jwt.ExpiredSignatureError:
        raise AuthError("Token expired", code=401)
    except jwt.InvalidTokenError as e:
        raise AuthError(f"Invalid token: {e}", code=401)
```

---

## API Keys

### BLT-API Key (Backend Authentication)

The BLT-MCP server uses a static API key to authenticate with the BLT-API backend:

```python
# Configuration via environment (never hardcoded)
BLT_API_KEY = os.environ.get("BLT_API_KEY")
if not BLT_API_KEY:
    raise ConfigurationError("BLT_API_KEY environment variable required")

# Used for all BLT-API requests
headers = {
    "X-BLT-API-Key": BLT_API_KEY,
    "Content-Type": "application/json"
}
```

[Source: BLT-API PR #89, github.com/OWASP-BLT/BLT-API/pull/89]

**Security rules:**
- API key is stored in environment, never in code or config files committed to git
- API key is never returned in any tool response or resource content
- MCP client never has access to the raw API key
- API key can be rotated independently of MCP server restarts

### MCP-Level API Key (HTTP Transport)

For HTTP transport, an optional API key can be configured for the MCP server itself:

| Header | Description |
|--------|-------------|
| `X-MCP-Api-Key` | Optional static API key for MCP server access |

---

## Secrets Management

### What Must Be Protected

| Secret | Location | Risk if Exposed |
|--------|----------|-----------------|
| `BLT_API_KEY` | Server environment | Full access to BLT-API data |
| `JWT_SECRET` | Server environment | Token forgery |
| `OAUTH_CLIENT_SECRET` | Server environment | OAuth flow compromise |
| `BLT_API_BASE` | Server environment | (low risk, but should not be modified) |

### Rules

1. **Never expose secrets to the LLM.** The AI model should never have access to raw credentials.
2. **Never return secrets in tool results.** Tool handlers must filter out sensitive data.
3. **Never hardcode secrets.** All secrets come from environment variables or a secrets manager.
4. **Short-lived tokens over long-lived keys.** When possible, use OAuth tokens with short expiry.
5. **Credential vault for production.** Consider HashiCorp Vault or similar for enterprise deployments.

[Source: OWASP MCP Security Cheat Sheet]

---

## Rate Limiting

### Strategy: Token Bucket

```python
class RateLimiter:
    """Token bucket rate limiter per client identity."""

    def __init__(self):
        # Per-client rate limits
        self.buckets: dict[str, TokenBucket] = {}

    async def check_rate_limit(self, client_id: str) -> RateLimitResult:
        bucket = self.buckets.get(client_id)
        if not bucket:
            bucket = TokenBucket(
                capacity=100,     # max 100 requests
                refill_rate=10,   # 10 per second refill
                refill_interval=1.0
            )
            self.buckets[client_id] = bucket

        allowed, remaining = bucket.consume()
        return RateLimitResult(
            allowed=allowed,
            remaining=remaining,
            reset_after=bucket.time_until_full()
        )
```

### Limits

| Resource | Default Limit | Rationale |
|----------|---------------|-----------|
| Tools/call | 100 req/min per client | Prevent unintended automation loops |
| Resources/read | 200 req/min per client | Read operations are lighter |
| Prompts/get | 50 req/min per client | Prompts are interactive |
| Per-tool limits | Configurable per tool | Destructive tools get stricter limits |

### Response on Rate Limit

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rate limit exceeded",
    "data": {
      "retry_after": 5,
      "limit": 100,
      "remaining": 0,
      "reset": 1719398400
    }
  }
}
```

---

## Permissions

### Permission Model

```
User/Client
    |
    +-- Authentication (who you are)
    |       stdio: process identity
    |       HTTP: OAuth 2.1 token
    |
    +-- Authorization (what you can do)
    |       |
    |       +-- Tool-level scopes
    |       |       tools:read  ->  search_bugs, list_issues
    |       |       tools:write ->  submit_issue, add_comment
    |       |       tools:admin ->  award_bacon, update_issue_status
    |       |
    |       +-- Resource-level access
    |               blt://issues/*        (read all issues)
    |               blt://users/{id}      (read specific user)
    |
    +-- Action gating
            Read operations: auto-approve
            Write operations: require human confirmation
            Admin operations: require human + secondary approval
```

### Permission Matrix

| Tool | Read Scope | Write Scope | Admin Scope | Consent Required |
|------|-----------|-------------|-------------|------------------|
| `search_bugs` | ✅ | - | - | No |
| `list_issues` | ✅ | - | - | No |
| `get_user_profile` | ✅ | - | - | No |
| `submit_issue` | - | ✅ | - | Yes (default) |
| `add_comment` | - | ✅ | - | Yes |
| `update_issue_status` | - | - | ✅ | Yes |
| `award_bacon` | - | - | ✅ | Yes (strong) |

[Source: OWASP MCP Security Cheat Sheet — "Require explicit user confirmation for destructive/financial operations"]

---

## Audit Logging

### Log Schema

Every tool call, resource read, and prompt invocation is logged:

```json
{
  "timestamp": "2026-06-26T12:00:00.000Z",
  "event_type": "tool_call",
  "session_id": "sess_abc123",
  "client_id": "claude-desktop-user-xyz",
  "client_ip": "203.0.113.42",
  "transport": "stdio",
  "method": "tools/call",
  "tool": "submit_issue",
  "parameters": {
    "title": "XSS vulnerability",
    "severity": "high"
  },
  "result": {
    "success": true,
    "issue_id": 1234
  },
  "duration_ms": 234,
  "auth_method": "env_api_key",
  "error": null
}
```

### Log Events

| Event | When Triggered |
|-------|---------------|
| `session_start` | Client initializes connection |
| `session_end` | Connection closed |
| `tool_call` | Tool invoked (before execution) |
| `tool_result` | Tool completed (with outcome) |
| `resource_read` | Resource accessed |
| `prompt_get` | Prompt retrieved |
| `auth_failure` | Authentication failed |
| `rate_limit_hit` | Rate limit exceeded |
| `validation_error` | Input validation failed |

### Log Storage

| Environment | Storage | Retention |
|-------------|---------|-----------|
| Development | stdout/console | N/A |
| Production | Structured logging to SIEM | Per organizational policy |
| Audit | Append-only log file | 90 days minimum |

[Source: OWASP MCP Top 10 MCP08:2025 — "Maintain detailed logs of tool invocations, context changes, and user-agent interactions"]

---

## OWASP Risks

### MCP01: Token Mismanagement

**Risk:** BLT-API key could be exposed through tool output, error messages, or LLM context leakage.

**Mitigations:**
- API key stored in environment only
- Tool handlers never return raw credentials
- Error messages sanitized (no secret leakage)
- Key rotation supported via environment variable update

### MCP02: Privilege Escalation

**Risk:** An agent could use a read-only tool to perform write operations, or access data beyond its scope.

**Mitigations:**
- Tool-level scope enforcement
- Every tool call is independently authorized
- Deny-by-default: unrecognized scopes are blocked
- Resource URI validation prevents path traversal

### MCP03: Tool Poisoning

**Risk:** Malicious tool descriptions could manipulate the LLM into performing unintended actions.

**Mitigations:**
- Tool descriptions are static, defined in code (not user-generated)
- Tool schemas use `additionalProperties: false` and strict `enum` types
- Tool definitions pinned and verified at startup
- `mcp-scan` compatible for automated scanning

[Source: OWASP MCP Top 10 MCP03:2025]

### MCP04: Supply Chain

**Risk:** Vulnerable dependencies could compromise the MCP server.

**Mitigations:**
- Python dependencies pinned to specific versions
- Regular `pip audit` or `safety` scanning
- Dependabot or similar automated dependency updates
- Minimal dependency footprint

### MCP05: Command Injection

**Risk:** Malicious tool parameters could inject commands or manipulate backend calls.

**Mitigations:**
- Strict JSON Schema validation before any execution
- `additionalProperties: false` on all input schemas
- Parameter type coercion and validation
- No direct system command execution
- BLT-API calls use safe HTTP client

### MCP07: Insufficient Auth

**Risk:** Unauthenticated access to the MCP server could allow unauthorized BLT data access.

**Mitigations:**
- stdio: Process isolation (OS-level security)
- HTTP: OAuth 2.1 with PKCE (mandatory)
- Fallback: API key authentication for simpler deployments
- All protected operations require authentication

### MCP08: Lack of Audit

**Risk:** Without logging, security incidents cannot be investigated.

**Mitigations:**
- Every tool call logged with full context
- Resource reads logged with URI and client identity
- Authentication failures logged with timestamps
- Structured log format for SIEM integration

---

## Future Security Improvements

### Short-Term (GSoC 2026)

| Improvement | Priority | Effort |
|-------------|----------|--------|
| OAuth 2.1 with PKCE for HTTP transport | Critical | 2 weeks |
| Input validation with `additionalProperties: false` | Critical | 1 week |
| Rate limiting per client | High | 1 week |
| Audit logging for all MCP operations | High | 1 week |
| Tool-level scope enforcement | High | 1 week |
| Environment variable sanitization from tool output | Medium | 2 days |

### Medium-Term

| Improvement | Description |
|-------------|-------------|
| mTLS | Mutual TLS for HTTP transport (MCP07 recommendation) |
| JIT Access | Just-in-time credential issuance per session |
| mcp-scan Integration | Automated tool description scanning in CI/CD |
| Behavioral Anomaly Detection | Detect unusual tool call patterns |
| Credential Vault Integration | HashiCorp Vault / AWS Secrets Manager |

### Long-Term

| Improvement | Description |
|-------------|-------------|
| RBAC/ABAC | Full role-based and attribute-based access control |
| Federation | OIDC integration with organizational IdP |
| Policy Decision Point | Centralized authorization engine |
| Zero Trust Architecture | Continuous validation on every request |
| HITL Approval Workflows | Human-in-the-loop for sensitive operations |

---

## References

1. OWASP MCP Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
2. OWASP GenAI Security — Guide for Secure MCP Server Development — https://genai.owasp.org/resource/a-practical-guide-for-secure-mcp-server-development/
3. OWASP MCP Top 10 — https://owasp.org/www-project-mcp-top-10/
4. CSA Agentic MCP Security Best Practices — https://labs.cloudsecurityalliance.org/agentic/agentic-mcp-security-best-practices-v1/
5. Descope — MCP Server Security Best Practices — https://www.descope.com/blog/post/mcp-server-security-best-practices
6. OWASP GenAI — Securing Third-Party MCP Servers — https://genai.owasp.org/resource/cheatsheet-a-practical-guide-for-securely-using-third-party-mcp-servers-1-0/
7. MCP Server Permissions and Access Control — https://readfa.com/blog/mcp-server-permissions/
8. MCP Specification (2025-06-18) — https://modelcontextprotocol.io/specification/2025-06-18
9. BLT-API PR #89 (API Key Auth) — https://github.com/OWASP-BLT/BLT-API/pull/89
10. MCP Python SDK — https://py.sdk.modelcontextprotocol.io/
