# Rate Limiting

Rate limits are enforced per-endpoint-type. Defaults, from `core/config.py`:

| Setting                   | Default   | Applies to                                                                                           |
| ------------------------- | --------- | ---------------------------------------------------------------------------------------------------- |
| `public_rps`              | 3 req/sec | Public endpoints (`/sub/{token}`)                                                                    |
| `internal_no_token_rps`   | 1 req/sec | Internal endpoints (`/api/v1/...`) called without a valid `API-Token`                                |
| `internal_with_token_rps` | 3 req/sec | Internal endpoints called with a valid `API-Token`                                                   |
| —                         | No limit  | Admin endpoints (protected instead by IP allowlist + HMAC — see [Authentication](authentication.md)) |

Rate limiting is scoped per client IP address (via the same `get_client_ip` helper used by the admin IP allowlist and ban system).

**Exceeded limit** raises `RateLimitError`, which maps to `429` with `error: "too_many_requests"` and, when available, a `retry_after` value under `details` — see [Error Handling](errors.md).

```json
{
  "error": "too_many_requests",
  "message": "Too many requests",
  "details": {
    "retry_after": 1.5
  }
}
```
