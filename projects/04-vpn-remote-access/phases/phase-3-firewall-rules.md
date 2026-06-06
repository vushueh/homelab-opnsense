# Phase 3 — Firewall Rules for VPN Access

## Goal

Create firewall rules that allow VPN clients to reach lab VLANs (30, 40, 250) with
least-privilege. Block VPN access to production VLANs (10, 20). Open WAN port for VPN.

---

## Track A — GUI Steps

[LEONEL]

### 3.1 Create WireGuard Interface in OPNsense

Assigning a WireGuard interface is recommended for clean per-interface firewall rules. OPNsense can use the default WireGuard group instead, but this lab uses a dedicated `VPN_WG` interface so screenshots and rules are easier to audit.

1. OPNsense → **Interfaces → Assignments**
2. Find the WireGuard device in the unassigned list (`wg1` is common for the first modern OPNsense instance; use whatever `wgX` appears) → assign it
3. Name it: `VPN_WG`
4. Open the new interface page
5. Enable and lock the interface
6. Leave IPv4/IPv6 configuration type as **None**; the tunnel IP stays on the WireGuard Instance
7. Save and apply
8. Restart WireGuard from **VPN → WireGuard → General** if the interface does not appear active

### 3.2 Allow VPN Clients to Reach Lab VLANs

1. OPNsense → **Firewall → Rules → VPN_WG**
2. Add rule:
   - Action: Pass
   - Source: `10.200.200.0/24` (VPN subnet)
   - Destination: `192.168.30.0/24` (VLAN 30)
   - Description: VPN → VLAN30
3. Repeat for VLAN 40 (192.168.40.0/24) and VLAN 250 (192.168.250.0/24)

### 3.3 Block VPN Access to Production VLANs

Add DENY rules ABOVE the allow rules:

1. Add rule:
   - Action: **Block**
   - Source: `10.200.200.0/24`
   - Destination: `192.168.10.0/24` (VLAN 10 — production infra)
   - Description: BLOCK VPN → VLAN10
2. Add rule:
   - Action: **Block**
   - Source: `10.200.200.0/24`
   - Destination: `192.168.20.0/24` (VLAN 20 — management)
   - Description: BLOCK VPN → VLAN20

**Rule order matters:** Block rules must be above Allow rules in OPNsense.

### 3.4 Open WAN Port for WireGuard

1. OPNsense → **Firewall → Rules → WAN**
2. Add rule:
   - Action: Pass
   - Protocol: UDP
   - Destination: WAN address
   - Destination Port: `51820`
   - Description: WireGuard VPN inbound

---

## Track B — Verification

[CODEX/CLAUDE — after approval]

```sh
# Confirm WireGuard listening on UDP 51820
sockstat -4 -l | grep 51820

# Check WireGuard peers from OPNsense CLI
wg show

# Confirm the assigned interface exists
ifconfig | grep -A3 '^wg'
```

From VPN client (after enabling tunnel with WAN endpoint):
```bash
# Reachable
ping 192.168.30.1     # VLAN 30 gateway — should succeed
ping 192.168.40.1     # VLAN 40 gateway — should succeed
ping 192.168.250.1    # VLAN 250 gateway — should succeed

# Blocked
ping 192.168.10.1     # VLAN 10 — should FAIL (block rule)
ping 192.168.20.254   # MGMT — should FAIL (block rule)
```

---

## Screenshots to Capture

- VPN_WG firewall rules showing Allow (VLAN 30/40/250) and Block (VLAN 10/20) in correct order
- WAN rule showing UDP 51820 open
- Ping test results: VLAN 30 success, VLAN 10 failure

## Documentation Checklist

- [ ] WireGuard interface (VPN_WG) assigned and enabled
- [ ] Allow rules for VLANs 30, 40, 250
- [ ] Block rules for VLANs 10, 20 (above allow rules)
- [ ] WAN UDP 51820 open
- [ ] Connectivity tests confirm correct allow/block behavior
