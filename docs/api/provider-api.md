# Provider API

Endpoints for providers to request access to a user's account and, once approved, manage subscriptions on that user's behalf. All endpoints require the `API-Token` header identifying the **provider** — see [Authentication → Provider Authentication](authentication.md#provider-authentication).

**Base Path**: `/api/v1/providers`

## Request Access to User

Create a pending connection request to manage a user's subscriptions. If no account exists yet for `user_id`, one is created.

**Endpoint**: `POST /api/v1/providers/{user_id}`

**Parameters**:

- `user_id` (path, required): Target user ID

**Response** (201 Created):

```json
{
  "user_id": 12345,
  "status": "pending",
  "connection_link": "https://v2hub.link/api/v1/connect/abc123"
}
```

**Notes**:

- The user must approve the request (via [Approve Provider Connection](user-self-service.md#approve-provider-connection)) before the provider can manage their subscriptions.
- `connection_link` can be shared with the user to direct them to the approval flow.

**Error Responses**:

- `409 Conflict`: Connection already exists

---

## Get Connection Status

**Endpoint**: `GET /api/v1/providers/{user_id}`

**Parameters**:

- `user_id` (path, required): Target user ID

**Response** (200 OK):

```json
{
  "user_id": 12345,
  "status": "approved"
}
```

**Error Responses**:

- `404 Not Found`: Connection not found

---

## Revoke Connection

Revoke the provider's own access to a user's account, without deleting the authorization record (it can be re-approved later).

**Endpoint**: `DELETE /api/v1/providers/{user_id}`

**Parameters**:

- `user_id` (path, required): Target user ID

**Response** (200 OK):

```json
{
  "user_id": 12345,
  "status": "revoked"
}
```

**Error Responses**:

- `404 Not Found`: Connection not found

---

## Provider-Scoped Subscription Endpoints

Once a connection is `approved`, every subscription and source endpoint documented in [Subscription Management](subscription-management.md) is available under the provider-scoped base path, operating on the target user's subscriptions instead of the provider's own:

```
/api/v1/providers/{user_id}/subs
/api/v1/providers/{user_id}/subs/{token}
/api/v1/providers/{user_id}/subs/{token}/sources
/api/v1/providers/{user_id}/subs/{token}/config
/api/v1/providers/{user_id}/subs/{token}/comments
/api/v1/providers/{user_id}/subs/{token}/refresh
```

Request bodies, response schemas, and error codes are identical to their `/api/v1/subs/...` counterparts — see [Subscription Management](subscription-management.md) for the full reference. The only difference is the additional `user_id` path segment and the requirement that the calling provider hold an `approved` connection for that `user_id`.

**Additional Error Response** (all endpoints in this group):

- `403 Forbidden`: Connection not approved for this `user_id`
