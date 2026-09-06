# Enumerations

## SourceType

```python
class SourceType(str, Enum):
    CONFIG = "config"
    EXTERNAL_URL = "external_url"
    INTERNAL_TOKEN = "internal_token"
```

| Value             | Description                                          |
| ------------------ | ------------------------------------------------------ |
| `CONFIG`            | Direct proxy configuration (vless://, vmess://, etc.) |
| `EXTERNAL_URL`      | HTTPS URL to fetch subscription content from            |
| `INTERNAL_TOKEN`    | Reference to another subscription token on this server |

The `source_type` of a `Source` is determined server-side from the value of its `data` field — it is never set directly by the client.

## ProviderAuthorizationStatus

```python
class ProviderAuthorizationStatus(str, Enum):
    PENDING = "pending"
    APPROVED = "approved"
    REVOKED = "revoked"
```

| Value        | Description                                                  |
| ------------- | --------------------------------------------------------------- |
| `PENDING`      | Provider has requested access; awaiting user approval            |
| `APPROVED`     | User has approved; provider can manage subscriptions             |
| `REVOKED`      | Access was approved but has since been revoked                   |

Client libraries built against this API (see the [official client documentation](https://v2hub.dev)) may additionally define an `UNKNOWN` fallback value for forward compatibility with status values introduced after the client was released — that fallback is a client-side convenience and is never returned by the API itself, which only ever returns one of the three values above.
