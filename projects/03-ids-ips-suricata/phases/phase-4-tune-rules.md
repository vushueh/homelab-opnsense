# Phase 4 — Tune Rules and Reduce Noise

## Goal

Review the alerts generated in Phase 3. Suppress false positives, document chosen rule sets, and establish a baseline noise level before considering IPS blocking mode.

---

## Track A — GUI Steps

[LEONEL]

### 4.1 Review Enabled Rule Sets

1. OPNsense → **Services → Intrusion Detection → Download**
2. Document which rule sets are enabled (ET Open, abuse.ch, etc.)
3. Note total rule count

### 4.2 Identify Noisy Rules

1. OPNsense → **Services → Intrusion Detection → Alerts**
2. Sort by count or rule name
3. Identify any rules firing on non-lab traffic (DNS lookups, normal browsing, etc.)
4. Record SIDs of noisy rules

### 4.3 Suppress False Positives

For each noisy rule that is not security-relevant, use the OPNsense IDS policy workflow. In OPNsense 24.x, routine tuning is done from **Policy** for bulk rule actions and **Rules** for manual per-SID changes; do not assume a separate suppression-list menu exists.

1. OPNsense → **Services → Intrusion Detection → Policy**
2. Create or edit a policy that matches the noisy SID, rule category, or metadata
3. Set the policy action to disable the rule or keep it alert-only; do not set noisy rules to drop
4. For a single SID, use **Services → Intrusion Detection → Rules**, search the SID, and apply a manual disable/action change
5. Document the SID, action, and reason in `verification/rule-tuning-notes.md`

If a true Suricata threshold/suppress rule is needed by source/destination IP pair, treat it as advanced custom configuration and send it to Claude for review before applying it live.

### 4.4 Verify Suppression Works

1. Regenerate the traffic that caused the false positive
2. Confirm the alert no longer appears (suppression active)
3. Confirm legitimate attack signatures still fire (Phase 3 nmap/curl still triggers alerts)

---

## Track B — CLI Rule Analysis

[CODEX/CLAUDE — after approval]

```sh
# Count alerts per SID — find noisiest rules
grep '"event_type":"alert"' /var/log/suricata/eve.json | \
  python3 -c "
import sys, json, collections
sids = collections.Counter()
for l in sys.stdin:
  try:
    d = json.loads(l)
    sig = d.get('alert',{}).get('signature_id','?')
    sids[sig] += 1
  except: pass
for sid,count in sids.most_common(20): print(count, sid)
"

# View full alert detail for specific SID
grep '"signature_id":2001219' /var/log/suricata/eve.json | head -1 | python3 -m json.tool
```

---

## Rule Set Documentation Template

Fill this in for the repo:

| Rule Set | Enabled | Reason | Alert Count (24h) |
|----------|---------|--------|-------------------|
| ET Open — emerging-scan | Yes | Detects nmap, scanning | |
| ET Open — emerging-web-server | Yes | SQLi, web attacks | |
| abuse.ch — botcc | Yes | C2 communication detection | |
| ET Open — emerging-dns | No | Too noisy for lab DNS traffic | |

---

## What Good Looks Like

- False positive rate reduced — known benign traffic no longer fires alerts
- Security-relevant rules still active and confirmed working
- Rule set choices documented with reasoning
- Suppression list documented

---

## Screenshots to Capture

- Rule sets enabled list (Download tab)
- Suppression list with entries
- Alert page after tuning — reduced noise visible
- Before/after alert count comparison

## Documentation Checklist

- [ ] Enabled rule sets documented with reasoning
- [ ] Noisy SIDs identified and suppressed
- [ ] Suppression tested — false positives no longer appear
- [ ] Security rules still trigger on Kali test traffic
- [ ] Rule tuning notes saved to `verification/rule-tuning-notes.md`
