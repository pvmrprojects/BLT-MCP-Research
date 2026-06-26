# BLT-MCP Development Plan

> **Document Version:** 1.0
> **Status:** Draft
> **Date:** 2026-06-26
> **Total Duration:** 14 weeks
> **Target:** GSoC 2026 Completion

---

## Overview

The BLT-MCP project is split into 9 sprints following the GSoC 2026 timeline. Each sprint is 1-2 weeks and delivers a specific set of capabilities. The plan follows a build-up approach: foundation → resources → tools → prompts → hardening → documentation → release.

---

## Sprint Schedule

```
Sprint 0: Research (Pre-GSoC)
Sprint 1: Foundation - MCP Server Skeleton (Week 1-2)
Sprint 2: Resources Layer (Week 3-5)
Sprint 3: Tools Layer (Week 6-8)
Sprint 4: Prompts Layer (Week 9-10)
Sprint 5: Authentication & Security (Week 11)
Sprint 6: Testing & Hardening (Week 12)
Sprint 7: Documentation (Week 13)
Sprint 8: Release & Polish (Week 14)
```

---

## Sprint 0: Research (Pre-GSoC)

**Duration:** Pre-GSoC (completed)
**Focus:** Understanding the BLT ecosystem, MCP protocol, and existing codebase

### Deliverables

| Deliverable | Description | Location |
|-------------|-------------|----------|
| MCP Sources Research | Official MCP spec, SDK, lifecycle, schema | `research/mcp_sources.md` |
| BLT Sources Research | BLT repos, docs, GSoC ideas | `research/blt_sources.md` |
| API Sources Research | BLT-API endpoints, auth, existing MCP | `research/api_sources.md` |
| Security Sources Research | OWASP MCP Top 10, cheat sheets | `research/security_sources.md` |
| Architecture Sources Research | MCP patterns, production architectures | `research/architecture_sources.md` |
| Project Overview | Executive summary, goals, scope, timeline | `docs/00_PROJECT_OVERVIEW.md` |
| Problem Statement | Why MCP, current limitations, measurable improvements | `docs/01_PROBLEM_STATEMENT.md` |
| MCP Handbook | Complete MCP protocol reference | `docs/02_MCP_RESEARCH.md` |
| BLT Handbook | Complete BLT ecosystem reference | `docs/03_BLT_RESEARCH.md` |
| API Reference | All BLT-API endpoints documented | `docs/04_API_REFERENCE.md` |
| Architecture Design | Component, deployment, sequence diagrams | `docs/07_ARCHITECTURE.md` |
| Security Architecture | Threat model, auth, permissions, audit | `docs/10_SECURITY.md` |
| Development Plan | This document | `docs/12_DEVELOPMENT_PLAN.md` |

### Effort: 40 hours (completed pre-GSoC)

---

## Sprint 1: MCP Server Foundation

**Duration:** Weeks 1-2 (14 days)
**Focus:** Project scaffolding, MCP lifecycle implementation, BLT-API client

### Goals
- Set up Python project with FastMCP SDK
- Implement MCP lifecycle (initialize → initialized → operation)
- Create BLT-API client library
- Implement stdio transport
- Write initial configuration and entry point
- Basic error handling

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| Initialize Python project structure (pyproject.toml, dependencies) | 4 | None |
| Install and configure MCP Python SDK (pin `mcp>=1.27,<2`) | 2 | Project setup |
| Create FastMCP server with basic lifecycle | 8 | SDK installed |
| Implement `initialize` request handler | 4 | Server skeleton |
| Implement capability negotiation | 4 | Initialize handler |
| Create BLT-API HTTP client class | 8 | None |
| Implement stdio transport configuration | 4 | Server skeleton |
| Add environment variable loading (BLT_API_KEY, BLT_API_BASE) | 2 | Project setup |
| Implement basic error handling and JSON-RPC error responses | 6 | Server skeleton |
| Write integration test for lifecycle | 4 | Server complete |
| **Total** | **46** | |

### Definition of Done
- [x] Python project runs with `uv run blt-mcp`
- [x] MCP server initializes and responds to `initialize` request
- [x] Capability negotiation works (server declares resources, tools, prompts capabilities)
- [x] BLT-API client can authenticate and make basic requests
- [x] stdio transport functions correctly
- [x] Environment variables are loaded

### Key Files
```
blt-mcp/
├── pyproject.toml
├── src/blt_mcp/
│   ├── __init__.py
│   ├── server.py          # FastMCP server
│   ├── config.py           # Environment config
│   └── client/
│       ├── __init__.py
│       └── blt_api.py      # BLT-API HTTP client
├── tests/
│   ├── __init__.py
│   └── test_lifecycle.py
└── .env.example
```

---

## Sprint 2: Resources Layer

**Duration:** Weeks 3-5 (21 days)
**Focus:** Implementing all blt:// resource URIs with templates and pagination

### Goals
- Implement all read-only data URIs
- Support resource templates (parameterized URIs)
- Pagination and filtering for list resources
- Resource subscription support (listChanged)
- Error handling for missing resources

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| Implement `blt://issues` — list all issues (paginated) | 8 | BLT-API client |
| Implement `blt://issues/{id}` — single issue details | 4 | Issues list |
| Implement `blt://contributors` — list all contributors | 6 | BLT-API client |
| Implement `blt://contributors/{id}` — single contributor | 4 | Contributors list |
| Implement `blt://repos` — list tracked repositories | 6 | BLT-API client |
| Implement `blt://repos/{id}` — single repository | 4 | Repos list |
| Implement `blt://leaderboards` — leaderboard rankings | 6 | BLT-API client |
| Implement `blt://rewards` — rewards and bacon points | 6 | BLT-API client |
| Implement `blt://workflows` — list workflows | 4 | BLT-API client |
| Implement `blt://workflows/{id}` — single workflow | 4 | Workflows list |
| Add resource template support (URI patterns) | 8 | Any resource |
| Add pagination helper for list resources | 4 | Any list resource |
| Add resource subscription (listChanged notification) | 6 | Resources complete |
| Write resource tests | 8 | All resources |
| **Total** | **78** | |

### Resource URI Map

| URI | Description | BLT-API Endpoint |
|-----|-------------|------------------|
| `blt://issues` | All issues (paginated) | `GET /v2/bugs` |
| `blt://issues/{id}` | Single issue | `GET /v2/bugs/{id}` |
| `blt://contributors` | All contributors | `GET /v2/users` |
| `blt://contributors/{id}` | Single contributor | `GET /v2/users/{id}` |
| `blt://repos` | All repositories | TBD |
| `blt://repos/{id}` | Single repository | TBD |
| `blt://leaderboards` | Leaderboard rankings | `GET /v2/users?sort=reputation` |
| `blt://rewards` | Rewards/BACON data | BACON API |
| `blt://workflows` | All workflows | TBD |
| `blt://workflows/{id}` | Single workflow | TBD |

### Definition of Done
- [x] All 10 resource URIs return correct data
- [x] Resource templates work with parameter substitution
- [x] List resources support pagination with `page`/`per_page`
- [x] Resources return appropriate MIME types
- [x] Missing resources return proper MCP error responses
- [x] Resource tests pass

---

## Sprint 3: Tools Layer

**Duration:** Weeks 6-8 (21 days)
**Focus:** Implementing all MCP tools with validation and error handling

### Goals
- Implement all tool handlers
- Strict JSON Schema validation
- Meaningful error messages
- Tool call audit logging
- Rate limiting integration

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| Implement `submit_issue` tool | 8 | BLT-API client |
| Implement `award_bacon` tool | 6 | BLT-Rewards API client |
| Implement `update_issue_status` tool | 6 | BLT-API client |
| Implement `add_comment` tool | 6 | BLT-API client |
| Add `search_bugs` tool (derived from resource) | 4 | BLT-API client |
| Implement strict JSON Schema validation for all tools | 8 | All tools |
| Add `additionalProperties: false` enforcement | 2 | Schema validation |
| Add enum constraints for categorical fields | 2 | Schema validation |
| Implement tool-specific error messages | 4 | All tools |
| Add audit logging for every tool call | 4 | All tools |
| Integrate rate limiter with tool calls | 6 | Rate limiter |
| Add response size caps | 2 | All tools |
| Write tool tests | 10 | All tools |
| **Total** | **68** | |

### Tool Definitions

| Tool | Parameters | BLT-API Call |
|------|-----------|-------------|
| `submit_issue` | title, description, url, severity, domain_id, tags | `POST /v2/bugs` |
| `award_bacon` | user_id, amount, reason | BACON API |
| `update_issue_status` | issue_id, status, reason | `PATCH /v2/bugs/{id}` |
| `add_comment` | issue_id, comment_body | Comment API |
| `search_bugs` | query, status, domain | `GET /v2/bugs/search` |

### JSON Schema Example

```python
@mcp.tool()
def submit_issue(
    title: str,
    description: str,
    url: str = None,
    severity: str = "medium",
    domain_id: int = None,
    tags: list[str] = None
) -> str:
    """
    Submit a new bug report to the BLT platform.

    Args:
        title: Short descriptive title of the issue
        description: Detailed description of the bug
        url: URL where the bug was found (must be http/https)
        severity: Severity level (low, medium, high, critical)
        domain_id: ID of the domain (optional)
        tags: List of tags to categorize the issue
    """
    # Implementation calls BLT-API
```

Generated JSON Schema:
```json
{
  "type": "object",
  "properties": {
    "title": { "type": "string", "description": "Short descriptive title" },
    "description": { "type": "string", "description": "Detailed description" },
    "url": { "type": "string", "description": "URL where the bug was found", "pattern": "^https?://" },
    "severity": { "type": "string", "enum": ["low", "medium", "high", "critical"] },
    "domain_id": { "type": "integer" },
    "tags": { "type": "array", "items": { "type": "string" } }
  },
  "required": ["title", "description"],
  "additionalProperties": false
}
```

### Definition of Done
- [x] All 5 tools execute successfully against BLT-API
- [x] JSON Schema validation rejects invalid inputs with clear messages
- [x] `additionalProperties: false` enforced on all tool schemas
- [x] Audit log captures every tool call
- [x] Rate limiting prevents abuse
- [x] Tool tests pass (>80% coverage)

---

## Sprint 4: Prompts Layer

**Duration:** Weeks 9-10 (14 days)
**Focus:** Implementing reusable prompt templates for security workflows

### Goals
- Implement 3+ prompt templates
- Resource embedding in prompts
- Argument completion support
- Prompt chaining (linking prompts together)

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| Implement `triage_vulnerability` prompt | 8 | Resources layer |
| Implement `plan_remediation` prompt | 8 | Resources layer |
| Implement `review_contribution` prompt | 6 | Contributors resource |
| Add resource embedding in prompts (include issue data) | 6 | All prompts |
| Implement argument completion for prompt parameters | 6 | All prompts |
| Add prompt chaining support (next_prompt field) | 4 | All prompts |
| Write prompt tests | 6 | All prompts |
| **Total** | **44** | |

### Prompt Definitions

#### triage_vulnerability

```
Assess the severity and impact of a vulnerability report.
Steps:
1. Review the issue details (embedded from blt://issues/{id})
2. Check for CVE identifiers and known exploits
3. Assess severity based on OWASP criteria
4. Identify affected components
5. Recommend priority level
6. Suggest stakeholders to notify
```

#### plan_remediation

```
Create a step-by-step remediation plan for a verified issue.
Steps:
1. Understand the vulnerability context
2. Identify root cause
3. Propose fix strategy
4. Estimate effort and risk
5. Define verification criteria
6. Set remediation timeline
```

#### review_contribution

```
Review a security contribution for quality and correctness.
Steps:
1. Examine the contribution details
2. Verify against BLT contribution guidelines
3. Check for common security mistakes
4. Provide constructive feedback
5. Recommend approval or changes
```

### Prompt Schema

```json
{
  "name": "triage_vulnerability",
  "description": "Guide through triaging a vulnerability report",
  "arguments": [
    {
      "name": "issue_id",
      "description": "ID of the issue to triage",
      "required": true
    }
  ]
}
```

### Definition of Done
- [x] All 3 prompt templates return correct, context-aware prompts
- [x] Resource embedding works (issue data included in prompt)
- [x] Argument completion provides helpful suggestions
- [x] Prompts tests pass

---

## Sprint 5: Authentication & Security

**Duration:** Week 11 (7 days)
**Focus:** Security hardening, OAuth implementation, rate limiting, audit

### Goals
- Implement OAuth 2.1 with PKCE for HTTP transport
- Complete rate limiting implementation
- Add input sanitization and security headers
- Security review against OWASP MCP Security Cheat Sheet
- Address high-priority OWASP MCP Top 10 risks

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| Implement OAuth 2.1 authorization server metadata endpoint | 6 | None |
| Implement PKCE authorization code flow | 8 | Auth metadata |
| Implement token validation and introspection | 6 | Token generation |
| Add tool-level scope enforcement | 4 | Auth complete |
| Complete rate limiter with per-client buckets | 6 | Rate limiter |
| Add rate limit headers to responses | 2 | Rate limiter |
| Add input sanitization (strip control chars, limit lengths) | 4 | None |
| Add security headers (CORS, CSP, X-Content-Type-Options) | 2 | None |
| Security review against OWASP MCP Cheat Sheet checklist | 6 | All security |
| Run mcp-scan for tool description vulnerabilities | 3 | All tools |
| Fix any security findings | 6 | Review results |
| **Total** | **53** | |

### OWASP MCP Security Checklist

| Requirement | Status |
|-------------|--------|
| Tool schemas use `additionalProperties: false` | ✅ Sprint 3 |
| API keys stored in environment, not code | ✅ Sprint 1 |
| Secrets never exposed to LLM in tool results | ✅ Sprint 3 |
| Destructive operations require human confirmation | ✅ Sprint 5 |
| Audit logging enabled for all operations | ✅ Sprint 3 |
| OAuth 2.1 with PKCE for HTTP transport | ✅ Sprint 5 |
| Rate limiting implemented | ✅ Sprint 5 |
| mcp-scan passes clean | ✅ Sprint 5 |
| Dependency vulnerability scanning | ✅ Sprint 5 |

[Source: OWASP MCP Security Cheat Sheet]

### Definition of Done
- [x] OAuth 2.1 with PKCE works for HTTP transport
- [x] Tool-level scopes enforced
- [x] Rate limiting prevents abuse
- [x] Security headers configured
- [x] OWASP MCP Security Checklist passes
- [x] mcp-scan reports clean

---

## Sprint 6: Testing & Hardening

**Duration:** Week 12 (7 days)
**Focus:** Comprehensive testing, edge cases, performance, reliability

### Goals
- Full test suite (unit + integration)
- Edge case handling
- Performance optimization
- Error message refinement
- CI/CD pipeline configuration

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| Write unit tests for config module | 3 | All sprints |
| Write unit tests for BLT-API client | 6 | Sprint 1 |
| Write unit tests for resource handlers | 6 | Sprint 2 |
| Write unit tests for tool handlers | 6 | Sprint 3 |
| Write unit tests for prompt handlers | 4 | Sprint 4 |
| Write unit tests for auth module | 4 | Sprint 5 |
| Write unit tests for rate limiter | 3 | Sprint 5 |
| Write integration tests against BLT-API (mock) | 8 | All sprints |
| Write protocol compliance tests | 4 | All sprints |
| Add edge case handling (network errors, timeouts, partial data) | 4 | All sprints |
| Review and refine all error messages | 3 | All sprints |
| Performance profiling and optimization | 6 | All sprints |
| Configure CI/CD (GitHub Actions) | 4 | All sprints |
| **Total** | **61** | |

### Test Categories

| Category | Tests | Target Coverage |
|----------|-------|-----------------|
| Unit: Config | 5+ | 100% of config paths |
| Unit: API Client | 15+ | 90%+ |
| Unit: Resources | 20+ | 90%+ |
| Unit: Tools | 20+ | 90%+ |
| Unit: Prompts | 10+ | 90%+ |
| Unit: Auth | 15+ | 95%+ |
| Unit: Rate Limiter | 10+ | 95%+ |
| Integration: Lifecycle | 5+ | All paths |
| Integration: Resource Read | 10+ | All URIs |
| Integration: Tool Call | 10+ | All tools |
| Protocol Compliance | 10+ | All mandatory features |

### Definition of Done
- [x] Test coverage > 80% overall
- [x] All tests passing in CI
- [x] Edge cases handled (timeouts, network errors, partial data)
- [x] Error messages are descriptive and actionable
- [x] CI/CD pipeline configured with GitHub Actions
- [x] Performance meets latency targets (<5s for simple queries)

---

## Sprint 7: Documentation

**Duration:** Week 13 (7 days)
**Focus:** Comprehensive documentation for users and contributors

### Goals
- Setup guide for end users
- API reference (auto-generated)
- Integration examples for Claude Desktop, Cline, custom agents
- Contributor guide
- Security architecture overview

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| Write README.md with badges, quick start, features | 4 | All sprints |
| Write setup guide (installation, configuration) | 6 | Sprint 1 |
| Write API reference (resources, tools, prompts) | 8 | Sprints 2-4 |
| Write Claude Desktop integration guide | 4 | Sprint 1 |
| Write Cline integration guide | 3 | Sprint 1 |
| Write custom agent integration guide | 4 | Sprint 1 |
| Write security configuration guide | 4 | Sprint 5 |
| Write troubleshooting guide | 3 | All sprints |
| Add docstrings to all public interfaces | 4 | All sprints |
| Review and proofread all documentation | 4 | All docs |
| **Total** | **44** | |

### Document Structure

```
docs/
├── 00_PROJECT_OVERVIEW.md       # Executive summary, goals, scope
├── 01_PROBLEM_STATEMENT.md      # Problem analysis
├── 02_MCP_RESEARCH.md           # MCP protocol handbook
├── 03_BLT_RESEARCH.md           # BLT ecosystem handbook
├── 04_API_REFERENCE.md          # BLT-API endpoint reference
├── 07_ARCHITECTURE.md           # Architecture diagrams
├── 10_SECURITY.md               # Security architecture
├── 12_DEVELOPMENT_PLAN.md       # Sprint plan (this doc)
├── guides/
│   ├── SETUP.md                 # Installation and configuration
│   ├── INTEGRATION_CLAUDE.md    # Claude Desktop setup
│   ├── INTEGRATION_CLINE.md     # Cline setup
│   └── INTEGRATION_CUSTOM.md    # Custom agent setup
└── CONTRIBUTING.md              # Contributor guide
```

### Definition of Done
- [x] README.md complete with badges, quick start, feature list
- [x] Setup guide covers all environment configurations
- [x] API reference documents all resources, tools, prompts
- [x] Integration guides for Claude Desktop, Cline, custom agents
- [x] Security configuration documented
- [x] All public interfaces have docstrings
- [x] Documentation reviewed and proofread

---

## Sprint 8: Release & Polish

**Duration:** Week 14 (7 days)
**Focus:** Final testing, bug fixes, release preparation

### Goals
- End-to-end testing
- Bug fixes from testing
- Performance tuning
- Release packaging
- Demo preparation

### Task Breakdown

| Task | Hours | Dependencies |
|------|-------|-------------|
| End-to-end testing with Claude Desktop | 8 | All sprints |
| End-to-end testing with HTTP transport | 4 | Sprint 5 |
| Bug fixes from end-to-end testing | 8 | Testing results |
| Performance tuning and optimization | 6 | All sprints |
| Final security review | 4 | Sprint 5 |
| Package release (GitHub release, CHANGELOG) | 4 | All sprints |
| Prepare demo script and screencast | 6 | All sprints |
| Write GSoC final report | 4 | All sprints |
| **Total** | **44** | |

### Release Criteria

| Criterion | Requirement |
|-----------|-------------|
| Functional | All resources, tools, prompts work correctly |
| Protocol Compliance | Passes MCP specification compliance checks |
| Security | OWASP MCP Security Checklist passes |
| Performance | Tool calls complete in <5s for simple queries |
| Documentation | All docs complete and reviewed |
| Tests | >80% coverage, all passing |
| Integration | Works with Claude Desktop, Cline, HTTP transport |

### Definition of Done (GSoC Completion)
- [x] All end-to-end tests pass
- [x] All known bugs fixed
- [x] Security review completed
- [x] Release packaged on GitHub
- [x] CHANGELOG written
- [x] Demo prepared
- [x] GSoC final report submitted

---

## Effort Summary

| Sprint | Focus | Hours | Weeks |
|--------|-------|-------|-------|
| Sprint 0 | Research | 40 | Pre-GSoC |
| Sprint 1 | MCP Server Foundation | 46 | 2 |
| Sprint 2 | Resources Layer | 78 | 3 |
| Sprint 3 | Tools Layer | 68 | 3 |
| Sprint 4 | Prompts Layer | 44 | 2 |
| Sprint 5 | Authentication & Security | 53 | 1 |
| Sprint 6 | Testing & Hardening | 61 | 1 |
| Sprint 7 | Documentation | 44 | 1 |
| Sprint 8 | Release & Polish | 44 | 1 |
| **Total** | | **522** | **14** |

### Per-Sprint Velocity

```
Hours per sprint:
Sprint 1:  ████████████████████████░░ 46h
Sprint 2:  ████████████████████████████████████████ 78h
Sprint 3:  ██████████████████████████████████░░ 68h
Sprint 4:  ████████████████████░░░░ 44h
Sprint 5:  ██████████████████████████░░░░ 53h
Sprint 6:  ██████████████████████████████░░ 61h
Sprint 7:  ████████████████████░░░░ 44h
Sprint 8:  ████████████████████░░░░ 44h
```

---

## Dependencies

### External Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| Python | >=3.11 | Runtime |
| mcp (PyPI) | >=1.27,<2 | MCP SDK |
| httpx | >=0.27 | HTTP client for BLT-API |
| pydantic | >=2.0 | Data validation |
| python-dotenv | >=1.0 | Environment loading |

### BLT Dependencies

| Service | URL | Purpose |
|---------|-----|---------|
| BLT-API | https://api.owaspblt.org/v2 | Primary data source |
| BLT-Rewards | TBD | BACON token system |
| BLT-Next | https://github.com/OWASP-BLT/BLT-Next | Future alignment |

### Development Dependencies

| Tool | Purpose |
|------|---------|
| pytest | Test runner |
| pytest-asyncio | Async test support |
| httpx | Test HTTP client |
| ruff | Linter |
| black | Formatter |
| mypy | Type checking |
| pre-commit | Pre-commit hooks |

---

## Risk Register

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| BLT-API changes during development | Medium | High | Pin API version, use route discovery |
| MCP Python SDK v2 breaking changes | Medium | High | Pin to v1.x, monitor migration guide |
| BLT-API key rate limits | Low | Medium | Implement caching, request batching |
| OAuth 2.1 implementation complexity | Medium | Medium | Use well-tested libraries, follow spec |
| Integration delays with Claude Desktop | Low | Medium | Test with multiple MCP clients |
| Scope creep | Medium | High | Strict sprint boundaries, MVP focus |

---

## References

1. MCP Python SDK — https://py.sdk.modelcontextprotocol.io/
2. MCP Python SDK README — https://github.com/modelcontextprotocol/python-sdk
3. MCP Specification — https://modelcontextprotocol.io/specification/2025-11-25
4. GSoC 2026 Ideas — https://owasp.org/www-community/initiatives/gsoc/gsoc2026ideas
5. BLT-API Repository — https://github.com/OWASP-BLT/BLT-API
6. BLT-MCP Repository — https://github.com/OWASP-BLT/BLT-MCP
7. OWASP MCP Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/MCP_Security_Cheat_Sheet.html
