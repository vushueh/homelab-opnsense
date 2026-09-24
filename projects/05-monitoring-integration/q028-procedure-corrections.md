# Q028 procedure corrections to the P05 blueprint

Codex review (2026-09-23, repository reads only), verified by Claude against
Q018/Q027 evidence. The phase files were rewritten on 2026-09-24 around actual execution. This table retains the reviewed corrections to the earlier blueprint.

| Blueprint text | Correction |
|---|---|
| Management address `.254` | Current management address is `.253` (Q018 baseline). |
| VLAN 50 lab interface | No OPNsense VLAN 50 interface exists (Q018). |
| "Audit local-only logging" | A remote UDP syslog target to Wazuh already exists; Q027 added Suricata to it. Audit its current application/level selections instead. |
| Port `1514` for Wazuh syslog | TCP 1514 is the agent (secure) channel; syslog is UDP 514. |
| `nmap -sV` across a /24 | Use only the approved lab tuple and Q027's benign marker. |
| Leave all applications/facilities empty | Add only the observed missing applications; whole-form saves can drop entries. |
| `nc -u` success proves receipt | UDP send success proves nothing; verify manager receipt and indexing. |
| DHCP leases page proves DHCP logging | Lease display is not log forwarding evidence. |
| VPN connection saved search | P04 VPN is not built; out of scope. |
| NetFlow to the SIEM/collector | Not the SIEM: OPNsense already exports LAN-only v9 to its own dedicated collector instance (live since 2026-06-21). Extend to lab VLANs through gate G5. |
| Handoff to P06 | Queue successor is Q029 (CCNA P02). |
