# Phase 1 — Audit Current DNS and DHCP State

## Goal

Document the current OPNsense DNS resolver and DHCP configuration before making any changes. Read-only except for screenshots and notes.

---

## Track A — GUI Steps

[LEONEL]

### DNS Resolver Audit

1. OPNsense → **Services → Unbound DNS → General**
   - Note: enabled/disabled, listen interfaces, forwarding mode on/off
2. OPNsense → **Services → Unbound DNS → Advanced**
   - Note: DNSSEC, DNS over TLS settings, cache size
3. OPNsense → **Services → Unbound DNS → Query Forwarding**
   - Note: upstream DNS servers configured (if any)

### DHCP Audit — Per VLAN

For each active VLAN (30, 40, 50, 250):

1. OPNsense → **Services → DHCPv4 → [VLAN interface]**
   - Note: range start/end, DNS servers in scope options, default gateway, domain name
2. OPNsense → **Services → DHCPv4 → Leases**
   - Note: active leases, any existing static mappings

---

## Track B — CLI Verification

[CODEX/CLAUDE — after approval]

```sh
# Confirm Unbound is running
ps aux | grep unbound

# Check current upstream forwarders
cat /var/unbound/unbound.conf | grep "forward-addr"

# Check DHCP leases on all interfaces
cat /var/dhcpd/var/db/dhcpd.leases | head -60
```

---

## Audit Document Template

Save findings to `verification/pre-p07-audit.md`:

```
## DNS Resolver
- Status: enabled / disabled
- Forwarding mode: on / off
- Upstream DNS: [list]
- DNSSEC: enabled / disabled
- DNS over TLS: enabled / disabled

## DHCP — VLAN 30
- Range: x.x.x.x – x.x.x.x
- DNS option: [value]
- Gateway option: [value]
- Static mappings: [count]

## DHCP — VLAN 40 / 50 / 250
[repeat]
```

---

## Documentation Checklist

- [ ] DNS resolver settings documented
- [ ] DHCP settings per VLAN documented
- [ ] Active leases count noted
- [ ] pre-p07-audit.md saved to verification/
- [ ] No changes made in this phase
