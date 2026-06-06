---
name: opnsense-p03-suricata
description: OPNsense Project 03 IDS/IPS with Suricata workflow for Leonel's homelab. Use when the user says "opnsense p03", "project 03", "Suricata", "IDS", "IPS", "verify alerts", "tune rules", "break/fix Suricata", "eve.json", "ET Open", or asks to start, review, troubleshoot, document, or close the OPNsense IDS/IPS project.
---

# OPNsense P03 Suricata

## Start Here

Read, in order:

1. `AGENTS.md`
2. `CLAUDE-REVIEW.md`
3. `CODEX-LOG.md`
4. `projects/03-ids-ips-suricata/README.md`
5. The current phase file under `projects/03-ids-ips-suricata/phases/`

Use Project 03 to turn OPNsense into a lab IDS sensor first. Treat IPS blocking as a later decision, not the default outcome.

## Phase Map

- Phase 1: audit interfaces, backups, plugin/service state, and safety baseline.
- Phase 2: enable Suricata in IDS alert-only mode on lab interfaces only.
- Phase 3: generate controlled Kali VLAN 40 to VLAN 250 test traffic and verify alerts.
- Phase 4: tune noisy rules through OPNsense IDS policy/rule workflow.
- Phase 5: run break/fix scenarios for interface scope, stopped service, and disabled rules.
- Phase 6: decide whether IPS blocking is safe; defer if any safety gate fails.

## Safety Rules

- Do not select WAN for Suricata unless Leonel and Claude explicitly approve a separate WAN-sensor experiment.
- Do not include production VLAN 10 or management VLAN 20 in IDS/IPS scope.
- Do not enable IPS/drop/blocking mode until Phase 4 tuning is complete, a same-day backup exists, and Hyper-V console access is confirmed.
- Do not commit raw OPNsense XML exports, WAN identifiers, secrets, certificates, or password hashes.
- Keep test traffic controlled: Kali on VLAN 40 toward VLAN 250 targets.

## Operating Pattern

When guiding Leonel:

1. State the current phase and safety gate.
2. Give GUI-first steps for screenshots.
3. Give command/log verification only as a second track for Claude/approved CLI use.
4. Explain what good output looks like.
5. Name exactly what screenshot or note to save.
6. Include rollback for every live change.

## Technical Anchors

- OPNsense GUI: `Services -> Intrusion Detection`.
- Suricata EVE JSON path: `/var/log/suricata/eve.json`.
- Start with alert-only IDS mode.
- First rule scope should stay small, such as ET Open test rules, before broader categories.
- For tuning, prefer OPNsense `Policy` and `Rules` actions over hand-editing generated files.

## Closeout

Before marking the project complete:

- Confirm all phase checklists are complete.
- Confirm at least one controlled alert from VLAN 40 to VLAN 250 is documented.
- Confirm rule tuning notes exist.
- Confirm break/fix notes exist.
- Confirm the IPS decision is documented as enabled with rollback proof or deferred with reason.
- Append a session entry to `CODEX-LOG.md`.
