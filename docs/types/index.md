# Types Reference

Complete reference for every request/response model used by the V2Hub API, read directly from the Pydantic schema definitions in `src/v2hub_api/schemas/`.

**API Version**: `v1`

## Pages

| Page                                  | Covers                                                                                       |
| ------------------------------------- | -------------------------------------------------------------------------------------------- |
| [Enumerations](enumerations.md)       | `SourceType` and `ProviderAuthorizationStatus`                                               |
| [Request Models](request-models.md)   | Subscription, source, and provider request bodies                                            |
| [Response Models](response-models.md) | Subscription, source, connection, and error response shapes                                  |
| [Admin Models](admin-models.md)       | Admin-only request/response models (users, providers, authorization, bans, whitelist, stats) |

For the endpoints that use these types, see the [API Reference](../api/index.md).

## Base Identifier Formats

Common identifier shapes reused across many models — not Python type aliases in the source itself, but consistent formats worth knowing up front:

| Kind                                         | Format             | Length               | Example                                |
| -------------------------------------------- | ------------------ | -------------------- | -------------------------------------- |
| Subscription token                           | Unpadded Base64URL | 43 chars             | `dGhpcyBpcyBhIHRva2Vu...`              |
| API token                                    | Unpadded Base64URL | 43 chars             | `YW5vdGhlciB0b2tlbiBoZXJl...`          |
| Source ID (hash)                             | Hex                | 32 chars             | `a1b2c3d4e5f67890a1b2c3d4e5f67890`     |
| `user_hash` / `provider_hash` / `owner_hash` | UUID string        | 36 chars             | `3f2a1b9c-7d4e-4a1f-9c3e-8b2a1d4f5e6c` |
| `user_id`                                    | Integer            | 1 to 999,999,999,999 | `12345`                                |

All of the length/format constraints above are enforced by `Field(min_length=..., max_length=...)` (or `pattern=...`) on the actual Pydantic models — see [Configuration Limits](../api/configuration-limits.md) for the underlying constants.
