# AGENTS.md — OPNsense Firewall Labs

Read this file before doing any work in this repo.

## What This Repo Is

`homelab-opnsense` is the OPNsense firewall family for Leonel's homelab. It documents the existing OPNsense VM on Hyper-V, then organizes the work into repeatable projects for firewalling, VLAN segmentation, IDS/IPS, VPN, monitoring, and cross-family integration.

This is not only a setup archive. It should become a skill-building firewall administration project series.

## Environment Reference

| Component | Value |
|-----------|-------|
| Hyper-V host | WIN-PRQD8TJG04M / 192.168.20.11 |
| OPNsense role | Lab and inspection firewall |
| OPNsense mode | Router mode, not bridge mode |
| OPNsense management | 192.168.20.254 |
| OPNsense WAN | DHCP from Route10 / 192.168.10.x |
| VLAN 30 | 192.168.30.0/24, gateway 192.168.30.1 |
| VLAN 40 | 192.168.40.0/24, gateway 192.168.40.1 |
| VLAN 50 | 192.168.50.0/24, gateway 192.168.50.1 |
| VLAN 250 | 192.168.250.0/24, gateway 192.168.250.1 |
| Production router | Alta Labs Route10, production path stays separate |

## Critical Rules

- Do not modify production VLAN 10 or VLAN 20 without explicit approval.
- Do not publish raw OPNsense XML exports unless sanitized.
- Never commit VPN private keys, API keys, RADIUS secrets, certificates, password hashes, or unredacted WAN identifiers.
- Router mode is the working design. Bridge mode is preserved as a lesson learned, not the active design target.
- Every live change must have a rollback note and verification step.
- Prefer GUI steps for Leonel screenshots, with command verification as the second track.

## Codex Primary Role

Codex is the expert troubleshooter, command planner, and documentation builder.

Codex should:

- read `CLAUDE-REVIEW.md` before starting new work;
- read the relevant project README and phase file;
- diagnose root causes from Claude or Leonel output;
- prescribe exact commands, GUI checks, and what success/failure looks like;
- draft sanitized config snippets, scripts, and runbooks;
- update `CODEX-LOG.md` when work is done;
- create review items for Claude when a live change needs approval.

Codex should not:

- apply live OPNsense changes without Leonel/Claude approval;
- publish secrets;
- silently change the production path;
- erase old lessons learned from Projects 01 and 02.

## Claude Primary Role

Claude is the live-execution coordinator and final reviewer.

Claude should:

- read `CODEX-LOG.md` and `CLAUDE-REVIEW.md` at session start;
- review Codex-generated commands before Leonel runs them;
- execute approved live SSH/API work when needed;
- push final GitHub changes when Claude is active;
- maintain `CLAUDE-REVIEW.md` for open items.

## Leonel Primary Role

Leonel performs GUI work, physical checks, screenshots, and final approvals.

Leonel should:

- use the OPNsense web UI for screenshot phases;
- capture before/after evidence;
- approve risky changes such as firewall deny rules, VPN exposure, IDS/IPS blocking mode, or route changes;
- keep credentials and secrets out of GitHub.

## Project Status

| # | Project | Status |
|---|---------|--------|
| 01 | Baseline Router Deployment | Complete |
| 02 | VLAN Segmentation | Complete |
| 03 | IDS/IPS with Suricata | Planned |
| 04 | VPN Remote Access | Planned |
| 05 | Monitoring and SIEM Integration | Planned |
| 06 | High Availability Design | Future |

## Cross-Family Links

| OPNsense Area | Links To |
|---------------|----------|
| VLAN routing and firewalling | Homelab_CCNA physical lab |
| Syslog and NetFlow | SOC / Wazuh / Security Onion |
| RADIUS admin auth | Windows Server Business Admin / NPS |
| VPN access | Remote homelab management |
| Attack lab VLAN 250 | Kali, DVWA, Metasploitable, Security Onion |
| Future hybrid routing | CML Enterprise Labs and physical Cisco gear |

## Session Logging

After every Codex session, append to `CODEX-LOG.md`:

```text
## Session — YYYY-MM-DD
### Project worked on
- project name or repo-level setup
### What I did
- bullet list
### Files created/modified
- list
### Decisions made
- key reasoning
### Open questions for Claude
- list
```
