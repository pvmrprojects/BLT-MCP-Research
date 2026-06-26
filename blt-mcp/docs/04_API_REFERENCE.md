# BLT API Reference

> **Document Version:** 1.0
> **Status:** Draft
> **Date:** 2026-06-26
> **Base URL:** `https://api.owaspblt.org/v2`

---

## Overview

This document catalogs all public endpoints of the OWASP BLT API (BLT-API), which serves as the primary backend interface for the BLT-MCP server. The API is built with Python on Cloudflare Workers and uses D1 (SQLite) for data persistence.

**Key characteristics:**
- RESTful design with JSON responses
- Versioned under `/v2` prefix
- Shared static API key authentication via `X-BLT-API-Key` header
- Pagination support (page/per_page)
- Edge-deployed on Cloudflare's global network

[Source: BLT-API README, github.com/OWASP-BLT/BLT-API]

---

## Authentication

### API Key Authentication

All protected endpoints require a static API key sent via HTTP header:

```
X-BLT-API-Key: sk_xxxxx
```

**Public routes (no auth required):**
- `GET /` — API homepage
- `GET /v2` — API homepage (versioned)
- `GET /health` — Health check
- `GET /v2/health` — Health check
- `OPTIONS` — CORS preflight

**Authentication failure response:**
```json
{
  "error": "Unauthorized",
  "message": "Missing or invalid API key",
  "status": 401
}
```

[Source: BLT-API PR #89, github.com/OWASP-BLT/BLT-API/pull/89]

### Configuration

API key is configured via environment variable:

| Variable | Description |
|----------|-------------|
| `BLT_API_KEY` | Shared static API key for authentication |

---

## Endpoints Summary

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/` | No | API homepage |
| `GET` | `/v2` | No | Versioned homepage |
| `GET` | `/health` | No | Health check |
| `GET` | `/v2/health` | No | Health check |
| `GET` | `/v2/routes` | Yes | Programmatic route discovery |
| `GET` | `/v2/bugs` | Yes | List bugs |
| `GET` | `/v2/bugs/{id}` | Yes | Get bug details |
| `POST` | `/v2/bugs` | Yes | Create bug |
| `GET` | `/v2/bugs/search` | Yes | Search bugs |
| `GET` | `/v2/users` | Yes | List users |
| `POST` | `/v2/users` | Yes | Create user |
| `GET` | `/v2/users/{id}` | Yes | Get user |
| `GET` | `/v2/users/{id}/profile` | Yes | User profile with stats |
| `GET` | `/v2/users/{id}/bugs` | Yes | User's bugs |
| `GET` | `/v2/users/{id}/domains` | Yes | User's domains |
| `GET` | `/v2/users/{id}/followers` | Yes | User's followers |
| `GET` | `/v2/users/{id}/following` | Yes | User's following |
| `POST` | `/v2/auth/signup` | Yes | Create account |
| `POST` | `/v2/auth/signin` | Yes | Sign in |
| `GET` | `/v2/domains` | Yes | List domains |
| `GET` | `/v2/domains/{id}` | Yes | Get domain |
| `GET` | `/v2/domains/{id}/tags` | Yes | Domain tags |
| `GET` | `/v2/organizations` | Yes | List organizations |
| `GET` | `/v2/organizations/{id}` | Yes | Get organization |
| `GET` | `/v2/organizations/{id}/domains` | Yes | Organization domains |
| `GET` | `/v2/organizations/{id}/bugs` | Yes | Organization bugs |
| `GET` | `/v2/organizations/{id}/managers` | Yes | Organization managers |
| `GET` | `/v2/organizations/{id}/tags` | Yes | Organization tags |
| `GET` | `/v2/organizations/{id}/integrations` | Yes | Organization integrations |
| `GET` | `/v2/organizations/{id}/stats` | Yes | Organization statistics |
| `GET` | `/v2/hunts` | Yes | List bug bounty hunts |
| `GET` | `/v2/projects` | Yes | List projects |

---

## Endpoint Details

### GET / (Public)

API homepage with interactive documentation.

**Response:**
```json
{
  "name": "BLT API",
  "version": "2.0",
  "description": "Full-featured REST API for OWASP BLT",
  "documentation": "/v2/routes"
}
```

---

### GET /health (Public)

Health check endpoint for monitoring.

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2026-06-26T12:00:00Z",
  "version": "2.0.0"
}
```

---

### GET /v2/routes (Auth Required)

Programmatic API discovery — returns all registered routes.

**Response:**
```json
{
  "success": true,
  "data": [
    { "method": "GET", "path": "/v2/health" },
    { "method": "GET", "path": "/v2/bugs" },
    { "method": "GET", "path": "/v2/bugs/{id}" },
    { "method": "POST", "path": "/v2/bugs" },
    { "method": "GET", "path": "/v2/bugs/search" },
    { "method": "GET", "path": "/v2/users" },
    { "method": "POST", "path": "/v2/users" },
    ...
  ],
  "count": 28
}
```

[Source: BLT-API PR #68, github.com/OWASP-BLT/BLT-API/pull/68]

---

### GET /v2/bugs (Auth Required)

List all bugs with pagination and filtering.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page (max: 100) |
| `status` | string | - | Filter: open, closed, reviewing |
| `domain` | integer | - | Filter by domain ID |
| `verified` | boolean | - | Filter by verification status |

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1234,
      "title": "XSS vulnerability in login form",
      "description": "The login page reflects user input...",
      "url": "https://example.com/login",
      "status": "open",
      "severity": "high",
      "domain_id": 42,
      "reporter_id": 567,
      "verified": false,
      "created_at": "2026-06-25T10:30:00Z",
      "screenshots": [],
      "tags": ["xss", "security"]
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 1500,
    "total_pages": 75
  }
}
```

**Data Source:** Cloudflare D1 database [Source: BLT-API DATABASE.md]

**Used by BLT-MCP:**
- Resource: `blt://issues` → maps to `GET /v2/bugs`
- Tool: `submit_issue` → maps to `POST /v2/bugs`

---

### GET /v2/bugs/{id} (Auth Required)

Get a single bug with full details including screenshots and tags.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | Bug ID |

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 1234,
    "title": "XSS vulnerability in login form",
    "description": "The login page reflects user input...",
    "url": "https://example.com/login",
    "status": "open",
    "severity": "high",
    "domain_id": 42,
    "domain_name": "example.com",
    "reporter_id": 567,
    "reporter_username": "security_researcher",
    "verified": false,
    "verification_count": 3,
    "created_at": "2026-06-25T10:30:00Z",
    "updated_at": "2026-06-26T08:15:00Z",
    "screenshots": [
      {
        "id": 1,
        "url": "https://blt.owasp.org/media/screenshots/1234_1.png",
        "caption": "XSS proof of concept"
      }
    ],
    "tags": ["xss", "security", "high-priority"],
    "cve_id": "CVE-2026-1234",
    "cve_score": 7.5
  }
}
```

**Used by BLT-MCP:**
- Resource: `blt://issues/{id}` → maps to `GET /v2/bugs/{id}`

---

### POST /v2/bugs (Auth Required)

Create a new bug report.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Issue title |
| `description` | string | Yes | Detailed description |
| `url` | string | Yes | URL where bug was found |
| `severity` | string | No | Severity: low, medium, high, critical |
| `domain_id` | integer | No | Domain ID |
| `screenshots` | array | No | Base64-encoded screenshots |
| `tags` | array | No | List of tag strings |
| `cve_id` | string | No | CVE identifier |

**Validation Rules:**
- URL must use `http://` or `https://` protocol
- URL must contain a valid domain (netloc check)
- Fields validated at the API layer

[Source: BLT-API PR #35, github.com/OWASP-BLT/BLT-API/pull/35]

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": 1235,
    "title": "Broken link on homepage",
    "status": "open",
    "created_at": "2026-06-26T12:00:00Z"
  },
  "message": "Bug report created successfully"
}
```

**Error Responses:**

| Code | Meaning |
|------|---------|
| 400 | Validation error (invalid URL, missing fields) |
| 401 | Missing or invalid API key |
| 429 | Rate limit exceeded |

**Used by BLT-MCP:**
- Tool: `submit_issue` → maps to `POST /v2/bugs`

---

### GET /v2/bugs/search (Auth Required)

Search bugs by URL or description.

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Search query (searches URL and description) |
| `status` | string | Optional status filter |

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1234,
      "title": "XSS vulnerability in login form",
      "url": "https://example.com/login",
      "status": "open",
      "severity": "high",
      "created_at": "2026-06-25T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 5,
    "total_pages": 1
  }
}
```

---

### GET /v2/users (Auth Required)

List all users with pagination.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page (max: 100) |

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 567,
      "username": "security_researcher",
      "email": "user@example.com",
      "profile_picture": "https://gravatar.com/avatar/...",
      "reputation": 1250,
      "joined_at": "2025-01-15T08:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 850,
    "total_pages": 43
  }
}
```

**Used by BLT-MCP:**
- Resource: `blt://contributors` → maps to `GET /v2/users`

---

### POST /v2/users (Auth Required)

Create a new user account.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `username` | string | Yes | Username |
| `email` | string | Yes | Email address |
| `password` | string | Yes | Password |

**Characteristics:**
- Rate-limited
- Input validated
- Password hashed before storage

---

### GET /v2/users/{id} (Auth Required)

Get a specific user.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | User ID |

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 567,
    "username": "security_researcher",
    "email": "user@example.com",
    "profile_picture": "https://gravatar.com/avatar/...",
    "reputation": 1250,
    "joined_at": "2025-01-15T08:00:00Z",
    "bug_count": 47,
    "domain_count": 12
  }
}
```

**Used by BLT-MCP:**
- Resource: `blt://contributors/{id}` → maps to `GET /v2/users/{id}`

---

### GET /v2/users/{id}/profile (Auth Required)

Get user profile with statistics.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 567,
    "username": "security_researcher",
    "reputation": 1250,
    "rank": 15,
    "total_bugs": 47,
    "verified_bugs": 32,
    "domains_submitted": 12,
    "badges": ["bug-hunter", "verified-reporter"],
    "bacon_balance": 2500,
    "joined_at": "2025-01-15T08:00:00Z"
  }
}
```

---

### GET /v2/users/{id}/bugs (Auth Required)

Get bugs reported by a specific user.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | User ID |

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |
| `status` | string | - | Filter by status |

**Response:** Same format as `GET /v2/bugs` but filtered by reporter.

---

### GET /v2/users/{id}/domains (Auth Required)

Get domains submitted by a specific user.

---

### GET /v2/users/{id}/followers (Auth Required)

Get users following this user.

---

### GET /v2/users/{id}/following (Auth Required)

Get users this user follows.

---

### POST /v2/auth/signup (Auth Required)

Create a new account with authentication.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `username` | string | Yes | Username |
| `email` | string | Yes | Email address |
| `password` | string | Yes | Password |

---

### POST /v2/auth/signin (Auth Required)

Authenticate and receive a session token.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `username` | string | Yes | Username |
| `password` | string | Yes | Password |

---

### GET /v2/domains (Auth Required)

List all domains with pagination.

**Data Source:** Cloudflare D1 database. [Source: BLT-API]

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page |

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 42,
      "name": "example.com",
      "url": "https://example.com",
      "description": "Example company website",
      "organization_id": 7,
      "bug_count": 15,
      "created_at": "2025-03-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 500,
    "total_pages": 25
  }
}
```

---

### GET /v2/domains/{id} (Auth Required)

Get a specific domain.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | Domain ID |

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 42,
    "name": "example.com",
    "url": "https://example.com",
    "description": "Example company website",
    "organization_id": 7,
    "organization_name": "Example Corp",
    "bug_count": 15,
    "verified_bugs": 10,
    "created_at": "2025-03-01T00:00:00Z",
    "tags": ["ecommerce", "high-traffic"]
  }
}
```

---

### GET /v2/domains/{id}/tags (Auth Required)

Get tags for a specific domain.

**Data Source:** Cloudflare D1 database.

**Response:**
```json
{
  "success": true,
  "data": ["ecommerce", "high-traffic", "fintech"]
}
```

---

### GET /v2/organizations (Auth Required)

List organizations with search and filtering.

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | integer | Page number |
| `per_page` | integer | Items per page |
| `search` | string | Search by name, slug, description |
| `type` | string | Filter by organization type |
| `is_active` | boolean | Filter by active status |

[Source: BLT-API PR #41, github.com/OWASP-BLT/BLT-API/pull/41]

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 7,
      "name": "Example Corp",
      "slug": "example-corp",
      "description": "A software company",
      "type": "company",
      "is_active": true,
      "domain_count": 5,
      "bug_count": 47,
      "verified_bugs": 32,
      "manager_count": 3,
      "created_at": "2024-06-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 120,
    "total_pages": 6
  }
}
```

---

### GET /v2/organizations/{id} (Auth Required)

Get organization details.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | Organization ID |

---

### GET /v2/organizations/{id}/domains (Auth Required)

Get domains belonging to an organization.

---

### GET /v2/organizations/{id}/bugs (Auth Required)

Get bugs reported against an organization's domains.

---

### GET /v2/organizations/{id}/managers (Auth Required)

Get managers of an organization.

---

### GET /v2/organizations/{id}/tags (Auth Required)

Get tags associated with an organization.

---

### GET /v2/organizations/{id}/integrations (Auth Required)

Get integrations configured for an organization.

---

### GET /v2/organizations/{id}/stats (Auth Required)

Get statistics for an organization.

**Response:**
```json
{
  "success": true,
  "data": {
    "organization_id": 7,
    "domain_count": 5,
    "bug_count": 47,
    "verified_bugs": 32,
    "open_bugs": 12,
    "manager_count": 3,
    "average_severity": 6.5,
    "top_tags": ["security", "bug-bounty"]
  }
}
```

---

## Error Codes

| HTTP Status | Code | Meaning |
|-------------|------|---------|
| 400 | Bad Request | Invalid input parameters |
| 401 | Unauthorized | Missing or invalid API key |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side failure |

**Error Response Format:**
```json
{
  "success": false,
  "error": "validation_error",
  "message": "URL must use http:// or https:// protocol",
  "status": 400
}
```

---

## BLT-MCP to REST Mapping

| MCP Layer | MCP Element | REST Endpoint |
|-----------|-------------|---------------|
| **Resource** | `blt://issues` | `GET /v2/bugs` |
| **Resource** | `blt://issues/{id}` | `GET /v2/bugs/{id}` |
| **Resource** | `blt://contributors` | `GET /v2/users` |
| **Resource** | `blt://contributors/{id}` | `GET /v2/users/{id}` |
| **Resource** | `blt://repos` | TBD (GitHub integration) |
| **Resource** | `blt://leaderboards` | `GET /v2/users?sort=reputation` |
| **Resource** | `blt://rewards` | BACON system API |
| **Tool** | `submit_issue` | `POST /v2/bugs` |
| **Tool** | `award_bacon` | BACON system API |
| **Tool** | `update_issue_status` | `PATCH /v2/bugs/{id}` (if available) |
| **Tool** | `add_comment` | TBD (comment endpoint) |

---

## References

1. BLT-API Repository — https://github.com/OWASP-BLT/BLT-API
2. BLT-API PR #35 (URL Validation) — https://github.com/OWASP-BLT/BLT-API/pull/35
3. BLT-API PR #41 (Organizations) — https://github.com/OWASP-BLT/BLT-API/pull/41
4. BLT-API PR #68 (Routes Discovery) — https://github.com/OWASP-BLT/BLT-API/pull/68
5. BLT-API PR #89 (API Key Auth) — https://github.com/OWASP-BLT/BLT-API/pull/89
6. BLT-MCP Repository — https://github.com/OWASP-BLT/BLT-MCP
