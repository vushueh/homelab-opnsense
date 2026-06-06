# Phase 3 — Cross-Family Firewall Policy Matrix

## Goal

Design and apply firewall rules that enforce access policy across all connected lab families. OPNsense becomes the single enforcement point for inter-family traffic.

---

## Policy Matrix

| Source | Destination | Action | Reason |
|--------|-------------|--------|--------|
| VLAN 40 (Kali) | VLAN 250 (Vulnerable) | Allow | Intended attack lab traffic |
| VLAN 40 (Kali) | VLAN 30 (Lab) | Allow | Lab access |
| VLAN 40 (Kali) | VLAN 10/20 (Prod/Mgmt) | Block | Production isolation |
| VPN clients (10.200.200.0/24) | VLAN 30/40/250 | Allow | VPN lab access (P04) |
| VPN clients | VLAN 10/20 | Block | VPN must not reach production |
| VLAN 150 (CML bridge) | VLAN 30 | Allow | CML ↔ OPNsense lab traffic |
| VLAN 150 (CML bridge) | VLAN 10/20 | Block | CML must not reach production |
| Any | OPNsense RADIUS (UDP 1812) | Allow from VLAN 20 only | NPS/RADIUS queries |

---

## Track A — Apply Policy Rules

[LEONEL]

For each row in the policy matrix — verify or create the rule in OPNsense Firewall → Rules on the relevant interface. Use the API diff workflow from P08 to capture before/after state.

---

## Track B — Policy Verification Tests

Run from the relevant source host:

```bash
# VLAN 40 → VLAN 250 (should pass)
ping 192.168.250.10   # from Kali VM

# VLAN 40 → VLAN 20 (should block)
ping 192.168.20.254   # from Kali VM → no response

# VPN client → VLAN 30 (should pass)
ping 192.168.30.1     # from WireGuard client

# VPN client → VLAN 10 (should block)
ping 192.168.10.35    # from WireGuard client → no response
```

---

## Documentation Checklist

- [ ] Policy matrix documented in verification/policy-matrix.md
- [ ] All rules verified with ping tests
- [ ] API diff captured before/after rule changes
- [ ] No production VLAN (10/20) reachable from lab VLANs or VPN
