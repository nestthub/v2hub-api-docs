# V2Hub API

> Reference documentation for the V2Hub VPN Subscription API

The V2Hub API is a production-ready FastAPI service for managing, aggregating, and serving VPN proxy subscriptions, with multi-source aggregation, recursive resolution, provider delegation, and HMAC-secured administration endpoints.

**API Version**: 1.1.2 · **Base URL**: https://v2hub.link

## Explore the API

### [API Reference](api/index.md)

Every endpoint the API exposes: authentication, rate limiting, public access, subscription management, the user self-service API, the provider API, and admin endpoints — with request/response bodies, schemas, and error codes for each.

**[Get started →](api/index.md)**

### [Types Reference](types/index.md)

Every request model, response model, admin model, and enumeration used by the API, field by field.

**[Get started →](types/index.md)**

!!! note "Verified against source"
Every page on this site was written directly against the `v2hub-api` server source (routers, Pydantic schemas, and settings) rather than the repository's own `docs/` folder — see [API Reference → Overview](api/index.md) for why.

## Looking for a client library instead?

If you want to integrate with the API from Python rather than calling it directly, see the [V2Hub client library documentation](https://v2hub.dev) — it covers the `v2hub` Python client, the `v2hub-admin` administration extension, and the `v2hub-cli` command-line tool, all built on top of this API.

## Ecosystem

The V2Hub API is part of the broader [V2Hub Ecosystem](https://github.com/nestthub/nestthub/tree/main/ecosystems/v2hub), which includes client libraries, administration tools, a CLI, a web panel, and a Telegram bot. For self-hosting and operator resources, see [v2hub.link](https://v2hub.link).
