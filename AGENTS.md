## Shared workflow

Read `E:/Homelab-Repos/family-projects/AGENTS.md` once per session
(`/mnt/e/Homelab-Repos/family-projects/AGENTS.md` in WSL). If working outside
this workspace, fetch the shared contract from
`vushueh/family-projects-ai-playbook` before homelab operations.
It owns task-scoped reads and publication authority; this repo owns technical
constraints. For a named file task, read target files and related OPEN items.
For project selection/status/resume, use the shared goal skill and freshness
checks; preserve the active item, dependencies, queue order and WIP limits.
Either operating agent may publish the authorized package. Use explicit paths
and relevant checks, preserve dirty work and intentionally unpublished overlays.

For startup scope and publication roles, this Shared workflow section takes
precedence over older README ownership tables and phase-guide handoffs to Claude.
Those guides still own technical procedures, evidence and live safety requirements.
An unrelated OPEN item does not block a named-file task; honor related blockers.

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
| OPNsense management | Named unknown as of 2026-07-18; retained evidence favors `192.168.20.253`, while legacy pages say `.254` |
| OPNsense WAN | DHCP from Route10 / 192.168.10.x |
| VLAN 30 | 192.168.30.0/24, gateway 192.168.30.1 |
| VLAN 40 | 192.168.40.0/24, gateway 192.168.40.1 |
| VLAN 50 | 192.168.50.0/24, gateway 192.168.50.1 |
| VLAN 250 | 192.168.250.0/24, gateway 192.168.250.1 |
| Production router | Alta Labs Route10, production path stays separate |

The dated [U0-TRUTH management-address reconciliation](docs/u0-truth-management-address-2026-07-18.md)
is the current authority boundary. Do not rely on `.253` or `.254` as freshly
verified until Leonel completes the named manual console readback.

## Edit Tier Rules

All three parties (Claude, Codex, Leonel) must follow this model.

### Tier 1 — Local repo (default for all content work)
Use for: phase files, project READMEs, configs, scripts, docs, skill files.
Local path (Windows): `E:\Homelab-Repos\family-projects\homelab-opnsense\`
Local path (WSL):      `/mnt/e/Homelab-Repos/family-projects/homelab-opnsense/`
- Codex writes here directly (open this folder as the Codex workspace/project).
- Claude reads and edits files here directly.
- Start with scoped Git status; commit explicit reviewed paths and publish only when authorized.

### Tier 2 — GitHub API (exception only)
Use for: bridge file quick patches (CLAUDE-REVIEW.md, CODEX-LOG.md) between sessions when no local checkout is open.
Never: phase content, skill files, configs, or any file over ~5KB.
Either agent may publish the user-authorized package using available Git/GitHub tools.

### Tier 3 — Live infrastructure (approval required)
Use for: SSH commands, OPNsense web UI changes, firewall rule edits.
Who: Leonel performs all GUI actions. Claude coordinates SSH with explicit approval per action.
Never: Codex does not execute live infrastructure commands.

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
- publish final GitHub changes within the user-authorized package;
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

## Master Program Selection

Before selecting or advancing work, invoke the local `/goal` wrapper or read
`../docs/homelab-goals.yaml`. Resume the active primary item or take exactly
the lowest-sequence ready item; never offer a project menu. Reconcile this
repo's README/project state and `CLAUDE-REVIEW.md` lock first. A blocker must
be repaired, safely rescoped, or closed Deferred with a precise trigger before
advancing. Firewall/IDS live approval and detect-only-first rules remain.
