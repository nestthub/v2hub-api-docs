# Validation Rules

Cross-cutting validation rules referenced throughout the [Request Models](request-models.md) and [Admin Models](admin-models.md) pages.

## String Fields

- Leading/trailing whitespace is stripped before length validation is applied.
- Empty strings (after stripping) are rejected wherever a field is documented as "non-empty" or has a minimum length of 1.

## Subscription Name

- 1-64 characters after stripping.
- Must be unique per user (or per provider-managed user, for provider-created subscriptions) — a duplicate raises `duplicate_name` (`409 Conflict`).

## Subscription / Config Description & Comment

- `description` (on subscriptions): max 64 characters.
- `comment` (on individual sources, via [`SourceUpdateRequest`](request-models.md#sourceupdaterequest) / the deprecated [`CommentUpdateRequest`](request-models.md#commentupdaterequest-deprecated)): max 256 characters. Do not include a leading `#` — it is added automatically when the config is resolved.

## Sources List

- Maximum 150 items per subscription (`MAX_SOURCES_PER_SUB`).
- Duplicate entries (compared by stripped `data` value) are silently deduplicated, keeping the first occurrence, rather than rejected — this applies when creating a subscription with initial sources and when adding sources to an existing one.
- An empty array is only accepted by the *replace* endpoint (`PUT /api/v1/subs/{token}/sources`), where it clears all sources; it is not accepted by the *create* or *add* endpoints as a meaningful no-op.

## `max_depth`

- Integer, 0-3 inclusive.
- Governs how many additional levels of nested `internal_token` references are followed when resolving that specific source. `0` means the source itself is resolved but no further nested internal tokens beneath it are followed.
- Independent of `MAX_NESTING_DEPTH`, the server-wide ceiling — `max_depth` can only narrow resolution for a given source, never widen it beyond what the server otherwise permits.

## IP Addresses

- `ip_address` on ban endpoints ([Ban Request](admin-models.md#banrequest)) must be a valid IPv4 or IPv6 address (a single host, not a range).
- `ip_address` on whitelist endpoints ([Whitelist Request](admin-models.md#whitelistrequest)) accepts either a single address or a CIDR range (e.g. `10.0.0.0/24`).

## User / Provider Identifiers

- `user_id`: positive integer (`> 0`).
- `owner_hash`, `provider_hash`: opaque, server-generated hash strings — never constructed or guessed client-side; always obtained from a prior API response.

## Circular References & Nesting Depth

- The server detects circular references among `internal_token` sources automatically (e.g. subscription A referencing B, which references A again) and rejects the operation that would introduce the cycle with `circular_reference`, rather than allowing it to be created and failing later at resolution time.
- Exceeding the server's configured `MAX_NESTING_DEPTH` during resolution (independent of any single source's `max_depth`) results in `nesting_too_deep`.

## HMAC Signature & Timestamp (Admin)

- The `X-Timestamp` header must be a Unix timestamp in **milliseconds**.
- A request is only accepted if its timestamp is within the server's configured window (default ±60 seconds) of the server's current time — this bounds replay-attack exposure but requires reasonably synchronized clocks between client and server.
- The signature payload is the concatenation `{timestamp}{method}{path}{body}` (no separators), HMAC-SHA256'd with the admin secret key — see [Authentication → Admin Authentication](../api/authentication.md#admin-authentication) for a worked example.
