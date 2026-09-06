# v2hub API Documentation

> Official reference documentation for the v2hub VPN Subscription API

v2hub API Documentation provides the complete endpoint and type reference for the v2hub server itself — authentication, rate limiting, subscription management, the provider API, admin endpoints, and every request/response model behind them.

### 🌐 [Part of the v2hub Ecosystem](https://github.com/nestthub/nestthub/tree/main/ecosystems/v2hub)

This repository documents the **[v2hub API](https://github.com/nestthub/v2hub-api)** server directly — the HTTP endpoints and JSON payloads themselves, independent of any client library.

If you're integrating from Python rather than calling the API directly, see the **[v2hub client documentation](https://v2hub.dev)** instead, which covers the `v2hub` client library, the `v2hub-admin` administration extension, and the `v2hub-cli` command-line tool.

---

## 📚 Documentation

The full documentation is available at [**docs.v2hub.link**](https://docs.v2hub.link).

It covers:

- Authentication (user, provider, and HMAC-signed admin auth)
- Rate limiting
- Public, subscription, user self-service, provider, and admin endpoints
- Error codes and response formats
- Complete workflow examples (raw HTTP and Python client)
- Configuration limits and best practices
- Every request model, response model, admin model, enumeration, and validation rule

---

## 🛠️ Development

The documentation is built with [Zensical](https://zensical.org/).

Install dependencies:

```bash
uv sync
```

Run the documentation locally:

```bash
uv run zensical serve
```

Build the documentation:

```bash
uv run zensical build --clean --strict
```

### Deployment

Documentation is deployed automatically to GitHub Pages on every push to `main` (see `.github/workflows/docs.yml`).

---

## 📄 License

The documentation is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

See [`LICENSE`](LICENSE) for the full license text.
