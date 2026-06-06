# Phase 3 — Firewall Rule Export and Diff

## Goal

Export the current firewall rule set to JSON via API. Compare snapshots before and after a change to build a diff-based change-control habit.

---

## Export Script (Python)

```python
#!/usr/bin/env python3
"""
export-rules.py — Export OPNsense firewall rules to JSON.
"""
import os, json, requests
from datetime import datetime
from pathlib import Path

OPN_HOST   = os.environ["OPN_HOST"]
OPN_KEY    = os.environ["OPN_KEY"]
OPN_SECRET = os.environ["OPN_SECRET"]
OUT_DIR    = Path("exports/firewall-rules")
OUT_DIR.mkdir(parents=True, exist_ok=True)

resp = requests.get(
    f"https://{OPN_HOST}/api/firewall/filter/searchRule",
    auth=(OPN_KEY, OPN_SECRET),
    verify=False,
    timeout=30
)
resp.raise_for_status()

date_str = datetime.now().strftime("%Y%m%d-%H%M")
out_file = OUT_DIR / f"rules-{date_str}.json"
out_file.write_text(json.dumps(resp.json(), indent=2))
print(f"Exported {len(resp.json().get('rows', []))} rules to {out_file}")
```

---

## Diff Between Snapshots

```bash
# After making a rule change in GUI, export again then diff
python export-rules.py

# Compare two exports
diff exports/firewall-rules/rules-20260606-0900.json \
     exports/firewall-rules/rules-20260606-1000.json

# Or use jq for cleaner output
jq '.rows[] | {description, action, interface}' exports/firewall-rules/rules-20260606-1000.json
```

---

## Change Control Workflow

```
1. Export current rules → save as "before" snapshot
2. Make change in GUI
3. Export again → save as "after" snapshot
4. Run diff — confirm only intended change appears
5. Commit both snapshots to git with descriptive message
```

---

## Documentation Checklist

- [ ] export-rules.py script tested — JSON output saved
- [ ] Before/after diff run for at least one rule change
- [ ] Change control workflow documented in verification/change-control-workflow.md
- [ ] Script saved to `scripts/export-rules.py` (sanitized)
