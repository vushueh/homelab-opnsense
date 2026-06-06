---
name: opnsense-p04-vpn
description: OPNsense Project 04 VPN remote access workflow for Leonel's homelab. Use when the user says "opnsense p04", "project 04", "OPNsense VPN", "WireGuard", "OpenVPN", "remote access", "VPN firewall rules", "split tunnel", "VPN break/fix", or asks to start, review, troubleshoot, document, or close the OPNsense VPN project.
---

# OPNsense P04 VPN

## Start Here

Read, in order:

1. `AGENTS.md`
2. `CLAUDE-REVIEW.md`
3. `CODEX-LOG.md`
4. `projects/04-vpn-remote-access/README.md`
5. The current phase file under `projects/04-vpn-remote-access/phases/`

Use Project 04 to build lab-scoped remote access. Monitoring should ideally be working before WAN exposure.

## Phase Map

- Phase 1: audit existing access paths and define allowed/blocked networks.
- Phase 2: build WireGuard server and client peer locally with no WAN exposure.
- Phase 3: assign the WireGuard interface and add least-privilege firewall rules.
- Phase 4: test from outside the LAN and run safe break/fix scenarios.
- Phase 5: sanitize evidence and close the project.

## Safety Rules

- Stop before opening WAN UDP 51820 until Leonel explicitly approves the exposure.
- Never commit WireGuard private keys, preshared keys, real client configs, raw backups, or screenshots that show secrets.
- Block VPN access to VLAN 10 and VLAN 20; do not use them for intentional break/fix exposure tests.
- Use lab VLANs 30, 40, and 250 as the intended reachable networks.
- Keep Tailscale as a separate access path; do not erase or replace it silently.

## WireGuard Defaults

- Treat WireGuard as built into modern OPNsense.
- Use `VPN -> WireGuard -> Instances`, `Peers`, and `General`.
- Use tunnel subnet `10.200.200.0/24` unless a conflict is discovered.
- Use client address `/32` in client config and peer Allowed IP.
- If assigning an interface, use the visible `wgX` device, name it `VPN_WG`, enable/lock it, and leave interface IP configuration as `None`; the tunnel address belongs to the WireGuard instance.

## Operating Pattern

When guiding Leonel:

1. Confirm whether the phase is local-only or WAN-exposed.
2. Give GUI-first steps for screenshots.
3. Add CLI verification only for Claude/approved use.
4. Include a ping matrix for allowed and blocked networks.
5. Require rollback notes for WAN rules and VPN firewall rules.
6. Remind Leonel to crop or hide keys before saving screenshots.

## Closeout

Before marking the project complete:

- Confirm VPN reaches VLAN 30, VLAN 40, and VLAN 250 only.
- Confirm VLAN 10 and VLAN 20 are blocked from VPN clients.
- Confirm testing from a phone hotspot or separate network.
- Confirm sanitized templates are committed instead of real configs.
- Confirm break/fix notes exist.
- Append a session entry to `CODEX-LOG.md`.
