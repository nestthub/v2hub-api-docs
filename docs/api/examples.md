# Examples

Complete workflow examples using raw HTTP requests, plus equivalents using the official Python client libraries.

## Complete Workflow Example

### 1. Create a Subscription

```bash
curl -X POST https://v2hub.link/api/v1/subs \
  -H "API-Token: your_token_here" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My VPN",
    "sources": ["vless://uuid@server:port#Server1"]
  }'
```

### 2. Add More Sources

```bash
curl -X POST https://v2hub.link/api/v1/subs/abc123/sources \
  -H "API-Token: your_token_here" \
  -H "Content-Type: application/json" \
  -d '{
    "sources": ["https://provider.com/subscription"]
  }'
```

### 3. Get Resolved Subscription (Public)

```bash
curl https://v2hub.link/sub/abc123
```

### 4. Update Config Comment

```bash
curl -X PATCH https://v2hub.link/api/v1/subs/abc123/config \
  -H "API-Token: your_token_here" \
  -H "Content-Type: application/json" \
  -d '{
    "config_id": "hash1",
    "comment": "My Custom Server"
  }'
```

## Provider Integration Example

### 1. Request Access to User

```bash
curl -X POST https://v2hub.link/api/v1/providers/12345 \
  -H "API-Token: provider_token_here"
```

### 2. User Approves (from user's side)

```bash
curl -X POST https://v2hub.link/api/v1/me/providers/MyProvider/approve \
  -H "API-Token: user_token_here"
```

### 3. Provider Manages User's Subscription

```bash
curl -X POST https://v2hub.link/api/v1/providers/12345/subs \
  -H "API-Token: provider_token_here" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Provider-Managed VPN",
    "sources": ["vless://uuid@server:port#Server1"]
  }'
```

## Python Client Example

Using the official [`v2hub`](https://pypi.org/project/v2hub/) client library instead of raw HTTP requests:

```python
from v2hub import AsyncVPNClient

async with AsyncVPNClient(
    base_url="https://v2hub.link",
    api_token="your_token_here",
) as client:
    # Create subscription
    sub = await client.create_subscription(
        "My VPN",
        sources=["vless://uuid@server:port#Server1"],
    )

    # Add sources
    sub = await client.add_sources(sub.token, ["https://provider.com/subscription"])

    # Get resolved subscription
    public = await client.get_public_subscription(sub.token)
    print(public.decode())
```

See the [v2hub client documentation](https://v2hub.dev) for the full client reference, including the synchronous client, error handling, retries, and provider workflows.

## Admin Client Example

Using the official [`v2hub-admin`](https://pypi.org/project/v2hub-admin/) library for HMAC-signed administrative operations:

```python
from v2hub_admin import AsyncAdminClient

async with AsyncAdminClient(
    base_url="https://v2hub.link",
    secret_key="your-hmac-secret",
) as admin:
    user = await admin.create_user(user_id=12345)
    print(user.api_token)

    await admin.ban_ip("192.168.1.100", duration_seconds=3600)
```

See the [v2hub Admin documentation](https://v2hub.dev) for the full admin client reference.
