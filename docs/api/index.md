# API Reference

Complete reference for the V2Hub VPN Subscription API, generated directly from the server's source code (`nestthub/v2hub-api`).

## Overview

The VPN Subscription API provides a comprehensive solution for managing and aggregating VPN proxy subscriptions. It supports:

- **Multi-source aggregation**: Combine proxy configs from direct URIs, external URLs, and internal references to other subscriptions
- **Per-source comments, visibility & nesting control**: Each source can carry a custom comment, be hidden from resolved output (`is_hidden`), and cap how many further levels of nested subscription references are followed (`max_depth`, 0-3)
- **Recursive resolution**: Automatic resolution of nested subscription references, with circular-reference detection
- **Two-tier caching**: Redis + PostgreSQL for performance
- **Rate limiting**: Configurable limits for different endpoint types
- **HMAC-signed administration**: Admin endpoints require a signed request (see [Authentication](authentication.md)) plus an IP allowlist
- **Provider delegation**: External services can request permission to manage a user's subscriptions on their behalf, subject to a per-user cap on simultaneously-approved providers (see [Provider API](provider-api.md))

**Base URL**: your own deployment, e.g. `https://your-domain.com`

**API Version**: `v1`

## Pages

| Page                                                  | Covers                                                                              |
| ----------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [Authentication](authentication.md)                   | User/provider token header, admin HMAC signing, provider connection-invite HMAC     |
| [Rate Limiting](rate-limiting.md)                     | Per-endpoint-type rate limits                                                       |
| [Public Endpoints](public-endpoints.md)               | The unauthenticated resolved-subscription endpoint                                  |
| [Subscription Management](subscription-management.md) | Create, read, update, delete subscriptions and their sources                        |
| [User Self-Service API](user-self-service.md)         | The authenticated user's own account and connections (`/me`)                        |
| [Provider API](provider-api.md)                       | Provider connection lifecycle and delegated subscription management                 |
| [Admin Endpoints](admin-endpoints.md)                 | User, provider, provider-authorization, ban, whitelist, and stats administration    |
| [Error Handling](errors.md)                           | Error response shape, status codes, and error codes, as actually implemented        |
| [Configuration Limits](configuration-limits.md)       | Every configurable limit and default, read from the server's settings and constants |

For the underlying request/response types referenced throughout this section, see the [Types Reference](../types/index.md).

!!! note "Source of truth"
Every endpoint, field, status code, and limit on these pages was verified directly against the `v2hub-api` source (`src/v2hub_api/`) — routers, Pydantic schemas, `core/config.py`, and `core/constants.py` — rather than against the repository's own `docs/` folder, which was found to have drifted from the actual implementation in several places.
