# Project 07 — DNS and DHCP Services

## Status

Planned. Requires Project 02 (VLAN Segmentation) complete.

## Purpose

Harden and document OPNsense DNS and DHCP behavior across all lab VLANs.
Configure DNS resolver settings, DHCP static mappings, and validate that
each VLAN gets the correct DNS and gateway options. Build break/fix exercises
around common DNS and DHCP failure scenarios.

## Skills Practiced

- OPNsense Unbound DNS resolver configuration
- DNS over TLS (DoT) upstream configuration
- DHCP static mappings and lease management
- Per-VLAN DHCP options (gateway, DNS, domain)
- DNS query logging and troubleshooting
- Break/fix: DHCP scope exhaustion, DNS resolution failure, wrong gateway option

## Cross-Family Links

| Family | Link |
|--------|------|
| Windows Server | Compare OPNsense DNS to Windows AD-integrated DNS (Project 03) |
| Homelab_CCNA | Compare DHCP relay behavior to Cisco IOS DHCP relay (Project 01) |
| SOC / Blue Team | DNS query logs forwarded to Wazuh (Project 05) |

## Phase Index

| Phase | Goal |
|-------|------|
| [1](phases/phase-1-audit-dns-dhcp.md) | Audit current DNS resolver and DHCP scope settings |
| [2](phases/phase-2-harden-dns-resolver.md) | Harden Unbound: DNS over TLS, DNSSEC, block rebind attacks |
| [3](phases/phase-3-dhcp-static-mappings.md) | Create static DHCP mappings for all known lab hosts |
| [4](phases/phase-4-per-vlan-options.md) | Verify per-VLAN DHCP options (gateway, DNS, domain suffix) |
| [5](phases/phase-5-break-fix.md) | Break/fix: wrong DNS upstream, DHCP scope exhaustion, wrong gateway |
| [6](phases/phase-6-document-and-close.md) | Document all settings and close project |

## Safety Gates

Stop before:
- changing DNS settings on the WAN interface without Claude review;
- modifying VLAN 10 or VLAN 20 DHCP or DNS settings;
- enabling DNS rebind protection if it blocks legitimate lab traffic — test first.

## Completion Evidence

- screenshot of Unbound resolver settings with DoT upstream;
- screenshot of static DHCP mappings for lab hosts;
- screenshot of per-VLAN DHCP options;
- break/fix log with 3 scenarios documented;
- STAR summary complete.
