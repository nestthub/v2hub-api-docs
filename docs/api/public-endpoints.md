# Public Endpoints

Public endpoints are accessible without authentication (no `API-Token` required).

## Get Resolved Subscription

Retrieve a fully resolved subscription — all sources aggregated, recursively resolved, base64-encoded.

**Endpoint**: `GET /sub/{token}`

**Parameters**:

- `token` (path, required): Subscription token

**Response Headers**:

```http
Content-Type: text/plain; charset=utf-8
profile-title: base64:{base64-encoded description}
profile-update-interval: 12
Content-Disposition: attachment; filename="subscription.txt"
Cache-Control: no-store
```

`profile-title` is always base64-encoded, regardless of whether the underlying description contains non-ASCII characters — decode it the same way you decode the body.

**Response Body**: Base64-encoded proxy configuration lines, newline-joined before encoding (i.e. decode the whole body first, then split on `\n` to get individual configs).

**Example Request**:

```bash
curl https://your-domain.com/sub/abc123xyz456
```

**Example Response** (raw, before you base64-decode the body):

```
dmxlc3M6Ly91dWlkQHNlcnZlcjoxMjM0NT9lbmNyeXB0aW9uPW5vbmUmc2VjdXJpdHk9dGxzJnNu
aT1leGFtcGxlLmNvbSZ0eXBlPXRjcCZoZWFkZXJUeXBlPW5vbmUjTXlTZXJ2ZXIK
```

**Error Responses**:

- `404`: Subscription token not found (`subscription_not_found`)
- `422`: Resolution failed — circular reference (`circular_reference`) or nesting depth exceeded (`nesting_too_deep`)
- `429`: Rate limit exceeded — see [Rate Limiting](rate-limiting.md)

This endpoint is rate-limited at `public_rps` (default 3 req/sec) per client IP.
