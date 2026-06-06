# AGENTS.md — Codex Standing Orders
## OPNsense Firewall Labs Family

**Read this file before doing any work in this repo.**

---

## What This Repo Is

Structured lab projects for OPNsense firewall on Hyper-V. OPNsense runs as a VM on
Hyper-V (192.168.20.11). It is in **Router Mode** — not bridge mode.

This OPNsense VM is the inspection/lab firewall. It is separate from Route10
(Alta Labs at 192.168.70.1) which is the production internet router.

Projects 01–02 are complete. Projects 03–06 are planned.

## Your Role (Codex)

You are the **builder and documenter**. You write configs, scripts, and runbooks.
You do NOT push to GitHub — Claude does that.

| You own | Claude owns |
|---------|------------|
| OPNsense config generation | Config review before applying |
| Script drafts (bash, PowerShell) | All GitHub pushes |
| CODEX-LOG.md updates | CLAUDE-REVIEW.md |
| Lab guides and runbooks | Design decisions |
| Hyper-V setup steps | Cross-family link decisions |

## Before Starting Any Task

1. Read `CLAUDE-REVIEW.md` — resolve all OPEN items before new work
2. Read the relevant project README and phase file
3. Read `README.md` for current project status
4. Check the architecture diagram in `diagrams/` if unsure about topology

## Critical Rules

- **NEVER modify production VLANs 10 or 20** — these are on Route10, not OPNsense
- **NEVER push to GitHub** — Claude handles all pushes
- All configs reviewed by Claude before being applied to OPNsense
- OPNsense changes go through web UI or SSH — Claude executes live changes
- Hyper-V virtual switch changes must be reviewed carefully — can disrupt production

## Environment Reference

| Component | Value |
|-----------|-------|
| OPNsense VM host | Hyper-V (192.168.20.11) |
| OPNsense WAN | DHCP from Route10 (192.168.10.x) |
| OPNsense LAN | 192.168.30.1/24 (VLAN 30) |
| OPNsense OPT1 | 192.168.40.1/24 (VLAN 40) |
| OPNsense OPT2 | 192.168.50.1/24 (VLAN 50) |
| OPNsense OPT3 | 192.168.250.1/24 (VLAN 250 — Attack Lab) |
| OPNsense MGMT | 192.168.20.254 |
| Production router | Route10 (Alta Labs) — 192.168.70.1 — DO NOT TOUCH |

## Project Status

| # | Project | Status |
|---|---------|--------|
| 01 | Baseline Deployment | ✅ Complete |
| 02 | VLAN Segmentation | ✅ Complete |
| 03 | IDS/IPS — Suricata | ⬜ Planned |
| 04 | VPN Remote Access | ⬜ Planned |
| 05 | Monitoring Integration | ⬜ Planned |
| 06 | High Availability | 💡 Future |

## Cross-Family Links

OPNsense connects to other project families. Flag these to Claude before building:

| OPNsense Project | Links to |
|-----------------|----------|
| 05 Monitoring | Physical gear monitoring in Homelab_CCNA (syslog, NetFlow) |
| 05 Monitoring | SOC stack (Wazuh/TheHive) in Homelab_CCNA |
| OSPF future | R1-2900 in physical lab is an OSPF neighbour |

## Logging Work

After every session, append to `CODEX-LOG.md`:

```
## Session — YYYY-MM-DD
### Project worked on
- [project name]
### What I did
- bullet list
### Files created/modified
- list
### Open questions for Claude
- list
```
