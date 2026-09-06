# Types Reference

Complete reference for all request/response models used by the v2hub API.

## Overview

This document defines all data types, request models, and response models used throughout the API. Types are organized by category and include validation rules.

**API Version**: v1 (`1.1.2`)

## Pages

| Page | Covers |
| --- | --- |
| [Base Types](#base-types) *(this page)* | Shared primitive type aliases used across models |
| [Enumerations](enumerations.md) | `SourceType` and `ProviderAuthorizationStatus` |
| [Request Models](request-models.md) | Every request body model (subscriptions, sources, providers, admin) |
| [Response Models](response-models.md) | Every response model (subscriptions, sources, providers, connections, errors) |
| [Admin Models](admin-models.md) | Admin-only request/response models (users, providers, bans, whitelist, stats) |
| [Validation Rules](validation-rules.md) | Cross-cutting validation rules referenced throughout the other pages |

For the endpoints that use these types, see the [API Reference](../api/index.md).

## Base Types

Common type aliases used throughout the API.

```python
Token = str  # Subscription token (unique identifier)
Hash = str  # SHA-256 hash used for source IDs, provider hashes
Timestamp = datetime  # ISO 8601 formatted datetime
UserID = int  # Positive integer user identifier
```
