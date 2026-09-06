# Request Models

Request body models for subscription, source, and provider endpoints. For admin-only request models, see [Admin Models](admin-models.md).

## SourceCreate

Used within subscription/source creation and replacement requests.

```python
class SourceCreate(BaseModel):
    data: str
    is_hidden: bool = False
    max_depth: int = 3
```

| Field       | Type    | Default | Validation                          |
| ----------- | ------- | ------- | ------------------------------------- |
| `data`      | string  | —        | Non-empty                              |
| `is_hidden` | boolean | `false`   | —                                       |
| `max_depth` | integer | `3`       | 0-3                                     |

Also accepted as a plain string wherever a list of sources is expected — a bare string is equivalent to `{"data": "<string>"}` with defaults for the other fields.

## SubscriptionCreateRequest

```python
class SubscriptionCreateRequest(BaseModel):
    name: str
    description: str | None = None
    sources: list[str | SourceCreate] = []
```

| Field         | Type                          | Default | Validation                    |
| ------------- | ------------------------------ | ------- | -------------------------------- |
| `name`        | string                          | —        | 1-64 chars, non-empty after strip |
| `description` | string · null                  | `null`    | Max 64 chars                     |
| `sources`     | array[string \| SourceCreate]   | `[]`      | Max 150 items                    |

## SubscriptionUpdateRequest

```python
class SubscriptionUpdateRequest(BaseModel):
    name: str | None = None
    description: str | None = None
```

| Field         | Type            | Default | Validation                              |
| ------------- | ---------------- | ------- | ------------------------------------------ |
| `name`        | string · null    | `null`    | 1-64 chars if provided                     |
| `description` | string · null    | `null`    | Max 64 chars if provided                   |

At least one of `name` / `description` must be provided.

## SourceAddRequest

```python
class SourceAddRequest(BaseModel):
    sources: list[str | SourceCreate]
```

| Field     | Type                          | Validation |
| --------- | ------------------------------ | ------------ |
| `sources` | array[string \| SourceCreate]   | 1-150 items  |

## SourceReplaceRequest

```python
class SourceReplaceRequest(BaseModel):
    sources: list[str | SourceCreate]
```

| Field     | Type                          | Validation                          |
| --------- | ------------------------------ | -------------------------------------- |
| `sources` | array[string \| SourceCreate]   | Max 150 items; empty array allowed     |

## SourceRemoveRequest

```python
class SourceRemoveRequest(BaseModel):
    source_ids: list[str]
```

| Field         | Type          | Validation |
| ------------- | ------------- | ------------ |
| `source_ids`  | array[string] | Min 1 item   |

## SourceUpdateRequest

Used by `PATCH /api/v1/subs/{token}/config`. Supersedes [`CommentUpdateRequest`](#commentupdaterequest-deprecated): supports the same comment update plus `is_hidden` and `max_depth`. Only fields explicitly provided are changed; omitted fields are left unchanged.

```python
class SourceUpdateRequest(BaseModel):
    config_id: str
    comment: str | None = None
    is_hidden: bool | None = None
    max_depth: int | None = None
```

| Field        | Type             | Validation      |
| ------------ | ----------------- | ----------------- |
| `config_id`  | string             | Non-empty          |
| `comment`    | string · null      | Max 256 chars       |
| `is_hidden`  | boolean · null     | —                    |
| `max_depth`  | integer · null     | 0-3                  |

## CommentUpdateRequest <sup>Deprecated</sup>

> **Deprecated**: Used by `PATCH /api/v1/subs/{token}/comments`, which still works and continues to be fully supported. Prefer [`SourceUpdateRequest`](#sourceupdaterequest) / `PATCH /api/v1/subs/{token}/config` instead.

```python
class CommentUpdateRequest(BaseModel):
    config_id: str
    comment: str | None = None
```

| Field        | Type             | Validation      |
| ------------ | ----------------- | ----------------- |
| `config_id`  | string             | Non-empty          |
| `comment`    | string · null      | Max 256 chars       |

## ProviderConnectionRequest

Used internally by provider-connection endpoints (`POST /api/v1/providers/{user_id}`, and the corresponding admin endpoints).

```python
class ProviderConnectionRequest(BaseModel):
    user_id: int
```

| Field     | Type    | Validation |
| --------- | ------- | ------------ |
| `user_id` | integer | `> 0`        |
