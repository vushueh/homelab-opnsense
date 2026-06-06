# Phase 3 — DHCP Static Mappings for Lab Hosts

## Goal

Create static DHCP mappings for all known lab hosts so they always receive the same IP. Eliminates IP drift when VMs restart and makes firewall rules more reliable.

---

## Track A — GUI Steps

[LEONEL]

### 3.1 Identify Hosts That Need Static Mappings

From the DHCP lease table and known VM inventory, identify hosts on each VLAN:

| Host | VLAN | Current IP | MAC | Target Static IP |
|------|------|-----------|-----|-----------------|
| Kali VM | 40 | 192.168.40.x | [from leases] | 192.168.40.10 |
| Vulnerable VM | 250 | 192.168.250.x | [from leases] | 192.168.250.10 |
| Lab host VLAN 30 | 30 | 192.168.30.x | [from leases] | 192.168.30.10 |

### 3.2 Create Static Mappings

For each host:

1. OPNsense → **Services → DHCPv4 → [VLAN interface]**
2. Scroll to **Static ARP** / **Static Mappings** section
3. Click **+**
4. Enter:
   - **MAC address:** from lease table
   - **IP address:** chosen static IP (within scope but outside dynamic range)
   - **Hostname:** descriptive name (kali-vm, vuln-lab, etc.)
   - **Description:** lab purpose
5. Save → Apply

### 3.3 Verify Static Mapping Applied

1. Renew DHCP on the VM: `dhclient -r && dhclient eth0` (Linux) or `ipconfig /renew` (Windows)
2. Confirm VM received the static IP
3. OPNsense → Services → DHCPv4 → Leases — static entries marked differently from dynamic

---

## Track B — CLI Verification

[CODEX/CLAUDE — after approval]

```sh
# List current static mappings from DHCP config
grep -A5 "host " /var/dhcpd/etc/dhcpd.conf | head -40

# Check lease file for static binding confirmation
grep -A4 "hardware ethernet" /var/dhcpd/var/db/dhcpd.leases
```

---

## Documentation Checklist

- [ ] All known lab hosts identified with MAC addresses
- [ ] Static mappings created per VLAN
- [ ] Each VM confirmed receiving correct static IP after renew
- [ ] Static mapping table saved to verification/static-mappings.md
