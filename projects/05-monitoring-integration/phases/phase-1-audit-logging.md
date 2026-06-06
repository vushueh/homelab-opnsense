# Phase 1 — Audit Current Logging State

## Goal

Document what OPNsense is currently logging, where logs go, and what is missing before configuring any external targets.

---

## Track A — GUI Steps

[LEONEL]

1. OPNsense → **System → Log Files → General** — review current log retention settings
2. OPNsense → **System → Settings → Logging** — review local log settings and check the **Remote** tab for existing syslog destinations
3. OPNsense → **Firewall → Log Files → Live View** — confirm firewall logs are generating
4. OPNsense → **Services → Intrusion Detection → Alerts** — confirm Suricata alert logs exist (from P03)
5. OPNsense → **Services → DHCPv4 → Leases** — confirm DHCP logs present
6. Note OPNsense version: **System → Firmware → Status**

Document in `verification/pre-monitoring-audit.md`:
- Current syslog destination (local only, or existing remote?)
- Firewall log format (JSON vs syslog)
- Suricata alert log location
- OPNsense version (affects config paths and features)

---

## Target SIEM/Monitoring Tool

Choose before proceeding to Phase 2:

| Option | Notes |
|--------|-------|
| Wazuh | Already in Blue Team homelab — preferred for cross-family integration |
| Security Onion | Full SOC stack — heavier setup |
| Graylog | Syslog collector with search — lighter weight |
| Elastic / ELK | Flexible but complex to configure from scratch |

**Recommended:** Wazuh (already deployed in homelab, native syslog ingestion, Windows Server integration in P10).

Document choice and target IP in README.md.

---

## Log Source Priority

| Log Source | Priority | Notes |
|------------|----------|-------|
| Firewall rules (allow/block) | High | Core security telemetry |
| Suricata IDS alerts | High | From P03 |
| DHCP leases | Medium | Asset visibility |
| DNS queries | Medium | Lateral movement detection |
| VPN connections | Medium | From P04 |
| System/auth logs | Low | OPNsense admin logins |

---

## Documentation Checklist

- [ ] Current logging state documented
- [ ] Target SIEM/monitoring tool chosen
- [ ] SIEM IP and port confirmed reachable from OPNsense MGMT interface
- [ ] pre-monitoring-audit.md saved
