# Phase 2 — Configure Remote Syslog

## Goal

Send OPNsense system and firewall logs to the remote SIEM/monitoring target via syslog.
Verify logs are arriving before adding more sources.

---

## Track A — GUI Steps

[LEONEL]

### 2.1 Configure Remote Syslog Target

1. OPNsense → **System → Settings → Logging → Remote**
2. Click **+** to add a destination
3. Configure:
   - **Transport:** UDP (standard) or TCP (reliable)
   - **Hostname:** `<SIEM-IP>` (Wazuh manager IP or syslog collector IP)
   - **Port:** `514` (standard syslog) or `1514` (Wazuh)
   - **Levels:** Informational and above, or leave empty for all during the first test
   - **Facilities:** Leave empty for all during the first test
   - **Applications:** Leave empty for all during the first test; narrow later if needed
   - **Description:** `Wazuh-SIEM`
4. Save and apply

### 2.2 Enable Firewall Log Forwarding

1. OPNsense → **Firewall → Settings → Advanced**
2. Ensure **Log firewall default rules** is enabled
3. Check **Log packets matched from the default pass rules** if needed

### 2.3 Test with a Known Event

Trigger a log event to confirm receipt at SIEM:

1. Log into OPNsense as admin (generates auth log event)
2. Trigger a firewall rule hit: ping from VLAN 40 to VLAN 250
3. Check SIEM immediately for incoming events

---

## Track B — CLI Verification

[CODEX/CLAUDE — after approval]

```sh
# Confirm syslog service is sending to remote target
grep -R "<SIEM-IP>" /usr/local/etc/syslog-ng.conf /usr/local/etc/syslog-ng.conf.d /var/etc 2>/dev/null

# Tail OPNsense local log to see what's being generated
tail -n 20 /var/log/filter/latest.log  # firewall log
tail -n 20 /var/log/system/latest.log  # system log
```

**On SIEM/Wazuh side:**
```bash
# Confirm events arriving at Wazuh manager
tail -f /var/ossec/logs/archives/archives.log | grep "opnsense\|192.168.20.254"
```

---

## Common Syslog Formats from OPNsense

OPNsense firewall logs arrive in this format:
```
<134>1 2026-06-06T10:23:45Z opnsense.chongong.local filterlog - - - 4,,,0,igb0.30,match,block,in,4,0x0,,64,12345,0,none,6,tcp,...
```

Key fields:
- `filterlog` = firewall log
- `match,block,in` = action, direction
- `igb0.30` = interface (VLAN 30)
- Source/dest IP and port follow

Note: older OPNsense references may mention `clog /var/log/filter.log`. Current OPNsense uses regular log files with `latest.log` symlinks under `/var/log/<application>/`; use `tail` unless this firewall is confirmed to be on an older circular-log format.

---

## Screenshots to Capture

- OPNsense Logging Targets page with remote SIEM configured
- SIEM dashboard or log view showing OPNsense events arriving
- Firewall log entry visible in SIEM with correct source IP

## Documentation Checklist

- [ ] Remote syslog target configured (SIEM IP, port, transport)
- [ ] Firewall log forwarding enabled
- [ ] Test event triggered and confirmed received at SIEM
- [ ] Syslog format documented in verification/log-format-notes.md
