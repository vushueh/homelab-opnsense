# Phase 1 — Readiness Audit

## Goal

Verify all P01-P08 dependencies are complete before starting the capstone. Document the integration readiness of each connected lab family.

---

## Dependency Checklist

| Dependency | Required From | Check |
|------------|--------------|-------|
| OPNsense baseline (router mode, VLAN 30/40/50/250) | P01 | |
| VLAN segmentation and firewall rules | P02 | |
| Suricata IDS running on lab VLANs | P03 | |
| WireGuard VPN operational | P04 | |
| OPNsense syslog forwarding to Wazuh | P05 | |
| DNS/DHCP hardened | P07 | |
| API backup running | P08 | |
| Windows NPS/RADIUS server reachable | Windows Server P13 | |
| Wazuh SIEM running and receiving OPNsense logs | SOC family | |
| CML VLAN 150 bridge operational | Homelab_CCNA P01 | |

---

## Track A — Integration Reachability Tests

[LEONEL]

```bash
# From OPNsense CLI — reach Windows NPS server
ping 192.168.20.11    # WIN-PRQD8TJG04M

# Reach Wazuh manager
ping <wazuh-manager-IP>

# Confirm Suricata running
ps aux | grep suricata

# Confirm syslog forwarding active
grep -R "Wazuh\|<siem-ip>" /usr/local/etc/syslog-ng.conf.d/
```

---

## Gap Documentation

For any dependency not met — document the gap and decide:
- **Block:** capstone cannot proceed until resolved
- **Defer:** integrate that family in a later phase, proceed with others

Save findings to `verification/readiness-audit.md`.

---

## Documentation Checklist

- [ ] All dependency checks run
- [ ] Gaps documented with block/defer decision
- [ ] readiness-audit.md saved
- [ ] Proceed to Phase 2 only when core dependencies met (P01-P05, Windows NPS, Wazuh)
