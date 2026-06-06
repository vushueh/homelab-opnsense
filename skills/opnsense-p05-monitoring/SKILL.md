---
name: opnsense-p05-monitoring
description: OPNsense Project 05 monitoring and SIEM integration workflow for Leonel's homelab. Use when the user says "opnsense p05", "project 05", "OPNsense monitoring", "remote syslog", "Wazuh", "Security Onion", "Suricata forwarding", "NetFlow", "SIEM dashboard", "log pipeline", or asks to start, review, troubleshoot, document, or close the OPNsense monitoring project.
---

# OPNsense P05 Monitoring

## Start Here

Read, in order:

1. `AGENTS.md`
2. `CLAUDE-REVIEW.md`
3. `CODEX-LOG.md`
4. `projects/05-monitoring-integration/README.md`
5. The current phase file under `projects/05-monitoring-integration/phases/`

Use Project 05 to make OPNsense telemetry visible in a SIEM before relying on VPN or IDS events operationally.

## Phase Map

- Phase 1: audit current logging state and choose the SIEM/collector.
- Phase 2: configure remote syslog and prove firewall/system logs arrive.
- Phase 3: add Suricata EVE syslog forwarding and built-in NetFlow export.
- Phase 4: validate dashboards and break/fix a log-pipeline failure.
- Phase 5: sanitize evidence and close the project.

## Safety Rules

- Do not commit SIEM credentials, API keys, raw logs with private data, or screenshots with secrets.
- Do not expose management services to collect logs; OPNsense should send telemetry outward.
- Do not hand-edit generated syslog-ng files as the first approach.
- Keep collector destinations on lab/SIEM networks approved by Leonel.
- Treat VLAN 10 and VLAN 20 as production/management context; observe only, do not modify routing or access.

## Technical Anchors

- Remote syslog GUI path: `System -> Settings -> Logging -> Remote`.
- Firewall log GUI path: `Firewall -> Log Files -> Live View`.
- Modern log check pattern: `tail -n 20 /var/log/filter/latest.log`.
- Suricata EVE path: `/var/log/suricata/eve.json`.
- Prefer built-in Suricata EVE syslog output before custom agents or tail jobs.
- NetFlow path: `Reporting -> NetFlow`; do not use `os-softflowd` for this project.

## Operating Pattern

When guiding Leonel:

1. Identify the source, transport, collector, and expected event.
2. Give GUI-first setup and screenshot steps.
3. Provide SIEM-side validation queries or log checks.
4. Explain expected latency, usually within about 60 seconds for lab validation.
5. Use break/fix to prove local logs continue even when remote forwarding breaks.
6. Document field names and dashboards in `verification/`.

## Closeout

Before marking the project complete:

- Confirm firewall allow/block logs arrive in SIEM.
- Confirm system/auth logs arrive in SIEM.
- Confirm Suricata alerts arrive after controlled Kali traffic.
- Confirm NetFlow or flow dashboard evidence exists.
- Confirm saved SIEM queries are documented.
- Confirm log pipeline break/fix notes exist.
- Append a session entry to `CODEX-LOG.md`.
