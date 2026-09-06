# Configuration Limits

Every value on this page was read directly from `core/config.py` (`Settings`, environment-variable-driven) or `core/constants.py` (fixed constants, not configurable per deployment) in the `v2hub-api` source. Where a setting is environment-configurable, its `validation_alias` (the environment variable name) is given — your specific deployment may override the default shown.

## Environment-Configurable Settings (`core/config.py`)

| Setting                        | Env Var                        | Default                       | Description                                                                                                           |
| ------------------------------ | ------------------------------ | ----------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `max_name_length`              | `MAX_NAME_LENGTH`              | 64                            | Max length for subscription names                                                                                     |
| `max_description_length`       | `MAX_DESCRIPTION_LENGTH`       | 255                           | Declared max length for descriptions — see note below                                                                 |
| `max_comment_length`           | `MAX_COMMENT_LENGTH`           | 255                           | Declared max length for config comments — see note below                                                              |
| `max_nesting_depth`            | `MAX_NESTING_DEPTH`            | 3                             | Max recursion depth for nested subscription references                                                                |
| `max_subscriptions_per_user`   | `MAX_SUBSCRIPTIONS_PER_USER`   | 3                             | Max subscriptions per user                                                                                            |
| `max_configs_per_subscription` | `MAX_CONFIGS_PER_SUBSCRIPTION` | 150                           | Max resolved configs in one subscription                                                                              |
| `max_sources_per_subscription` | `MAX_SOURCES_PER_SUBSCRIPTION` | 150                           | Max sources per subscription                                                                                          |
| `refresh_cooldown`             | `REFRESH_COOLDOWN`             | 900 (15 min)                  | Minimum interval between automatic lazy-refreshes of `external_url` sources                                           |
| `fetch_timeout`                | `FETCH_TIMEOUT`                | 1 second                      | HTTP timeout when fetching an external subscription URL                                                               |
| `fetch_user_agent`             | `FETCH_USER_AGENT`             | `v2hub/1.0`                   | User-Agent sent when fetching external subscription URLs                                                              |
| `fetch_max_redirects`          | `FETCH_MAX_REDIRECTS`          | 1                             | Max redirects followed when fetching an external URL                                                                  |
| `max_providers_per_user`       | `MAX_PROVIDERS_PER_USER`       | 5                             | Max simultaneously-approved providers per user                                                                        |
| `connection_link_prefix`       | `CONNECTION_LINK_PREFIX`       | `https://t.me/v2hubot?start=` | Prefix used to build the provider connection-invite link — see [Provider API](provider-api.md#request-access-to-user) |
| `redis_ttl`                    | `REDIS_TTL`                    | 600 (10 min)                  | Default TTL for cached items                                                                                          |
| `public_rps`                   | —                              | 3 req/sec                     | Rate limit for `/sub/{token}` — see [Rate Limiting](rate-limiting.md)                                                 |
| `internal_no_token_rps`        | —                              | 1 req/sec                     | Rate limit for `/api/v1/...` without a valid token                                                                    |
| `internal_with_token_rps`      | —                              | 3 req/sec                     | Rate limit for `/api/v1/...` with a valid token                                                                       |

!!! note "`max_description_length` / `max_comment_length` are not currently wired up"
These two `Settings` fields exist and default to 255, but the actual Pydantic request models that validate subscription descriptions and config comments (`SubscriptionCreateRequest.description`, `SourceUpdateRequest.comment`, etc.) import their `max_length` from the **fixed constants** below instead (`SUBSCRIPTION_DESCRIPTION_MAX_LENGTH = 64`, `COMMENT_MAX_LENGTH = 255`). In practice, subscription descriptions are capped at **64** characters, not the 255 this setting might suggest — the setting appears to be effectively unused dead configuration in the version of the source this documentation was written against. Comments genuinely are capped at 255, so that one happens to agree by coincidence. Verify against your deployed version if this matters for your integration.

## Fixed Constants (`core/constants.py`)

These are not environment-configurable — changing them requires a code change and redeploy.

| Constant                              | Value           | Description                                                                                                                       |
| ------------------------------------- | --------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `HASH_LENGTH`                         | 32              | Length of hex source-hash IDs (16 raw bytes)                                                                                      |
| `API_TOKEN_LENGTH`                    | 43              | Length of unpadded Base64URL API tokens (32 raw bytes)                                                                            |
| `SUBSCRIPTION_TOKEN_LENGTH`           | 43              | Length of unpadded Base64URL subscription tokens (32 raw bytes)                                                                   |
| `UUID_LENGTH`                         | 36              | Length of `user_hash`/`provider_hash`/`owner_hash` (standard UUID string)                                                         |
| `PROVIDER_NAME_MIN_LENGTH`            | 4               | Minimum provider name length                                                                                                      |
| `PROVIDER_NAME_MAX_LENGTH`            | 16              | Maximum provider name length                                                                                                      |
| `URL_MAX_LENGTH`                      | 255             | Maximum length for a stored URL (provider URL, etc.)                                                                              |
| `USER_ID_MIN`                         | 1               | Minimum accepted `user_id`                                                                                                        |
| `USER_ID_MAX`                         | 999,999,999,999 | Maximum accepted `user_id`                                                                                                        |
| `COMMENT_MAX_LENGTH`                  | 255             | Maximum config comment length                                                                                                     |
| `SUBSCRIPTION_NAME_MAX_LENGTH`        | 64              | Maximum subscription name length (this one _is_ actually enforced, matching `max_name_length` above)                              |
| `SUBSCRIPTION_DESCRIPTION_MAX_LENGTH` | 64              | Maximum subscription description length (**actually enforced** — see the note above)                                              |
| `AUTH_HMAC_LENGTH`                    | 24              | Truncated length of the provider connection-invite HMAC — see [Authentication](authentication.md#provider-connection-invite-hmac) |

## Security & Timing

| Setting                          | Value                        | Configurable?                                                                                                                                     |
| -------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Default IP ban duration          | 3600 seconds (1 hour)        | Yes, per-request via `duration_seconds` on [Ban IP Address](admin-endpoints.md#ban-ip-address); the 1-hour value is the code default when omitted |
| Admin HMAC timestamp window      | ±60 seconds                  | No — hardcoded in `verify_request_signature`, not an environment variable in the version this was verified against                                |
| Provider `provider_name` pattern | `^[a-z0-9]+(?:-[a-z0-9]+)*$` | No — enforced by Pydantic `pattern` on the relevant request models                                                                                |
