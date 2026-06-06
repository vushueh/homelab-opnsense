# Phase 6 — IPS Blocking Mode Decision and Project Close

## Goal

Evaluate whether enabling IPS blocking mode is appropriate for this lab. Document the decision
with reasoning. If enabled, run a controlled blocking test and verify rollback works.
Complete STAR summary and save all evidence.

---

## IPS Decision Framework

Answer each question before enabling blocking mode:

| Question | Answer before enabling IPS |
|----------|---------------------------|
| Are suppression rules tuned? (Phase 4 complete?) | Must be Yes |
| Do you have console/Hyper-V access if GUI locks out? | Must be Yes |
| Is a config backup from today available? | Must be Yes |
| Is MGMT (VLAN 20) excluded from IPS scope? | Must be Yes |
| Is WAN excluded from IPS scope? | Must be Yes |
| Are test VLANs (30, 40, 250) isolated from production? | Must be Yes |

**If any answer is No → do not enable IPS blocking. Document "Deferred — reason" and close the project.**

---

## Track A — If Enabling IPS (all checks passed)

[LEONEL]

1. OPNsense → **Services → Intrusion Detection → Administration**
2. Enable **IPS mode** (blocking)
3. Confirm scope is VLAN 40 and VLAN 250 only — NOT MGMT or WAN
4. Apply changes

**Controlled blocking test:**

From Kali VLAN 40 — run the same nmap scan used in Phase 3:
```bash
nmap -sV 192.168.250.0/24
```

**Expected with IPS active:**
- Scan either times out or receives resets faster than before
- OPNsense Alerts tab shows **drop** action (not just alert)
- Kali may see filtered/closed ports it previously saw open

**Verify blocking in logs:**
```sh
grep '"action":"blocked"' /var/log/suricata/eve.json | tail -5
```

**Verify management access not affected:**
- OPNsense GUI still reachable at 192.168.20.254
- SSH to OPNsense still works

---

## IPS Rollback

If blocking mode causes unexpected behavior (management lockout, production impact):

**Option A — GUI (if reachable):**
- Services → Intrusion Detection → Administration → disable IPS mode → apply

**Option B — Hyper-V Console:**
- Open OPNsense VM console
- Log in as root
- `pluginctl -s suricata stop`
- Restore config backup from System → Configuration → Backups

---

## STAR Summary — Complete in README.md

```
Action:
  - Installed and enabled Suricata in IDS alert-only mode on lab VLANs 40 and 250
  - Generated controlled test traffic from Kali to confirm alert generation
  - Tuned rule sets to suppress false positives
  - Completed 3 break/fix scenarios (interface scope, service stop, rule disable)
  - [If IPS enabled] Enabled IPS blocking mode with controlled verification
  - [If deferred] Documented IPS deferral with reason

Result:
  - OPNsense now detects and logs attacks crossing lab VLANs
  - Alert pipeline verified with real Kali traffic
  - Tuned rule set documented for future reference
  - Break/fix scenarios prove ability to diagnose IDS failures
  - Foundation ready for Project 05 (SIEM integration of Suricata alerts)
```

---

## Final Evidence Checklist

```
[ ] phases/phase-1 through phase-6 all complete
[ ] verification/ contains: rule-tuning-notes.md, alert-screenshots
[ ] troubleshooting/break-fix-log.md has all 3 scenarios
[ ] configs/ has OPNsense backup from before project start
[ ] IPS decision documented (enabled or deferred with reason)
[ ] README.md status updated to ✅ Complete
[ ] Tell Claude → Claude pushes to GitHub
```
