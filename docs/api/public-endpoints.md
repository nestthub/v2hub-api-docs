# Public Endpoints

Public endpoints are accessible without authentication.

## Get Resolved Subscription

Retrieve a fully resolved subscription with all configs aggregated.

**Endpoint**: `GET /sub/{token}`

**Parameters**:

- `token` (path, required): Subscription token

**Response Headers**:

```http
Content-Type: text/plain; charset=utf-8
profile-title: base64:{encoded_description}
profile-update-interval: 12
Content-Disposition: attachment; filename="subscription.txt"
Cache-Control: no-store
```

**Response Body**: Base64-encoded proxy configurations (one per line)

**Example Request**:

```bash
curl https://v2hub.link/sub/abc123xyz456
```

**Example Response**:

```
dmxlc3M6Ly91dWlkQHNlcnZlcjoxMjM0NT9lbmNyeXB0aW9uPW5vbmUmc2VjdXJpdHk9dGxzJnNu
aT1leGFtcGxlLmNvbSZ0eXBlPXRjcCZoZWFkZXJUeXBlPW5vbmUjTXlTZXJ2ZXIK
...
```

**Error Responses**:

- `404 Not Found`: Subscription token not found
- `500 Internal Server Error`: Resolution failed (circular reference, nesting too deep, etc.)
