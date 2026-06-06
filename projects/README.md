# OPNsense Project Index

This directory contains the OPNsense firewall family projects. Each project should be practical, screenshot-friendly, and connected to a real operational skill.

## Current Projects

| # | Project | Status | Main Skill |
|---|---------|--------|------------|
| 01 | [Baseline Router Deployment](01-baseline-deployment/) | Complete | Deploy OPNsense on Hyper-V in router mode |
| 02 | [VLAN Segmentation](02-vlan-segmentation/) | Complete | Build routed lab VLANs with DHCP, NAT, and firewall rules |
| 03 | [IDS/IPS with Suricata](03-ids-ips-suricata/) | Planned | Detect and block lab attacks using Suricata |
| 04 | [VPN Remote Access](04-vpn-remote-access/) | Planned | Build controlled remote access to lab networks |
| 05 | [Monitoring and SIEM Integration](05-monitoring-integration/) | Planned | Export logs/flows to Wazuh or other monitoring tools |
| 06 | [High Availability Design](06-high-availability/) | Future | Design and test HA requirements when hardware is available |

## Recommended Order

1. Finish Project 03 first because it turns the firewall into a security sensor.
2. Then Project 05, because monitoring makes Suricata and firewall rule behavior visible.
3. Then Project 04, because remote access should be built after logging and alerting exist.
4. Keep Project 06 as design-only until a second firewall instance or host exists.

## Standard Project Folder

```text
project-name/
|-- README.md
|-- phases/
|   |-- phase-1-audit.md
|   |-- phase-2-build.md
|   |-- phase-3-verify.md
|   |-- phase-4-breakfix.md
|   `-- phase-5-document.md
|-- configs/
|-- verification/
`-- troubleshooting/
```

## Standard Phase Pattern

Every phase should say who owns the work:

- **[LEONEL]** GUI clicks, screenshots, physical cabling, console work.
- **[CLAUDE]** live SSH/API execution, final config review, GitHub push.
- **[CODEX]** command drafting, configs, runbooks, troubleshooting logic.

Every project should include:

- audit / current state capture
- build steps
- verification steps
- break/fix challenge
- documentation checklist
- rollback or recovery note

## Security Rules

- Never publish raw OPNsense XML exports unless sanitized.
- Remove VPN private keys, API keys, shared secrets, account hashes, certificates, and public WAN details before committing.
- Do not modify production VLAN 10 or VLAN 20 without a dedicated approved migration project.
- Treat Route10 as the production gateway unless a future project explicitly changes that.
