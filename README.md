# OPNsense Firewall Labs

> A structured OPNsense firewall lab family for Hyper-V, VLAN segmentation, IDS/IPS, VPN, monitoring, and cross-family homelab integration.

[![Status](https://img.shields.io/badge/Status-Active-blue)]()
[![Platform](https://img.shields.io/badge/Platform-Hyper--V-blue)]()
[![OPNsense](https://img.shields.io/badge/OPNsense-25.7-orange)]()

## Purpose

This repo is the OPNsense firewall family for Leonel's homelab portfolio. It documents the working OPNsense deployment, then turns it into a project series that can be repeated, tested, broken, fixed, and connected to the other lab families.

OPNsense is used as the inspection and lab firewall. It does not replace the production Route10 gateway. The design keeps production networks separate while giving the lab a real routed firewall, VLAN boundaries, DHCP, NAT, IDS/IPS, VPN, logging, and monitoring.

## Quick Links

- [Workflow](WORKFLOW.md)
- [Agent Rules](AGENTS.md)
- [Open Review Items](CLAUDE-REVIEW.md)
- [Codex Log](CODEX-LOG.md)
- [Project Index](projects/README.md)
- [Family Skill](skills/opnsense-family.md)
- [Cross-Family Integration](docs/cross-family-integration.md)
- [Troubleshooting](troubleshooting/)

## Current Architecture

```text
Internet
    |
Alta Labs Route10
    |-- Production networks: VLAN 10 / VLAN 20
    |
    |-- OPNsense WAN: 192.168.10.x
            |
        OPNsense VM on Hyper-V
            |-- MGMT: 192.168.20.254
            |-- VLAN 30: 192.168.30.1/24   Blue team / services
            |-- VLAN 40: 192.168.40.1/24   Kali / attacker tools
            |-- VLAN 50: 192.168.50.1/24   Reserved lab segment
            |-- VLAN 250: 192.168.250.1/24 Vulnerable lab
            |
        Cisco Catalyst 2960G / Hyper-V vSwitch / lab VMs
```

## Family Project Map

| # | Project | Status | Purpose |
|---|---------|--------|---------|
| 01 | [Baseline Router Deployment](projects/01-baseline-deployment/) | Complete | Preserve the bridge-mode failure analysis and final router-mode success |
| 02 | [VLAN Segmentation](projects/02-vlan-segmentation/) | Complete | Document VLANs, DHCP, routing, firewall rules, and lab segmentation |
| 03 | [IDS/IPS with Suricata](projects/03-ids-ips-suricata/) | Planned | Turn OPNsense into a detection and prevention sensor for lab traffic |
| 04 | [VPN Remote Access](projects/04-vpn-remote-access/) | Planned | Build controlled remote access into lab networks |
| 05 | [Monitoring and SIEM Integration](projects/05-monitoring-integration/) | Planned | Send logs, NetFlow, and firewall telemetry to Wazuh/SOC tooling |
| 06 | [High Availability Design](projects/06-high-availability/) | Future | Design CARP/pfsync HA requirements and failover testing |
| 07 | [DNS and DHCP Services](projects/07-dns-dhcp-services/) | Future | Harden DNS/DHCP design and document resolver behavior |
| 08 | [Policy Automation](projects/08-policy-automation/) | Future | Explore API/export workflows, backups, and repeatable change control |
| 09 | [Cross-Family Firewall Capstone](projects/09-cross-family-firewall-capstone/) | Future | Connect CML, Physical CCNA, Windows AD, SOC, and OPNsense policies |

## Standard Project Structure

Each project should follow the same shape used by the other homelab families:

```text
projects/project-name/
|-- README.md              # Objectives, topology, status, phase index
|-- phases/                # Step-by-step phase guides
|-- configs/               # Sanitized config exports, snippets, templates
|-- verification/          # Test outputs, screenshots, evidence
`-- troubleshooting/       # Break/fix notes and recovery steps
```

## Cross-Family Role

OPNsense is not isolated. It is the firewall and routing lab that connects several families:

| Family | OPNsense Link |
|--------|---------------|
| Homelab_CCNA | VLAN routing, firewall policy, syslog, NetFlow, Wireshark validation |
| CML Enterprise Labs | External connectivity, NAT/ACL comparisons, future hybrid routing |
| Windows Server Business Admin | Future RADIUS/NPS authentication and log forwarding |
| Proxmox / Hyper-V Management | VM network placement, VLAN reachability, firewall access controls |
| SOC / Blue Team | Suricata, firewall logs, DNS logs, Wazuh/SIEM telemetry |
| SNML VirtualBox | Parallel firewall/security concepts without merging platforms |

## Key Design Principles

- Production VLANs stay safe. Do not make OPNsense the dependency for the home production path until a dedicated migration project exists.
- Router mode is the working architecture. Bridge mode is documented as a lesson learned, not a current goal.
- Every project needs a GUI path, a verification path, a rollback note, and a break/fix exercise.
- Sensitive exports must be sanitized before committing. Do not publish private keys, VPN secrets, RADIUS secrets, API keys, or full unredacted config XML.
- OPNsense changes should be documented before and after, with screenshots or command evidence.

## Status

The repo has been converted into a family-project layout. Projects 01 and 02 preserve the existing completed work. Projects 03 through 06 now have starter folders and phase guides so they can be completed one at a time.

**Trigger phrase:** `opnsense project`
