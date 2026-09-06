# Error Handling

## Error Response Format

Nearly all errors across the API follow one consistent JSON shape, returned directly as the response body (not wrapped in a `detail` key, despite being built from FastAPI's `HTTPException` internally — a custom exception handler unwraps it before sending):

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {}
}
```

| Field     | Type   | Description                                                                             |
| --------- | ------ | --------------------------------------------------------------------------------------- |
| `error`   | string | Machine-readable error code (see table below)                                           |
| `message` | string | Human-readable description                                                              |
| `details` | object | Additional context — shape varies by error type; empty object if there's nothing to add |

!!! warning "One documented exception"
[Usage Statistics](admin-endpoints.md#usage-statistics) (`GET /api/v1/admin/stats`) does **not** follow this shape — its error responses are FastAPI's raw default, `{"detail": "<plain string message>"}`. This is the only endpoint on this site confirmed to differ; if you write shared error-handling code against `error`/`message`/`details`, special-case this endpoint or fall back gracefully when those keys are absent.

## Status Code → Error Code Mapping

This is the exact mapping implemented in `to_http_exception()` in the server's source — not a general REST convention, and in a few places it differs from what you might expect from the HTTP status code alone:

| Status | Error Code (`error` field)                 | Raised when                                                                                                         |
| ------ | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| 401    | `invalid_token`                            | `API-Token` missing/invalid; admin signature missing/invalid/expired                                                |
| 403    | `forbidden`                                | Authorization error (access denied for a reason other than "not authenticated")                                     |
| 404    | `not_found`                                | Generic not-found                                                                                                   |
| 404    | `subscription_not_found`                   | Subscription token doesn't exist                                                                                    |
| 404    | `source_not_found`                         | Source ID doesn't exist within a subscription                                                                       |
| 409    | `duplicate_name`                           | Subscription/provider name already exists                                                                           |
| 409    | `invalid_authorization_status`             | A provider-authorization operation requires the connection to be in a specific status (e.g. `pending`) and it isn't |
| 422    | `invalid_config`                           | Generic validation failure, and specifically an invalid proxy config                                                |
| 422    | `invalid_url`                              | A URL failed validation (malformed, or rejected by SSRF protections)                                                |
| 422    | `circular_reference`                       | A subscription reference chain would create a cycle                                                                 |
| 422    | `nesting_too_deep`                         | Nested subscription resolution exceeded the configured max depth                                                    |
| 422    | `too_many_configs`                         | Resolved config count for a subscription would exceed the limit                                                     |
| 422    | `too_many_sources`                         | Source count for a subscription would exceed the limit                                                              |
| 422    | `too_many_subscriptions`                   | User's subscription count would exceed the limit                                                                    |
| 422    | `too_many_providers`                       | User's approved-provider count would exceed the limit                                                               |
| 422    | (Pydantic request validation)              | `validation_error` — malformed request body, wrong types, out-of-range fields, etc.                                 |
| 400    | `fetch_error`                              | Fetching an `external_url` source failed                                                                            |
| 429    | `too_many_requests`                        | Rate limit exceeded (see [Rate Limiting](rate-limiting.md))                                                         |
| 500    | `cache_error`                              | A cache read/write failed server-side                                                                               |
| 500    | `database_error`                           | An unhandled SQLAlchemy/database error propagated up                                                                |
| 500    | `internal_error` / `internal_server_error` | Any other unhandled exception                                                                                       |

**Important**: most "business rule" violations that might intuitively feel like `400 Bad Request` or `403 Forbidden` — circular references, nesting limits, all the "too many X" quota errors, invalid configs/URLs — are actually returned as **`422 Unprocessable Content`** in this API, because they're implemented as subclasses that get validation-style status codes. Don't branch client-side logic on `400` vs `422` as if they mean "structurally malformed" vs "semantically invalid" — check the `error` field instead.

## Example Error Responses

**Validation error** (Pydantic-level, e.g. sending `name: ""`):

```json
{
  "error": "validation_error",
  "message": "Request validation failed",
  "details": [
    {
      "type": "string_too_short",
      "loc": ["body", "name"],
      "msg": "String should have at least 1 character"
    }
  ]
}
```

Note `details` here is a **list** (Pydantic's own `.errors()` output), not an object — this is the one case where `details`'s shape differs from a plain dict, since it comes from a different code path (caught `PydanticValidationError`, not one of the API's own exception classes).

**Not found**:

```json
{
  "error": "subscription_not_found",
  "message": "Subscription 'abc123' not found",
  "details": {
    "resource": "Subscription",
    "identifier": "abc123"
  }
}
```

**Rate limit**:

```json
{
  "error": "too_many_requests",
  "message": "Too many requests",
  "details": {
    "retry_after": 1.5
  }
}
```

**Circular reference** (`422`, not `400`):

```json
{
  "error": "circular_reference",
  "message": "Circular reference: abc123 → def456 → abc123",
  "details": {
    "chain": ["abc123", "def456", "abc123"]
  }
}
```

**Too many sources** (`422`):

```json
{
  "error": "too_many_sources",
  "message": "Source count (151) exceeds maximum allowed (150)",
  "details": {
    "count": 151,
    "max_count": 150
  }
}
```

**Duplicate name** (`409`):

```json
{
  "error": "duplicate_name",
  "message": "Subscription name 'My VPN' already exists",
  "details": {
    "conflicting_field": "name"
  }
}
```
