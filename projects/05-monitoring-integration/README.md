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
| [1](phases/phase-1-audit-logging.md) | Audit current logging state and choose SIEM target |
| [2](phases/phase-2-syslog-config.md) | Configure remote syslog to SIEM |
| [3](phases/phase-3-suricata-netflow.md) | Add Suricata alert forwarding and NetFlow |
| [4](phases/phase-4-dashboards-and-breakfix.md) | Validate SIEM dashboards and break/fix log pipeline |
| [5](phases/phase-5-document-and-close.md) | Sanitize evidence and close project |

## Cross-Family Links

This project links directly to the SOC stack, Homelab_CCNA monitoring, and Windows Server event forwarding projects.
