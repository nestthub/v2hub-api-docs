# Response Models

Response models for subscription, source, provider-connection, and user endpoints. For admin-only response models, see [Admin Models](admin-models.md).

## SourceOut

An individual, resolved source within a subscription's `sources` list.

```python
class SourceOut(BaseModelConfig):
    id: str                      # 32 hex chars
    source_type: SourceType
    data: str
    order_index: int             # >= 0
    is_hidden: bool = False
    max_depth: int = 3           # 0-3, default settings.max_nesting_depth
    created_at: datetime
    updated_at: datetime
```

| Field         | Type                                     | Description                                                                       |
| ------------- | ---------------------------------------- | --------------------------------------------------------------------------------- |
| `id`          | string                                   | Source hash, 32 hex chars                                                         |
| `source_type` | [SourceType](enumerations.md#sourcetype) | `config`, `external_url`, or `internal_token`                                     |
| `data`        | string                                   | The source's content — see below for exactly what this contains per `source_type` |
| `order_index` | integer                                  | Display order (`>= 0`)                                                            |
| `is_hidden`   | boolean                                  | Whether excluded from resolved public output                                      |
| `max_depth`   | integer                                  | Max further nesting depth to follow for this source (0-3)                         |
| `created_at`  | datetime                                 | Creation timestamp                                                                |
| `updated_at`  | datetime                                 | Last update timestamp                                                             |

**What `data` contains, by `source_type`** (per the server's own `convert_sources_to_out` logic):

- `config`: the raw proxy config string, with `#{comment}` appended if a comment is set on this source within this subscription (comments are per-subscription, so the same underlying config can carry different comments in different subscriptions).
- `external_url`: the external subscription URL, unchanged.
- `internal_token`: reconstructed as `https://{domain}/sub/{token}` (i.e. you get back a full public-subscription URL, not a bare token) — `{domain}` is the server's configured `DOMAIN` setting.

## SubscriptionResponse

Full subscription detail, returned by create/get/update and every source-mutation endpoint.

```python
class SubscriptionResponse(SubscriptionBase):
    sources: list[SourceOut] = []
```

Where `SubscriptionBase` provides:

| Field           | Type                           | Description                                                                                        |
| --------------- | ------------------------------ | -------------------------------------------------------------------------------------------------- |
| `token`         | string                         | 43-char Base64URL subscription token                                                               |
| `name`          | string                         | Max 64 chars                                                                                       |
| `provider_name` | string · null                  | Name of the provider that manages this subscription, if any                                        |
| `description`   | string · null                  | Max 64 chars                                                                                       |
| `sources_count` | integer                        | Count of fully **resolved** configs (follows nested `internal_token` sources) — not `len(sources)` |
| `created_at`    | datetime                       | —                                                                                                  |
| `updated_at`    | datetime                       | —                                                                                                  |
| `sources`       | array[[SourceOut](#sourceout)] | _(added by `SubscriptionResponse` itself, not present on `SubscriptionListItem`)_                  |

## SubscriptionListItem

Identical to `SubscriptionBase` above (i.e. every `SubscriptionResponse` field **except** `sources`) — returned by the list-subscriptions endpoint. Fetch a subscription individually via [Get Subscription Details](../api/subscription-management.md#get-subscription-details) to see its resolved source list.

## RefreshSubscriptionResponse

```python
class RefreshSubscriptionResponse(BaseModelConfig):
    refreshed: int = 0
    failed: int = 0
    skipped: int = 0
    message: str | None = None
    errors: list[str] = []

    @computed_field
    def total(self) -> int:
        return self.refreshed + self.failed + self.skipped
```

| Field       | Type          | Description                                                                                                                    |
| ----------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `refreshed` | integer       | Sources successfully refreshed                                                                                                 |
| `failed`    | integer       | Sources that failed to refresh                                                                                                 |
| `skipped`   | integer       | Sources skipped (always includes every `config`/`internal_token` source, since only `external_url` sources are ever refreshed) |
| `total`     | integer       | **Computed**, not stored — always `refreshed + failed + skipped`                                                               |
| `message`   | string · null | Optional human-readable summary                                                                                                |
| `errors`    | array[string] | Per-source error details                                                                                                       |

## MeResponse

```python
class MeResponse(BaseModelConfig):
    user_id: int
    is_active: bool
```

Deliberately minimal — per the source's own docstring, this exposes only information relevant to the self-service API and does not include internal identifiers like `user_hash`.

## ConnectionResponse

A single provider connection, from the connected user's point of view.

```python
class ConnectionResponse(BaseModelConfig):
    provider_name: str          # 4-16 chars
    provider_url: str | None    # max 255 chars
    is_authorized: bool
    status: ProviderAuthorizationStatus | None = None
```

`is_authorized` is `true` exactly when `status == ProviderAuthorizationStatus.APPROVED` — it's derived at response-construction time, not an independently stored field.

## ConnectionsResponse

```python
class ConnectionsResponse(BaseModelConfig):
    connections: list[ConnectionResponse]
```

Returned by [User Self-Service → List My Connections](../api/user-self-service.md#list-my-connections) and the admin equivalent, [Admin → List User's Connections](../api/admin-endpoints.md#list-users-connections).

## ProviderConnectionResponse

The provider-side view of a connection (as opposed to `ConnectionResponse`, which is the user-side view of the same relationship).

```python
class ProviderConnectionResponse(BaseModelConfig):
    user_id: int
    status: ProviderAuthorizationStatus
```

## ProviderConnectionCreateResponse

Extends [`ProviderConnectionResponse`](#providerconnectionresponse) with an optional connection link.

```python
class ProviderConnectionCreateResponse(ProviderConnectionResponse):
    connection_link: str | None
```

`connection_link` is only non-null when [Request Access to User](../api/provider-api.md#request-access-to-user) had to fall back to the HMAC invite-link flow. See [Authentication → Provider Connection-Invite HMAC](../api/authentication.md#provider-connection-invite-hmac).

## ProviderConnectionDeleteResponse

Response returned after deleting a provider connection.

```python
class ProviderConnectionDeleteResponse(BaseModelConfig):
    detail: str
```

## ProviderInfoResponse

```python
class ProviderInfoResponse(BaseModelConfig):
    provider_name: str    # 4-16 chars
    provider_url: str     # max 255 chars, required (not optional here)
```

A smaller, provider-facing shape distinct from the admin-only [`ProviderResponse`](admin-models.md#providerresponse) — notably, `provider_url` is required (non-optional) on this model, unlike most other places `provider_url` appears.
