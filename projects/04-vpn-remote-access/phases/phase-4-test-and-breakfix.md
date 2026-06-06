# Phase 4 — Test Remote Access and Break/Fix

## Goal

Test VPN from outside the lab network. Run break/fix scenarios to build troubleshooting
muscle memory for common VPN failures.

---

## Track A — Full Remote Access Test

[LEONEL]

### 4.1 Connect from Outside LAN

Use a phone hotspot or different network to simulate true remote access:

1. Disconnect laptop from lab LAN / WiFi
2. Connect to phone hotspot (different public IP)
3. Start WireGuard tunnel on laptop using the full config with WAN Endpoint
4. Verify tunnel established:

```
wg show
# Expected: latest handshake: X seconds ago
#           transfer: received X MiB, sent X MiB
```

### 4.2 Access Lab Resources via VPN

```bash
# From laptop over VPN
ping 192.168.30.1          # VLAN 30 gateway
ping 192.168.250.1         # VLAN 250

# SSH to a lab host on VLAN 30
ssh user@192.168.30.x

# Confirm no access to production VLANs
ping 192.168.10.35         # Proxmox — should timeout
ping 192.168.20.254        # OPNsense MGMT — should timeout
```

### 4.3 Check VPN Logs on OPNsense

OPNsense → **VPN → WireGuard → Peers** — confirm handshake timestamp updated
OPNsense → **Firewall → Log Files → Live View** — confirm VPN traffic hitting allow/block rules

---

## Break Scenario A — Wrong Endpoint IP

**Break:** Change client config Endpoint to a wrong IP
**Symptom:** No handshake, `wg show` shows `(none)` for latest handshake
**Fix:** Correct Endpoint IP in client config

---

## Break Scenario B — Firewall Rule Order Wrong

Use a lab VLAN for this exercise. Do not intentionally allow VPN access to VLAN 10 or VLAN 20.

**Break:** Add a temporary block rule for VLAN 250 below the VLAN 250 allow rule
**Symptom:** VPN client can still ping 192.168.250.1 because the allow rule matches first
**Diagnose:** OPNsense Firewall → Rules → VPN_WG — rule order visible
**Fix:** Move the temporary VLAN 250 block rule above the allow rule, confirm ping fails, then remove the temporary block rule and confirm normal VLAN 250 access returns
**Lesson:** OPNsense processes rules top-to-bottom; first match wins

---

## Break Scenario C — WAN Port Closed

**Break:** Remove the UDP 51820 WAN rule
**Symptom:** From outside LAN, tunnel never establishes
**Diagnose:** `wg show` — no handshake. OPNsense Firewall → Log — blocked packet visible on WAN
**Fix:** Re-add WAN UDP 51820 rule

---

## Break/Fix Log Template

Save to `troubleshooting/break-fix-log.md`:

```
## Scenario A — Wrong Endpoint — YYYY-MM-DD
Symptom: No handshake established
Root cause: Endpoint IP typo in client config
Fix: Corrected IP in config, reconnected
Verified: wg show shows active handshake
```

---

## Screenshots to Capture

- wg show with active handshake from outside LAN
- Ping success to VLAN 30/250 from VPN client
- Ping failure to VLAN 10 from VPN client
- Firewall live log showing VPN traffic

## Documentation Checklist

- [ ] VPN tested from outside LAN (phone hotspot)
- [ ] All 3 break/fix scenarios completed
- [ ] Break-fix-log.md committed
- [ ] Firewall rule order verified correct
- [ ] Screenshots saved
