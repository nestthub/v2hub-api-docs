# Enumerations

Both enums below are `StrEnum` subclasses in the source (`core/enums.py`) — they serialize as their plain string value, and compare equal to that string directly (`SourceType.CONFIG == "config"`).

## SourceType

```python
class SourceType(StrEnum):
    CONFIG = "config"
    EXTERNAL_URL = "external_url"
    INTERNAL_TOKEN = "internal_token"
```

| Value            | Description                                                                    |
| ---------------- | ------------------------------------------------------------------------------ |
| `config`         | Direct proxy configuration (`vless://`, `vmess://`, etc.)                      |
| `external_url`   | HTTPS URL to fetch a third-party subscription from                             |
| `internal_token` | Reference to another subscription's token on this server, resolved recursively |

`source_type` is always derived server-side from the value of `data` when a source is created — it is never set directly by the client, and there is no request field for it.

## ProviderAuthorizationStatus

```python
class ProviderAuthorizationStatus(StrEnum):
    PENDING = "pending"
    APPROVED = "approved"
    REVOKED = "revoked"
```

| Value      | Description                                                                                                                                                                                                                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pending`  | A connection request exists; awaiting approval                                                                                                                                                                                                                                                  |
| `approved` | The provider is authorized to manage the user's subscriptions                                                                                                                                                                                                                                   |
| `revoked`  | Previously approved, then revoked — the record is kept (rather than deleted) when the user has subscription history with this provider; see [Provider API → Reject Provider Connection](../api/admin-endpoints.md#reject-provider-connection) for one of the two paths that produces this state |

Several response models (e.g. [`ConnectionResponse`](response-models.md#connectionresponse), [`ProviderAuthorizationInfoResponse`](admin-models.md#providerauthorizationinforesponse)) type their `status` field as `ProviderAuthorizationStatus | None` — `null`/`None` specifically means "no authorization record exists at all," which is a distinct, fourth state from all three enum values above (for example, after a full delete rather than a revoke — see [Reject Provider Connection](../api/admin-endpoints.md#reject-provider-connection)). There is no `UNKNOWN` or similar catch-all member in the source; unrecognized status values are not expected to occur since the enum is the sole source of truth for this field server-side.
