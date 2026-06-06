# Project 09 — Cross-Family Firewall Capstone

## Status

Planned. Requires Projects 01–08 complete. Coordinates with:
- Windows Server P13 (Enterprise Identity Integration)
- Homelab_CCNA P04 (Physical AAA)
- SOC / Blue Team family

## Purpose

Prove that OPNsense is the unified security enforcement point for the entire homelab.
Connect CML virtual networks, physical Cisco gear, Windows AD identity, and SOC telemetry
through OPNsense firewall policy. Demonstrate that a single firewall policy change
can be observed across all lab families simultaneously.

## What Gets Integrated

| Integration | How |
|------------|-----|
| CML virtual routers | VLAN 150 bridge (from Homelab_CCNA P01) — OPNsense enforces inter-VLAN policy |
| Physical Cisco gear | OPNsense Route10 is the default gateway — all physical traffic passes through |
| Windows AD RADIUS | OPNsense admin auth → NPS/RADIUS → AD group (Windows Server P13) |
| Wazuh SOC | OPNsense Suricata alerts + firewall logs flowing to Wazuh (P05) |
| VPN clients | WireGuard VPN users subject to same firewall policy as LAN users (P04) |

## Capstone Test Scenario

One change — block VLAN 40 (Kali) from reaching VLAN 250 (vulnerable lab) via OPNsense rule — and observe:

1. Kali can no longer reach VLAN 250 targets
2. Suricata (P03) logs the blocked attempt
3. Wazuh (P05) receives and alerts on the block event
4. Windows AD audit log (P13) shows no auth bypass attempted
5. Physical Cisco gear is unaffected (separate VLAN plane)
6. VPN client subject to same block (VPN_WG interface rule)

## Phase Index

| Phase | Goal |
|-------|------|
| [1](phases/phase-1-readiness-audit.md) | Verify all P01-P08 dependencies met; document integration readiness |
| [2](phases/phase-2-ad-radius-auth.md) | Configure OPNsense admin auth via Windows NPS/RADIUS |
| [3](phases/phase-3-cross-vlan-policy.md) | Design and apply cross-family firewall policy matrix |
| [4](phases/phase-4-capstone-test.md) | Execute capstone test: one rule change observed across all families |
| [5](phases/phase-5-break-fix.md) | Break/fix: RADIUS unreachable, Wazuh gap, VPN policy bypass |
| [6](phases/phase-6-document-and-close.md) | Full STAR summary, cross-family diagram, project close |

## Safety Gates

Stop before:
- any change to Route10 (production gateway) without explicit Claude approval;
- enabling RADIUS auth on OPNsense before a local fallback admin account is confirmed;
- any rule that could block management access to VLAN 10/20.

## Completion Evidence

- OPNsense admin login via AD credentials confirmed;
- capstone test executed — block rule observed in Suricata, Wazuh, and firewall log;
- cross-family integration diagram committed;
- STAR summary complete across all 9 OPNsense projects.
