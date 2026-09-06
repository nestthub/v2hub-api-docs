# Subscription Management

All endpoints on this page require the `API-Token` header (see [Authentication](authentication.md)) and are mounted at two parallel base paths that share identical request/response shapes:

- **Self-service**: `/api/v1/subs` — operates on the calling user's own subscriptions.
- **Provider-scoped**: `/api/v1/providers/{user_id}/subs` — operates on `user_id`'s subscriptions, requires the caller to hold an approved connection for that user (see [Provider API](provider-api.md)).

Everything below is written against the self-service path; substitute `/api/v1/providers/{user_id}/subs` wherever a provider needs to act on a specific user's behalf. Rate limit: `internal_with_token_rps` (default 3 req/sec) per client IP.

## Create Subscription

**Endpoint**: `POST /api/v1/subs`

**Request Body** ([`SubscriptionCreateRequest`](../types/request-models.md#subscriptioncreaterequest)):

```json
{
  "name": "My VPN Subscription",
  "description": "Personal VPN configs",
  "sources": [
    "vless://uuid@server:port?encryption=none#MyServer",
    "https://provider.com/subscription",
    {
      "data": "https://your-domain.com/sub/another-token",
      "is_hidden": true,
      "max_depth": 1
    }
  ]
}
```

| Field         | Type                          | Required | Constraints                                                  |
| ------------- | ----------------------------- | -------- | ------------------------------------------------------------ |
| `name`        | string                        | Yes      | Non-empty after stripping whitespace; max 64 chars           |
| `description` | string · null                 | No       | Max 64 chars                                                 |
| `sources`     | array[string \| SourceObject] | No       | Max 150 items (`MAX_SOURCES_PER_SUBSCRIPTION`); default `[]` |

Each `sources` item can be a plain string (shorthand for `{"data": "<string>"}`) or a [`SourceCreateRequest`](../types/request-models.md#sourcecreaterequest) object — see that page for `is_hidden`/`max_depth` details. Duplicate `data` values (after normalization) are silently deduplicated, keeping the first occurrence; if the whole list becomes empty after deduplication, the request is rejected with a validation error.

**Response** `201 Created` ([`SubscriptionResponse`](../types/response-models.md#subscriptionresponse)):

```json
{
  "token": "abc123xyz456...",
  "name": "My VPN Subscription",
  "provider_name": null,
  "description": "Personal VPN configs",
  "sources": [
    {
      "id": "a1b2c3d4...",
      "source_type": "config",
      "data": "vless://uuid@server:port?encryption=none#MyServer",
      "order_index": 0,
      "is_hidden": false,
      "max_depth": 3,
      "created_at": "2026-04-27T10:00:00Z",
      "updated_at": "2026-04-27T10:00:00Z"
    }
  ],
  "sources_count": 15,
  "created_at": "2026-04-27T10:00:00Z",
  "updated_at": "2026-04-27T10:00:00Z"
}
```

`sources_count` is the number of **fully resolved** configs (i.e. it follows nested `internal_token` sources), not merely `len(sources)` — it's computed by actually resolving the subscription, the same way [`GET /sub/{token}`](public-endpoints.md) does.

**Error Responses**:

- `422`: Validation failed (empty name, sources list empty after dedup, invalid config/URL format, etc.)
- `422`: `too_many_sources` if the resolved source count would exceed the limit; `too_many_subscriptions` if the user is already at `MAX_SUBSCRIPTIONS_PER_USER`
- `409`: `duplicate_name` — a subscription with this name already exists for the user

---

## List Subscriptions

**Endpoint**: `GET /api/v1/subs`

**Response** `200 OK` (array of [`SubscriptionListItem`](../types/response-models.md#subscriptionlistitem)):

```json
[
  {
    "token": "abc123xyz456...",
    "name": "My VPN Subscription",
    "provider_name": null,
    "description": "Personal VPN configs",
    "sources_count": 15,
    "created_at": "2026-04-27T10:00:00Z",
    "updated_at": "2026-04-27T10:00:00Z"
  }
]
```

`SubscriptionListItem` has every field of `SubscriptionResponse` except `sources` — use [Get Subscription Details](#get-subscription-details) to retrieve the resolved source list for one subscription.

---

## Get Subscription Details

**Endpoint**: `GET /api/v1/subs/{token}`

**Response** `200 OK`: [`SubscriptionResponse`](../types/response-models.md#subscriptionresponse), same shape as [Create Subscription](#create-subscription)'s response.

**Error Responses**:

- `404`: `subscription_not_found`

---

## Update Subscription Metadata

**Endpoint**: `PATCH /api/v1/subs/{token}`

**Request Body** ([`SubscriptionUpdateRequest`](../types/request-models.md#subscriptionupdaterequest)):

```json
{
  "name": "Updated Name",
  "description": "Updated description"
}
```

| Field         | Type          | Required                                                 |
| ------------- | ------------- | -------------------------------------------------------- |
| `name`        | string · null | No — max 64 chars, non-empty after stripping if provided |
| `description` | string · null | No — max 64 chars if provided                            |

Both fields are optional and independent — the endpoint does **not** require at least one to be non-null; sending `{}` is accepted and is a no-op.

**Response** `200 OK`: [`SubscriptionResponse`](../types/response-models.md#subscriptionresponse)

**Error Responses**:

- `404`: `subscription_not_found`
- `409`: `duplicate_name` if the new name collides with another of the user's subscriptions
- `422`: Validation failed

---

## Delete Subscription

**Endpoint**: `DELETE /api/v1/subs/{token}`

**Response** `204 No Content`

**Error Responses**:

- `404`: `subscription_not_found`

---

## Add Sources

**Endpoint**: `POST /api/v1/subs/{token}/sources`

**Request Body** ([`SourcesAddRequest`](../types/request-models.md#sourcesaddrequest)):

```json
{
  "sources": [
    "vless://uuid@server:port#NewServer",
    { "data": "https://new-provider.com/sub", "is_hidden": true }
  ]
}
```

| Field     | Type                          | Required | Constraints                     |
| --------- | ----------------------------- | -------- | ------------------------------- |
| `sources` | array[string \| SourceObject] | Yes      | 1-150 items after deduplication |

**Response** `200 OK`: [`SubscriptionResponse`](../types/response-models.md#subscriptionresponse) with the updated source list.

**Error Responses**:

- `404`: `subscription_not_found`
- `422`: Empty after deduplication, invalid source, `too_many_sources`

---

## Replace All Sources

**Endpoint**: `PUT /api/v1/subs/{token}/sources`

**Request Body** ([`SourcesReplaceRequest`](../types/request-models.md#sourcesreplacerequest)):

```json
{
  "sources": ["vless://uuid@server:port#Server1"]
}
```

Unlike [Add Sources](#add-sources), an **empty array is accepted** here (`sources: []`), clearing all sources from the subscription.

**Response** `200 OK`: [`SubscriptionResponse`](../types/response-models.md#subscriptionresponse)

**Error Responses**:

- `404`: `subscription_not_found`
- `422`: Invalid source, `too_many_sources`

---

## Remove Sources

**Endpoint**: `DELETE /api/v1/subs/{token}/sources`

**Request Body** ([`SourcesRemoveRequest`](../types/request-models.md#sourcesremoverequest)):

```json
{
  "source_ids": ["a1b2c3d4e5f6...", "b2c3d4e5f6a1..."]
}
```

| Field        | Type          | Required | Constraints                                     |
| ------------ | ------------- | -------- | ----------------------------------------------- |
| `source_ids` | array[string] | Yes      | At least 1 item; each a 32-char hex source hash |

**Response** `200 OK`: [`SubscriptionResponse`](../types/response-models.md#subscriptionresponse)

**Error Responses**:

- `404`: `subscription_not_found`, `source_not_found`
- `422`: Empty list

---

## Update Config

Partially updates a single source's comment, hidden state, and/or nesting depth. Only fields explicitly present in the request body are changed; omitted (or `null`) fields are left untouched.

**Endpoint**: `PATCH /api/v1/subs/{token}/config`

**Request Body** ([`SourceUpdateRequest`](../types/request-models.md#sourceupdaterequest)):

```json
{
  "config_id": "a1b2c3d4e5f6...",
  "comment": "My custom comment",
  "is_hidden": false,
  "max_depth": 2
}
```

| Field       | Type           | Required | Constraints                                                                            |
| ----------- | -------------- | -------- | -------------------------------------------------------------------------------------- |
| `config_id` | string         | Yes      | 32-char hex source hash. **Aliased**: `config_hash` is also accepted as the field name |
| `comment`   | string · null  | No       | Max 255 chars                                                                          |
| `is_hidden` | boolean · null | No       | —                                                                                      |
| `max_depth` | integer · null | No       | 0-3                                                                                    |

Unlike the deprecated [Update Config Comment](#update-config-comment-deprecated), passing `comment: null` here leaves the existing comment **unchanged** — it does not clear it or fall back to a default.

**Response** `204 No Content`

**Error Responses**:

- `404`: `subscription_not_found`, `source_not_found`
- `422`: Invalid `config_id` or field values

---

## Update Config Comment <sup>Deprecated</sup>

> **Deprecated**: Superseded by [Update Config](#update-config), which supports the same comment update plus `is_hidden` and `max_depth`. Still fully functional.

**Endpoint**: `PATCH /api/v1/subs/{token}/comments`

**Request Body** ([`CommentUpdateRequest`](../types/request-models.md#commentupdaterequest-deprecated)):

```json
{
  "config_id": "a1b2c3d4e5f6...",
  "comment": "My custom comment"
}
```

| Field       | Type          | Required | Constraints                                    |
| ----------- | ------------- | -------- | ---------------------------------------------- |
| `config_id` | string        | Yes      | 32-char hex source hash (alias: `config_hash`) |
| `comment`   | string · null | No       | Max 255 chars                                  |

**Behavioral difference from [Update Config](#update-config)**: if `comment` is omitted or `null`, this endpoint sets the comment to the server's configured domain name (`settings.domain`) rather than leaving it unchanged.

**Response** `204 No Content`

**Error Responses**: Same as [Update Config](#update-config).

---

## Refresh Subscription

Force an immediate refresh of all `external_url` sources in a subscription, bypassing the normal lazy-refresh cooldown.

**Endpoint**: `POST /api/v1/subs/{token}/refresh`

**Response** `200 OK` ([`RefreshSubscriptionResponse`](../types/response-models.md#refreshsubscriptionresponse)):

```json
{
  "refreshed": 5,
  "failed": 1,
  "skipped": 2,
  "total": 8,
  "message": "Refresh completed with some failures",
  "errors": ["https://dead-provider.com/sub: Connection timeout"]
}
```

`total` is a **computed** field (`refreshed + failed + skipped`), not independently settable or stored. `config`/`internal_token` sources are always counted under `skipped`, since only `external_url` sources are ever refreshed by this endpoint.

**Error Responses**:

- `404`: `subscription_not_found`
