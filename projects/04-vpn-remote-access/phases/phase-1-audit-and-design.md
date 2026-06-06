# Phase 1 — Audit Current Access Paths and VPN Design

## Goal

Document all existing remote access paths into the lab. Choose VPN technology.
Define which networks the VPN will reach and which it will not.

---

## Track A — Audit Existing Access

[LEONEL]

1. OPNsense GUI → **Interfaces → Assignments** — document all interfaces and IPs
2. OPNsense GUI → **Firewall → Rules → WAN** — document any existing port forwards or WAN allow rules
3. OPNsense GUI → **VPN** — confirm no existing WireGuard or OpenVPN configured
4. Note current Tailscale access (this remains separate — not replaced by this project)

Document findings in `verification/pre-vpn-audit.md`:
- Current WAN IP / interface
- Any existing inbound rules
- Which VLANs need to be reachable via VPN
- Which VLANs must NOT be reachable (VLAN 10, VLAN 20)

---

## VPN Technology Decision

| Criteria | WireGuard | OpenVPN |
|----------|-----------|---------|
| Setup complexity | Low | Medium |
| Certificate management | No (key pairs only) | Yes (CA + certs) |
| Performance | High | Medium |
| Learning value | Modern protocol, industry growing | Established, widely deployed |
| OPNsense native support | ✅ Built-in | ✅ Built-in |

**Recommendation:** WireGuard for speed and simplicity. Choose OpenVPN if certificate management is the target skill.

Document choice and reason in README.md.

---

## Access Scope Definition

Define before building:

| Network | VPN Reachable? | Reason |
|---------|---------------|--------|
| VLAN 30 (192.168.30.0/24) | Yes | Lab access |
| VLAN 40 (192.168.40.0/24) | Yes (Kali) | Controlled lab only |
| VLAN 250 (192.168.250.0/24) | Yes | Vulnerable lab |
| VLAN 10 (192.168.10.0/24) | No | Production infrastructure |
| VLAN 20 (192.168.20.0/24) | No | Management — Tailscale only |
| Internet via VPN | No (split tunnel) | Avoid routing all traffic through lab |

---

## Safety Gates for This Phase

- Do not add any WAN inbound rules yet
- Do not expose port to internet without Leonel approval
- Document scope limits before proceeding to Phase 2

## Documentation Checklist

- [ ] Existing access paths documented
- [ ] VPN technology chosen and reason recorded
- [ ] Access scope table completed
- [ ] pre-vpn-audit.md saved to verification/
