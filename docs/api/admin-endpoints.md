# Admin Endpoints

All admin endpoints require [Admin Authentication](authentication.md#admin-authentication) (IP whitelist + HMAC signature).

**Base Path**: `/api/v1/admin`

## User Management

### Create User

**Endpoint**: `POST /api/v1/admin/users`

**Request Body**:

```json
{
  "user_id": 12345
}
```

**Response** (201 Created):

```json
{
  "user_id": 12345,
  "api_token": "generated_token_here",
  "is_active": true,
  "created_at": "2026-04-27T10:00:00Z"
}
```

**Error Responses**:

- `409 Conflict`: User already exists

---

### Get User

**Endpoint**: `GET /api/v1/admin/users/{user_id}`

**Response** (200 OK):

```json
{
  "user_id": 12345,
  "is_active": true,
  "created_at": "2026-04-27T10:00:00Z"
}
```

**Error Responses**:

- `404 Not Found`: User not found

---

### Update User Status

**Endpoint**: `PATCH /api/v1/admin/users/{user_id}`

**Request Body**:

```json
{
  "is_active": false
}
```

**Response** (200 OK): Same as [Get User](#get-user)

---

### Delete User

**Endpoint**: `DELETE /api/v1/admin/users/{user_id}`

**Response** (204 No Content): Empty body

**Notes**: Cascades — deletes all subscriptions, provider connections, and related data owned by the user.

---

### Refresh User Token

Generate a new API token for a user, invalidating the old one.

**Endpoint**: `POST /api/v1/admin/users/{user_id}/refresh-token`

**Response** (200 OK):

```json
{
  "user_id": 12345,
  "new_api_token": "new_generated_token"
}
```

---

## Provider Management

### Create Provider

**Endpoint**: `POST /api/v1/admin/providers`

**Request Body**:

```json
{
  "owner_hash": "user_hash_here",
  "provider_name": "MyProvider",
  "provider_url": "https://myprovider.com"
}
```

**Response** (201 Created):

```json
{
  "provider_hash": "generated_hash",
  "provider_name": "MyProvider",
  "provider_url": "https://myprovider.com",
  "api_token": "provider_api_token",
  "is_active": true,
  "created_at": "2026-04-27T10:00:00Z"
}
```

---

### List All Providers

**Endpoint**: `GET /api/v1/admin/providers`

**Response** (200 OK):

```json
{
  "provider_hashes": {
    "MyProvider": "hash1",
    "AnotherProvider": "hash2"
  }
}
```

---

### Get Provider by Name

**Endpoint**: `GET /api/v1/admin/providers/by-name/{provider_name}`

**Response** (200 OK): Same as [Create Provider](#create-provider) response

---

### Get Provider by Owner

**Endpoint**: `GET /api/v1/admin/providers/by-owner/{owner_id}`

**Response** (200 OK): Same as [Create Provider](#create-provider) response

---

### Update Provider Status

**Endpoint**: `PATCH /api/v1/admin/providers/{provider_hash}`

**Request Body**:

```json
{
  "is_active": false
}
```

**Response** (200 OK): Same as [Create Provider](#create-provider) response

---

### Delete Provider

**Endpoint**: `DELETE /api/v1/admin/providers/{provider_hash}`

**Response** (204 No Content): Empty body

**Notes**: Cascades — deletes all subscriptions managed by this provider.

---

### Refresh Provider Token

**Endpoint**: `POST /api/v1/admin/providers/{provider_hash}/refresh-token`

**Response** (200 OK):

```json
{
  "provider_hash": "hash1",
  "new_api_token": "new_provider_token"
}
```

---

## Provider Authorization Management

### Process Authorization Request

Manually process a provider authorization request (typically called by the provider's system after receiving an HMAC-signed invite).

**Endpoint**: `POST /api/v1/admin/providers/{provider_name}/authorize/{user_id}`

**Request Body**:

```json
{
  "hmac": "signature_from_provider"
}
```

**Response** (200 OK):

```json
{
  "user_id": 12345,
  "status": "pending"
}
```

---

### Get User's Providers

**Endpoint**: `GET /api/v1/admin/users/{user_id}/providers`

**Response** (200 OK): Same shape as [List My Provider Connections](user-self-service.md#list-my-provider-connections)

---

### Get Specific User-Provider Connection

**Endpoint**: `GET /api/v1/admin/users/{user_id}/providers/{provider_name}`

**Response** (200 OK): Same shape as [Get Specific Provider Connection](user-self-service.md#get-specific-provider-connection)

---

### Approve Provider Authorization

**Endpoint**: `POST /api/v1/admin/users/{user_id}/providers/{provider_name}/approve`

**Response** (200 OK): Same shape as [Get Specific Provider Connection](user-self-service.md#get-specific-provider-connection), with `status: "approved"`

**Error Responses**:

- `409 Conflict`: Not in pending status, or user has reached `MAX_PROVIDERS_PER_USER`

---

### Reject Provider Authorization

**Endpoint**: `POST /api/v1/admin/users/{user_id}/providers/{provider_name}/reject`

**Response** (200 OK): Same shape as [Get Specific Provider Connection](user-self-service.md#get-specific-provider-connection)

**Notes**: Same delete-vs-revoke behavior as [Reject Provider Connection](user-self-service.md#reject-provider-connection).

**Error Responses**:

- `409 Conflict`: Not in pending status

---

## IP Ban Management

### Ban IP Address

**Endpoint**: `POST /api/v1/admin/ban`

**Request Body**:

```json
{
  "ip_address": "192.168.1.100",
  "duration_seconds": 3600
}
```

`duration_seconds` is optional; if omitted, the server's default ban duration is used.

**Response** (201 Created):

```json
{
  "ip_address": "192.168.1.100",
  "banned_until": "2026-04-27T11:00:00Z",
  "remaining_seconds": 3600
}
```

---

### Check Ban Status

**Endpoint**: `GET /api/v1/admin/ban/{ip_address}`

**Response** (200 OK):

```json
{
  "ip_address": "192.168.1.100",
  "is_banned": true,
  "banned_until": "2026-04-27T11:00:00Z",
  "remaining_seconds": 1800
}
```

---

### Unban IP Address

**Endpoint**: `DELETE /api/v1/admin/ban/{ip_address}`

**Response** (200 OK):

```json
{
  "ip_address": "192.168.1.100",
  "was_banned": true
}
```

`was_banned` is `false` if the address wasn't actually banned — safe to call unconditionally.

---

### List All Bans

**Endpoint**: `GET /api/v1/admin/bans`

**Response** (200 OK):

```json
{
  "total": 2,
  "entries": [
    {
      "ip_address": "192.168.1.100",
      "banned_until": "2026-04-27T11:00:00Z"
    },
    {
      "ip_address": "203.0.113.5",
      "banned_until": "2026-04-27T12:00:00Z"
    }
  ]
}
```

---

## Whitelist Management

Whitelisted ranges take precedence over bans and are exempt from IP-based rate limiting.

### Add to Whitelist

**Endpoint**: `POST /api/v1/admin/whitelist`

**Request Body**:

```json
{
  "ip_address": "10.0.0.0/24",
  "description": "Office network"
}
```

**Response** (201 Created):

```json
{
  "message": "IP range added to whitelist"
}
```

---

### List Whitelist

**Endpoint**: `GET /api/v1/admin/whitelist`

**Response** (200 OK):

```json
{
  "entries": [
    {
      "ip_address": "10.0.0.0/24",
      "description": "Office network",
      "added_at": "2026-04-27T10:00:00Z"
    }
  ]
}
```

---

### Remove from Whitelist

**Endpoint**: `DELETE /api/v1/admin/whitelist/{ip_address}`

**Response** (200 OK):

```json
{
  "ip_address": "10.0.0.0/24",
  "was_whitelisted": true
}
```

---

## Usage Statistics

**Endpoint**: `GET /api/v1/admin/stats`

**Query Parameters**:

| Parameter    | Type   | Required | Description                                        |
| ------------ | ------ | -------- | --------------------------------------------------- |
| `period`     | string | No       | Predefined period: `day`, `week`, or `month`         |
| `start_date` | string | No       | ISO 8601 start of an explicit range                  |
| `end_date`   | string | No       | ISO 8601 end of an explicit range                    |

Omit all parameters to use the API's default range. `period` and an explicit `start_date`/`end_date` range are mutually exclusive ways of specifying the same thing.

**Response** (200 OK):

```json
{
  "period": {
    "start": "2026-04-20T00:00:00Z",
    "end": "2026-04-27T00:00:00Z"
  },
  "total_requests": 154302,
  "total_subscriptions": 421,
  "total_users": 98,
  "total_providers": 6
}
```

The exact set of fields returned may be extended over time; treat unknown fields as informational.
