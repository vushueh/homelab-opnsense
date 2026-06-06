# Phase 1 — Enable OPNsense API and Test with curl

## Goal

Enable the OPNsense REST API, create an API key pair, and confirm basic API calls work from the automation host before writing any scripts.

---

## Track A — GUI Steps

[LEONEL]

### 1.1 Enable API Access

1. OPNsense → **System → Access → Users**
2. Edit the admin user (or create a dedicated `api-automation` user)
3. Scroll to **API keys** section → click **+** to generate a new key pair
4. Download the key file — contains `key` and `secret`
5. **Store securely — never commit to git**

### 1.2 Verify API is Reachable

From management host (VLAN 10 or Tailscale):

```bash
# Basic API test — get firmware info
curl -k -u "<key>:<secret>" \
  https://192.168.20.254/api/core/firmware/info

# Expected: JSON response with OPNsense version info
```

---

## Track B — API Key Security

[LEONEL]

Store API credentials as environment variables — never hardcode in scripts:

```bash
# Add to ~/.bashrc or use a .env file (never committed)
export OPN_KEY="your-api-key"
export OPN_SECRET="your-api-secret"
export OPN_HOST="192.168.20.254"

# Use in scripts:
curl -k -u "$OPN_KEY:$OPN_SECRET" "https://$OPN_HOST/api/..."
```

Add to `.gitignore`:
```
.env
*.key
*secret*
api-credentials*
```

---

## Useful API Endpoints to Explore

| Endpoint | Purpose |
|----------|---------|
| `GET /api/core/firmware/info` | OPNsense version |
| `GET /api/firewall/filter/searchRule` | List firewall rules |
| `GET /api/dhcpv4/leases/searchLease` | List DHCP leases |
| `GET /api/ids/service/getAlertsLogs` | Suricata alert logs |
| `POST /api/core/backup/download` | Download config backup |

---

## Documentation Checklist

- [ ] API key generated (key/secret stored securely, NOT in repo)
- [ ] curl test successful — JSON response received
- [ ] .gitignore updated to exclude credential files
- [ ] API endpoint reference saved to verification/api-endpoints.md
