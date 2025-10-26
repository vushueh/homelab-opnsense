# OPNsense Lab Network - Quick Reference Guide

**Quick access to the most important information from the complete documentation**

---

## 🌐 Network Overview

```
Internet → Route10 → OPNsense (Firewall/Router) → Cisco Switch → VMs (Hyper-V & Proxmox)
              │                    │
              │                    └─→ Multiple VLANs (30, 40, 70, 250)
              │
              └─→ Production Network (VLAN 10, 20)
```

---

## 📊 VLAN Quick Reference

| VLAN | Network | Gateway | DHCP Range | Purpose | Hypervisors |
|------|---------|---------|------------|---------|-------------|
| **10** | 192.168.10.0/24 | .10.1 (Route10) | .10-.252 | WAN/Internet | Hyper-V |
| **20** | 192.168.20.0/24 | .20.1 (Route10) | .10-.252 | Production/Mgmt | Hyper-V |
| **30** | 192.168.30.0/24 | .30.1 (OPNsense) | .100-.200 | Lab Primary | Both |
| **40** | 192.168.40.0/24 | .40.1 (OPNsense) | .100-.200 | Lab Secondary | Both |
| **70** | 192.168.70.0/24 | .70.1 (OPNsense) | .100-.200 | Lab Additional | Both |
| **250** | 192.168.250.0/24 | .250.1 (OPNsense) | .100-.200 | Attack Lab | Both |

---

## 🔌 Interface Mapping

### OPNsense Interfaces

| Interface | Name | IP Address | Physical NIC | vSwitch | Purpose |
|-----------|------|------------|--------------|---------|---------|
| **hn2** | WAN | 192.168.10.32/24 | Intel I350 Port 1 | vSwitch-WAN | Internet |
| **hn1** | LAN | 192.168.30.1/24 | Intel I350 Port 2 | vSwitch-LAN | Lab VLANs (trunk) |
| **hn0** | OPT1 (Mgmt) | 192.168.10.199/24 | Intel I217-LM | External-VLAN-Trunk | Management |
| **vlan01** | LAN_VLAN30 | 192.168.30.1/24 | (on hn1) | - | VLAN 30 |
| **vlan02** | OPT2_VLAN40 | 192.168.40.1/24 | (on hn1) | - | VLAN 40 |
| **vlan03** | OPT3_VLAN70 | 192.168.70.1/24 | (on hn1) | - | VLAN 70 |
| **vlan04** | OPT4_VLAN250 | 192.168.250.1/24 | (on hn1) | - | VLAN 250 |

### Cisco Switch Ports

| Port | Description | Mode | VLANs | Connected To |
|------|-------------|------|-------|--------------|
| **Gi0/11** | Hyper-V-LAN-Trunk | Trunk | 30,40,70,250 | Hyper-V vSwitch-LAN |
| **Gi0/1** | Proxmox-eno1-Trunk | Trunk | 30,40,70,250 | Proxmox eno1 |

---

## 🔥 Firewall Rules Summary

### WAN Interface (hn2)

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | **Pass** | 192.168.10.0/24<br>192.168.20.0/24 | 192.168.250.0/24 | Any | Allow internal → attack lab |
| 2 | **Pass** | Any | 192.168.10.0/24<br>192.168.20.0/24 | Any | Return traffic |
| 3 | **Pass** | 192.168.10.0/24<br>192.168.20.0/24 | This Firewall | 443, 22 | Management access |
| * | **Block** | Any | Any | Any | Default deny (implicit) |

### LAN Interface (hn1 - VLAN 30)

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | **Pass** | LAN net | Any | Any | Allow all outbound |
| 2 | **Block** | LAN net | RFC1918 Private | Any | Block other private nets |

### Management Interface (hn0 - OPT1)

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | **Pass** | 192.168.20.0/24 | This Firewall | 443, 22 | Admin access |
| 2 | **Block** | OPT1 net | Any | Any | NO internet access |

---

## 🚀 Quick Start Commands

### Access OPNsense

```bash
# WebGUI
https://192.168.10.199   # Management interface (from VLAN 10/20)
https://192.168.30.1     # LAN interface (from VLAN 30)

# SSH
ssh root@192.168.10.199
ssh root@192.168.30.1

# Console (Hyper-V)
vmconnect.exe localhost OPNsense
```

### OPNsense Command Cheatsheet

```bash
# Interface status
ifconfig -a
ifconfig hn1
ifconfig vlan01

# Routing
netstat -rn
route -n show

# Firewall
pfctl -sr          # Show rules
pfctl -sn          # Show NAT
pfctl -ss          # Show states

# Packet capture
tcpdump -i hn1
tcpdump -i vlan01 icmp
tcpdump -i hn2 host 192.168.250.28

# Services
service -e
service sshd restart
service dhcpd restart

# Logs
clog /var/log/filter.log
clog /var/log/dhcpd.log
```

### Cisco Switch Commands

```cisco
# Access
ssh admin@[switch-ip]
enable
configure terminal

# VLAN Status
show vlan brief
show interface trunk
show interface Gi0/11 switchport

# MAC Addresses
show mac address-table
show mac address-table interface Gi0/1
show mac address-table vlan 30

# Troubleshooting
show interface status
show interface counters errors
show logging
show spanning-tree

# Save
write memory
copy running-config startup-config
```

### Proxmox Commands

```bash
# Network interfaces
ip link show
ip addr show eno1
ip link show vmbr_trunk

# Bridge VLAN info
bridge link show
bridge vlan show dev vmbr_trunk

# VMs
qm list
qm start [VMID]
qm stop [VMID]
qm config [VMID]

# Apply network changes
ifreload -a

# Logs
journalctl -xe
dmesg | grep -i network
```

### Hyper-V PowerShell

```powershell
# VM info
Get-VM "OPNsense"
Get-VMNetworkAdapter -VMName "OPNsense"
Get-VMNetworkAdapterVlan -VMName "OPNsense"

# vSwitch info
Get-VMSwitch
Get-VMSwitch "vSwitch-LAN" | Format-List *

# Start/Stop
Start-VM "OPNsense"
Stop-VM "OPNsense"
Restart-VM "OPNsense"

# Connect
vmconnect.exe localhost OPNsense
```

---

## 🔍 Troubleshooting Checklist

### VM Not Getting IP (DHCP Issue)

```bash
# On OPNsense
□ Check DHCP enabled: Services → DHCPv4 → [Interface]
□ Check DHCP range: Verify .100-.200 available
□ Check firewall: Interface rules allow DHCP (UDP 67/68)
□ Monitor DHCP: tcpdump -i [interface] port 67 or port 68

# On VM
□ Interface UP: ip link set eth0 up
□ Request DHCP: dhclient -v eth0
□ Check lease: cat /var/lib/dhcp/dhclient.leases
```

### Can't Ping Gateway

```bash
# On VM
□ Check gateway config: ip route
□ Correct gateway IP: 192.168.X.1
□ Ping test: ping -c4 192.168.X.1

# On OPNsense
□ Interface UP: ifconfig [interface]
□ Monitor traffic: tcpdump -i [interface] icmp
□ Check firewall: Firewall → Rules → [Interface]

# On Switch
□ Trunk configured: show interface Gi0/X switchport
□ VLAN allowed: switchport trunk allowed vlan X
□ Port status: show interface Gi0/X status
```

### No Internet Access

```bash
# On VM
□ Gateway reachable: ping 192.168.X.1  ✓
□ DNS configured: cat /etc/resolv.conf
□ Internet by IP: ping 8.8.8.8
□ Internet by name: ping google.com

# On OPNsense
□ WAN interface UP: ifconfig hn2
□ WAN has IP: 192.168.10.32
□ Internet from OPNsense: ping 8.8.8.8
□ NAT configured: Firewall → NAT → Outbound
□ Firewall allows: Firewall → Rules → [Interface]
```

### Inter-VLAN Routing Not Working

```bash
# Test connectivity
□ Ping from VLAN 30 to VLAN 250: ping 192.168.250.28
□ Traceroute shows path: traceroute 192.168.250.28

# On OPNsense
□ VLANs created: Interfaces → Other Types → VLAN
□ VLANs assigned: Interfaces → Assignments
□ VLANs enabled: Interfaces → [VLANX] → Enable ✓
□ Firewall allows: Firewall → Rules → Check both interfaces

# On Route10 (if crossing through it)
□ Static route exists: ip route show | grep 250
□ Route correct: 192.168.250.0/24 via 192.168.10.32
□ Ping OPNsense: ping 192.168.10.32
```

---

## 📈 Verification Tests

### Test 1: DHCP Functionality

```bash
# From test VM
dhclient -r eth0      # Release
dhclient -v eth0      # Request
ip addr show eth0     # Verify

# Expected: IP in range 192.168.X.100-200
```

### Test 2: Gateway Reachability

```bash
ping -c4 192.168.30.1    # VLAN 30 gateway
ping -c4 192.168.40.1    # VLAN 40 gateway

# Expected: 4 packets, 0% loss, <1ms latency
```

### Test 3: Internet Access

```bash
ping -c4 8.8.8.8         # Google DNS (by IP)
ping -c4 google.com      # Google (by name - tests DNS)
curl -I https://google.com   # HTTP test

# Expected: All succeed
```

### Test 4: Inter-VLAN Communication

```bash
# From VLAN 30 VM (192.168.30.120)
ping 192.168.250.28      # To VLAN 250

# From VLAN 250 VM (192.168.250.28)
ping 192.168.30.120      # Back to VLAN 30

# Expected: Both succeed (if firewall allows)
```

### Test 5: Cross-Hypervisor

```bash
# From Hyper-V VM on VLAN 40
ping 192.168.40.115      # Proxmox VM same VLAN

# Expected: Success (same VLAN, layer 2)
```

---

## 🎯 Common Challenges & Quick Fixes

### Challenge: VLANs Created in CLI Don't Show in GUI

**Problem**: Used `ifconfig vlan create` but VLANs missing from WebGUI

**Quick Fix**:
```bash
# Remove CLI VLANs
ifconfig vlan2 destroy
sysrc -x ifconfig_vlan2

# Use WebGUI instead: Interfaces → Other Types → VLAN → Add
```

### Challenge: Static Route Not Working

**Problem**: Can't reach VLAN 250 from VLAN 10/20

**Quick Fix**:
```bash
# On Route10
ip route add 192.168.250.0/24 via 192.168.10.32

# Verify
ip route show | grep 250
ping 192.168.250.1
```

### Challenge: Firewall Blocking Traffic

**Problem**: Traffic reaches OPNsense but doesn't pass through

**Quick Fix**:
- Check WAN rules: Firewall → Rules → WAN
- Add rule: Pass | Source: 192.168.10.0/24 | Dest: 192.168.250.0/24
- NOT: Destination → "This Firewall"
- YES: Destination → Specific network or "Any"

### Challenge: Proxmox VMs No Network

**Problem**: VMs created with VLAN tags can't communicate

**Quick Fix**:
```bash
# Edit /etc/network/interfaces
nano /etc/network/interfaces

# Add VLAN-aware bridge
auto vmbr_trunk
iface vmbr_trunk inet manual
    bridge-ports eno1
    bridge-vlan-aware yes
    bridge-vids 1-4094

# Apply
ifreload -a
```

---

## 📋 Configuration Backup Locations

### OPNsense

```
WebGUI: System → Configuration → Backups → Download
File: config-OPNsense-[date].xml
```

### Cisco Switch

```cisco
show running-config
copy running-config tftp:
# Or manually copy output to text file
```

### Proxmox

```bash
cat /etc/network/interfaces > proxmox-network-config.txt
```

### Hyper-V

```powershell
Get-VMSwitch | Export-Csv -Path "C:\Backup\HyperV-Switches.csv"
Get-VM "OPNsense" | Get-VMNetworkAdapter | Export-Csv -Path "C:\Backup\OPNsense-NICs.csv"
```

---

## 🔐 Security Checklist

- [ ] Changed default OPNsense password
- [ ] Management interface has NO internet access
- [ ] WAN interface has proper firewall rules
- [ ] Each VLAN has appropriate firewall rules
- [ ] NAT configured for all internal VLANs
- [ ] SSH key-based auth enabled (optional but recommended)
- [ ] Regular config backups scheduled
- [ ] IDS/IPS enabled on inspection interfaces (future)
- [ ] Logging configured and monitored
- [ ] Unused switch ports disabled

---

## 📞 Need More Details?

This is a **quick reference**. For detailed explanations, see:

- **Full Documentation**: `OPNsense-Lab-Network-Documentation.md`
- **Challenges & Solutions**: Main doc, "Challenges & Solutions" section
- **Complete Configs**: Main doc, "Step-by-Step Configuration" section
- **Traffic Flow Analysis**: Main doc, "Traffic Flow Analysis" section

---

## 🎓 Key Takeaways

1. **Always use WebGUI for VLAN creation** on OPNsense
2. **Static routes are essential** for multi-router setups
3. **Firewall rules must explicitly allow** inter-VLAN traffic
4. **VLAN tags must match** across hypervisor, switch, and firewall
5. **Separate management from data** to prevent lockouts
6. **Test incrementally** - verify each step before proceeding

---

**Document Version**: 1.0  
**Last Updated**: October 26, 2025  
**Quick Reference For**: OPNsense 24.7, Hyper-V, Proxmox VE 8.x

**Tip**: Keep this file open while configuring for quick command reference!
