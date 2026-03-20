---
name: restful-api-guidelines
description: Applies guidelines when writing or reviewing RESTful API code. Automatically activated when writing API endpoint code, implementing controllers/handlers, or requesting API design review.
---

# RESTful API Guidelines Skill

## Trigger Conditions

Apply this skill in the following situations:
- Writing or modifying API endpoint code
- Implementing REST controllers/handlers
- Designing API response structures
- API code review requests

---

## Code Writing Mode

Follow the rules below when writing API code.

### URL Naming Rules

```
# Plural nouns + kebab-case
GET    /articles
POST   /articles
GET    /articles/{id}
PUT    /articles/{id}
PATCH  /articles/{id}
DELETE /articles/{id}

# Nested resources
GET    /users/{userId}/comments

# Action pattern (for operations that cannot be expressed as CRUD)
POST   /articles/{id}:publish
POST   /orders/{id}:cancel
POST   /users/{id}:deactivate

# URL path segments: only ASCII lowercase letters, digits, hyphens
# Correct: /user-profiles, /article-categories
# Incorrect: /user_profiles, /UserProfiles, /사용자

# Query parameters: use same parameter name repeated for array values (OR condition)
GET /articles?status=PUBLISHED&status=DRAFT

# Query parameters MUST NOT be used for state-changing operations
# Bad:  POST /articles?action=publish
# Good: POST /articles/123:publish

# Sensitive information (tokens, passwords) MUST NOT be placed in query parameters
```

### HTTP Method to Status Code Mapping

| Method | Success Code | Description |
|--------|-------------|-------------|
| GET | 200 OK | Retrieval success |
| POST (create) | 201 Created + Location header | Resource created |
| POST (action) | 200 OK | Action performed |
| POST (long-running) | 201 Created + Location header | Long-running task — domain resource created immediately with status field |
| PUT | 200 OK | Full replacement success |
| PATCH | 200 OK | Partial update success |
| DELETE | 204 No Content | Deletion success (no body) |

✅ Required: GET requests MUST NOT modify server state (safe method)
✅ Required: PUT requests MUST be idempotent — identical requests must produce the same result
✅ Required: Include `Content-Type: application/json` header when request body is present
✅ Required: Include `Content-Type` header in all responses with a body

### Error Response Format Template

All error responses use RFC 7807/9457 Problem Details structure.
`Content-Type: application/problem+json`

```json
{
  "type": "https://api.example.com/errors/error-type",
  "title": "Short summary of the error type",
  "status": 400,
  "detail": "Specific error description for this request",
  "instance": "/request/path",
  "errors": [
    {
      "field": "fieldName",
      "message": "Specific problem with this field"
    }
  ],
  "traceId": "request_trace_id"
}
```

**Commonly used error type URIs:**

| HTTP Status | type | When to use |
|-------------|------|-------------|
| 400 | `.../errors/invalid-request` | Malformed request |
| 400 | `.../errors/validation-failed` | Validation failure (include errors array) |
| 401 | `.../errors/unauthorized` | Authentication required |
| 403 | `.../errors/forbidden` | Insufficient permissions |
| 404 | `.../errors/resource-not-found` | Resource not found |
| 409 | `.../errors/conflict` | Duplicate or conflict |
| 429 | `.../errors/too-many-requests` | Rate limit exceeded |
| 500 | `.../errors/internal-server-error` | Internal server error |

### JSON Field Naming Rules

```json
// Correct example
{
  "id": "123",
  "userId": "456",
  "isActive": true,
  "createdAt": "2024-01-20T10:00:00Z",
  "updatedAt": "2024-01-20T15:00:00Z",
  "status": "PUBLISHED"
}

// Incorrect example — strictly prohibited
{
  "user_id": "456",       // snake_case forbidden
  "is_active": true,      // snake_case forbidden
  "created_at": "...",    // snake_case forbidden
  "status": "published"   // enums must be UPPER_SNAKE_CASE
}

// Type rules
{
  "isActive": true,    // Boolean: use native JSON true/false, NOT "true"/"false" or 1/0
  "count": 42,         // Number: use JSON number type, NOT string "42"
  "isEnabled": false   // Boolean fields MUST use is/has/can prefix
}
```

✅ Required: Field names MUST start with a lowercase letter (a-z) — no UpperCamelCase (e.g., `userId` not `UserId`)

### Collection/Pagination Pattern

**Response structure (RFC 5988 Link header + top-level array):**

```
HTTP/1.1 200 OK
Link: <https://api.example.com/articles?pageSize=20&pageToken=abc>; rel="next",
      <https://api.example.com/articles?pageSize=20>; rel="first",
      <https://api.example.com/articles?pageSize=20&pageToken=xyz>; rel="last"
X-Total-Count: 100

[
  { "id": "1", "title": "Item 1" },
  { "id": "2", "title": "Item 2" }
]
```

**Request parameters (camelCase):**

```
GET /articles?pageSize=20&pageToken=eyJ...
GET /articles?pageSize=20&page=2
GET /articles?status=PUBLISHED&orderBy=createdAt:desc
GET /articles?orderBy=createdAt:desc,title:asc
```

### Standard Resource Fields

Fields automatically managed by the server on create/update:

```json
{
  "id": "Server-generated, not modifiable by client",
  "createdAt": "RFC 3339, server-generated",
  "updatedAt": "RFC 3339, auto-updated by server"
}
```

✅ Required: All resources MUST have a unique identifier (`id`)
✅ Required: Server-managed read-only fields (`id`, `createdAt`, `updatedAt`) MUST be silently ignored if included in the client's request body
✅ Required: PUT replaces the entire resource — mutable fields not included in the request body are reset to default or null

### Date/Time

**Server response (✅ Required: UTC)**
```json
{
  "createdAt": "2024-01-20T10:00:00Z",
  "scheduledAt": "2024-01-25T00:30:00Z",
  "birthDate": "1990-05-15"
}
```

**Client request (⚠️ Recommended: UTC, offset allowed)**
```json
{ "scheduledAt": "2024-01-25T00:30:00Z" }         // Recommended
{ "scheduledAt": "2024-01-25T09:30:00+09:00" }     // Allowed — server normalizes to UTC
```

**Prohibited**
```json
{
  "createdAt": 1705744800000,  // Unix timestamp forbidden
  "createdAt": "2024-01-20"   // Must include time (except date-only fields)
}
```

### Authentication

**Auth header (✅ Required: use Authorization header)**

```
# Bearer Token
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...

# API Key
Authorization: ApiKey your-api-key-here
```

**401 vs 403 distinction:**

| Status | Cause | WWW-Authenticate Header |
|--------|-------|------------------------|
| 401 | Missing/expired/malformed token | ✅ Required |
| 403 | Authenticated but insufficient permissions | Not needed |

### Idempotency-Key

Required for POST operations with risk of duplicate execution (payments, orders, etc.):

```
POST /orders
Idempotency-Key: a8098c1a-f86e-11da-bd1a-00112444be1e
```

- Client MUST generate the key as a UUID v4 (e.g., using UUID.randomUUID() in Java/Kotlin)
- On re-request with same key → return existing result as-is (no reprocessing)
- Key validity period: minimum 24 hours

### Rate Limiting

**Response headers (✅ Required: include both)**

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1742342450
RateLimit: limit=100, remaining=99, reset=50
RateLimit-Policy: 100;w=3600
```

**429 response (✅ Required: Retry-After + Problem Details)**

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
  "title": "Rate limit exceeded",
  "status": 429,
  "detail": "You have exceeded the allowed request limit. Please retry after 50 seconds."
}
```

**Client retry (✅ Required / ⚠️ Recommended)**

- ✅ Required: On 429, wait for the `Retry-After` header value before retrying
- ⚠️ Recommended: If no `Retry-After`, use exponential backoff + jitter (`min(60, 1 × 2^(n-1)) + random(0,1)`, n starting from 1)
- Maximum 3–5 retries; immediate retry forbidden

---

## Code Review Mode

When reviewing API code, identify violations using the checklist below and suggest fixes.

### Review Checklist

#### URL Design
- [ ] Lowercase kebab-case used in paths
- [ ] Resource names are plural nouns
- [ ] No verbs in paths (actions use `:action` pattern)
- [ ] No file extensions in URLs
- [ ] No version in URL path (`/v1/`, `/v2/`, etc.)
- [ ] URL path segments use only ASCII lowercase letters, digits, and hyphens (no underscores, no uppercase, no non-ASCII)
- [ ] Array values passed by repeating the same parameter name (e.g., ?status=A&status=B), not comma-separated
- [ ] Query parameters not used for state-changing operations (use POST + action pattern instead)
- [ ] Sensitive information (tokens, API keys, passwords) not included in query parameters

#### Versioning
- [ ] API version delivered via `X-API-Version` header, not URL path
- [ ] `X-API-Version` value uses ISO 8601 date format (`YYYY-MM-DD`)
- [ ] Breaking changes (field removal/rename, type change, required field addition) are accompanied by a new X-API-Version date
- [ ] Non-breaking changes (new optional fields, new endpoints, new Enum values) do not require version bump

#### HTTP Methods & Status Codes
- [ ] POST create → 201 + Location header
- [ ] GET, PUT, PATCH success → 200
- [ ] DELETE success → 204 (no body)
- [ ] GET requests do not modify server state
- [ ] PUT request is idempotent (same request sent multiple times produces the same result)
- [ ] GET requests do not include a request body
- [ ] HEAD requests do not include a request body
- [ ] DELETE requests do not include a request body
- [ ] 200 not returned for error conditions

#### Request/Response Headers
- [ ] `Content-Type: application/json` included in requests with a body
- [ ] `Content-Type` header included in all responses with a body

#### JSON Response
- [ ] All fields are camelCase
- [ ] Boolean fields use `is`/`has`/`can` prefix
- [ ] Date/time in RFC 3339 format
- [ ] All time values in server response are UTC (`Z`)
- [ ] Offset input is normalized to UTC by server (not an error)
- [ ] Enum values are UPPER_SNAKE_CASE
- [ ] Null-valued fields excluded from response
- [ ] Resource has `id`, `createdAt`, `updatedAt` fields
- [ ] Server silently ignores `id`, `createdAt`, `updatedAt` if sent in request body (not rejected with error)
- [ ] PUT request body absence of a mutable field resets it to default/null (not left unchanged)
- [ ] Boolean values use native JSON true/false (not strings "true"/"false" or numbers 1/0)
- [ ] Numeric values use JSON number type (not string-wrapped numbers)
- [ ] Field names start with a lowercase letter (no UpperCamelCase field names)

#### Error Handling
- [ ] Error responses use RFC 7807/9457 Problem Details structure (`type`, `title`, `status`, `detail`)
- [ ] `Content-Type: application/problem+json` used
- [ ] `type` is a URI identifying the error type
- [ ] All validation errors returned at once (`errors` extension field)
- [ ] Internal implementation details (stack traces, DB errors) not exposed

#### Collections
- [ ] Collection response body is a top-level array (no envelope)
- [ ] Query parameters are camelCase (pageSize, pageToken, orderBy)
- [ ] Pagination metadata delivered via `Link` header (rel: next, prev, first, last)
- [ ] `X-Total-Count` header used when providing total item count
- [ ] `rel="next"` excluded from `Link` header when no next page

#### Rate Limiting
- [ ] Rate limit response includes `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` headers
- [ ] Rate limit response includes `RateLimit`, `RateLimit-Policy` headers
- [ ] 429 response includes `Retry-After` header (delta-seconds format)
- [ ] 429 response body uses Problem Details structure (`type`, `title`, `status`, `detail`)
- [ ] 429 response `Content-Type` is `application/problem+json`
- [ ] Client retry respects `Retry-After` value (immediate retry forbidden)

#### Authentication & Security
- [ ] Auth token delivered via `Authorization` header (query parameter forbidden)
- [ ] Bearer token: `Authorization: Bearer <token>` format
- [ ] 401 response includes `WWW-Authenticate` header
- [ ] 401 (authentication failure) / 403 (authorization failure) properly distinguished
- [ ] `Idempotency-Key` supported for duplicate-risk POST operations (payments, orders, etc.)
- [ ] Idempotency-Key value is a client-generated UUID v4 format

#### Long-Running Tasks
- [ ] Long-running task returns 201 Created + Location header pointing to domain resource
- [ ] Domain resource has `status` field (`PENDING`/`IN_PROGRESS`/`COMPLETED`/`FAILED`)
- [ ] No generic `/operations` endpoint — status tracked on the domain resource itself
- [ ] `FAILED` status includes RFC 7807 error structure in the resource body

#### Deprecation
- [ ] Deprecated endpoints include `Deprecation: true`, `Sunset`, and `Link` headers in responses
- [ ] Deprecation notice provided at least 6 months before sunset date

#### Filtering
- [ ] Range filters use `After`/`Before` suffix (`createdAfter`, `createdBefore`)
- [ ] Repeated same parameter treated as OR condition
- [ ] Partial match filter uses `Contains` suffix or `q` parameter

#### Custom Headers
- [ ] New custom headers do not use `X-` prefix (deprecated per RFC 6648)

### Violation Report Format

Report violations in the following format:

```
❌ [Violated Rule] Current code
   → Suggested fix

Examples:
❌ [URL snake_case] GET /user_profiles
   → GET /user-profiles or GET /users

❌ [POST 201 missing] return ResponseEntity.ok(created)
   → return ResponseEntity.created(location).body(created)

❌ [Error structure] return Map.of("message", "Not found")
   → Use Problem Details structure:
      { "type": ".../errors/resource-not-found", "title": "Resource not found",
        "status": 404, "detail": "...", "instance": "/path" }
      Content-Type: application/problem+json
```

---

## Code Examples

### Spring Boot (Java/Kotlin)

```kotlin
@RestController
@RequestMapping("/articles")
class ArticleController {

    @GetMapping
    fun getArticles(
        @RequestParam(defaultValue = "20") pageSize: Int,
        @RequestParam(required = false) pageToken: String?,
        @RequestParam(required = false) status: ArticleStatus?
    ): ResponseEntity<List<Article>> {
        val result = articleService.getArticles(pageSize, pageToken, status)
        val headers = HttpHeaders()
        buildLinkHeader(result, pageSize).let { headers.set("Link", it) }
        result.totalCount?.let { headers.set("X-Total-Count", it.toString()) }
        return ResponseEntity.ok().headers(headers).body(result.items)
    }

    @PostMapping
    fun createArticle(@RequestBody request: CreateArticleRequest): ResponseEntity<Article> {
        val article = articleService.create(request)
        val location = URI.create("/articles/${article.id}")
        return ResponseEntity.created(location).body(article)
    }

    @PatchMapping("/{id}")
    fun updateArticle(
        @PathVariable id: String,
        @RequestBody request: UpdateArticleRequest
    ): ResponseEntity<Article> {
        val article = articleService.update(id, request)
        return ResponseEntity.ok(article)
    }

    @DeleteMapping("/{id}")
    fun deleteArticle(@PathVariable id: String): ResponseEntity<Void> {
        articleService.delete(id)
        return ResponseEntity.noContent().build()
    }

    @PostMapping("/{id}:publish")
    fun publishArticle(@PathVariable id: String): ResponseEntity<Article> {
        val article = articleService.publish(id)
        return ResponseEntity.ok(article)
    }
}
```

### Error Response Structure

```kotlin
// Content-Type: application/problem+json (RFC 7807/9457)
data class ProblemDetail(
    val type: String,           // Error type URI
    val title: String,          // Short summary of error type
    val status: Int,            // HTTP status code
    val detail: String,         // Request-specific description
    val instance: String? = null,  // Request path
    val errors: List<FieldError>? = null,  // Extension: field-level errors
    val traceId: String? = null    // Extension: request trace ID
) {
    data class FieldError(
        val field: String,
        val message: String
    )
}

// Usage example
throw ApiException(
    status = HttpStatus.NOT_FOUND,
    type = "https://api.example.com/errors/resource-not-found",
    title = "Resource not found",
    detail = "The requested article could not be found.",
    instance = request.requestURI
)
```

### Collection Response Helper

```kotlin
// Internal helper — HTTP response body is List<T>, metadata delivered via headers
data class PageResult<T>(
    val items: List<T>,
    val totalCount: Long? = null,
    val nextPageToken: String? = null,
    val prevPageToken: String? = null
)

fun <T> buildLinkHeader(result: PageResult<T>, pageSize: Int): String {
    val base = "https://api.example.com/articles?pageSize=$pageSize"
    val links = mutableListOf<String>()
    result.nextPageToken?.let { links += "<$base&pageToken=$it>; rel=\"next\"" }
    result.prevPageToken?.let { links += "<$base&pageToken=$it>; rel=\"prev\"" }
    links += "<$base>; rel=\"first\""
    return links.joinToString(", ")
}
```

### Authentication Handling

```kotlin
// 401 response — token expired
return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
    .header("WWW-Authenticate", "Bearer realm=\"api\", error=\"token_expired\"")
    .contentType(MediaType.parseMediaType("application/problem+json"))
    .body(ProblemDetail(
        type = "https://api.example.com/errors/unauthorized",
        title = "Authentication required",
        status = 401,
        detail = "The access token has expired."
    ))

// 403 response — insufficient permissions
return ResponseEntity.status(HttpStatus.FORBIDDEN)
    .contentType(MediaType.parseMediaType("application/problem+json"))
    .body(ProblemDetail(
        type = "https://api.example.com/errors/forbidden",
        title = "Access denied",
        status = 403,
        detail = "You do not have permission to access this resource."
    ))
```

### Idempotency-Key Handling

```kotlin
@PostMapping("/orders")
fun createOrder(
    @RequestHeader("Idempotency-Key") idempotencyKey: String?,
    @RequestBody request: CreateOrderRequest
): ResponseEntity<Order> {
    // Check for existing result
    idempotencyKey?.let { key ->
        idempotencyStore.find(key)?.let { cached ->
            return ResponseEntity.status(cached.statusCode).body(cached.body)
        }
    }

    val order = orderService.create(request)
    val location = URI.create("/orders/${order.id}")

    // Save result (TTL: 24 hours)
    idempotencyKey?.let { key ->
        idempotencyStore.save(key, statusCode = 201, body = order, ttl = Duration.ofHours(24))
    }

    return ResponseEntity.created(location).body(order)
}
```

### Rate Limit Response Header Generation

```kotlin
fun addRateLimitHeaders(headers: HttpHeaders, limit: Int, remaining: Int, resetAt: Instant) {
    val resetUnix = resetAt.epochSecond
    val resetDelta = resetAt.epochSecond - Instant.now().epochSecond

    // Legacy headers
    headers.set("X-RateLimit-Limit", limit.toString())
    headers.set("X-RateLimit-Remaining", remaining.toString())
    headers.set("X-RateLimit-Reset", resetUnix.toString())

    // IETF standard headers
    headers.set("RateLimit", "limit=$limit, remaining=$remaining, reset=$resetDelta")
    headers.set("RateLimit-Policy", "$limit;w=3600")
}

// 429 response
fun rateLimitExceededResponse(retryAfterSeconds: Long): ResponseEntity<ProblemDetail> {
    val headers = HttpHeaders()
    headers.set("Retry-After", retryAfterSeconds.toString())
    addRateLimitHeaders(headers, limit = 100, remaining = 0, resetAt = Instant.now().plusSeconds(retryAfterSeconds))

    return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
        .headers(headers)
        .contentType(MediaType.parseMediaType("application/problem+json"))
        .body(ProblemDetail(
            type = "https://api.example.com/errors/too-many-requests",
            title = "Rate limit exceeded",
            status = 429,
            detail = "You have exceeded the allowed request limit. Please retry after ${retryAfterSeconds} seconds."
        ))
}
```

### API Version Header Handling

```
// Backward-compatible changes (allowed without version bump):
// - Adding new optional fields
// - Adding new endpoints
// - Adding new Enum values

/// Breaking changes (REQUIRE version bump):
// - Removing or renaming fields
// - Changing field types
// - Adding required fields
// - Removing Enum values
// - Changing status code semantics
```

```kotlin
// Extract version from request
@GetMapping("/articles")
fun getArticles(
    @RequestHeader("X-API-Version", required = false) apiVersion: String?
): ResponseEntity<List<Article>> {
    val version = apiVersion ?: "2024-01-20" // Default version
    val articles = articleService.getArticles(version)

    return ResponseEntity.ok()
        .header("X-API-Version", version)  // Echo applied version in response
        .body(articles)
}
```

### Deprecation Header Handling

```kotlin
// Add headers to deprecated endpoints
fun addDeprecationHeaders(headers: HttpHeaders, sunsetDate: String, successorUrl: String) {
    headers.set("Deprecation", "true")
    headers.set("Sunset", sunsetDate)  // e.g. "Sat, 01 Jan 2025 00:00:00 GMT"
    headers.set("Link", "<$successorUrl>; rel=\"successor-version\"")
}
```
