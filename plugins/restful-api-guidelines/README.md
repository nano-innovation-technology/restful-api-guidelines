# RESTful API Guidelines

> [한국어](README.ko.md)

RESTful API design guidelines.

---

## Compliance Levels

| Symbol | Level | Description |
|--------|-------|-------------|
| ✅ **Required** | MUST / DO | Rules that must be followed |
| ⚠️ **Recommended** | SHOULD / MAY | Rules that should be followed when possible |
| ❌ **Prohibited** | MUST NOT / DO NOT | Patterns that must not be used |

---

## Table of Contents

1. [Overview](#1-overview)
2. [HTTP Fundamentals](#2-http-fundamentals)
   - [URL Design](#21-url-design)
   - [HTTP Request/Response Patterns](#22-http-requestresponse-patterns)
   - [Query Parameters](#23-query-parameters)
   - [HTTP Headers](#24-http-headers)
3. [REST Principles](#3-rest-principles)
   - [Resource Schema](#31-resource-schema)
   - [Field Mutability](#32-field-mutability)
   - [Create/Update/Replace Handling](#33-createupdatereplace-handling)
   - [Error Handling](#34-error-handling)
4. [JSON Rules](#4-json-rules)
   - [Field Naming](#41-field-naming)
   - [Type System](#42-type-system)
   - [Dates and Times](#43-dates-and-times)
   - [Enum Handling](#44-enum-handling)
5. [Common API Patterns](#5-common-api-patterns)
   - [Actions](#51-actions)
   - [Collections and Pagination](#52-collections-and-pagination)
   - [Filtering and Sorting](#53-filtering-and-sorting)
   - [API Versioning](#54-api-versioning)
   - [Deprecation](#55-deprecation)
   - [Rate Limiting](#56-rate-limiting)
   - [Long-Running Operations](#57-long-running-operations)
6. [Authentication & Security](#6-authentication--security)
   - [Authentication Methods](#61-authentication-methods)
   - [401 vs 403 Distinction](#62-401-vs-403-distinction)
   - [Idempotency-Key](#63-idempotency-key)

---

## 1. Overview

### Purpose

- Ensure consistency, predictability, and maintainability across all RESTful APIs.
- APIs are products consumed by developers.
  - They must be intuitively understandable.
  - They must provide clear messages when errors occur.
  - They must maintain backward compatibility across versions.
- Follows Roy Fielding's RESTful principles ([Architectural Styles and the Design of Network-based Software Architectures](https://roy.gbiv.com/pubs/dissertation/fielding_dissertation.pdf)).
  - HATEOAS is not implemented.

### Scope

- All new HTTP/HTTPS APIs developed within the organization
- Applied as much as possible when improving existing APIs

---

## 2. HTTP Fundamentals

### 2.1 URL Design

#### Basic Structure

```
https://{host}/{service-root}/{resource-collection}/{resource-id}
```

Example:

```
https://api.example.com/users/articles/123
```

#### URL Casing

✅ **Required**: Use lowercase kebab-case for URL paths.

```
# Good
GET /user-profiles
GET /product-categories/123

# Bad
GET /userProfiles
GET /UserProfiles
GET /user_profiles
```

✅ **Required**: Use plural nouns for resource collection names.

```
# Good
GET /articles
GET /users/123/comments

# Bad
GET /article
GET /user/123/comment
```

❌ **Prohibited**: Do not include verbs in URLs. Express actions using HTTP methods.

```
# Bad
POST /createUser
GET /getArticles
DELETE /deleteComment/123

# Good
POST /users
GET /articles
DELETE /comments/123
```

❌ **Prohibited**: Do not include file extensions (`.json`, `.xml`) in URLs. Use the `Accept` header for content negotiation.

#### Allowed Characters

✅ **Required**: Use only lowercase ASCII letters, digits, and hyphens (`-`) in URL path segments.

✅ **Required**: Use camelCase for query parameter names.

```
# Good
GET /articles?pageSize=20&sortOrder=desc

# Bad
GET /articles?page_size=20&sort_order=desc
```

#### URL Length

⚠️ **Recommended**: Keep URLs under 2000 characters. If longer URLs are needed, consider moving query parameters to the request body.

---

### 2.2 HTTP Request/Response Patterns

#### HTTP Methods

| Method | Meaning | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve a resource | ✅ | ✅ |
| POST | Create a resource or perform an action | ❌ | ❌ |
| PUT | Fully replace a resource | ✅ | ❌ |
| PATCH | Partially update a resource | ❌ | ❌ |
| DELETE | Delete a resource | ✅ | ❌ |
| HEAD | Retrieve headers only | ✅ | ✅ |

✅ **Required**: GET requests must not modify server state.

✅ **Required**: PUT requests must be idempotent — sending the same request multiple times must produce the same result.

⚠️ **Recommended**: Use PATCH instead of PUT for partial updates.

❌ **Prohibited**: Do not include a request body in GET, HEAD, or DELETE requests.

#### Status Codes

✅ **Required**: Use the standard HTTP status codes below with their precise meanings.

**2xx Success**

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 OK | Success | Successful GET, PUT, PATCH, POST (action) |
| 201 Created | Created | Successful resource creation via POST |
| 204 No Content | No Content | Successful DELETE, no response body |

**4xx Client Errors**

| Code | Meaning | When to Use |
|------|---------|-------------|
| 400 Bad Request | Bad Request | Malformed request, validation failure |
| 401 Unauthorized | Authentication Required | Missing or expired authentication token |
| 403 Forbidden | Access Denied | Authenticated but lacks permission |
| 404 Not Found | Not Found | Resource does not exist |
| 409 Conflict | Conflict | Duplicate resource, optimistic lock failure |
| 422 Unprocessable Entity | Unprocessable | Semantic validation failure |
| 429 Too Many Requests | Too Many Requests | Rate limit exceeded |

**5xx Server Errors**

| Code | Meaning | When to Use |
|------|---------|-------------|
| 500 Internal Server Error | Server Error | Unexpected server error |
| 503 Service Unavailable | Service Unavailable | Temporary server overload or maintenance |

✅ **Required**: Include the URL of the created resource in the `Location` header for 201 Created responses.

```
HTTP/1.1 201 Created
Location: https://api.example.com/users/articles/456
Content-Type: application/json

{
  "id": "456",
  "title": "New Article Title"
}
```

❌ **Prohibited**: Do not return 200 OK for error situations.

---

### 2.3 Query Parameters

✅ **Required**: Use camelCase for query parameter names.

✅ **Required**: Pass array values by repeating the same parameter name.

```
GET /articles?tag=tech&tag=design
```

⚠️ **Recommended**: Design query parameters as optional. Include required values in the path.

⚠️ **Recommended**: Do not include sensitive information (passwords, tokens, etc.) in query parameters, as they may be recorded in server logs.

❌ **Prohibited**: Do not use query parameters for operations that modify server state.

---

### 2.4 HTTP Headers

#### Request Headers

✅ **Required**: Include the `Content-Type` header when the request has a body.

```
Content-Type: application/json
```

⚠️ **Recommended**: Use the `Accept` header for response format negotiation.

```
Accept: application/json
```

#### Response Headers

✅ **Required**: Include the `Content-Type` header when the response has a body.

⚠️ **Recommended**: Use the `Cache-Control` header to specify caching strategy.

```
Cache-Control: no-cache
Cache-Control: max-age=3600
```

⚠️ **Recommended**: Use the RFC 5988 `Link` header for collection pagination responses.

```
Link: <https://api.example.com/articles?pageSize=20&pageToken=abc>; rel="next",
      <https://api.example.com/articles?pageSize=20>; rel="first"
```

⚠️ **Recommended**: Use the `X-Total-Count` header when providing total item count.

```
X-Total-Count: 100
```

#### Custom Headers

⚠️ **Recommended**: Use clear names without the `X-` prefix for custom headers. The `X-` prefix was deprecated in RFC 6648 (2012).

```
Request-Id: abc-123
Correlation-Id: xyz-789
```

> **Note**: Headers like `X-Request-Id` and `X-Correlation-Id` that have become de facto standards are allowed for legacy compatibility. Do not use the `X-` prefix for new custom headers.

❌ **Prohibited**: Do not redefine the meaning of standard HTTP headers.

---

## 3. REST Principles

### 3.1 Resource Schema

Resources are the core entities exposed by a service. Each resource must be accessible via a unique URL.

✅ **Required**: Every resource must have a unique identifier (`id`).

✅ **Required**: Resource schemas must maintain a consistent structure.

**Standard Resource Fields**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique resource identifier |
| `createdAt` | string (RFC 3339) | Creation timestamp |
| `updatedAt` | string (RFC 3339) | Last modification timestamp |

Example:

```json
{
  "id": "123",
  "title": "RESTful API Design",
  "content": "...",
  "createdAt": "2024-01-15T09:00:00Z",
  "updatedAt": "2024-01-20T14:30:00Z"
}
```

⚠️ **Recommended**: Design resource identifiers as opaque strings. Clients should not parse or depend on the structure of identifiers.

❌ **Prohibited**: Do not include null-valued fields in responses. Omit fields with no value from the response.

```json
// Bad
{
  "id": "123",
  "title": "Title",
  "deletedAt": null
}

// Good
{
  "id": "123",
  "title": "Title"
}
```

---

### 3.2 Field Mutability

Fields are classified by whether they can be changed after creation.

| Classification | Description | Examples |
|----------------|-------------|---------|
| **Create-only** | Can only be set at creation, cannot be changed afterward | `id`, `createdAt` |
| **Read-only** | Managed by the server, cannot be modified by clients | `updatedAt` |
| **Mutable** | Can be modified by clients | `title`, `content` |

✅ **Required**: Ignore server-managed read-only fields (`id`, `createdAt`, `updatedAt`) if included by the client in the request body.

⚠️ **Recommended**: Document the mutability of each field in the API documentation.

---

### 3.3 Create/Update/Replace Handling

#### POST — Create Resource

✅ **Required**: Return 201 Created with the created resource upon successful resource creation.

```
POST /articles
Content-Type: application/json

{
  "title": "New Article Title",
  "content": "Article body content"
}

---

HTTP/1.1 201 Created
Location: /articles/456
Content-Type: application/json

{
  "id": "456",
  "title": "New Article Title",
  "content": "Article body content",
  "createdAt": "2024-01-20T10:00:00Z",
  "updatedAt": "2024-01-20T10:00:00Z"
}
```

#### PUT — Full Resource Replacement

✅ **Required**: PUT requests fully replace the resource. Mutable fields not included in the request body are treated as default or null values.

✅ **Required**: PUT requests must be idempotent.

#### PATCH — Partial Update

⚠️ **Recommended**: PATCH requests modify only the fields included in the request body.

```
PATCH /articles/456
Content-Type: application/json

{
  "title": "Updated Title"
}
```

#### DELETE — Delete Resource

✅ **Required**: Return 204 No Content upon successful deletion.

⚠️ **Recommended**: Return either 404 Not Found or 204 No Content for re-deletion of already-deleted resources. Decide based on service characteristics.

---

### 3.4 Error Handling

#### Error Response Structure

✅ **Required**: All error responses must follow the RFC 7807 / RFC 9457 (Problem Details for HTTP APIs) standard.

✅ **Required**: Use `application/problem+json` as the `Content-Type` for error responses.

```json
{
  "type": "https://api.example.com/errors/resource-not-found",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "The requested article could not be found.",
  "instance": "/articles/999",
  "traceId": "abc-123-xyz"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `type` | ✅ Required | URI identifying the error type (serves as documentation link; `about:blank` allowed) |
| `title` | ✅ Required | Short, human-readable summary of the error type |
| `status` | ✅ Required | HTTP status code (numeric) |
| `detail` | ✅ Required | Specific error description for this request (in language the user can understand) |
| `instance` | ⚠️ Recommended | Request path where the problem occurred |
| `errors` | ⚠️ Recommended | Extension field — list of field-level validation error details |
| `traceId` | ⚠️ Recommended | Extension field — request trace ID (for debugging) |

> **References**: [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807), [RFC 9457](https://datatracker.ietf.org/doc/html/rfc9457)

⚠️ **Recommended**: Return all validation errors at once on validation failure (do not return one at a time).

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Request data failed validation.",
  "instance": "/articles",
  "errors": [
    { "field": "title", "message": "Title is required." },
    { "field": "content", "message": "Content must be at least 10 characters." }
  ],
  "traceId": "abc-123-xyz"
}
```

❌ **Prohibited**: Do not expose internal implementation details in error responses, such as stack traces, internal system paths, or database error messages.

---

## 4. JSON Rules

### 4.1 Field Naming

✅ **Required**: Use camelCase for JSON field names.

```json
// Good
{
  "userId": "123",
  "createdAt": "2024-01-20T10:00:00Z",
  "isActive": true
}

// Bad
{
  "user_id": "123",
  "created_at": "2024-01-20T10:00:00Z",
  "is_active": true
}
```

✅ **Required**: Field names must start with a lowercase letter.

❌ **Prohibited**: Do not overuse abbreviations in field names. Prefer clear, complete words.

```json
// Bad
{
  "usr": "john",
  "ts": "2024-01-20T10:00:00Z",
  "cnt": 5
}

// Good
{
  "username": "john",
  "timestamp": "2024-01-20T10:00:00Z",
  "count": 5
}
```

---

### 4.2 Type System

#### Boolean

✅ **Required**: Use JSON `true`/`false` for boolean values. Do not use strings `"true"`/`"false"` or numbers `1`/`0`.

✅ **Required**: Use prefixes like `is`, `has`, `can` for boolean field names.

```json
{
  "isActive": true,
  "hasPermission": false,
  "canEdit": true
}
```

#### Number

✅ **Required**: Use JSON number type for numeric values.

⚠️ **Recommended**: Return large integers that exceed JavaScript's safe integer range (2^53 - 1) as strings.

```json
{
  "count": 42,
  "price": 19.99,
  "largeId": "9007199254740993"
}
```

#### String

⚠️ **Recommended**: Distinguish between empty string (`""`) and `null`. Omit the field for meaningful "no value" cases; use empty string only when the value is intentionally empty.

---

### 4.3 Dates and Times

✅ **Required**: Represent all date/time values as strings in RFC 3339 format (ISO 8601 profile).

✅ **Required**: Always include the timezone when present. Use `Z` for UTC.

✅ **Required**: Return all time values in server responses as UTC (`Z`). Clients handle conversion to local timezone.

⚠️ **Recommended**: Send time values in client requests as UTC (`Z`). If an offset is included, the server normalizes to UTC before storing.

⚠️ **Recommended**: Use `YYYY-MM-DD` format without timezone for date-only fields (e.g., date of birth).

❌ **Prohibited**: Do not use Unix timestamps (epoch milliseconds/seconds) as the default time format.

#### Server Response Example

```json
{
  "createdAt": "2024-01-20T10:00:00Z",
  "scheduledAt": "2024-01-25T00:30:00Z",
  "birthDate": "1990-05-15"
}
```

#### Client Request Example

```json
// ⚠️ Recommended: UTC
{ "scheduledAt": "2024-01-25T00:30:00Z" }

// Allowed: with offset → server normalizes to UTC before storing
{ "scheduledAt": "2024-01-25T09:30:00+09:00" }
```

#### Server Normalization Rules

✅ **Required**: When a client sends a time with an offset, the server converts it to UTC before storing. Do not return an error.

✅ **Required**: Responses after server normalization must always be returned as UTC (`Z`).

⚠️ **Recommended**: If timezone information is business-critical (e.g., preserving the user's original timezone), use a separate `timeZone` field.

```json
{
  "scheduledAt": "2024-01-25T00:30:00Z",
  "timeZone": "Asia/Seoul"
}
```

---

### 4.4 Enum Handling

✅ **Required**: Use UPPER_SNAKE_CASE strings for enum values.

```json
{
  "status": "PUBLISHED",
  "priority": "HIGH"
}
```

⚠️ **Recommended**: Design for clients to receive unknown enum values. Handle new enum values being added without breaking existing clients.

❌ **Prohibited**: Do not use numbers or unclear abbreviations for enum values.

```json
// Bad
{
  "status": 1,
  "priority": "hi"
}

// Good
{
  "status": "PUBLISHED",
  "priority": "HIGH"
}
```

---

## 5. Common API Patterns

### 5.1 Actions

Use the action pattern for operations that are difficult to express as CRUD (e.g., approve, send, lock).

✅ **Required**: Express actions as `:action` appended to the resource URL.

```
POST /articles/123:publish
POST /users/456:deactivate
POST /orders/789:cancel
```

✅ **Required**: Use the POST method for action endpoints.

⚠️ **Recommended**: Use verb infinitives for action names (publish, cancel, approve).

**Request Example:**

```
POST /articles/123:publish
Content-Type: application/json

{}

---

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "123",
  "status": "PUBLISHED",
  "publishedAt": "2024-01-20T15:00:00Z"
}
```

---

### 5.2 Collections and Pagination

#### Collection Response Structure

✅ **Required**: Return a resource array (top-level JSON array) as the collection response body.

```
HTTP/1.1 200 OK
Link: <https://api.example.com/articles?pageSize=20&pageToken=abc>; rel="next",
      <https://api.example.com/articles?pageSize=20>; rel="first",
      <https://api.example.com/articles?pageSize=20&pageToken=xyz>; rel="last"
X-Total-Count: 100

[
  { "id": "1", "title": "First Article" },
  { "id": "2", "title": "Second Article" }
]
```

| Header | Required | Description |
|--------|----------|-------------|
| `Link` | ⚠️ Recommended | Pagination navigation (RFC 5988) |
| `X-Total-Count` | ⚠️ Recommended | Total item count |

| rel value | Description |
|-----------|-------------|
| `next` | Next page |
| `prev` | Previous page |
| `first` | First page |
| `last` | Last page |

#### Cursor-Based Pagination (Recommended)

⚠️ **Recommended**: Use cursor-based pagination instead of offset-based for large datasets.

**Request:**

```
GET /articles?pageSize=20&pageToken=eyJwYWdlIjoyf...
```

**Response:**

```
HTTP/1.1 200 OK
Link: <https://api.example.com/articles?pageSize=20&pageToken=abc>; rel="next",
      <https://api.example.com/articles?pageSize=20>; rel="first"

[
  { "id": "1", "title": "First Article" },
  ...
]
```

✅ **Required**: Exclude `rel="next"` from the `Link` header when there is no next page.

#### Offset-Based Pagination

Offset-based pagination is also acceptable for small datasets.

**Request:**

```
GET /articles?page=2&pageSize=20
```

**Response:**

```
HTTP/1.1 200 OK
Link: <https://api.example.com/articles?pageSize=20&page=1>; rel="first",
      <https://api.example.com/articles?pageSize=20&page=1>; rel="prev",
      <https://api.example.com/articles?pageSize=20&page=3>; rel="next",
      <https://api.example.com/articles?pageSize=20&page=5>; rel="last"
X-Total-Count: 100

[
  { "id": "21", "title": "Article 21" },
  ...
]
```

⚠️ **Recommended**: Set the default page size (`pageSize`) to 20.

⚠️ **Recommended**: Limit the maximum page size to 100.

---

### 5.3 Filtering and Sorting

#### Filtering

⚠️ **Recommended**: Express filters as query parameters.

**Equality filter:**

```
GET /articles?status=PUBLISHED&authorId=123
```

**Range filter:** Use `After`/`Before` suffixes.

```
GET /articles?createdAfter=2024-01-01T00:00:00Z
GET /articles?createdBefore=2024-02-01T00:00:00Z
GET /articles?createdAfter=2024-01-01T00:00:00Z&createdBefore=2024-02-01T00:00:00Z
```

**Multi-value filter (IN):** Repeat the same parameter for OR conditions.

```
GET /articles?status=PUBLISHED&status=DRAFT
GET /articles?authorId=123&authorId=456
```

✅ **Required**: Treat repeated same parameters as OR conditions. Combinations of different parameters are AND conditions.

```
# status=PUBLISHED OR status=DRAFT, AND authorId=123
GET /articles?status=PUBLISHED&status=DRAFT&authorId=123
```

**Partial match (contains):** Use the `q` parameter or append `Contains` to the field name.

```
GET /articles?q=REST              # full-text search
GET /articles?titleContains=REST  # partial match on specific field
```

❌ **Prohibited**: Do not treat repeated same parameters as AND conditions.

#### Sorting

⚠️ **Recommended**: Use the `orderBy` parameter for sorting, combining field name and direction.

```
GET /articles?orderBy=createdAt:desc
GET /articles?orderBy=title:asc
GET /articles?orderBy=createdAt:desc,title:asc
```

⚠️ **Recommended**: Document the default sort order in the API documentation.

---

### 5.4 API Versioning

#### Version Notation

❌ **Prohibited**: Do not include the API version in the URL path.

> **Reason**: A URL is a resource identifier. `/v1/articles` and `/v2/articles` represent the same resource but with different URLs, which violates REST principles. URL versioning also forces clients to rewrite their code entirely. Header versioning allows clients to migrate gradually and gives the server flexibility to apply a default version when no version header is specified.

✅ **Required**: Specify the version in the `X-API-Version` header using ISO 8601 (`YYYY-MM-DD`) date format.

```
X-API-Version: 2024-01-20
```

⚠️ **Recommended**: Apply the latest stable version to requests without a version header, and include the applied version in the response.

```
HTTP/1.1 200 OK
X-API-Version: 2024-01-20
```

#### Backward Compatibility

✅ **Required**: Maintain backward compatibility within the same version.

**Backward-compatible changes (allowed):**
- Adding new optional fields
- Adding new endpoints
- Adding new enum values

**Backward-incompatible changes (requires version bump):**
- Renaming or removing fields
- Changing field types
- Adding required fields
- Removing enum values
- Changing status code semantics

⚠️ **Recommended**: Maintain the previous version for at least 6 months before a version bump.

---

### 5.5 Deprecation

✅ **Required**: Notify clients of deprecated APIs via response headers.

```
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/users/articles>; rel="successor-version"
```

⚠️ **Recommended**: Announce deprecation at least 6 months before the end-of-life date.

⚠️ **Recommended**: Notify clients with `Deprecation`, `Sunset`, and `Link` headers on deprecated API calls.

```
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/users/articles>; rel="successor-version"

[
  { "id": "1", "title": "Item 1" },
  ...
]
```

---

### 5.6 Rate Limiting

API servers limit request frequency per client to ensure service stability.

#### Response Headers

✅ **Required**: Include the following headers in all responses where rate limiting applies.

**Legacy headers (X-RateLimit-\*)**

| Header | Description | Example |
|--------|-------------|---------|
| `X-RateLimit-Limit` | Maximum requests allowed within the time window | `100` |
| `X-RateLimit-Remaining` | Remaining requests in the current time window | `99` |
| `X-RateLimit-Reset` | Time when the window resets (Unix timestamp, seconds) | `1742342450` |

**IETF standard headers (draft-ietf-httpapi-ratelimit-headers)**

| Header | Description | Example |
|--------|-------------|---------|
| `RateLimit` | Current rate limit state (Structured Field) | `limit=100, remaining=99, reset=50` |
| `RateLimit-Policy` | Active rate limit policy | `100;w=3600` |

> **Note**: The `reset` value in the `RateLimit` header is **delta-seconds** (seconds until window reset), while `X-RateLimit-Reset` is a **Unix timestamp**. Be careful not to confuse them.
>
> **`RateLimit-Policy` structure**: `100;w=3600` — `100` is the maximum allowed requests, `w=3600` is the window size in seconds. Include in all responses.

**Normal response example:**

```
HTTP/1.1 200 OK
Content-Type: application/json
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1742342450
RateLimit: limit=100, remaining=99, reset=50
RateLimit-Policy: 100;w=3600

[
  { "id": "1", "title": "Item 1" }
]
```

#### 429 Too Many Requests Response

✅ **Required**: Return `429 Too Many Requests` when rate limit is exceeded.

✅ **Required**: Include the `Retry-After` header in 429 responses. Use delta-seconds (seconds to wait before retrying).

✅ **Required**: Use RFC 7807 Problem Details structure for 429 response bodies.

**429 response example:**

```
HTTP/1.1 429 Too Many Requests
Content-Type: application/problem+json
Retry-After: 50
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1742342450
RateLimit: limit=100, remaining=0, reset=50
RateLimit-Policy: 100;w=3600
```

```json
{
  "type": "https://api.example.com/errors/too-many-requests",
  "title": "Rate Limit Exceeded",
  "status": 429,
  "detail": "You have exceeded the allowed request limit. Please try again in 50 seconds."
}
```

#### Client Retry Strategy

✅ **Required**: Clients must wait for the duration specified in the `Retry-After` header before retrying after a 429 response.

⚠️ **Recommended**: When the `Retry-After` header is absent or other transient errors occur (503, etc.), use an exponential backoff + jitter strategy. `attempt` starts at 1 (1 = first retry).

```
wait_time = min(maxDelay, baseDelay × 2^(attempt - 1)) + random(0, jitterRange)
```

Example: attempt=1 → `min(60, 1 × 2^0) + random(0,1)` = 1~2 seconds

| Parameter | Recommended Value | Description |
|-----------|-------------------|-------------|
| `baseDelay` | 1 second | Wait time for the first retry |
| `maxDelay` | 60 seconds | Maximum wait time cap |
| `jitterRange` | 0 ~ 1 second | Random delay (to prevent thundering herd) |
| Max retry count | 3 ~ 5 times | Prevent infinite retries |

❌ **Prohibited**: Do not retry immediately or at fixed intervals after receiving a 429 response.

❌ **Prohibited**: Do not ignore the `Retry-After` header and use your own wait time when it is present.

---

### 5.7 Long-Running Operations

For operations that do not complete immediately (report generation, data import, etc.), create a domain resource immediately and track processing progress via the resource's status field.

✅ **Required**: Upon receiving a long-running operation request, create the domain resource immediately and return `201 Created` + `Location` header.

✅ **Required**: Include a `status` field in the domain resource to represent processing state.

⚠️ **Recommended**: Use the following status values:

| Status | Description |
|--------|-------------|
| `PENDING` | Operation is waiting to start |
| `IN_PROGRESS` | Operation is being processed |
| `COMPLETED` | Operation has completed |
| `FAILED` | Operation has failed |

⚠️ **Recommended**: When status is `FAILED`, include an RFC 7807 error structure in the resource.

**Example:**

```
# Start operation — immediately create domain resource
POST /reports  →  201 Created
                  Location: /reports/123
                  { "id": "123", "status": "PENDING" }

# Poll status
GET /reports/123  →  { "id": "123", "status": "IN_PROGRESS" }
GET /reports/123  →  { "id": "123", "status": "COMPLETED", ... }
GET /reports/123  →  { "id": "123", "status": "FAILED", "error": { ... } }
```

❌ **Prohibited**: Do not use a separate generic `/operations` resource. Track status within the domain resource itself.

---

## 6. Authentication & Security

### 6.1 Authentication Methods

#### Bearer Token (JWT)

✅ **Required**: Use the `Authorization` header for authentication tokens. Do not include in query parameters or request body.

```
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...
```

#### API Key

⚠️ **Recommended**: Use the `Authorization` header for API Key authentication.

```
Authorization: ApiKey your-api-key-here
```

❌ **Prohibited**: Do not pass API Keys as query parameters. URLs may be recorded in server logs.

```
# Bad
GET /articles?apiKey=secret-key
```

---

### 6.2 401 vs 403 Distinction

| Status Code | Meaning | When to Use |
|-------------|---------|-------------|
| `401 Unauthorized` | Authentication failure | Missing token, expired token, malformed token |
| `403 Forbidden` | Authorization failure | Authenticated but lacks permission for the resource/action |

✅ **Required**: Include the `WWW-Authenticate` header in 401 responses.

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api", error="token_expired"
Content-Type: application/problem+json

{
  "type": "https://api.example.com/errors/unauthorized",
  "title": "Authentication Required",
  "status": 401,
  "detail": "The access token has expired."
}
```

⚠️ **Recommended**: Do not reveal resource existence in 403 responses. For security-sensitive resources, you may respond with 404 as if the resource does not exist.

```
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json

{
  "type": "https://api.example.com/errors/forbidden",
  "title": "Access Denied",
  "status": 403,
  "detail": "You do not have permission to access this resource."
}
```

---

### 6.3 Idempotency-Key

POST is not idempotent, so retrying after a network error can result in duplicate resource creation. Use the `Idempotency-Key` header for APIs where duplicate execution is dangerous, such as payments, orders, and email sending.

✅ **Required**: Support the `Idempotency-Key` header for POST endpoints where duplicate execution is risky.

```
POST /orders
Content-Type: application/json
Idempotency-Key: a8098c1a-f86e-11da-bd1a-00112444be1e

{
  "productId": "123",
  "quantity": 2
}
```

**Server Behavior:**
- First request: Process normally and store the result
- Re-request with same key: Return the stored result without reprocessing
- Same request with different key: Treat as a separate request

✅ **Required**: Use client-generated UUID v4 for `Idempotency-Key` values.

⚠️ **Recommended**: Return the same status code and body as the original response for re-requests with the same key.

⚠️ **Recommended**: Set the validity period of `Idempotency-Key` to at least 24 hours.

❌ **Prohibited**: Do not design POST endpoints with financial impact (payments, orders, etc.) without `Idempotency-Key` support.

---

## References

- [Microsoft Azure REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)
- [RFC 3339 - Date and Time on the Internet](https://datatracker.ietf.org/doc/html/rfc3339)
- [HTTP/1.1 (RFC 7231)](https://datatracker.ietf.org/doc/html/rfc7231)
- [JSON:API Specification](https://jsonapi.org/)
- [Day1, 2-2. 그런 REST API로 괜찮은가](https://www.youtube.com/watch?v=RP_f5dMoHFc)
- [Architectural Styles and the Design of Network-based Software Architectures - Roy Fielding](https://roy.gbiv.com/pubs/dissertation/fielding_dissertation.pdf)
- [RFC 8288 - Web Linking](https://datatracker.ietf.org/doc/html/rfc8288)
- [RFC 6585 - Additional HTTP Status Codes (429)](https://datatracker.ietf.org/doc/html/rfc6585#section-4)
- [IETF draft-ietf-httpapi-ratelimit-headers - RateLimit Header Fields](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/)
