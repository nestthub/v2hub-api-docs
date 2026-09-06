# Response Models

Response models for subscription, source, provider, and user endpoints. For admin-only response models, see [Admin Models](admin-models.md).

## Source

An individual, resolved source within a subscription's `sources` list.

```python
class Source(BaseModel):
    id: str
    source_type: SourceType
    data: str
    order_index: int
    is_hidden: bool
    max_depth: int
    created_at: datetime
    updated_at: datetime
```

| Field         | Type                   | Description                                              |
| ------------- | ----------------------- | ----------------------------------------------------------- |
| `id`          | string                   | Unique source identifier (hash)                              |
| `source_type` | [SourceType](enumerations.md#sourcetype) | `config`, `external_url`, or `internal_token`            |
| `data`        | string                   | The source data (config URI, URL, or token)                 |
| `order_index` | integer                  | Display order (`>= 0`)                                        |
| `is_hidden`   | boolean                  | Whether hidden from resolved public output                    |
| `max_depth`   | integer                  | Max nesting depth to follow (0-3)                             |
| `created_at`  | datetime                 | Creation timestamp                                            |
| `updated_at`  | datetime                 | Last update timestamp                                         |

## Subscription

Complete subscription with all details.

```python
class Subscription(BaseModel):
    token: str
    name: str
    provider_name: str | None
    description: str | None
    sources: list[Source]
    sources_count: int
    created_at: datetime
    updated_at: datetime
```

| Field            | Type              | Description                                          |
| ----------------- | ------------------ | ------------------------------------------------------- |
| `token`            | string              | Unique subscription token                                |
| `name`             | string              | User-defined name                                        |
| `provider_name`    | string · null       | Name of the provider managing this subscription, if any |
| `description`      | string · null       | Optional description                                     |
| `sources`          | array[[Source](#source)] | Resolved list of sources                            |
| `sources_count`    | integer             | Total resolved configs count                              |
| `created_at`        | datetime            | Creation timestamp                                        |
| `updated_at`        | datetime            | Last update timestamp                                     |

## SubscriptionListItem

Identical shape to [`Subscription`](#subscription); returned by `GET /api/v1/subs` (list endpoint).

## RefreshSubscriptionResponse

Returned by `POST /api/v1/subs/{token}/refresh`.

```python
class RefreshSubscriptionResponse(BaseModel):
    refreshed: int
    failed: int
    skipped: int
    total: int
    message: str | None
    errors: list[str] | None
```

| Field         | Type              | Description                                    |
| ------------- | ------------------ | ------------------------------------------------- |
| `refreshed`    | integer             | Number of successfully refreshed sources           |
| `failed`       | integer             | Number of sources that failed to refresh           |
| `skipped`      | integer             | Number of sources skipped                          |
| `total`        | integer             | Total URLs processed                                |
| `message`      | string · null       | Optional status message                             |
| `errors`       | array[string] · null | Per-URL error details                              |

## MeResponse

Returned by `GET /api/v1/me`.

```python
class MeResponse(BaseModel):
    user_id: int
    is_active: bool
```

| Field         | Type      | Description                       |
| ------------- | ---------- | ------------------------------------ |
| `user_id`      | integer     | The authenticated user's numeric ID  |
| `is_active`    | boolean     | Whether the account is active         |

## ConnectionResponse

A single provider connection from the current user's point of view.

```python
class ConnectionResponse(BaseModel):
    provider_name: str
    provider_url: str | None
    is_authorized: bool
    status: ProviderAuthorizationStatus | None
```

| Field            | Type                                    | Description                        |
| ----------------- | ---------------------------------------- | ------------------------------------- |
| `provider_name`    | string                                    | Public provider name                  |
| `provider_url`     | string · null                             | Provider's URL, if published            |
| `is_authorized`    | boolean                                   | Whether the provider is currently authorized |
| `status`           | [ProviderAuthorizationStatus](enumerations.md#providerauthorizationstatus) · null | Current authorization status |

## ConnectionsResponse

Returned by `GET /api/v1/me/providers`.

```python
class ConnectionsResponse(BaseModel):
    connections: list[ConnectionResponse]
```

## ProviderConnectionResponse

A provider↔user connection from the **provider's** point of view.

```python
class ProviderConnectionResponse(BaseModel):
    user_id: int
    status: ProviderAuthorizationStatus
```

## ProviderConnectionCreateResponse

Returned by `POST /api/v1/providers/{user_id}`. Extends [`ProviderConnectionResponse`](#providerconnectionresponse) with one additional field.

```python
class ProviderConnectionCreateResponse(ProviderConnectionResponse):
    connection_link: str | None
```

| Field               | Type            | Description                                                |
| -------------------- | ---------------- | -------------------------------------------------------------- |
| `user_id`             | integer           | *(inherited)* The end-user's numeric ID                          |
| `status`              | [ProviderAuthorizationStatus](enumerations.md#providerauthorizationstatus) | *(inherited)* Current authorization status |
| `connection_link`     | string · null     | Link the end-user can use to view/approve the connection        |

## ErrorResponse

Shape of the JSON error body returned on any non-2xx response — see [Error Handling](../api/errors.md) for the full list of error codes.

```python
class ErrorResponse(BaseModel):
    error: str
    message: str
    details: dict | None
```

| Field       | Type              | Description                             |
| ----------- | ------------------ | ------------------------------------------ |
| `error`      | string              | Error code/type; non-empty                  |
| `message`    | string              | Human-readable error message; non-empty     |
| `details`    | object · null       | Additional error details                     |
