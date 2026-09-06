# Request Models

Request body models used by subscription, source, and provider endpoints. All are Pydantic models with `BaseModelConfig` as their base (`str_strip_whitespace=True`, `populate_by_name=True`, `use_enum_values=True`). For admin-only request models, see [Admin Models](admin-models.md).

## SourceCreateRequest

Used within [`SourcesAddRequest`](#sourcesaddrequest), [`SourcesReplaceRequest`](#sourcesreplacerequest), and [`SubscriptionCreateRequest`](#subscriptioncreaterequest).

```python
class SourceCreateRequest(BaseModelConfig):
    data: str
    is_hidden: bool | None = None
    max_depth: int | None = None  # 0-3
```

| Field       | Type           | Default | Constraints |
| ----------- | -------------- | ------- | ----------- |
| `data`      | string         | —       | Required    |
| `is_hidden` | boolean · null | `None`  | —           |
| `max_depth` | integer · null | `None`  | 0-3         |

Also accepted as a plain string wherever a list of sources is expected (e.g. `"vless://..."` instead of `{"data": "vless://..."}`) — normalized to the object form before validation via a `field_validator`.

## SubscriptionCreateRequest

```python
class SubscriptionCreateRequest(BaseModelConfig):
    name: str
    description: str | None = None
    sources: list[SourceCreateRequest] = []
```

| Field         | Type                                                         | Default | Constraints                                                                                                                                                                                                                           |
| ------------- | ------------------------------------------------------------ | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`        | string                                                       | —       | Max 64 chars; stripped and validated non-empty by a `field_validator` (no `Field(min_length=...)`, but effectively required to be non-blank)                                                                                          |
| `description` | string · null                                                | `None`  | Max 64 chars                                                                                                                                                                                                                          |
| `sources`     | array[string \| [SourceCreateRequest](#sourcecreaterequest)] | `[]`    | Max 150 items (`settings.max_sources_per_subscription`); deduplicated by `data` before length validation, keeping first occurrence; rejected if the list becomes empty after deduplication and at least one item was originally given |

## SubscriptionUpdateRequest

```python
class SubscriptionUpdateRequest(BaseModelConfig):
    name: str | None = None
    description: str | None = None
```

| Field         | Type          | Default | Constraints                                                  |
| ------------- | ------------- | ------- | ------------------------------------------------------------ |
| `name`        | string · null | `None`  | Max 64 chars; stripped and validated non-empty _if provided_ |
| `description` | string · null | `None`  | Max 64 chars                                                 |

Both fields are independently optional — there is no cross-field validator requiring at least one to be set.

## SourcesAddRequest

```python
class SourcesAddRequest(BaseModelConfig):
    sources: list[SourceCreateRequest]  # min 1, max settings.max_sources_per_subscription
```

Same normalization/deduplication as `SubscriptionCreateRequest.sources`, but with a hard minimum: the request itself requires `min_length=1`, and deduplication down to an empty list raises a validation error.

## SourcesReplaceRequest

```python
class SourcesReplaceRequest(BaseModelConfig):
    sources: list[SourceCreateRequest] = []  # max settings.max_sources_per_subscription
```

Same shape as [`SourcesAddRequest`](#sourcesaddrequest), except an empty list is explicitly allowed (`default_factory=list`, no minimum) — this is what lets [Replace All Sources](../api/subscription-management.md#replace-all-sources) clear every source in one call.

## SourcesRemoveRequest

```python
class SourcesRemoveRequest(BaseModelConfig):
    source_ids: list[str]  # min 1 item, each HASH_LENGTH chars
```

| Field        | Type          | Constraints                                     |
| ------------ | ------------- | ----------------------------------------------- |
| `source_ids` | array[string] | At least 1 item; each item exactly 32 hex chars |

IDs are deduplicated (preserving first occurrence) before the minimum-length check; an empty result raises a validation error.

## SourceUpdateRequest

Backs [Update Config](../api/subscription-management.md#update-config). Only fields explicitly present and non-`null` in the request are applied; everything else is left unchanged server-side.

```python
class SourceUpdateRequest(BaseModelConfig):
    config_hash: str      # alias: config_id
    comment: str | None = None
    is_hidden: bool | None = None
    max_depth: int | None = None  # 0-3
```

| Field         | Type           | Constraints                                                                                                                                                                  |
| ------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `config_hash` | string         | Exactly 32 hex chars. **Field name aliasing**: the request body may use either `config_hash` or `config_id` as the JSON key — both populate this same field (`AliasChoices`) |
| `comment`     | string · null  | Max 255 chars                                                                                                                                                                |
| `is_hidden`   | boolean · null | —                                                                                                                                                                            |
| `max_depth`   | integer · null | 0-3                                                                                                                                                                          |

## CommentUpdateRequest <sup>Deprecated</sup>

Backs the deprecated [Update Config Comment](../api/subscription-management.md#update-config-comment-deprecated) endpoint. The class itself is marked `@deprecated` in source (`typing_extensions.deprecated`), which some type checkers and IDEs will surface as a warning if you construct it directly.

```python
class CommentUpdateRequest(BaseModelConfig):
    config_hash: str      # alias: config_id
    comment: str | None = None
```

Same `config_hash`/`config_id` aliasing as [`SourceUpdateRequest`](#sourceupdaterequest).

## ProviderConnectionRequest

```python
class ProviderConnectionRequest(BaseModelConfig):
    user_id: int  # 1 to 999,999,999,999
```

Underlies the path-parameter-driven provider endpoints ([Request Access to User](../api/provider-api.md#request-access-to-user), etc.) — in practice `user_id` arrives via the URL path on those routes rather than this model being used as a JSON body directly, but it defines the same validation constraints.
