---
description: "Use when designing, implementing, or reviewing REST APIs — URL structure, HTTP methods, status codes, JSON format, error responses, and headers. Source: github.com/ppzxc/restful-api-guidelines"
user-invocable: true
---

# RESTful API Guidelines

Source: https://github.com/ppzxc/restful-api-guidelines

---

## URL Design

- **kebab-case** for path segments: `/user-profiles`, `/product-categories/123`
- **Plural nouns** for collections: `/articles` not `/article`
- **No verbs** in URLs — HTTP methods convey the action
- **No file extensions** (`.json`, `.xml`)
- **camelCase** for query parameters: `pageSize=20&sortOrder=desc`
- ASCII lowercase letters, numerals, and hyphens only in path segments
- Repeat parameter names for arrays: `?tag=tech&tag=design`
- **Max nesting depth: 2 levels** — `/{resource}/{resourceId}/{innerResource}/{innerResourceId}`

**Nesting depth rule:**

Use at most one level of sub-resource nesting. For deeper relationships, use a flat top-level route instead.

| Situation | ✅ Do | ❌ Don't |
|-----------|-------|---------|
| Order items under an order | `/orders/{orderId}/items/{itemId}` | `/users/{userId}/orders/{orderId}/items/{itemId}` |
| Reviews on an order item | `/order-items/{orderItemId}/reviews/{reviewId}` | `/users/{userId}/orders/{orderId}/items/{itemId}/reviews/{reviewId}` |
| Delivery zones under address | `/addresses/{addressId}/delivery-zones/{zoneId}` | `/users/{userId}/addresses/{addressId}/delivery-zones/{zoneId}` |

## HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|-----------|------|
| GET | Retrieve | Yes | Yes |
| POST | Create / execute | No | No |
| PUT | Full replacement | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove | Yes | No |

GET, HEAD, DELETE must not include request bodies.

## Status Codes

**2xx Success:**
- `200 OK` — standard success
- `201 Created` — creation success; include `Location` header with new resource URL
- `204 No Content` — success with no body (DELETE, etc.)

**4xx Client Error:**
- `400 Bad Request` — malformed request, validation failure
- `401 Unauthorized` — missing/expired authentication
- `403 Forbidden` — authenticated but not authorized
- `404 Not Found` — resource doesn't exist
- `409 Conflict` — duplicate or optimistic lock failure
- `422 Unprocessable Entity` — semantic validation failure
- `429 Too Many Requests` — rate limit exceeded

**5xx Server Error:**
- `500 Internal Server Error` — unexpected failure
- `503 Service Unavailable` — temporary unavailability

## JSON Format

- **camelCase** field names: `userId`, `createdAt`, `isActive`
- Never snake_case or abbreviations
- Omit null/missing fields entirely (do not send `"field": null`)
- Standard resource fields: `id`, `createdAt` (create-only), `updatedAt` (read-only)
- Servers must ignore read-only fields in request bodies

## Error Response (RFC 7807/9457 Problem Details)

```json
{
  "type": "https://api.example.com/errors/resource-not-found",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "User-friendly explanation",
  "instance": "/articles/999",
  "traceId": "abc-123-xyz"
}
```

- `Content-Type: application/problem+json`
- Include **all** validation failures at once, not incrementally
- Never expose stack traces, internal paths, or DB errors

## Headers

- `Content-Type: application/json` for bodies
- `Accept: application/json` for content negotiation
- `Location` header on 201 Created
- `X-Total-Count` for collection size
- RFC 5988 `Link` header for pagination
- Custom headers omit deprecated `X-` prefix (RFC 6648)

## CRUD Behavior

**POST (Create):** Return `201` with full resource + `Location` header.

**PUT (Full Replace):** Idempotent; omitted mutable fields reset to defaults.

**PATCH (Partial Update):** Only modify fields present in body; others unchanged.

**DELETE:** Return `204`; re-deletion policy is per-service (404 or 204).

## API Versioning

**URL path versioning is PROHIBITED** — `/v1/articles` violates REST principles (same resource must have one URL)

**Header versioning is REQUIRED:**
```
X-API-Version: 2024-01-20   (ISO 8601 date format)
```

- Requests without version header receive the latest stable version
- Responses always include the applied version
- Maintain previous versions for minimum 6 months before deprecation
- **Breaking changes** (require version bump): field deletion, type changes, new required fields, enum removal, status code semantics change
- **Compatible changes**: new optional fields, new endpoints, new enum values

## Deprecation

Deprecated APIs MUST include these response headers:

```
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/new-resource>; rel="successor-version"
```

- Deprecation notice must be given at least 6 months before the sunset date

## Key Principles

- Resource identifiers are opaque strings — clients must not parse structure
- Avoid storing sensitive data in query strings (they get logged)
- `Cache-Control` header specifies caching strategy
