# Phase 4 — Push a Change via API Script

## Goal

Use the OPNsense API to create a firewall alias (IP group) programmatically. Verify the alias appears in the GUI. Practice the read → change → apply → verify workflow.

---

## Safe Change: Create a Firewall Alias

An alias is low-risk — it holds IP addresses/networks and can be referenced in rules.
Creating one via API has no immediate traffic impact.

```python
#!/usr/bin/env python3
"""
create-alias.py — Create a firewall alias via OPNsense API.
"""
import os, json, requests

OPN_HOST   = os.environ["OPN_HOST"]
OPN_KEY    = os.environ["OPN_KEY"]
OPN_SECRET = os.environ["OPN_SECRET"]

alias_data = {
    "alias": {
        "enabled":     "1",
        "name":        "Lab-Kali-Hosts",
        "type":        "host",
        "content":     "192.168.40.10\n192.168.40.11",
        "description": "Kali lab VMs on VLAN 40 — created by API script"
    }
}

# Create alias
resp = requests.post(
    f"https://{OPN_HOST}/api/firewall/alias/addItem",
    auth=(OPN_KEY, OPN_SECRET),
    json=alias_data,
    verify=False,
    timeout=30
)
print("Create response:", resp.status_code, resp.json())

# Apply changes
apply = requests.post(
    f"https://{OPN_HOST}/api/firewall/alias/reconfigure",
    auth=(OPN_KEY, OPN_SECRET),
    verify=False,
    timeout=30
)
print("Apply response:", apply.status_code, apply.json())
```

---

## Verify in GUI

1. OPNsense → **Firewall → Aliases**
2. Confirm `Lab-Kali-Hosts` alias appears with correct IPs
3. Screenshot for evidence

---

## Cleanup After Test

```python
# Delete the test alias via API after verification
resp = requests.post(
    f"https://{OPN_HOST}/api/firewall/alias/delItem/<uuid>",
    auth=(OPN_KEY, OPN_SECRET),
    verify=False
)
# Then reconfigure to apply deletion
```

---

## Documentation Checklist

- [ ] create-alias.py script tested successfully
- [ ] Alias visible in OPNsense GUI after apply
- [ ] Test alias cleaned up after verification
- [ ] Script saved to `scripts/create-alias.py` (sanitized)
- [ ] Change control diff run before/after alias creation
