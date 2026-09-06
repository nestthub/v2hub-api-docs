# Rate Limiting

The API implements tiered rate limiting:

| Endpoint Type         | Rate Limit | Scope                 |
| --------------------- | ---------- | --------------------- |
| Public (`/sub/*`)     | 3 req/sec  | Per IP                |
| Internal (no token)   | 1 req/sec  | Per IP                |
| Internal (with token) | 3 req/sec  | Per IP                |
| Admin                 | No limit   | IP whitelist required |

**Rate Limit Headers** (returned on all requests):

```http
X-RateLimit-Limit: 3
X-RateLimit-Remaining: 2
X-RateLimit-Reset: 1714234567
```

**429 Response** (rate limit exceeded):

```json
{
  "error": "too_many_requests",
  "message": "Rate limit exceeded",
  "details": {
    "retry_after": 1.5
  }
}
```
