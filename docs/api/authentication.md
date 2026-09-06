# Authentication

There are three independent authentication/verification mechanisms in the API, used in different places. They are not interchangeable.

## User & Provider Authentication

Self-service (`/api/v1/subs`, `/api/v1/me`) and provider-scoped (`/api/v1/providers/{user_id}/subs`) endpoints authenticate via a single header:

```http
API-Token: {api_token}
```

The token identifies either a regular user account or a provider account — which one determines what the token can do, not how it's sent. A provider's own `api_token` additionally requires an **approved** connection for a given `user_id` before provider-scoped calls for that user succeed — see [Provider API](provider-api.md).

Token format: 43-character unpadded Base64URL string (32 random bytes).

<a id="admin-authentication"></a>

## Admin Authentication (HMAC request signing)

All `/api/v1/admin/...` endpoints require **two** independent checks, both enforced as FastAPI dependencies on every admin route:

1. **IP allowlist** — the request's source IP must appear in the server's `ADMIN_ALLOWED_IPS` setting. If that setting is empty, admin access is refused entirely (`403`), not silently opened up.
2. **HMAC-SHA256 request signature** — via two required headers:

   ```http
   X-Signature: {hex_hmac_sha256}
   X-Timestamp: {unix_timestamp_milliseconds}
   ```

**Signature payload** — the exact string that gets HMAC'd is the concatenation, with no separators, of:

```
{timestamp}{method}{path}{body}
```

- `timestamp` is the raw `X-Timestamp` header value (string, milliseconds).
- `method` is the HTTP method (`"POST"`, `"PATCH"`, etc.).
- `path` is the request path **including the query string** if one is present (`"{path}?{query}"`) — this matters for admin `GET` endpoints that take query parameters.
- `body` is the raw request body decoded as UTF-8 (empty string for bodyless requests).

```python
import hashlib
import hmac
import time

timestamp = str(int(time.time() * 1000))
method = "POST"
path = "/api/v1/admin/users"
body = '{"user_id": 12345}'

payload = f"{timestamp}{method}{path}{body}"
signature = hmac.new(
    admin_secret_key.encode(),
    payload.encode(),
    hashlib.sha256,
).hexdigest()
```

**Timestamp window**: the request is rejected with `401` if `abs(current_time_ms - X-Timestamp) > 60_000` (i.e. a fixed ±60 second window; this is not currently configurable via an environment variable).

**Missing headers**: if either `X-Signature` or `X-Timestamp` is absent, the request fails with `401` before signature verification is even attempted.

**Comparison**: signatures are compared with `hmac.compare_digest` (constant-time), not `==`.

## Provider Connection-Invite HMAC

A third, separate HMAC mechanism secures the _invite link_ generated when a provider requests access to a `user_id` that has no account yet (see [Provider API → Request Access to User](provider-api.md#request-access-to-user)). This is unrelated to the admin request-signing scheme above — different secret, different payload, different purpose.

- **Secret**: `settings.auth_hmac_secret` (a separate configuration value from the admin `secret_key`).
- **Payload signed**: `"{user_id}:{provider_hash}"`.
- **Digest**: HMAC-SHA256, **truncated to 24 hex characters** (`AUTH_HMAC_LENGTH`) — short enough to fit Telegram's 64-byte `?start=` deep-link payload limit alongside the rest of the invite string.
- **Invite format**: `conn_{hmac}_{provider_name}`, embedded in a link of the form `{connection_link_prefix}conn_{hmac}_{provider_name}` (by default a `t.me/...?start=...` Telegram deep link, configured via `CONNECTION_LINK_PREFIX`).
- **Verification**: whatever trusted service resolves the invite (e.g. a Telegram bot) calls the admin API's [Process Provider Connection Request](admin-endpoints.md#process-provider-connection-request) endpoint with the extracted `hmac`, which the server re-derives and compares with `hmac.compare_digest`.

This proves the `(user_id, provider_hash)` pair genuinely came from an invite this API generated, rather than from a caller guessing a `user_id`. Because the digest is truncated, its effective security margin is bounded by 24 hex characters (96 bits) rather than the full 256-bit SHA-256 output — still infeasible to brute-force, but worth knowing if you're reasoning about the guarantees this mechanism provides.
