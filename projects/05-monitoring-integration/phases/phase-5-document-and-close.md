# Phase 5 — Document and Project Close

## Goal

Complete STAR summary, sanitize all evidence, update project status, hand off to Claude for GitHub push.

---

## Sanitization Checklist

```
[ ] No SIEM credentials in committed files
[ ] No OPNsense admin credentials in logs or screenshots
[ ] siem-queries.md references only log field names — no raw private data
[ ] NetFlow config doc does not contain collector credentials
[ ] Screenshots do not show sensitive IP ranges beyond lab scope
```

---

## Evidence to Commit

```
verification/
  pre-monitoring-audit.md       ← Phase 1 findings
  log-format-notes.md           ← OPNsense syslog format documented
  suricata-forwarding-method.md ← which method used to forward alerts
  siem-queries.md               ← saved SIEM queries for each log source
  screenshots/                  ← dashboard and event screenshots

troubleshooting/
  break-fix-log.md              ← log pipeline failure scenario

phases/phase-1 through phase-5  ← all present
```

---

## STAR Summary — Complete in README.md

```
Action:
  - Audited existing OPNsense logging (local only, no remote target)
  - Chose Wazuh as SIEM target (already deployed in Blue Team homelab)
  - Configured remote syslog forwarding (UDP 514 / TCP 1514 to Wazuh)
  - Confirmed firewall allow/block, system, and DHCP logs arriving
  - Enabled Suricata EVE syslog forwarding from Project 03 alert pipeline
  - Configured built-in OPNsense NetFlow v9 exporter to SIEM collector
  - Built saved SIEM queries for blocks, IDS alerts, DHCP, VPN
  - Ran break/fix: log pipeline failure (wrong port) — diagnosed and restored

Result:
  - OPNsense firewall, IDS, DHCP, and NetFlow all visible in single SIEM
  - Attack timeline provable: Kali scan → firewall log → Suricata alert → NetFlow
  - Break/fix proves ability to diagnose syslog pipeline failures
  - Cross-family integration: OPNsense telemetry joins Windows Server events in Wazuh
  - Foundation complete for Project 06 (High Availability — design only)
```

---

## Cross-Family Note

This project creates the logging foundation for:
- **Windows Server P10** — Windows event logs join OPNsense logs in same Wazuh instance
- **Homelab_CCNA P05** — physical Cisco syslog can target same SIEM
- **SOC/Blue Team** — unified view of east-west (OPNsense) + north-south (Windows) telemetry

---

## Documentation Checklist

- [ ] Sanitization checklist complete
- [ ] STAR summary written in README.md
- [ ] README.md status updated to ✅ Complete
- [ ] All phase files present (1–5)
- [ ] All verification files committed
- [ ] Tell Claude → Claude pushes to GitHub
