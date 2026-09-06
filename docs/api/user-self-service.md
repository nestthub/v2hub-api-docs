# User Self-Service API

Endpoints for the authenticated user to inspect their own account and manage their own provider connections. All endpoints require the `API-Token` header — see [Authentication](authentication.md).

**Base Path**: `/api/v1/me`

## Get Current User Info

**Endpoint**: `GET /api/v1/me`

**Response** (200 OK):

```json
{
  "user_id": 12345,
  "is_active": true
}
```

---

## List My Provider Connections

Get all provider connections for the current user (pending and approved; revoked connections are excluded).

**Endpoint**: `GET /api/v1/me/providers`

**Response** (200 OK):

```json
{
  "connections": [
    {
      "provider_name": "MyVPNProvider",
      "provider_url": "https://provider.com",
      "is_authorized": true,
      "status": "approved"
    },
    {
      "provider_name": "AnotherProvider",
      "provider_url": null,
      "is_authorized": false,
      "status": "pending"
    }
  ]
}
```

---

## Get Specific Provider Connection

**Endpoint**: `GET /api/v1/me/providers/{provider_name}`

**Parameters**:

- `provider_name` (path, required): Provider name

**Response** (200 OK):

```json
{
  "provider_name": "MyVPNProvider",
  "provider_url": "https://provider.com",
  "is_authorized": true,
  "status": "approved"
}
```

**Error Responses**:

- `404 Not Found`: Provider connection not found

---

## Approve Provider Connection

Approve a pending provider connection request.

**Endpoint**: `POST /api/v1/me/providers/{provider_name}/approve`

**Parameters**:

- `provider_name` (path, required): Provider name

**Response** (200 OK): Same shape as [Get Specific Provider Connection](#get-specific-provider-connection), with `status: "approved"`

**Error Responses**:

- `404 Not Found`: Provider connection not found
- `409 Conflict`: Connection is not in pending status, or the user has reached `MAX_PROVIDERS_PER_USER`

---

## Reject Provider Connection

Reject a pending provider connection request.

**Endpoint**: `POST /api/v1/me/providers/{provider_name}/reject`

**Parameters**:

- `provider_name` (path, required): Provider name

**Response** (200 OK): Same shape as [Get Specific Provider Connection](#get-specific-provider-connection)

**Notes**:

- If the user never had subscriptions from this provider, the authorization record is deleted outright.
- If the user already had subscriptions from this provider, the record is kept with `status: "revoked"` instead, so past subscriptions remain traceable.

**Error Responses**:

- `404 Not Found`: Provider connection not found
- `409 Conflict`: Connection is not in pending status

---

## Revoke Provider Connection

Revoke a previously approved provider connection.

**Endpoint**: `POST /api/v1/me/providers/{provider_name}/revoke`

**Parameters**:

- `provider_name` (path, required): Provider name

**Response** (204 No Content): Empty body

**Notes**:

- Existing subscriptions created by that provider remain available; only future delegated management is revoked.

**Error Responses**:

- `404 Not Found`: Provider connection not found
