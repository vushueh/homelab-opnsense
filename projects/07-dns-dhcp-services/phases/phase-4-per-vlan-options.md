# Phase 4 — Verify Per-VLAN DHCP Options

## Goal

Confirm each VLAN delivers the correct gateway, DNS server, and domain suffix to clients. Catch misconfigured scope options before they cause hard-to-diagnose routing or DNS failures.

---

## Expected DHCP Options Per VLAN

| VLAN | Interface | Gateway | DNS | Domain Suffix |
|------|-----------|---------|-----|---------------|
| 30 | LAN / OPT-30 | 192.168.30.1 | 192.168.30.1 (Unbound) | lab.local |
| 40 | OPT-40 | 192.168.40.1 | 192.168.40.1 (Unbound) | lab.local |
| 50 | OPT-50 | 192.168.50.1 | 192.168.50.1 (Unbound) | lab.local |
| 250 | OPT-250 | 192.168.250.1 | 192.168.250.1 (Unbound) | lab.local |

---

## Track A — GUI Steps

[LEONEL]

For each VLAN interface:

1. OPNsense → **Services → DHCPv4 → [VLAN interface]**
2. Confirm **Gateway** is set to the correct VLAN gateway IP
3. Confirm **DNS servers** field contains the OPNsense IP for that VLAN
4. Confirm **Domain name** is set (e.g. `lab.local`)
5. If any option is wrong or blank — correct it and apply

---

## Track B — Client-Side Verification

[LEONEL — on a VM in each VLAN]

```bash
# Linux — show DHCP-received options
ip route show          # default gateway correct?
cat /etc/resolv.conf   # DNS server correct?

# Windows
ipconfig /all          # shows DHCP server, gateway, DNS, domain suffix
```

---

## Break/Fix Embedded — Wrong Gateway Option

**Break:** Temporarily set VLAN 40 gateway option to wrong IP (e.g. 192.168.40.99)
**Symptom:** Kali VM on VLAN 40 gets no internet after DHCP renew — pings local hosts OK but not external
**Diagnose:** `ip route show` — default route points to wrong gateway; `ipconfig /all` on Windows shows wrong gateway
**Fix:** Correct gateway in DHCP scope, renew on client
**Lesson:** Wrong gateway option is silent at DHCP time — only fails when routing off-VLAN

---

## Documentation Checklist

- [ ] All 4 VLANs verified: gateway, DNS, domain suffix correct
- [ ] Client-side verification output saved per VLAN
- [ ] Break/fix scenario completed and documented
- [ ] Any corrections applied and re-verified
