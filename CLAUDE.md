# CLAUDE.md — homelab-opnsense

Shared rules: [../AGENTS.md](../AGENTS.md) ·
[../AI-HOMELAB-PLAYBOOK.md](../AI-HOMELAB-PLAYBOOK.md). Last updated:
2026-07-04.

## What this repo owns (source of truth for)

- OPNsense 25.7 Hyper-V VM: Route10-facing 192.168.10.32 (hn2), VLAN-20 leg
  192.168.20.253 (hn0), default route via 192.168.10.1
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
