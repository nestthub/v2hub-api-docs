# Provider API

Endpoints for a provider to request access to a user's account and, once approved, manage that user's subscriptions. All endpoints require `API-Token` identifying the **provider** (see [Authentication](authentication.md)).

**Base Path**: `/api/v1/providers`

## Request Access to User

Create (or, for an existing pending/approved connection, re-fetch) a connection to `user_id`.

**Endpoint**: `POST /api/v1/providers/{user_id}`

**Behavior branches on whether the user already has an account**:

- **User exists**: a `pending` authorization record is created (if one doesn't already exist) — no HMAC involved.
- **User does not exist**: the API cannot create a pending authorization for a user it doesn't know about. Instead it returns a signed invite payload for a trusted service (e.g. a Telegram bot) to resolve — see [Authentication → Provider Connection-Invite HMAC](authentication.md#provider-connection-invite-hmac).

**Response** `200 OK` ([`ProviderConnectionCreateResponse`](../types/response-models.md#providerconnectioncreateresponse)):

```json
{
  "user_id": 12345,
  "status": "pending",
  "connection_link": "https://t.me/v2hubot?start=conn_a1b2c3d4e5f6a7b8c9d0e1f2_myprovider"
}
```

`connection_link` is only populated in the invite-payload branch (user doesn't exist yet); when the user already exists, `connection_link` is `null` and `status` directly reflects the (possibly pre-existing) authorization.

**Error Responses**:

- `409`: A connection already exists in a state this endpoint doesn't re-issue an invite for

---

## Get Connection Status

**Endpoint**: `GET /api/v1/providers/{user_id}`

**Response** `200 OK` ([`ProviderConnectionResponse`](../types/response-models.md#providerconnectionresponse)):

```json
{
  "user_id": 12345,
  "status": "approved"
}
```

**Error Responses**:

- `404`: No connection (or no user) found for `user_id`

---

## Revoke Connection

Revoke the provider's access **without** deleting the authorization record — it can later be re-approved (e.g. by the admin approve endpoint) without going through the invite flow again.

**Endpoint**: `POST /api/v1/providers/{user_id}/revoke`

**Response** `200 OK` ([`ProviderConnectionResponse`](../types/response-models.md#providerconnectionresponse)), with `status: "revoked"`.

**Error Responses**:

- `404`: Connection not found

---

## Delete Connection

Permanently delete the authorization record between this provider and `user_id`. This is a distinct, stronger operation from [Revoke Connection](#revoke-connection) above — deletion removes the record entirely rather than marking it `revoked`.

**Endpoint**: `DELETE /api/v1/providers/{user_id}`

**Response** `200 OK` ([`ProviderConnectionDeleteResponse`](../types/response-models.md#providerconnectiondeleteresponse)):

```json
{
  "detail": "Connection deleted"
}
```

**Error Responses**:

- `404`: Connection not found

---

## Provider-Scoped Subscription Endpoints

Once a connection's status is `approved`, every endpoint documented in [Subscription Management](subscription-management.md) is available under:

```
/api/v1/providers/{user_id}/subs
/api/v1/providers/{user_id}/subs/{token}
/api/v1/providers/{user_id}/subs/{token}/sources
/api/v1/providers/{user_id}/subs/{token}/config
/api/v1/providers/{user_id}/subs/{token}/comments
/api/v1/providers/{user_id}/subs/{token}/refresh
```

Request bodies, response schemas, and error codes are identical to their `/api/v1/subs/...` counterparts. Subscriptions created this way carry the acting provider's name in their `provider_name` field when later fetched by the user or the provider — see [`SubscriptionResponse`](../types/response-models.md#subscriptionresponse).
