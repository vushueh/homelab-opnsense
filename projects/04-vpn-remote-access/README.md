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
| 1 | Audit current remote access paths and define allowed networks |
| 2 | Choose VPN technology and document why |
| 3 | Build VPN server/profile in GUI |
| 4 | Create firewall rules for least-privilege access |
| 5 | Test remote client access |
| 6 | Break/fix: wrong route or blocked VPN rule |
| 7 | Document and sanitize evidence |

## Safety Gates

Stop before adding WAN exposure, port forwards, or broad access to VLAN 10/20. VPN private keys and client configs must not be committed.
