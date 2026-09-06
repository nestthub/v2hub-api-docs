# Admin Endpoints

All admin endpoints require both an IP allowlist match and a valid HMAC request signature — see [Authentication → Admin Authentication](authentication.md#admin-authentication). Not rate-limited by the standard tiers (protection here is the allowlist + signature instead).

**Base Path**: `/api/v1/admin`

## User Management

### Create User

**Endpoint**: `POST /api/v1/admin/users`

**Request Body** ([`UserCreateRequest`](../types/admin-models.md#usercreaterequest)):

```json
{ "user_id": 12345 }
```

**Response** `201 Created` ([`UserCreateResponse`](../types/admin-models.md#usercreateresponse), identical shape to [`UserResponse`](../types/admin-models.md#userresponse)):

```json
{
  "user_hash": "3f2a1b9c-...",
  "user_id": 12345,
  "api_token": "generated-43-char-token",
  "is_active": true,
  "provider_hash": null
}
```

`provider_hash` is always `null` on creation — a freshly created user cannot yet own a provider.

**Error Responses**:

- `409`: `duplicate_name` — a user with this `user_id` already exists

---

### Get User

**Endpoint**: `GET /api/v1/admin/users/{user_id}`

**Response** `200 OK` ([`UserResponse`](../types/admin-models.md#userresponse)): same shape as [Create User](#create-user)'s response. `api_token` is always included here too — this endpoint returns the user's _current_ live token, not just at creation time. `provider_hash` is populated if this user owns a provider account (looked up by owner hash), otherwise `null`.

**Error Responses**:

- `404`: User not found

---

### List User's Connections

Despite the path resembling "providers owned by this user," this returns the providers **this user is connected to as an end-user** (i.e. the same relationship as [User Self-Service → List My Connections](user-self-service.md#list-my-connections), but looked up by an admin on the user's behalf) — not any provider account the user itself owns.

**Endpoint**: `GET /api/v1/admin/users/{user_id}/providers`

**Response** `200 OK` ([`ConnectionsResponse`](../types/response-models.md#connectionsresponse)): identical shape to [User Self-Service → List My Connections](user-self-service.md#list-my-connections).

**Error Responses**:

- `404`: User not found

---

### Get User's Specific Connection

**Endpoint**: `GET /api/v1/admin/users/{user_id}/providers/{provider_name}`

**Response** `200 OK` ([`ConnectionResponse`](../types/response-models.md#connectionresponse)).

**Error Responses**:

- `404`: User or provider not found

---

### Delete User

**Endpoint**: `DELETE /api/v1/admin/users/{user_id}`

**Response** `204 No Content`

**Notes**: Cascades to the user's subscriptions and connection records.

**Error Responses**:

- `404`: User not found

---

### Update User Status

**Endpoint**: `PATCH /api/v1/admin/users/{user_id}/status`

**Request Body** ([`UserStatusUpdateRequest`](../types/admin-models.md#userstatusupdaterequest)):

```json
{ "is_active": false }
```

**Response** `200 OK` ([`UserResponse`](../types/admin-models.md#userresponse))

**Error Responses**:

- `404`: User not found

---

### Refresh User Token

Note the path — `user_id` goes in the **request body**, not the URL, unlike most other single-resource admin operations on this page.

**Endpoint**: `POST /api/v1/admin/users/refresh-token`

**Request Body** ([`TokenRefreshRequest`](../types/admin-models.md#tokenrefreshrequest)):

```json
{ "user_id": 12345 }
```

**Response** `200 OK` ([`TokenRefreshResponse`](../types/admin-models.md#tokenrefreshresponse)):

```json
{ "user_id": 12345, "new_api_token": "new-43-char-token" }
```

**Error Responses**:

- `404`: User not found

---

## Provider Management

### Create Provider

**Endpoint**: `POST /api/v1/admin/providers`

**Request Body** ([`ProviderCreateRequest`](../types/admin-models.md#providercreaterequest)):

```json
{
  "owner_hash": "3f2a1b9c-...",
  "provider_name": "my-provider",
  "provider_url": "https://t.me/examplebot"
}
```

| Field           | Type          | Required | Constraints                                                                          |
| --------------- | ------------- | -------- | ------------------------------------------------------------------------------------ |
| `owner_hash`    | string        | Yes      | Must be an existing user's `user_hash` (UUID)                                        |
| `provider_name` | string        | Yes      | 4-16 chars, pattern `^[a-z0-9]+(?:-[a-z0-9]+)*$` (lowercase, digits, single hyphens) |
| `provider_url`  | string · null | No       | Max 255 chars; validated as a safe external URL (SSRF protections apply)             |

**Response** `201 Created` ([`ProviderCreateResponse`](../types/admin-models.md#providercreateresponse), identical shape to [`ProviderResponse`](../types/admin-models.md#providerresponse)):

```json
{
  "provider_hash": "9c8b7a6f-...",
  "owner_hash": "3f2a1b9c-...",
  "provider_name": "my-provider",
  "api_token": "generated-43-char-token",
  "provider_url": "https://t.me/examplebot",
  "is_active": true
}
```

**Error Responses**:

- `422`: Invalid `provider_name` pattern/length, or `provider_url` rejected by URL validation (`invalid_url`)
- `409`: Provider name already taken

---

### List All Providers

**Endpoint**: `GET /api/v1/admin/providers`

**Response** `200 OK` ([`AllProvidersResponse`](../types/admin-models.md#allprovidersresponse)):

```json
{
  "provider_hashes": {
    "my-provider": "9c8b7a6f-...",
    "another-one": "1a2b3c4d-..."
  }
}
```

Mapping of `provider_name → provider_hash` for every provider on the instance.

---

### Get Provider by Name

**Endpoint**: `GET /api/v1/admin/providers/name/{provider_name}`

**Response** `200 OK` ([`ProviderResponse`](../types/admin-models.md#providerresponse)): same shape as [Create Provider](#create-provider)'s response, including the live `api_token`.

**Error Responses**:

- `404`: Provider not found

---

### Get Provider by Owner

Looks up the owning user first — if `owner_id` doesn't correspond to an existing user, this fails as a user lookup, not (yet) as a provider lookup.

**Endpoint**: `GET /api/v1/admin/providers/owner/{owner_id}`

**Response** `200 OK` ([`ProviderResponse`](../types/admin-models.md#providerresponse))

**Error Responses**:

- `404`: User with this `owner_id` not found, or that user doesn't own a provider

---

### Get Provider by Hash

**Endpoint**: `GET /api/v1/admin/providers/{provider_hash}`

**Response** `200 OK` ([`ProviderResponse`](../types/admin-models.md#providerresponse))

**Error Responses**:

- `404`: Provider not found

---

### Delete Provider

**Endpoint**: `DELETE /api/v1/admin/providers/{provider_hash}`

**Response** `204 No Content`

**Notes**: Cascades to subscriptions this provider manages.

**Error Responses**:

- `404`: Provider not found

---

### Update Provider Status

**Endpoint**: `PATCH /api/v1/admin/providers/{provider_hash}/status`

**Request Body** ([`ProviderStatusUpdateRequest`](../types/admin-models.md#providerstatusupdaterequest)):

```json
{ "is_active": false }
```

**Response** `200 OK` ([`ProviderResponse`](../types/admin-models.md#providerresponse))

**Error Responses**:

- `404`: Provider not found

---

### Update Provider URL

**Endpoint**: `PATCH /api/v1/admin/providers/{provider_hash}/url`

**Request Body** ([`ProviderURLUpdateRequest`](../types/admin-models.md#providerurlupdaterequest)):

```json
{ "provider_url": "https://new-url.com" }
```

**Response** `200 OK` ([`ProviderResponse`](../types/admin-models.md#providerresponse))

**Error Responses**:

- `404`: Provider not found
- `422`: `invalid_url`

---

### Update Provider Name

**Endpoint**: `PATCH /api/v1/admin/providers/{provider_hash}/name`

**Request Body** ([`ProviderNameUpdateRequest`](../types/admin-models.md#providernameupdaterequest)):

```json
{ "provider_name": "new-name" }
```

Same pattern/length constraints as [Create Provider](#create-provider)'s `provider_name`.

**Response** `200 OK` ([`ProviderResponse`](../types/admin-models.md#providerresponse))

**Error Responses**:

- `404`: Provider not found
- `409`: New name already taken
- `422`: Invalid name pattern/length

---

### Refresh Provider Token

Like [Refresh User Token](#refresh-user-token), the identifier goes in the body, not the URL.

**Endpoint**: `POST /api/v1/admin/providers/refresh-token`

**Request Body** ([`ProviderTokenRefreshRequest`](../types/admin-models.md#providertokenrefreshrequest)):

```json
{ "provider_hash": "9c8b7a6f-..." }
```

**Response** `200 OK` ([`ProviderTokenRefreshResponse`](../types/admin-models.md#providertokenrefreshresponse)):

```json
{ "provider_hash": "9c8b7a6f-...", "new_api_token": "new-43-char-token" }
```

**Error Responses**:

- `404`: Provider not found

---

## Provider Authorization Management

Mounted under `/api/v1/admin/providers/auth` (nested inside the provider router above). This is the admin-side counterpart to the invite flow described in [Provider API → Request Access to User](provider-api.md#request-access-to-user) and [Authentication → Provider Connection-Invite HMAC](authentication.md#provider-connection-invite-hmac).

### Get Authorization Status

**Endpoint**: `GET /api/v1/admin/providers/auth/{provider_name}/{user_id}`

**Response** `200 OK` ([`ProviderAuthorizationInfoResponse`](../types/admin-models.md#providerauthorizationinforesponse)):

```json
{
  "provider_name": "my-provider",
  "provider_url": "https://t.me/examplebot",
  "user_id": 12345,
  "status": "pending"
}
```

`status` is `null` if no authorization record exists between this provider and user.

**Error Responses**:

- `404`: Provider not found, or user not found

---

### Process Provider Connection Request

Finalizes a connection-invite link generated by [Provider API → Request Access to User](provider-api.md#request-access-to-user), verifying the embedded HMAC. Also used for a user that already exists (no HMAC needed in that case — see below).

**Endpoint**: `POST /api/v1/admin/providers/auth`

**Request Body** ([`ProviderAuthorizationRequest`](../types/admin-models.md#providerauthorizationrequest)):

```json
{
  "user_id": 12345,
  "provider_name": "my-provider",
  "hmac": "a1b2c3d4e5f6a7b8c9d0e1f2"
}
```

| Field           | Required | Notes                                                                                                                                                                     |
| --------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `user_id`       | Yes      | Target user                                                                                                                                                               |
| `provider_name` | Yes      | Pattern `^[a-z0-9]+(?:-[a-z0-9]+)*$`                                                                                                                                      |
| `hmac`          | No       | 24-char hex digest from the invite link; required only when creating a **new** pending authorization for a user that has no prior authorization record with this provider |

**Behavior**:

1. Looks up the provider by name first — an unknown `provider_name` fails immediately, before touching any user record (so an invalid provider name can't be used to probe for user existence or create a user as a side effect).
2. Looks up (or creates, if missing) the user for `user_id`.
3. If no authorization record exists yet **and** `hmac` is provided, verifies it against `(user_id, provider_hash)` and creates a `pending` authorization on success.
4. If an authorization already exists, its current status is returned as-is — this endpoint doesn't re-verify the HMAC or change status for an existing record.

**Response** `200 OK` ([`ProviderAuthorizationInfoResponse`](../types/admin-models.md#providerauthorizationinforesponse))

**Error Responses**:

- `404`: Provider not found
- `401`: `hmac` provided but invalid (`AuthenticationError` — "Invalid or expired connection invite")

---

### Approve Provider Connection

**Endpoint**: `POST /api/v1/admin/providers/auth/approve`

**Request Body** ([`ProviderAuthorizationDecisionRequest`](../types/admin-models.md#providerauthorizationdecisionrequest)):

```json
{ "user_id": 12345, "provider_name": "my-provider" }
```

**Behavior**: if the authorization is already in a non-`pending` state, this endpoint returns the current status as-is (idempotent no-op) rather than erroring.

**Response** `200 OK` ([`ProviderAuthorizationInfoResponse`](../types/admin-models.md#providerauthorizationinforesponse)), `status: "approved"` (or the pre-existing status if it wasn't `pending`).

**Error Responses**:

- `404`: Provider not found, user not found, or no authorization record exists at all

---

### Reject Provider Connection

**Endpoint**: `POST /api/v1/admin/providers/auth/reject`

**Request Body** ([`ProviderAuthorizationDecisionRequest`](../types/admin-models.md#providerauthorizationdecisionrequest)):

```json
{ "user_id": 12345, "provider_name": "my-provider" }
```

**Behavior branches on subscription history**:

- If the user already has subscriptions created by this provider, the authorization is set to `revoked` (kept, not deleted) so those subscriptions remain traceable to a provider relationship — response `status: "revoked"`.
- If the user has no subscriptions from this provider, the authorization record is **deleted outright** — response `status: null`.

**Response** `200 OK` ([`ProviderAuthorizationInfoResponse`](../types/admin-models.md#providerauthorizationinforesponse))

**Error Responses**:

- `404`: Provider not found, user not found, or no authorization record exists

---

## IP Ban Management

### Ban IP Address

**Endpoint**: `POST /api/v1/admin/bans`

**Request Body** ([`IPBanRequest`](../types/admin-models.md#ipbanrequest)):

```json
{ "ip_address": "192.168.1.100", "duration_seconds": 3600 }
```

`duration_seconds` is optional; the server's default ban duration (3600 seconds / 1 hour) is used if omitted.

**Response** `201 Created` ([`IPBanStatusResponse`](../types/admin-models.md#ipbanstatusresponse)):

```json
{
  "ip_address": "192.168.1.100",
  "is_banned": true,
  "banned_until": "2026-04-27T11:00:00",
  "remaining_seconds": 3600
}
```

---

### Check Ban Status

**Endpoint**: `GET /api/v1/admin/bans/{ip_address}`

**Response** `200 OK` ([`IPBanStatusResponse`](../types/admin-models.md#ipbanstatusresponse)):

```json
{
  "ip_address": "192.168.1.100",
  "is_banned": true,
  "banned_until": "2026-04-27T11:00:00",
  "remaining_seconds": 1800
}
```

`banned_until` is a plain ISO-8601 string (not a JSON datetime type distinction — treat it as a string on the wire either way).

---

### Unban IP Address

Unlike most other admin single-resource deletes on this page, the IP address goes in the **request body**, not the URL path.

**Endpoint**: `DELETE /api/v1/admin/bans`

**Request Body** ([`IPUnbanRequest`](../types/admin-models.md#ipunbanrequest)):

```json
{ "ip_address": "192.168.1.100" }
```

**Response** `200 OK` ([`IPUnbanResponse`](../types/admin-models.md#ipunbanresponse)):

```json
{
  "ip_address": "192.168.1.100",
  "was_banned": true,
  "message": "IP unbanned successfully"
}
```

`was_banned: false` if the address wasn't actually banned — safe to call unconditionally.

---

### List All Bans

**Endpoint**: `GET /api/v1/admin/bans`

**Response** `200 OK` ([`IPBanListResponse`](../types/admin-models.md#ipbanlistresponse)):

```json
{
  "entries": [
    { "ip_address": "192.168.1.100", "banned_until": "2026-04-27T11:00:00" }
  ],
  "total": 1
}
```

---

## Whitelist Management

Whitelisted IPs/ranges bypass rate limiting and take precedence over bans.

### Add to Whitelist

**Endpoint**: `POST /api/v1/admin/whitelist`

**Request Body** ([`WhitelistAddRequest`](../types/admin-models.md#whitelistaddrequest)):

```json
{ "ip_address": "10.0.0.0/24", "description": "Office network" }
```

**Response** `201 Created` ([`WhitelistAddResponse`](../types/admin-models.md#whitelistaddresponse)):

```json
{
  "ip_address": "10.0.0.0/24",
  "description": "Office network",
  "message": "IP added to whitelist"
}
```

---

### List Whitelist

**Endpoint**: `GET /api/v1/admin/whitelist`

**Response** `200 OK` ([`WhitelistListResponse`](../types/admin-models.md#whitelistlistresponse)):

```json
{
  "entries": [
    {
      "ip_address": "10.0.0.0/24",
      "description": "Office network",
      "added_at": "2026-04-27T10:00:00"
    }
  ],
  "total": 1
}
```

---

### Remove from Whitelist

IP address goes in the request body, not the URL — same pattern as [Unban IP Address](#unban-ip-address).

**Endpoint**: `DELETE /api/v1/admin/whitelist`

**Request Body** ([`WhitelistRemoveRequest`](../types/admin-models.md#whitelistremoverequest)):

```json
{ "ip_address": "10.0.0.0/24" }
```

**Response** `200 OK` ([`WhitelistRemoveResponse`](../types/admin-models.md#whitelistremoveresponse)):

```json
{
  "ip_address": "10.0.0.0/24",
  "was_whitelisted": true,
  "message": "IP removed from whitelist"
}
```

---

## Usage Statistics

**Endpoint**: `GET /api/v1/admin/stats`

**Query Parameters** (all optional; defaults to all-time stats if none are given):

| Parameter    | Type                             | Notes                                                     |
| ------------ | -------------------------------- | --------------------------------------------------------- |
| `start_date` | datetime (ISO 8601)              | —                                                         |
| `end_date`   | datetime (ISO 8601)              | —                                                         |
| `period`     | `"day"` \| `"week"` \| `"month"` | Predefined period, as an alternative to an explicit range |

**Response** `200 OK` ([`StatsResponse`](../types/admin-models.md#statsresponse)):

```json
{
  "general": {
    "total_users": 1542,
    "new_users": 45,
    "new_subscriptions": 12
  }
}
```

**Error Responses**:

- `400`: `start_date` is after `end_date` — note that this endpoint's error responses do **not** follow the standard `{"error", "message", "details"}` shape used elsewhere in the API (see [Error Handling](errors.md)); the body here is FastAPI's default `{"detail": "start_date cannot be after end_date"}`, a plain string rather than a structured object.
- `500`: Statistics aggregation failed — same non-standard `{"detail": "Failed to aggregate statistics"}` shape.
