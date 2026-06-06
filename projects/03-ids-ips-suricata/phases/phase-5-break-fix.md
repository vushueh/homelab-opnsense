# Phase 5 — Break/Fix Exercise

## Goal

Intentionally misconfigure Suricata or interface scope, diagnose the problem from symptoms,
and restore correct operation. Builds real troubleshooting skill for IDS failures.

---

## Break Scenario A — Wrong Interface Scope

**What to do:**

1. OPNsense → **Services → Intrusion Detection → Administration**
2. Remove VLAN 40 from the selected interfaces
3. Apply changes / restart IDS

**Expected symptom:**
- Kali traffic from VLAN 40 generates NO alerts
- IDS is running but blind to that segment

**Diagnose:**

```
[ ] Check Alerts tab — no new entries despite active Kali scan
[ ] Check Administration tab — VLAN 40 interface NOT in selected list
[ ] Confirm IDS service still running (green status)
[ ] Confirm Kali actually on VLAN 40: ip addr show on Kali VM
```

**Fix:**
Add VLAN 40 back to selected interfaces, apply, restart service. Verify alerts return.

---

## Break Scenario B — IDS Service Stopped

**What to do:**

[CODEX/CLAUDE — after approval]
```sh
# Stop Suricata service directly
service suricata stop
```

**Expected symptom:**
- No alerts generated for any traffic
- OPNsense Alerts tab shows no new entries
- GUI may show service as stopped

**Diagnose:**
```sh
ps aux | grep suricata     # no process
ls -la /var/log/suricata/  # log file timestamp not updating
```

From GUI:
- OPNsense → Services → Intrusion Detection → Administration
- Status shows stopped or error

**Fix:**
Click Start/Apply from GUI. Verify service restarts and alerts resume within 60 seconds.

---

## Break Scenario C — All Rules Disabled

**What to do:**

1. OPNsense → **Services → Intrusion Detection → Download**
2. Disable ALL rule sets (uncheck everything)
3. Apply and sync rules

**Expected symptom:**
- IDS running, no rules loaded, no alerts for any traffic
- Log shows Suricata started but rule count = 0

**Diagnose:**
```sh
grep "rules loaded" /var/log/suricata/suricata.log | tail -3
# Expected: 0 rules
```

**Fix:**
Re-enable ET Open rule sets, sync/apply. Verify rule count returns to expected value and alerts resume.

---

## Documentation

After each scenario, record in `troubleshooting/break-fix-log.md`:

```
## Break Scenario A — YYYY-MM-DD
### Symptom
[what was observed]
### Root cause
[wrong interface scope]
### Fix applied
[steps taken]
### Verification
[alert returned after fix]
```

---

## Screenshots to Capture

- Scenario A: Alerts tab empty during Kali scan
- Scenario A: Administration tab showing VLAN 40 missing
- After fix: Alert appearing again
- Scenario B: Service stopped state in GUI
- Scenario C: Rule count = 0 in log

## Documentation Checklist

- [ ] All 3 scenarios completed
- [ ] Each symptom, root cause, and fix documented
- [ ] break-fix-log.md committed to troubleshooting/
