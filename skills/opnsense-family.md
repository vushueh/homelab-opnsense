# OPNsense Firewall Labs — Family Skill

## Trigger

Use this skill when Leonel says:

- `opnsense project`
- `start opnsense project 03`
- `Suricata`
- `OPNsense VPN`
- `firewall rules`
- `NetFlow`
- `OPNsense monitoring`
- `VLAN 30`, `VLAN 40`, `VLAN 50`, or `VLAN 250`
- `Route10` together with OPNsense
- asks to connect OPNsense with CML, physical Cisco, Windows Server, Proxmox, Hyper-V, Wazuh, Security Onion, or SNML

## Project Purpose

This family turns the existing OPNsense deployment into a structured firewall administration and security operations lab.

The repo must teach real skills:

- firewall rule design;
- routed VLAN segmentation;
- NAT and DHCP behavior;
- IDS/IPS tuning;
- VPN design;
- telemetry export;
- break/fix troubleshooting;
- cross-family integration.

## Current Design

OPNsense is the lab/inspection firewall. Route10 remains the production gateway.

```text
Route10 production gateway
    |-- Production VLANs 10 and 20
    `-- OPNsense WAN
            |
        OPNsense router-mode VM on Hyper-V
            |-- MGMT 192.168.20.254
            |-- VLAN 30 192.168.30.0/24
            |-- VLAN 40 192.168.40.0/24
            |-- VLAN 50 192.168.50.0/24
            `-- VLAN 250 192.168.250.0/24
```

## Safety Rules

- Do not modify production VLAN 10 or VLAN 20 without explicit approval.
- Do not make OPNsense the production gateway unless a future migration project is approved.
- Do not publish raw OPNsense config XML.
- Sanitize all secrets before committing: VPN keys, RADIUS secrets, API keys, certificates, password hashes, and WAN identifiers.
- Do not enable IPS blocking mode until logging-only IDS mode has been verified.
- Do not add WAN port forwards without approval and rollback.

## Workflow

At the start of a session:

1. Read `AGENTS.md`.
2. Read `CLAUDE-REVIEW.md`.
3. Read `CODEX-LOG.md`.
4. Read `projects/README.md`.
5. Read the active project README and phase file.

Every phase should include:

- GUI steps for screenshots;
- command or log verification;
- rollback note;
- documentation checklist;
- break/fix idea when appropriate.

## Current Projects

| # | Project | Status |
|---|---------|--------|
| 01 | Baseline Router Deployment | Complete |
| 02 | VLAN Segmentation | Complete |
| 03 | IDS/IPS with Suricata | Planned |
| 04 | VPN Remote Access | Planned |
| 05 | Monitoring and SIEM Integration | Planned |
| 06 | High Availability Design | Future |

## Cross-Family Integration Map

| Integration | Purpose |
|-------------|---------|
| Homelab_CCNA | Use OPNsense for real firewall/routing behavior beyond CML |
| CML Enterprise Labs | Compare ASAv/CML firewall concepts with OPNsense behavior |
| Windows Server Business Admin | Later use NPS/RADIUS for OPNsense admin auth and event forwarding |
| Proxmox / Hyper-V | Place VMs into VLANs and verify reachability through OPNsense |
| SOC / Wazuh / Security Onion | Export firewall logs, Suricata alerts, DNS/DHCP events, and flows |
| SNML | Keep VirtualBox labs separate but map firewall concepts across both |

## Recommended Next Project

Project 03: IDS/IPS with Suricata.

Reason: it turns OPNsense from a router/firewall into a security sensor, and it gives useful output for the later SOC and monitoring projects.

## Response Standard

When Leonel pastes output or asks what to do next, respond with:

1. what the output means;
2. the likely root cause or current state;
3. exact GUI path or command to run;
4. what good output looks like;
5. what to screenshot or save;
6. rollback or safety warning if needed.
