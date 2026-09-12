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

# CLAUDE.md — homelab-opnsense

Shared rules: [../AGENTS.md](../AGENTS.md) ·
[../AI-HOMELAB-PLAYBOOK.md](../AI-HOMELAB-PLAYBOOK.md). Last updated:
2026-07-04.

## What this repo owns (source of truth for)

- OPNsense 25.7 Hyper-V VM: Route10-facing 192.168.10.32 (hn2), default route
  via 192.168.10.1; the VLAN-20 leg is a named unknown as of 2026-07-18.
  Retained evidence favors 192.168.20.253 on hn0 and identifies .254 as a Cisco
  path, but a fresh manual console readback is still required.
- OPNsense-owned lab VLANs and DHCP scopes: 30/40/70/100/200/250 as
  192.168.x.0/24 (distinct from Route10's own VLAN 30 = 10.30.0.0/24!)
- Unbound behavior: system domain `internal` (interface self-registration
  only — NOT an authoritative zone; not reachable from VLAN 20; verified
  2026-07-03)

## What this repo must NOT touch

- Route10 VLANs/routes → route10 repo · Windows DNS/DHCP → windows repo
- Live OPNsense changes (rules, Suricata, VPN) without approved change-window
  — this firewall fronts live lab attack networks AND is reachable from the
  household side

## Current status source

Current project status lives in
[../docs/state.yaml](../docs/state.yaml). Keep this file focused on repo
ownership, settled facts, hazards, and standards.

## Repo standards

- Evidence per opnsense-evidence-documentation skill conventions (global
  skill); projects/<p>/ folders; no secrets — WireGuard/OpenVPN keys and
  pre-shared keys never committed
- Review file: CLAUDE-REVIEW.md (existing style)
- Technical how-to: global skills homelab-opnsense-projects,
  opnsense-p03-suricata, opnsense-p04-vpn, opnsense-p05-monitoring,
  opnsense-p06-ha-design

## Cross-repo links

- Route10 static routes hand these VLANs to 192.168.10.32 (route10 repo owns
  that table)
- P05 monitoring feeds Wazuh (homelab-management owns the Wazuh workflows)
- If Unbound is ever exposed to VLAN 20 with real records, revisit the
  Windows conditional-forwarder design (windows repo P03 Phase 5 trigger)

## `/goal` Session Start

Run `/goal next` before project work. The local wrapper loads the canonical
skill; if unavailable, read the family root `docs/homelab-goals.yaml`. Return
one queue-selected project, reconcile `CLAUDE-REVIEW.md`, and do not advance
past an unresolved blocker. Suricata remains detect-only first.
