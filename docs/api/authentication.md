# Authentication

## User Authentication

Most endpoints require authentication via the `API-Token` header:

```http
API-Token: {provider_api_token}
```

**Example**:

```http
API-Token: a1b2c3d4e5f6g7h8i9j0
```

## Provider Authentication

Provider-scoped endpoints (`/api/v1/providers/...`) also authenticate via the `API-Token` header, but the token identifies a **provider** account rather than a user account. Unlike the user token, it is a single opaque string — it is not prefixed with the provider's hash or any other identifier:

```http
API-Token: {provider_api_token}
```

**Example**:

```http
API-Token: a1B2c3D4e5F6g7H8i9J0
```

The provider (and its `provider_hash`) is looked up from this token alone. A provider must additionally hold an **approved** authorization for the target `user_id` before it can manage that user's subscriptions (`/api/v1/providers/{user_id}/subs/...`). A connection request starts out `pending` via `POST /api/v1/providers/{user_id}` and needs a separate confirmation step to become `approved`; it can be revoked at any time thereafter — see [Provider API](provider-api.md).

## Admin Authentication

Admin endpoints require two security layers:

1. **IP Whitelist**: Request must originate from an allowed IP address
2. **HMAC Signature**: Request must include valid signature headers

**Required Headers**:

```http
X-Signature: {hmac_signature}
X-Timestamp: {unix_timestamp_ms}
```

**Signature Calculation**:

```python
import hmac
import hashlib
import time

timestamp = str(int(time.time() * 1000))
method = "POST"
path = "/api/v1/admin/users"
body = '{"user_id": 12345}'

payload = f"{timestamp}{method}{path}{body}"
signature = hmac.new(
    admin_secret_key.encode(),
    payload.encode(),
    hashlib.sha256
).hexdigest()
```

**Timestamp Validation**:

- Timestamps are valid within ±1 minute window
- Prevents replay attacks
