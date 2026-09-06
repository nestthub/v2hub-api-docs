# User Self-Service API

Endpoints for the authenticated user to inspect their own account and manage connections to providers that want to manage their subscriptions. All endpoints require `API-Token` (see [Authentication](authentication.md)).

**Base Path**: `/api/v1/me`

## Get Current User Info

**Endpoint**: `GET /api/v1/me`

**Response** `200 OK` ([`MeResponse`](../types/response-models.md#meresponse)):

```json
{
  "user_id": 12345,
  "is_active": true
}
```

---

## List My Connections

Get every provider connection for the current user — pending, approved, or unspecified (a provider the caller has interacted with but that has no recorded status is returned with `status: null` rather than being excluded).

**Endpoint**: `GET /api/v1/me/connections`

**Response** `200 OK` ([`ConnectionsResponse`](../types/response-models.md#connectionsresponse)):

```json
{
  "connections": [
    {
      "provider_name": "myprovider",
      "provider_url": "https://provider.com",
      "is_authorized": true,
      "status": "approved"
    },
    {
      "provider_name": "another",
      "provider_url": null,
      "is_authorized": false,
      "status": "pending"
    }
  ]
}
```

`is_authorized` is `true` exactly when `status == "approved"`; it's a derived convenience field, not independent state.

---

## Get Specific Connection

**Endpoint**: `GET /api/v1/me/connections/{provider_name}`

**Response** `200 OK` ([`ConnectionResponse`](../types/response-models.md#connectionresponse)): same shape as one item of [List My Connections](#list-my-connections).

**Error Responses**:

- `404`: Provider with this name doesn't exist

---

## Approve Connection

Approve a pending connection request from a provider.

**Endpoint**: `POST /api/v1/me/connections/{provider_name}/approve`

**Response** `200 OK` ([`ConnectionResponse`](../types/response-models.md#connectionresponse)), with `status: "approved"`.

**Error Responses**:

- `404`: Provider or connection not found
- `409`: `invalid_authorization_status` — the connection isn't currently `pending`

---

## Reject Connection

Reject a pending connection request from a provider.

**Endpoint**: `POST /api/v1/me/connections/{provider_name}/reject`

**Response** `200 OK` ([`ConnectionResponse`](../types/response-models.md#connectionresponse)).

**Error Responses**:

- `404`: Provider or connection not found
- `409`: `invalid_authorization_status` — the connection isn't currently `pending`

---

## Revoke Connection

Revoke a previously approved connection.

**Endpoint**: `DELETE /api/v1/me/connections/{provider_name}`

**Response** `204 No Content`

Subscriptions the provider already created for this user are **not** deleted or hidden by revocation; only the provider's ability to manage them going forward is removed.

**Error Responses**:

- `404`: Provider or connection not found
