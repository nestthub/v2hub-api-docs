# v2hub API

> Reference documentation for the v2hub VPN Subscription API

The v2hub API is a production-ready FastAPI service for managing, aggregating, and serving VPN proxy subscriptions, with multi-source aggregation, recursive resolution, provider delegation, and HMAC-secured administration endpoints.

**API Version**: 1.1.2 · **Base URL**: https://v2hub.link

## Explore the API

### [API Reference](api/index.md)

Every endpoint the API exposes: authentication, rate limiting, public access, subscription management, the user self-service API, the provider API, and admin endpoints — with request/response bodies, schemas, and error codes for each.

**[Get started →](api/index.md)**

### [Types Reference](types/index.md)

Every request model, response model, admin model, and enumeration used by the API, plus the shared validation rules and type aliases behind them.

**[Get started →](types/index.md)**

## Looking for a client library instead?

If you want to integrate with the API from Python rather than calling it directly, see the [v2hub client library documentation](https://v2hub.dev) — it covers the `v2hub` Python client, the `v2hub-admin` administration extension, and the `v2hub-cli` command-line tool, all built on top of this API.

The v2hub API is part of the broader [v2hub Ecosystem](https://github.com/nestthub/nestthub/tree/main/ecosystems/v2hub), which includes client libraries, administration tools, a CLI, a web panel, and a Telegram bot.
