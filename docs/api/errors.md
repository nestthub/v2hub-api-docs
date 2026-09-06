# Error Handling

All errors follow a consistent JSON structure.

## Error Response Format

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {
    "field": "additional_context"
  }
}
```

| Field     | Type          | Description                                              |
| --------- | ------------- | --------------------------------------------------------- |
| `error`   | string        | Machine-readable error code                                |
| `message` | string        | Human-readable description                                 |
| `details` | object · null | Additional context, when available (e.g. `retry_after`)   |

## Common Error Codes

| HTTP Status | Error Code             | Description                        |
| ----------- | ----------------------- | ----------------------------------- |
| 400         | `validation_error`       | Invalid request data                |
| 401         | `authentication_error`   | Invalid or missing credentials      |
| 403         | `authorization_error`    | Insufficient permissions            |
| 404         | `not_found`              | Resource not found                  |
| 409         | `conflict`               | Resource conflict (e.g. duplicate) |
| 413         | `payload_too_large`      | Request body exceeds limits          |
| 429         | `too_many_requests`      | Rate limit exceeded                  |
| 500         | `internal_server_error`  | Unexpected server error              |
| 503         | `service_unavailable`    | Service temporarily unavailable      |

## Specific Error Types

### Subscription Errors

| Error Code               | Description                             |
| -------------------------- | ----------------------------------------- |
| `subscription_not_found`    | Subscription token doesn't exist          |
| `duplicate_name`            | Subscription name already exists          |
| `max_subscriptions_reached` | User has reached subscription limit       |

### Source Errors

| Error Code            | Description                             |
| ------------------------ | ----------------------------------------- |
| `source_not_found`       | Source ID(s) not found                    |
| `circular_reference`     | Circular reference detected between sources |
| `nesting_too_deep`       | Max nesting depth exceeded                |
| `invalid_url`            | URL rejected (e.g. SSRF protection)       |
| `invalid_config`         | Config format is invalid                  |
| `too_many_sources`       | Per-subscription source limit reached     |
| `too_many_configs`       | Per-subscription resolved-config limit reached |

### Provider Errors

| Error Code                       | Description                                    |
| ----------------------------------- | ------------------------------------------------- |
| `provider_not_found`                | Provider doesn't exist                             |
| `connection_not_found`              | Provider connection doesn't exist                  |
| `invalid_authorization_status`      | Connection not in the expected status for this operation |
| `too_many_providers`                | User has reached `MAX_PROVIDERS_PER_USER`          |

### Admin Errors

| Error Code            | Description                             |
| ------------------------ | ----------------------------------------- |
| `invalid_signature`      | HMAC signature verification failed        |
| `timestamp_expired`      | Request timestamp outside valid window    |
| `ip_not_whitelisted`     | Request IP not in admin whitelist         |

## Example Error Responses

**Validation Error**:

```json
{
  "error": "validation_error",
  "message": "Invalid request data",
  "details": {
    "name": "Field required",
    "sources": "Ensure this value has at most 150 items"
  }
}
```

**Not Found**:

```json
{
  "error": "subscription_not_found",
  "message": "Subscription with token 'abc123' not found"
}
```

**Rate Limit**:

```json
{
  "error": "too_many_requests",
  "message": "Rate limit exceeded",
  "details": {
    "retry_after": 1.5
  }
}
```

**Circular Reference**:

```json
{
  "error": "circular_reference",
  "message": "Circular reference detected: token 'abc' references itself through 'def'"
}
```
