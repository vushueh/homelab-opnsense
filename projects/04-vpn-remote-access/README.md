# Project 04 — VPN Remote Access

## Status

Planned.

## Purpose

Build controlled remote access into selected lab networks without exposing the full home network. This project should be completed after logging and monitoring are in place.

## Candidate Technologies

- WireGuard for lightweight site/user access
- OpenVPN if certificate workflow is the learning target
- Tailscale remains separate and should not replace documenting OPNsense VPN skills

## Skills Practiced

- VPN server setup
- certificate or key handling
- firewall rule scoping
- split tunnel vs full tunnel decisions
- lab-only access policy
- remote access logging
- break-glass access planning

## Phase Index

| Phase | Goal |
|-------|------|
| [1](phases/phase-1-audit-and-design.md) | Audit current remote access paths and define allowed networks |
| [2](phases/phase-2-build-vpn-server.md) | Build WireGuard server instance and client key pair |
| [3](phases/phase-3-firewall-rules.md) | Create firewall rules for least-privilege access |
| [4](phases/phase-4-test-and-breakfix.md) | Test remote client access and break/fix scenarios |
| [5](phases/phase-5-document-and-close.md) | Sanitize evidence and close project |

## Safety Gates

Stop before adding WAN exposure, port forwards, or broad access to VLAN 10/20. VPN private keys and client configs must not be committed.
