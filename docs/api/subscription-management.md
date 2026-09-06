# Subscription Management

All subscription endpoints require authentication via `API-Token` header.

**Base Path**: `/api/v1/subs`

## Create Subscription

Create a new subscription with optional initial sources.

**Endpoint**: `POST /api/v1/subs`

**Request Body**:

```json
{
  "name": "My VPN Subscription",
  "description": "Personal VPN configs",
  "sources": [
    "vless://uuid@server:port?encryption=none#MyServer",
    "https://provider.com/subscription",
    {
      "data": "https://v2hub.link/api/v1/sub/another-token",
      "is_hidden": true,
      "max_depth": 1
    }
  ]
}
```

**Request Schema**:

| Field         | Type                          | Required | Description       | Constraints                 |
| ------------- | ----------------------------- | -------- | ----------------- | --------------------------- |
| `name`        | string                        | Yes      | Subscription name | 1-64 chars, unique per user |
| `description` | string                        | No       | Description       | Max 64 chars                |
| `sources`     | array[string \| SourceObject] | No       | Initial sources   | Max 150 items               |

Each item in `sources` can be a plain string (shorthand for `{"data": "<string>"}`) or a source object:

| Field       | Type    | Required | Description                                            | Constraints      |
| ----------- | ------- | -------- | ------------------------------------------------------ | ---------------- |
| `data`      | string  | Yes      | Config URI, URL, or internal token                     | Non-empty        |
| `is_hidden` | boolean | No       | Omit this source's configs from resolved output        | Default `false`  |
| `max_depth` | integer | No       | Max recursion depth for nested subscription references | 0-3, default `3` |

**Response** (201 Created):

```json
{
  "token": "abc123xyz456",
  "name": "My VPN Subscription",
  "description": "Personal VPN configs",
  "sources": [
    {
      "id": "hash1",
      "source_type": "config",
      "data": "vless://uuid@server:port?encryption=none#MyServer",
      "order_index": 0,
      "is_hidden": false,
      "max_depth": 3,
      "created_at": "2026-04-27T10:00:00Z",
      "updated_at": "2026-04-27T10:00:00Z"
    },
    {
      "id": "hash2",
      "source_type": "external_url",
      "data": "https://provider.com/subscription",
      "order_index": 1,
      "is_hidden": false,
      "max_depth": 3,
      "created_at": "2026-04-27T10:00:00Z",
      "updated_at": "2026-04-27T10:00:00Z"
    },
    {
      "id": "hash3",
      "source_type": "internal_token",
      "data": "https://v2hub.link/api/v1/sub/another-token",
      "order_index": 2,
      "is_hidden": true,
      "max_depth": 1,
      "created_at": "2026-04-27T10:00:00Z",
      "updated_at": "2026-04-27T10:00:00Z"
    }
  ],
  "sources_count": 15,
  "created_at": "2026-04-27T10:00:00Z",
  "updated_at": "2026-04-27T10:00:00Z"
}
```

**Error Responses**:

- `400 Bad Request`: Invalid input (validation failed)
- `409 Conflict`: Subscription name already exists
- `403 Forbidden`: Max subscriptions limit reached (default: 3)

---

## List Subscriptions

Get all subscriptions for the authenticated user.

**Endpoint**: `GET /api/v1/subs`

**Response** (200 OK):

```json
[
  {
    "token": "abc123xyz456",
    "name": "My VPN Subscription",
    "description": "Personal VPN configs",
    "sources_count": 15,
    "created_at": "2026-04-27T10:00:00Z",
    "updated_at": "2026-04-27T10:00:00Z"
  },
  {
    "token": "def789uvw012",
    "name": "Work VPN",
    "description": null,
    "sources_count": 8,
    "created_at": "2026-04-26T15:30:00Z",
    "updated_at": "2026-04-27T09:00:00Z"
  }
]
```

---

## Get Subscription Details

Retrieve detailed information about a specific subscription.

**Endpoint**: `GET /api/v1/subs/{token}`

**Parameters**:

- `token` (path, required): Subscription token

**Response** (200 OK):

```json
{
  "token": "abc123xyz456",
  "name": "My VPN Subscription",
  "description": "Personal VPN configs",
  "sources": [
    {
      "id": "hash1",
      "source_type": "config",
      "data": "vless://uuid@server:port#MyServer",
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

**Error Responses**:

- `404 Not Found`: Subscription not found
- `403 Forbidden`: Not owned by current user

---

## Update Subscription Metadata

Update subscription name and/or description.

**Endpoint**: `PATCH /api/v1/subs/{token}`

**Parameters**:

- `token` (path, required): Subscription token

**Request Body**:

```json
{
  "name": "Updated Name",
  "description": "Updated description"
}
```

**Request Schema**:

| Field         | Type   | Required | Description     | Constraints                 |
| ------------- | ------ | -------- | --------------- | --------------------------- |
| `name`        | string | No       | New name        | 1-64 chars, unique per user |
| `description` | string | No       | New description | Max 64 chars                |

**Note**: At least one field must be provided.

**Response** (200 OK): Same as [Get Subscription Details](#get-subscription-details)

**Error Responses**:

- `400 Bad Request`: No fields provided or validation failed
- `404 Not Found`: Subscription not found
- `409 Conflict`: Name already exists

---

## Delete Subscription

Permanently delete a subscription and all its sources.

**Endpoint**: `DELETE /api/v1/subs/{token}`

**Parameters**:

- `token` (path, required): Subscription token

**Response** (204 No Content): Empty body

**Error Responses**:

- `404 Not Found`: Subscription not found
- `403 Forbidden`: Not owned by current user

---

## Add Sources

Add new sources to an existing subscription.

**Endpoint**: `POST /api/v1/subs/{token}/sources`

**Parameters**:

- `token` (path, required): Subscription token

**Request Body**:

```json
{
  "sources": [
    "vless://uuid@server:port#NewServer",
    { "data": "https://new-provider.com/sub", "is_hidden": true }
  ]
}
```

**Request Schema**:

| Field     | Type                          | Required | Description    | Constraints |
| --------- | ----------------------------- | -------- | -------------- | ----------- |
| `sources` | array[string \| SourceObject] | Yes      | Sources to add | 1-150 items |

Each item can be a plain string or a source object with `data` (required), `is_hidden` (default `false`), and `max_depth` (default `3`, range 0-3) — see [Create Subscription](#create-subscription) for the object shape.

**Notes**:

- Duplicates are automatically filtered out (by `data`)
- Sources can include comments using `#` syntax
- URLs starting with `https://v2hub.link/api/v1/sub/` are treated as internal references

**Response** (200 OK): Same as [Get Subscription Details](#get-subscription-details) (with updated sources)

**Error Responses**:

- `400 Bad Request`: Invalid sources or validation failed
- `404 Not Found`: Subscription not found
- `413 Payload Too Large`: Too many sources

---

## Replace All Sources

Replace all sources in a subscription atomically.

**Endpoint**: `PUT /api/v1/subs/{token}/sources`

**Parameters**:

- `token` (path, required): Subscription token

**Request Body**:

```json
{
  "sources": [
    "vless://uuid@server:port#Server1",
    { "data": "vmess://uuid@server:port#Server2", "max_depth": 0 }
  ]
}
```

**Request Schema**:

| Field     | Type                          | Required | Description | Constraints                        |
| --------- | ----------------------------- | -------- | ----------- | ---------------------------------- |
| `sources` | array[string \| SourceObject] | Yes      | New sources | Max 150 items, empty array allowed |

Same item shape as [Add Sources](#add-sources).

**Note**: This is an atomic operation - all existing sources are deleted before new ones are added.

**Response** (200 OK): Same as [Get Subscription Details](#get-subscription-details)

**Error Responses**:

- `400 Bad Request`: Invalid sources
- `404 Not Found`: Subscription not found

---

## Remove Sources

Remove specific sources by their IDs.

**Endpoint**: `DELETE /api/v1/subs/{token}/sources`

**Parameters**:

- `token` (path, required): Subscription token

**Request Body**:

```json
{
  "source_ids": ["hash1", "hash2", "hash3"]
}
```

**Request Schema**:

| Field        | Type          | Required | Description   | Constraints |
| ------------ | ------------- | -------- | ------------- | ----------- |
| `source_ids` | array[string] | Yes      | IDs to remove | Min 1 item  |

**Response** (200 OK): Same as [Get Subscription Details](#get-subscription-details) (with sources removed)

**Error Responses**:

- `400 Bad Request`: Empty array or invalid IDs
- `404 Not Found`: Subscription or source IDs not found

---

## Update Config

Partially update settings (comment, visibility, nesting depth) for a specific config source within a subscription.

**Endpoint**: `PATCH /api/v1/subs/{token}/config`

**Parameters**:

- `token` (path, required): Subscription token

**Request Body**:

```json
{
  "config_id": "config_hash_value",
  "comment": "My custom comment",
  "is_hidden": false,
  "max_depth": 2
}
```

**Request Schema**:

| Field       | Type    | Required | Description                                         | Constraints                         |
| ----------- | ------- | -------- | --------------------------------------------------- | ----------------------------------- |
| `config_id` | string  | Yes      | Config hash                                         | Min 1 char                          |
| `comment`   | string  | No       | Comment text                                        | Max 256 chars, no `#` prefix needed |
| `is_hidden` | boolean | No       | Hide this source's configs from resolved output     | -                                   |
| `max_depth` | integer | No       | Max nesting depth for source visibility propagation | 0-3                                 |

Only the fields provided in the request are modified; omitted fields are left unchanged.

**Notes**:

- Same proxy config can have different comments in different subscriptions
- If `comment` is set to null or empty, uses domain name as default
- Comments are appended to configs using `#` when resolving

**Response** (204 No Content): Empty body

**Error Responses**:

- `404 Not Found`: Subscription or config not found
- `400 Bad Request`: Invalid config_id

---

## Update Config Comment <sup>Deprecated</sup>

> **Deprecated**: This endpoint still works and continues to be fully supported by the API and official client libraries, but it will not receive further updates and may be removed in a future major version. Use [Update Config](#update-config) instead, which supports the same comment update plus `is_hidden` and `max_depth`.

Update or set comment for a specific config within a subscription.

**Endpoint**: `PATCH /api/v1/subs/{token}/comments`

**Parameters**:

- `token` (path, required): Subscription token

**Request Body**:

```json
{
  "config_id": "config_hash_value",
  "comment": "My custom comment"
}
```

**Request Schema**:

| Field       | Type   | Required | Description  | Constraints                         |
| ----------- | ------ | -------- | ------------ | ----------------------------------- |
| `config_id` | string | Yes      | Config hash  | Min 1 char                          |
| `comment`   | string | No       | Comment text | Max 256 chars, no `#` prefix needed |

**Notes**:

- Same proxy config can have different comments in different subscriptions
- If `comment` is null or empty, uses domain name as default
- Comments are appended to configs using `#` when resolving

**Response** (204 No Content): Empty body

**Error Responses**:

- `404 Not Found`: Subscription or config not found
- `400 Bad Request`: Invalid config_id

---

## Refresh Subscription

Manually refresh all external URL sources in a subscription.

**Endpoint**: `POST /api/v1/subs/{token}/refresh`

**Parameters**:

- `token` (path, required): Subscription token

**Response** (200 OK):

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

**Response Schema**:

| Field       | Type          | Description                                        |
| ----------- | ------------- | -------------------------------------------------- |
| `refreshed` | integer       | Number of successfully refreshed sources           |
| `failed`    | integer       | Number of sources that failed to refresh           |
| `skipped`   | integer       | Number of sources skipped (CONFIG, INTERNAL_TOKEN) |
| `total`     | integer       | Total sources processed                            |
| `message`   | string        | Status message                                     |
| `errors`    | array[string] | List of error messages for failed sources          |

**Notes**:

- Only affects EXTERNAL_URL sources
- CONFIG and INTERNAL_TOKEN sources are skipped
- Updates cache for refreshed sources
- Background worker refreshes sources automatically every 15 minutes

**Error Responses**:

- `404 Not Found`: Subscription not found
