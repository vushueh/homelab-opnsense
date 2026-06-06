# Phase 4 — Capstone Test: One Change, All Families

## Goal

Execute the capstone proof: make one firewall rule change on OPNsense and observe the effect across all connected lab families simultaneously.

---

## The Test

**Action:** Block VLAN 40 (Kali) from reaching VLAN 250 (Vulnerable Lab) via OPNsense rule.

**Observe across all families:**

| Family | What to Observe | Tool |
|--------|----------------|------|
| OPNsense | Rule matches, traffic blocked | Firewall → Log Files → Live View |
| Suricata (P03) | Blocked attempt logged as alert | Services → IDS → Alerts |
| Wazuh (P05) | Block event appears in SIEM | Wazuh dashboard |
| Windows AD (P13) | No auth bypass attempted | Windows Event Viewer → Security |
| Physical Cisco (CCNA) | Unaffected — separate VLAN plane | show interfaces on R1 |
| VPN client (P04) | Same block applies via VPN_WG rules | ping 192.168.250.x from VPN |

---

## Step-by-Step Execution

[LEONEL]

1. **Before screenshot** — Kali can reach VLAN 250: `ping 192.168.250.10` → success
2. **Apply block rule** — OPNsense → Firewall → Rules → OPT-40 → Add block rule: source VLAN 40, dest VLAN 250
3. **Immediately test** — `ping 192.168.250.10` from Kali → should fail
4. **Check each family** in order — screenshot each observation
5. **Remove block rule** — restore original allow rule
6. **Confirm restored** — ping succeeds again

---

## Timeline Documentation

Record timestamps for each observation — proves the event pipeline is working end-to-end:

```
HH:MM:SS — Block rule applied in OPNsense GUI
HH:MM:SS — Firewall live log shows block event
HH:MM:SS — Suricata alert appears (if rule triggers)
HH:MM:SS — Wazuh receives and indexes the event
HH:MM:SS — Block rule removed, ping restored
```

---

## Documentation Checklist

- [ ] Before/after ping screenshots
- [ ] Firewall live log screenshot showing block
- [ ] Suricata alert screenshot (if triggered)
- [ ] Wazuh event screenshot
- [ ] Timeline documented in verification/capstone-test-results.md
