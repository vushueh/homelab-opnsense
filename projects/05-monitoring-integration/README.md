# Project 05 — Monitoring and SIEM Integration

## Status

Planned.

## Purpose

Send OPNsense telemetry to monitoring and security platforms so firewall decisions, DHCP/DNS behavior, Suricata alerts, and traffic flows can be investigated from one place.

## Target Integrations

- remote syslog
- firewall logs
- Suricata alert logs
- NetFlow / flow telemetry where practical
- Wazuh, Security Onion, or another SOC target
- dashboard screenshots for evidence

## Skills Practiced

- log source configuration
- SIEM ingestion validation
- event field interpretation
- firewall rule hit correlation
- flow visibility
- alert-to-traffic timeline building

## Phase Index

| Phase | Goal |
|-------|------|
| 1 | Audit current logging and monitoring targets |
| 2 | Configure remote syslog safely |
| 3 | Send firewall and DHCP/DNS logs |
| 4 | Add Suricata alert forwarding after Project 03 |
| 5 | Validate events in SIEM/monitoring tool |
| 6 | Break/fix: logs stop arriving |
| 7 | Document dashboards and runbook |

## Cross-Family Links

This project links directly to the SOC stack, Homelab_CCNA monitoring, and Windows Server event forwarding projects.
