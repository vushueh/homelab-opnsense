# Phase 2 — Automated Config Backup via API

## Goal

Write a script that pulls the OPNsense config backup via API and commits it to a local git repo on a schedule. Provides version-controlled history of all firewall changes.

---

## Backup Script (Python)

```python
#!/usr/bin/env python3
"""
backup-opnsense.py — Pull OPNsense config and commit to git.
Credentials loaded from environment variables — never hardcoded.
"""
import os, subprocess, requests
from datetime import datetime
from pathlib import Path

OPN_HOST   = os.environ["OPN_HOST"]
OPN_KEY    = os.environ["OPN_KEY"]
OPN_SECRET = os.environ["OPN_SECRET"]
BACKUP_DIR = Path("backups/opnsense")
BACKUP_DIR.mkdir(parents=True, exist_ok=True)

# Pull config backup
resp = requests.post(
    f"https://{OPN_HOST}/api/core/backup/download",
    auth=(OPN_KEY, OPN_SECRET),
    verify=False,   # self-signed cert in lab — acceptable
    timeout=30
)
resp.raise_for_status()

# Save with timestamp
date_str  = datetime.now().strftime("%Y%m%d-%H%M")
out_file  = BACKUP_DIR / f"opnsense-config-{date_str}.xml"
out_file.write_bytes(resp.content)
print(f"Saved: {out_file}")

# Git commit
subprocess.run(["git", "add", str(BACKUP_DIR)], check=True)
subprocess.run(["git", "commit", "-m", f"OPNsense backup {date_str}"], check=True)
print("Committed to git.")
```

**IMPORTANT:** The XML backup contains firewall rules, DHCP leases, and VPN keys.
Only commit to a **private** repo. Never push to a public GitHub repo.

---

## Schedule with Cron (on automation host)

```bash
# Run backup daily at 3am
0 3 * * * cd /home/leonel/netauto && source .env && python backup-opnsense.py
```

---

## Verify Backup Content

```bash
# Check XML is valid and non-empty
xmllint --noout backups/opnsense/opnsense-config-*.xml && echo "Valid XML"

# Check file size (should be >10KB for a real config)
ls -lh backups/opnsense/opnsense-config-*.xml | tail -3
```

---

## Documentation Checklist

- [ ] backup-opnsense.py script created and tested
- [ ] Backup XML saved and git-committed (to private repo only)
- [ ] Cron job configured on automation host
- [ ] .gitignore updated — backup XMLs excluded from public repo
- [ ] Script saved to `scripts/backup-opnsense.py` in this repo (sanitized — no credentials)
