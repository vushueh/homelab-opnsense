# Phase 4 — Validate Dashboards and Break/Fix

## Goal

Build a usable SIEM view for OPNsense telemetry. Run a break/fix scenario where logs
stop arriving and diagnose and restore the pipeline.

---

## Track A — SIEM Dashboard Validation

[LEONEL]

### 4.1 Confirm All Log Sources Visible in SIEM

| Source | Expected in SIEM | Confirmed |
|--------|-----------------|-----------|
| OPNsense firewall allow/block | Yes | |
| OPNsense system/auth | Yes | |
| Suricata IDS alerts | Yes | |
| DHCP leases | Yes | |
| NetFlow flows | Yes (if collector integrated) | |

### 4.2 Create Saved Searches / Views

In Wazuh / your SIEM, create saved views for:

1. **Firewall Blocks** — `filterlog` + action = block — shows all blocked traffic
2. **IDS Alerts** — `event_type: alert` from Suricata — attack timeline
3. **DHCP Leases** — new device detection — asset visibility
4. **VPN Connections** — WireGuard handshake events

Document query syntax for each in `verification/siem-queries.md`.

### 4.3 Test Event-to-Alert Timeline

1. From Kali VLAN 40 — run nmap scan against VLAN 250
2. In SIEM — verify you can see:
   - Firewall allow rule hit (packet crossed VLAN boundary)
   - Suricata alert for the scan
   - NetFlow entry showing the traffic volume
3. Document the timeline — events should appear within 60 seconds

---

## Break Scenario — Logs Stop Arriving

**Break:** Change the syslog target port to a wrong value (e.g. 5150 instead of 514)
1. OPNsense → System → Settings → Logging → Remote → edit target port
2. Save and apply

**Symptom:**
- SIEM shows no new OPNsense events
- Firewall log live view still works in OPNsense GUI (local logging unaffected)
- Suricata events also stop (if using syslog forwarding path)

**Diagnose:**
```sh
# On OPNsense
grep -R "<SIEM-IP>" /usr/local/etc/syslog-ng.conf /usr/local/etc/syslog-ng.conf.d /var/etc 2>/dev/null
# Shows wrong port

# Try manual netcat test
echo "test" | nc -u <SIEM-IP> 514
# If correct port: response or no error
# If wrong port: connection refused or timeout
```

**Fix:** Correct the port in OPNsense logging target, apply. Verify SIEM receives events within 60 seconds.

---

## Screenshots to Capture

- SIEM dashboard showing OPNsense firewall events
- SIEM view showing Suricata alert from Kali scan
- Break scenario: SIEM shows gap in events during wrong port period
- After fix: events resume in SIEM

## Documentation Checklist

- [ ] All 5 log sources confirmed in SIEM
- [ ] Saved queries documented in siem-queries.md
- [ ] Event-to-alert timeline documented
- [ ] Break/fix scenario completed
- [ ] break-fix-log.md updated with log pipeline failure scenario
