# API Reference

Complete reference for the v2hub VPN Subscription API.

## Overview

The VPN Subscription API provides a comprehensive solution for managing and aggregating VPN proxy subscriptions. It supports:

- **Multi-source aggregation**: Combine proxy configs from direct URIs, external URLs, and internal references
- **Per-subscription comments**: Add custom comments to configs within each subscription
- **Per-source visibility & nesting control**: Hide individual sources from resolved output and cap how deep internal references are followed (`is_hidden`, `max_depth`)
- **Recursive resolution**: Automatic resolution of nested subscription references
- **Two-tier caching**: Redis + PostgreSQL for optimal performance
- **Circular reference detection**: Prevents infinite loops in subscription chains
- **Rate limiting**: Configurable limits for different endpoint types
- **Security**: HMAC-SHA256 signature verification for admin endpoints
- **Provider integrations**: External services can request user consent to manage subscriptions on a user's behalf via an explicit pending → approved authorization flow (see [Provider API](provider-api.md)), subject to a per-user limit on how many providers can be simultaneously authorized (`MAX_PROVIDERS_PER_USER`, default 5)

**Base URL**: `https://v2hub.link`

**API Version**: v1 (`1.1.2`)

## Pages

| Page                                                       | Covers                                                                                 |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| [Authentication](authentication.md)                        | User, provider, and admin authentication — headers, HMAC signing, timestamp validation |
| [Rate Limiting](rate-limiting.md)                          | Per-endpoint-type rate limits and rate limit headers                                   |
| [Public Endpoints](public-endpoints.md)                    | The unauthenticated resolved-subscription endpoint                                     |
| [Subscription Management](subscription-management.md)      | Create, read, update, delete subscriptions and their sources                           |
| [User Self-Service API](user-self-service.md)              | The authenticated user's own account and provider connections                          |
| [Provider API](provider-api.md)                            | Provider connection lifecycle and delegated subscription management                    |
| [Admin Endpoints](admin-endpoints.md)                      | User, provider, ban, and whitelist administration                                      |
| [Error Handling](errors.md)                                | Error response format and error codes                                                  |
| [Examples](examples.md)                                    | Complete workflow examples, Python client and admin client samples                     |
| [Configuration Limits & Best Practices](best-practices.md) | Default configurable limits, performance, security, and error-handling recommendations |

For the underlying request/response types referenced throughout this section, see the [Types Reference](../types/index.md).
