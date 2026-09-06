# Admin Models

Request and response models used exclusively by [Admin Endpoints](../api/admin-endpoints.md). All are `AdminBaseModel` subclasses (`str_strip_whitespace=True`, `populate_by_name=True`; unlike the regular `BaseModelConfig`, this base does **not** set `use_enum_values=True`).

## User Models

### UserCreateRequest

```python
class UserCreateRequest(AdminBaseModel):
    user_id: int   # 1 to 999,999,999,999
```

### UserResponse

```python
class UserResponse(AdminBaseModel):
    user_hash: str            # UUID, 36 chars
    user_id: int
    api_token: str            # 43 chars
    is_active: bool
    provider_hash: str | None # UUID, 36 chars
```

`api_token` is populated on **every** response that returns a `UserResponse` — including a plain [Get User](../api/admin-endpoints.md#get-user) lookup, not just on creation. There is no "masked" or omitted-token variant of this model. `provider_hash` is non-`null` only if this user owns a provider account.

### UserCreateResponse

```python
class UserCreateResponse(UserResponse):
    pass
```

Identical shape to [`UserResponse`](#userresponse) — a distinct class name purely for endpoint-documentation purposes, with no additional or different fields.

### UserStatusUpdateRequest

```python
class UserStatusUpdateRequest(AdminBaseModel):
    is_active: bool
```

### TokenRefreshRequest

```python
class TokenRefreshRequest(AdminBaseModel):
    user_id: int
```

### TokenRefreshResponse

```python
class TokenRefreshResponse(AdminBaseModel):
    user_id: int
    new_api_token: str  # 43 chars
```

## Provider Models

### ProviderCreateRequest

```python
class ProviderCreateRequest(AdminBaseModel):
    owner_hash: str       # UUID, 36 chars
    provider_name: str    # 4-16 chars, pattern ^[a-z0-9]+(?:-[a-z0-9]+)*$
    provider_url: str | None = None  # max 255 chars
```

`provider_url`, if given, is validated with the same external-URL/SSRF check used elsewhere in the API (`validate_external_url`) — an invalid URL raises a Pydantic validation error at the field-validator level (surfaces as `422` with `invalid_url` — see [Error Handling](../api/errors.md)).

### ProviderResponse

```python
class ProviderResponse(AdminBaseModel):
    provider_hash: str      # UUID, 36 chars
    owner_hash: str         # UUID, 36 chars
    provider_name: str      # 4-16 chars
    api_token: str          # 43 chars
    provider_url: str | None  # max 255 chars
    is_active: bool
```

Like [`UserResponse`](#userresponse), `api_token` appears on every lookup of this model, not only on creation.

### ProviderCreateResponse

```python
class ProviderCreateResponse(ProviderResponse):
    pass
```

Same relationship to `ProviderResponse` as `UserCreateResponse` has to `UserResponse` — identical fields, distinct name.

### AllProvidersResponse

```python
class AllProvidersResponse(AdminBaseModel):
    provider_hashes: dict[str, str]  # provider_name -> provider_hash
```

Keys are constrained to 4-16 chars (matching `provider_name` rules); values to 36 chars (UUID format).

### ProviderStatusUpdateRequest

```python
class ProviderStatusUpdateRequest(AdminBaseModel):
    is_active: bool
```

### ProviderURLUpdateRequest

```python
class ProviderURLUpdateRequest(AdminBaseModel):
    provider_url: str | None  # max 255 chars, SSRF-validated
```

### ProviderNameUpdateRequest

```python
class ProviderNameUpdateRequest(AdminBaseModel):
    provider_name: str  # 4-16 chars, same pattern as ProviderCreateRequest
```

Three separate single-field request models, each backing its own `PATCH` endpoint (see [Admin Endpoints → Provider Management](../api/admin-endpoints.md#provider-management)) rather than one combined "update provider" request.

### ProviderTokenRefreshRequest

```python
class ProviderTokenRefreshRequest(AdminBaseModel):
    provider_hash: str  # UUID, 36 chars
```

### ProviderTokenRefreshResponse

```python
class ProviderTokenRefreshResponse(AdminBaseModel):
    provider_hash: str
    new_api_token: str  # 43 chars
```

## Provider Authorization Models

### ProviderAuthorizationInfoResponse

```python
class ProviderAuthorizationInfoResponse(AdminBaseModel):
    provider_name: str
    provider_url: str | None
    user_id: int
    status: ProviderAuthorizationStatus | None = None
```

`status: null` specifically means no authorization record exists at all between this provider and user — distinct from any of the three enum values. See [Enumerations → ProviderAuthorizationStatus](enumerations.md#providerauthorizationstatus).

### ProviderAuthorizationBaseRequest

```python
class ProviderAuthorizationBaseRequest(AdminBaseModel):
    user_id: int
    provider_name: str  # 4-16 chars, pattern ^[a-z0-9]+(?:-[a-z0-9]+)*$
```

Base class for the two request models below — not used directly as a request body itself.

### ProviderAuthorizationRequest

```python
class ProviderAuthorizationRequest(ProviderAuthorizationBaseRequest):
    hmac: str | None = None  # exactly 24 hex chars if provided
```

Backs [Process Provider Connection Request](../api/admin-endpoints.md#process-provider-connection-request). `hmac` is only required when creating a brand-new `pending` authorization for a user with no prior record with this provider — see [Authentication → Provider Connection-Invite HMAC](../api/authentication.md#provider-connection-invite-hmac).

### ProviderAuthorizationDecisionRequest

```python
class ProviderAuthorizationDecisionRequest(ProviderAuthorizationBaseRequest):
    pass
```

Identical fields to [`ProviderAuthorizationBaseRequest`](#providerauthorizationbaserequest) (`user_id`, `provider_name`) — backs both [Approve](../api/admin-endpoints.md#approve-provider-connection) and [Reject](../api/admin-endpoints.md#reject-provider-connection) Provider Connection, neither of which needs the `hmac` field since they act on an authorization that (by definition) already exists.

## IP Ban Models

### IPBanRequest

```python
class IPBanRequest(AdminBaseModel):
    ip_address: str
    duration_seconds: int | None = None
```

No format validation on `ip_address` beyond being a non-empty string at the Pydantic level (IPv4/IPv6 format correctness is presumably enforced deeper in the service layer, not visible in this schema itself).

### IPUnbanRequest

```python
class IPUnbanRequest(AdminBaseModel):
    ip_address: str  # min 8 chars
```

### IPUnbanResponse

```python
class IPUnbanResponse(AdminBaseModel):
    ip_address: str
    was_banned: bool
    message: str
```

### IPBanStatusResponse

```python
class IPBanStatusResponse(AdminBaseModel):
    ip_address: str
    is_banned: bool
    banned_until: str | None = None       # ISO 8601 string, not a datetime type
    remaining_seconds: int | None = None  # >= 0
```

Also the response shape for [Ban IP Address](../api/admin-endpoints.md#ban-ip-address), per that endpoint's `response_model`.

### IPBanEntry

```python
class IPBanEntry(AdminBaseModel):
    ip_address: str
    banned_until: str | None = None
```

### IPBanListResponse

```python
class IPBanListResponse(AdminBaseModel):
    entries: list[IPBanEntry]
    total: int
```

## Whitelist Models

### WhitelistAddRequest

```python
class WhitelistAddRequest(AdminBaseModel):
    ip_address: str
    description: str | None = None  # max 255 chars
```

### WhitelistAddResponse

```python
class WhitelistAddResponse(AdminBaseModel):
    ip_address: str
    description: str | None = None
    message: str
```

### WhitelistRemoveRequest

```python
class WhitelistRemoveRequest(AdminBaseModel):
    ip_address: str
```

### WhitelistRemoveResponse

```python
class WhitelistRemoveResponse(AdminBaseModel):
    ip_address: str
    was_whitelisted: bool
    message: str
```

### WhitelistEntry

```python
class WhitelistEntry(AdminBaseModel):
    ip_address: str
    description: str | None = None
    added_at: str  # ISO 8601 string
```

### WhitelistListResponse

```python
class WhitelistListResponse(AdminBaseModel):
    entries: list[WhitelistEntry] = []
    total: int  # >= 0
```

## Stats Models

### GeneralStats

```python
class GeneralStats(AdminBaseModel):
    total_users: int
    new_users: int
    new_subscriptions: int
```

### StatsResponse

```python
class StatsResponse(AdminBaseModel):
    general: GeneralStats
```
