# Configuration Limits & Best Practices

## Default Configurable Limits

These are the server's default limits. Actual values may be adjusted per deployment — check with your API operator if you're unsure which limits apply to your instance.

| Setting                    | Default | Description                                    |
| --------------------------- | ------- | ------------------------------------------------ |
| `MAX_SUBS_PER_USER`          | 3       | Maximum subscriptions per user                    |
| `MAX_SOURCES_PER_SUB`        | 150     | Maximum sources per subscription                  |
| `MAX_CONFIGS_PER_SUB`        | 1000    | Maximum resolved configs per subscription         |
| `MAX_PROVIDERS_PER_USER`     | 5       | Maximum providers a user can be connected to      |
| `MAX_NESTING_DEPTH`          | 3       | Maximum recursion depth for nested subscriptions  |
| `CACHE_TTL_SECONDS`          | 900     | Cache time-to-live for external URL sources (15 min) |
| `RATE_LIMIT_PUBLIC`          | 3/sec   | Rate limit for public endpoints                    |
| `RATE_LIMIT_INTERNAL_ANON`   | 1/sec   | Rate limit for unauthenticated internal requests  |
| `RATE_LIMIT_INTERNAL_AUTH`   | 3/sec   | Rate limit for authenticated internal requests    |
| `DEFAULT_BAN_DURATION`       | 3600    | Default IP ban duration in seconds (1 hour)       |
| `AUTH_TIMESTAMP_WINDOW`      | 60      | Admin HMAC timestamp validity window in seconds   |

## Best Practices

### For API Consumers

1. **Cache the public subscription endpoint** at your CDN/reverse-proxy layer where possible; it changes only when sources are added, removed, or refreshed.
2. **Use `is_hidden` instead of deleting sources** you want to temporarily exclude — it preserves the source for later reactivation.
3. **Keep nesting shallow.** `max_depth` exists as a safety valve, not a general hierarchy mechanism — a small number of shared "base" subscriptions referenced by a few others is the intended pattern.
4. **Respect rate limit headers.** Back off using `X-RateLimit-Reset` rather than polling on a fixed interval, and treat the `retry_after` in `429` responses as authoritative.
5. **Don't poll for provider authorization changes too aggressively.** A poll interval of a few minutes is normally sufficient; the API has no push/webhook mechanism for authorization state changes.

### For Integrators Building on the API

1. **Prefer the official client libraries** ([`v2hub`](https://pypi.org/project/v2hub/), [`v2hub-admin`](https://pypi.org/project/v2hub-admin/)) over raw HTTP calls where your stack allows it — they already implement retries, a circuit breaker, and a typed exception hierarchy on top of this API. See the [client documentation](https://v2hub.dev).
2. **Treat `4xx` responses as non-retryable** except `429` (rate limit). A `400`/`404`/`409` almost always indicates a bug in the calling code or a genuinely absent resource, not a transient condition.
3. **Retry `5xx` and network failures with exponential backoff and jitter**, capped at a sane maximum delay for your application's latency budget.
4. **Never log the raw `API-Token` or admin `secret_key`**, even at debug level.

### Security

1. **Treat the admin `secret_key` as highly sensitive** — it grants full administrative access. Load it from a secrets manager or environment variable, never hardcode it, and restrict which processes/environments can read it.
2. **Always use HTTPS** in production; the HMAC scheme protects request integrity, not confidentiality in transit.
3. **Keep the admin IP whitelist tight** — scope it to known administrative infrastructure, not broad ranges.
4. **Rotate tokens periodically**, and immediately if you suspect a token or secret has been exposed (see the token/secret rotation endpoints under [Admin Endpoints](admin-endpoints.md#user-management)).
5. **Validate any user-supplied subscription name/description in your own application layer** before sending it to the API, so you can give immediate, field-specific feedback rather than relying purely on the API's `400` response.

### Error Handling

1. **Branch on the `error` code, not just the HTTP status** — several distinct conditions share the same status code (e.g. many `400`s are `validation_error`, but `invalid_url`/`invalid_config`/`nesting_too_deep` are also possible depending on context). See [Error Handling](errors.md) for the full code list.
2. **Surface `details` selectively.** `details.retry_after` on a `429` is safe and useful to show a user directly; validation `details` are typically more appropriate for logs or developer-facing messages than end-user text.
3. **Distinguish "doesn't exist" from "server is unhealthy."** A `404` is often a normal, expected outcome (e.g. a stale token) — don't treat it with the same urgency as a `500` or `503`.
