---
name: opnsense-p06-ha-design
description: OPNsense Project 06 high availability design workflow for Leonel's homelab. Use when the user says "opnsense p06", "project 06", "OPNsense HA", "CARP", "pfsync", "XMLRPC sync", "firewall failover", "split brain", or asks to design, review, document, or plan OPNsense high availability without making live production changes.
---

# OPNsense P06 HA Design

## Start Here

Read, in order:

1. `AGENTS.md`
2. `CLAUDE-REVIEW.md`
3. `CODEX-LOG.md`
4. `projects/06-high-availability/README.md`
5. Any current phase/design file under `projects/06-high-availability/phases/`

Use Project 06 as a design-only project until a second suitable firewall instance, host, and network path exist.

## Phase Map

- Phase 1: audit hardware, Hyper-V networking, interfaces, and IP plan.
- Phase 2: design HA topology and failure domains.
- Phase 3: plan CARP VIPs, pfsync, and XMLRPC config sync.
- Phase 4: build a second node only after explicit approval and prerequisites.
- Phase 5: test failover in an isolated lab path.
- Phase 6: run break/fix for failed sync or split-brain symptoms.
- Phase 7: document whether production readiness is realistic or deferred.

## Safety Rules

- Do not build HA live just because the design exists.
- Do not move the production gateway role away from Route10.
- Do not change VLAN 10 or VLAN 20 without explicit approval.
- Do not create CARP VIPs on production paths without a reviewed maintenance plan.
- Do not assume HA is useful until hardware, switch paths, and address ownership are real.

## Design Checklist

Cover these areas in every serious HA design pass:

- Node placement and Hyper-V host dependency.
- WAN and LAN/interface symmetry.
- CARP VIP plan and non-conflicting IPs.
- pfsync network and state-sync scope.
- XMLRPC config sync scope and credentials handling.
- DHCP, DNS, VPN, IDS/IPS, and monitoring behavior during failover.
- Split-brain risks and how Leonel will detect them.
- Rollback path to single-node router mode.

## Operating Pattern

When guiding Leonel:

1. Separate "design", "lab simulation", and "live change" clearly.
2. Ask for missing topology facts only when they block safe design.
3. Prefer diagrams, tables, and evidence checklists.
4. Include exact prerequisites before any build phase.
5. Keep production impact analysis explicit.
6. Record deferred decisions instead of forcing implementation.

## Closeout

Before marking the project complete or ready to build:

- Confirm the design names all required hardware and network paths.
- Confirm CARP/pfsync/XMLRPC assumptions are documented.
- Confirm failure and rollback scenarios are written.
- Confirm Project 06 remains future/design-only unless Leonel approves a build.
- Append a session entry to `CODEX-LOG.md`.
