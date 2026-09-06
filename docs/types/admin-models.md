# Admin Models

Request and response models used exclusively by [Admin Endpoints](../api/admin-endpoints.md).

## User Models

### UserCreateRequest

```python
class UserCreateRequest(BaseModel):
    user_id: int
```

| Field     | Type    | Validation |
| --------- | ------- | ------------ |
| `user_id`  | integer  | `> 0`         |

### UserResponse

```python
class UserResponse(BaseModel):
    user_id: int
    api_token: str
    is_active: bool
    created_at: datetime
```

`api_token` is only populated on creation and token-refresh responses; it is not returned by `GET`/`PATCH` endpoints that merely inspect or update status.

### UserStatusUpdateRequest

```python
class UserStatusUpdateRequest(BaseModel):
    is_active: bool
```

### TokenRefreshResponse

```python
class TokenRefreshResponse(BaseModel):
    user_id: int
    new_api_token: str
```

Also used, with the equivalent `provider_hash`-keyed shape, for provider token refresh responses.

## Provider Models

### ProviderCreateRequest

```python
class ProviderCreateRequest(BaseModel):
    owner_hash: str
    provider_name: str
    provider_url: str | None = None
```

| Field             | Type            | Validation                              |
| ------------------ | ---------------- | ------------------------------------------ |
| `owner_hash`         | string            | Non-empty; must reference an existing user |
| `provider_name`      | string            | Non-empty, unique                           |
| `provider_url`       | string · null     | Optional                                    |

### ProviderResponse

```python
class ProviderResponse(BaseModel):
    provider_hash: str
    provider_name: str
    provider_url: str | None
    api_token: str
    is_active: bool
    created_at: datetime
```

Like [`UserResponse`](#userresponse), `api_token` is only populated on creation and token-refresh responses.

### ProviderStatusUpdateRequest

```python
class ProviderStatusUpdateRequest(BaseModel):
    is_active: bool
```

### ProviderHashesResponse

Returned by the list-all-providers endpoint.

```python
class ProviderHashesResponse(BaseModel):
    provider_hashes: dict[str, str]
```

A mapping of `provider_name` → `provider_hash`.

## IP Ban Models

### BanRequest

```python
class BanRequest(BaseModel):
    ip_address: str
    duration_seconds: int | None = None
```

| Field                | Type            | Validation                                   |
| --------------------- | ---------------- | ----------------------------------------------- |
| `ip_address`            | string            | Valid IPv4 or IPv6 address                        |
| `duration_seconds`      | integer · null     | Optional; server default used if omitted          |

### BanResponse

```python
class BanResponse(BaseModel):
    ip_address: str
    banned_until: datetime
    remaining_seconds: int
```

### BanStatusResponse

```python
class BanStatusResponse(BaseModel):
    ip_address: str
    is_banned: bool
    banned_until: datetime | None
    remaining_seconds: int | None
```

### UnbanResponse

```python
class UnbanResponse(BaseModel):
    ip_address: str
    was_banned: bool
```

`was_banned` is `false` if the address wasn't actually banned — the endpoint is safe to call unconditionally.

### BanListResponse

```python
class BanListEntry(BaseModel):
    ip_address: str
    banned_until: datetime


class BanListResponse(BaseModel):
    total: int
    entries: list[BanListEntry]
```

## Whitelist Models

### WhitelistRequest

```python
class WhitelistRequest(BaseModel):
    ip_address: str
    description: str | None = None
```

| Field           | Type            | Validation                        |
| ---------------- | ---------------- | -------------------------------------- |
| `ip_address`       | string            | Valid IPv4/IPv6 address or CIDR range   |
| `description`      | string · null     | Optional                                |

### WhitelistAddResponse

```python
class WhitelistAddResponse(BaseModel):
    message: str
```

### WhitelistEntry

```python
class WhitelistEntry(BaseModel):
    ip_address: str
    description: str | None
    added_at: datetime
```

### WhitelistResponse

```python
class WhitelistResponse(BaseModel):
    entries: list[WhitelistEntry]
```

### WhitelistRemoveResponse

```python
class WhitelistRemoveResponse(BaseModel):
    ip_address: str
    was_whitelisted: bool
```

## Stats Models

### StatsResponse

```python
class StatsPeriod(BaseModel):
    start: datetime
    end: datetime


class StatsResponse(BaseModel):
    period: StatsPeriod
    total_requests: int
    total_subscriptions: int
    total_users: int
    total_providers: int
```

The exact set of fields on `StatsResponse` may be extended over time; treat unknown fields as informational rather than relying on this being an exhaustive list.
