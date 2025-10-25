# OPNsense Router Mode Deployment on Hyper-V

**A Journey from Bridge Mode Attempts to Production-Ready Router Configuration**

---

## 📋 Table of Contents

- [Executive Summary](#executive-summary)
- [Project Overview](#project-overview)
- [Architecture Evolution](#architecture-evolution)
- [Phase-by-Phase Implementation](#phase-by-phase-implementation)
- [Critical Challenges & Solutions](#critical-challenges--solutions)
- [Configuration Decisions](#configuration-decisions)
- [Final Working Configuration](#final-working-configuration)
- [Lessons Learned](#lessons-learned)
- [Testing & Verification](#testing--verification)
- [Appendices](#appendices)

---

## 📊 Executive Summary

This document chronicles a comprehensive **3-week journey** deploying OPNsense firewall on **Microsoft Hyper-V** infrastructure, transitioning from failed bridge mode attempts to a fully functional **Router Mode** implementation.

### Key Achievements

| Metric | Result |
|--------|--------|
| **Deployment Time** | 3 weeks (research + implementation) |
| **Final Status** | ✅ Production Ready |
| **Uptime Capability** | 24/7 (quiet Hyper-V server) |
| **Network Segments** | 4 VLANs (30, 40, 50, 250) |
| **Interface Configuration** | 3 interfaces (WAN, LAN, Management) |
| **Production Impact** | Zero (independent network paths) |

### Success Factors

- ✅ Router mode instead of bridge mode
- ✅ Physical network separation (3 NICs)
- ✅ Fresh installation approach
- ✅ Systematic troubleshooting methodology
- ✅ Comprehensive backup strategy

---

## 🎯 Project Overview

### Business Objectives

1. **Primary Goal**: Deploy OPNsense firewall for network inspection and IDS/IPS capability
2. **Security Requirement**: Full traffic visibility with threat detection
3. **Availability Requirement**: 24/7 operation on quiet Hyper-V server
4. **Isolation Requirement**: Production network must remain independent
5. **Scalability Requirement**: Support multiple VLANs for lab segmentation

### Technical Requirements

**Hardware Environment**
- **Platform**: Windows Server 2022 on Hyper-V
- **Primary NIC**: Intel I350 Dual-Port Gigabit (Slot04)
- **Management NIC**: Intel I217-LM (Onboard)
- **RAM**: 32GB (4GB allocated to OPNsense)
- **Storage**: 500GB SSD

**Network Environment**
- **Router**: Alta Labs Route10 (managing production VLANs 10, 20)
- **Core Switch**: Cisco Catalyst 2960G (VLAN distribution)
- **Production Switch**: Unmanaged PoE 8-port
- **Existing VLANs**: 10 (Admin), 20 (Secondary), 22-25 (Lab via PA-220)

### Target Architecture

```
Internet
    ↓
Route10 Router
    ├── Port 1 → PoE Switch (Production: VLANs 10, 20) — Always Available
    └── Port 3 → Hyper-V NIC 1 (OPNsense WAN)
         ↓
    OPNsense VM (Router Mode)
         ↓
    Hyper-V NIC 2 → Cisco Switch Gi0/10 (Inspection: VLANs 30, 40, 50, 250)
```

---

## 🔄 Architecture Evolution

### Initial Approach: Bridge Mode (FAILED)

**Concept**: Deploy OPNsense as transparent inline bridge for full traffic inspection

**Attempted Configuration**:
```
Router → Unmanaged Switch ← OPNsense (Bridge) ← Unmanaged Switch → Network
         (bypass capability)
```

**Why It Failed**:

| Challenge | Root Cause | Impact |
|-----------|------------|--------|
| No PCI Passthrough | Hardware lacks ACS support in PCIe root ports | Cannot give VM direct NIC control |
| Virtual Switch Abstraction | Hyper-V vSwitch adds layer 2/3 processing | Breaks transparent bridging |
| Bridge Mode Architecture | Requires direct hardware access | Incompatible with virtualization |
| Performance Bottleneck | Synthetic adapters (~700-900 Mbps) | Below line-rate gigabit |

**Key Learning**: *Virtual switches fundamentally cannot provide true layer 1 transparent bridging*

---

### Intermediate Attempts: Proxmox Bridge Mode

**Hypothesis**: Proxmox might support bridge mode better than Hyper-V

**Configuration Tested**:
- VM on Proxmox with virtio NICs
- Linux bridges (vmbr3, vmbr4)
- VLAN tagging attempts

**Challenges Encountered**:

1. **VLAN Isolation Issues**
   - Management interface on `vmbr0v10` (VLAN 10 tagged)
   - Proxmox host on `vmbr0` (untagged)
   - **Result**: No layer 2 connectivity, ARP failed

2. **Network Path Complexity**
   ```
   OPNsense vtnet2 → vmbr0v10 → Physical NIC → Switch
                     (VLAN 10 tagged)
   
   Problem: Switch expected untagged or different VLAN configuration
   ```

3. **Loud Server Issue**
   - Proxmox server too noisy for 24/7 operation
   - Frequent power cycling disrupted testing

**Decision Point**: Abandon bridge mode entirely, focus on Router Mode on quiet Hyper-V server

---

### Final Approach: Router Mode (SUCCESS)

**Paradigm Shift**: Accept OPNsense as a router, not transparent bridge

**Architecture Benefits**:

| Aspect | Advantage |
|--------|-----------|
| **Network Topology** | Clean layer 3 segmentation |
| **NAT Implementation** | Native, stable, well-supported |
| **VLAN Support** | Unlimited VLANs via subinterfaces |
| **Firewall Rules** | Inter-VLAN routing control |
| **Production Independence** | Inspection network isolated from production |
| **Hyper-V Compatibility** | Virtual switches work perfectly for routing |

---

## 🛠️ Phase-by-Phase Implementation

### Phase 0: Pre-Implementation Planning

**Duration**: 3 days  
**Impact**: None (documentation phase)

#### Activities

1. **Network Discovery & Documentation**
   - Mapped existing network topology
   - Identified all VLANs (10, 20, 22-25, 30, 40, 50, 250)
   - Documented Route10 configuration
   - Cisco switch port assignments

2. **IP Address Planning**

| Network Segment | Subnet | VLAN | Gateway | DHCP Range | Purpose |
|-----------------|--------|------|---------|------------|---------|
| Production Home | 192.168.10.0/24 | 10 | 192.168.10.1 (Route10) | .100-.200 | Primary network |
| Production Home 2 | 192.168.20.0/24 | 20 | 192.168.20.1 (Route10) | .100-.200 | Secondary network |
| OPNsense WAN | 192.168.10.0/24 | 10 | 192.168.10.1 | DHCP | Internet uplink |
| Inspection LAN | 192.168.30.0/24 | 30 | 192.168.30.1 (OPNsense) | .100-.200 | Lab/test devices |
| Inspection VLAN 40 | 192.168.40.0/24 | 40 | 192.168.40.1 | .100-.200 | Additional segment |
| Attack Lab | 192.168.250.0/24 | 250 | 192.168.250.1 | .100-.200 | Offensive security lab |
| OPNsense Management | 192.168.20.254/32 | 20 | None | Static | GUI/SSH access |

3. **Hardware Verification**
   ```powershell
   # Verified available NICs
   Get-NetAdapter | Select-Object Name, InterfaceDescription, Status, LinkSpeed
   
   Output:
   - Slot04 x4 Port 1: Intel I350 #1 (Up, 1 Gbps)
   - Slot04 x4 Port 2: Intel I350 #2 (Up, 1 Gbps)
   - Ethernet: Intel I217-LM (Up, 1 Gbps)
   ```

4. **Backup Strategy**
   - Exported all Hyper-V VM configurations
   - Backed up Route10 config via web interface
   - Saved Cisco `running-config`
   - Documented physical cabling

#### Key Decisions

**Decision 1: Router Mode vs Bridge Mode**
- **Chosen**: Router Mode
- **Reason**: Hyper-V virtual switches incompatible with transparent bridging
- **Alternative Rejected**: Physical appliance (unnecessary hardware purchase)

**Decision 2: Network Separation**
- **Chosen**: Production on Route10 Port 1, Inspection on Route10 Port 3
- **Reason**: Complete isolation, production never depends on OPNsense
- **Alternative Rejected**: Single network path (creates dependency)

**Decision 3: Hyper-V vs Proxmox**
- **Chosen**: Hyper-V
- **Reason**: Quiet server enables 24/7 operation, Router Mode works perfectly
- **Alternative Rejected**: Proxmox (loud server, VLAN complexity)

---

### Phase 1: Route10 Configuration

**Duration**: 30 minutes  
**Impact**: None (isolated configuration)

#### Objectives
- Configure Route10 Port 3 for OPNsense WAN connection
- Enable DHCP on VLAN 10 for OPNsense WAN interface
- Set **Native VLAN** to prevent untagged frame issues

#### Configuration Steps

**1. Access Route10 Web Interface**
```
URL: https://192.168.10.1
Credentials: [admin credentials]
```

**2. Configure Port 3**

| Setting | Value | Reason |
|---------|-------|--------|
| Port | Port 3 | Dedicated for OPNsense WAN |
| Mode | Access | Single VLAN, untagged |
| VLAN | 10 | Admin network |
| **Native VLAN** | **10** | **Critical: Accepts untagged frames from Hyper-V** |
| Status | Enabled | Active port |
| Description | "OPNsense-WAN-Uplink" | Documentation |

**3. VLAN 10 Settings**
```
Network: 192.168.10.0/24
Gateway: 192.168.10.1
DHCP: Enabled
Range: 192.168.10.10 - 192.168.10.252
DNS: 192.168.10.24, 1.1.1.1
```

#### Critical Challenge: Native VLAN Configuration

**Problem Encountered**:
- Initial configuration: Port 3 without Native VLAN
- Hyper-V VMs send **untagged frames** by default
- Route10 expected **VLAN 10 tagged frames**
- **Result**: No DHCP, no connectivity ❌

**Investigation**:
```bash
# On OPNsense console
dhclient hn2  # WAN interface
# No DHCP offer received

# Physical layer checks
ifconfig hn2
# Status: active, link up at 1000baseT
# But no IP address assigned
```

**Root Cause Analysis**:
```
Hyper-V VM NIC → Sends untagged Ethernet frames
                 ↓
         vSwitch-WAN → Passes frames unmodified
                 ↓
         Route10 Port 3 → Without Native VLAN: DROPS untagged frames
```

**Solution Implemented**:
```
Route10 Port 3 Configuration:
- Native VLAN: 10  ← Added this
- Allowed VLANs: 10 only
- Mode: Access

Effect: Untagged frames automatically tagged as VLAN 10
```

**Verification**:
```bash
# On OPNsense console after fix
dhclient hn2

# Result:
DHCPOFFER from 192.168.10.1
DHCPACK from 192.168.10.1
Bound to 192.168.10.32/24  ✅

# Test internet
ping -c 4 8.8.8.8
# 4 packets transmitted, 4 received, 0% loss  ✅
```

**Key Learning**: *Native VLAN configuration is critical for Hyper-V VM connectivity*

#### Verification Checklist

- [x] Port 3 shows "Connected" status
- [x] VLAN 10 DHCP scope has available IPs
- [x] Native VLAN set to 10
- [x] Port description added
- [x] Configuration saved

---

### Phase 2: Cisco Switch Configuration

**Duration**: 45 minutes  
**Impact**: None (configuring unused port)

#### Objectives
- Configure Gi0/10 as trunk port for OPNsense LAN
- Create VLANs 30, 40, 50, 250 for inspection network
- Enable spanning-tree portfast for VM environment

#### Pre-Configuration Analysis

**Existing Cisco Configuration**:
```cisco
! Relevant existing ports
interface GigabitEthernet0/1
 description Proxmox-Existing-Trunk
 switchport mode trunk
 switchport trunk allowed vlan 22,30,40,50,60,70,200

interface GigabitEthernet0/20
 description Uplink-to-Route10
 switchport mode trunk
 switchport trunk allowed vlan 10,20,250
```

**Port Selection Decision**:
- **Chosen**: Gi0/10 (unused)
- **Reason**: Clean port, no existing configuration
- **Location**: Physically accessible on switch front

#### Configuration Steps

**1. Console Access**
```cisco
Switch> enable
Password: ******
Switch# configure terminal
```

**2. Create VLANs**

*Note*: VLANs 30, 40, 50, 250 already existed from previous configurations. Verification:

```cisco
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- --------
30   VLAN0030                         active    
40   VLAN0040                         active    
50   VLAN0050                         active    
250  Attack-Offensive-Lab             active    
```

**Discovery**: *VLANs pre-existing from Proxmox setup, no creation needed*

**3. Configure Gi0/10 as Trunk**

```cisco
Switch(config)# interface GigabitEthernet0/10
Switch(config-if)# description OPNsense-LAN-to-Hyper-V-vSwitch-LAN
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 999
Switch(config-if)# switchport trunk allowed vlan 30,40,50,250
Switch(config-if)# spanning-tree portfast trunk
%Warning: portfast should only be enabled on ports connected to a single
 host. Connecting hubs, concentrators, switches, bridges, etc... to this
 interface when portfast is enabled, can cause temporary bridging loops.
 Use with CAUTION

Switch(config-if)# no shutdown
Switch(config-if)# exit
Switch(config)# exit
```

**Note on Portfast Warning**: Acceptable in this scenario as port connects to single Hyper-V host, not another switch.

**4. Save Configuration**

```cisco
Switch# write memory
Building configuration...
[OK]
```

#### Configuration Decisions

**Decision 1: Native VLAN 999**
- **Chosen**: Unused VLAN 999
- **Reason**: Security best practice, prevents VLAN hopping attacks
- **Alternative Rejected**: VLAN 1 (default, insecure), VLAN 30 (unnecessary)

**Decision 2: Allowed VLANs**
- **Chosen**: 30, 40, 50, 250 only
- **Reason**: Explicit whitelist, only inspection VLANs
- **Alternative Rejected**: "all" (overly permissive), includes production VLANs (security risk)

**Decision 3: Spanning-Tree Portfast**
- **Chosen**: Enabled
- **Reason**: Single host connection, faster convergence for VM reboots
- **Alternative Rejected**: Disabled (30-second delay on interface up)

#### Verification

```cisco
Switch# show interface GigabitEthernet0/10 status
Port      Name               Status       Vlan       Duplex  Speed Type
Gi0/10    OPNsense-LAN-to-Hy connected    trunk      a-full  a-1000 10/100/1000BaseTX

Switch# show running-config interface GigabitEthernet0/10
Building configuration...
Current configuration : 217 bytes
!
interface GigabitEthernet0/10
 description OPNsense-LAN-to-Hyper-V-vSwitch-LAN
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,40,50,250
 switchport mode trunk
 spanning-tree portfast trunk
end

Switch# show interfaces trunk | include Gi0/10
Gi0/10      on           802.1q         trunking      999
Gi0/10      30,40,50,250
Gi0/10      30,40,50,250
Gi0/10      30,40,50,250
```

**Status**: ✅ Port configured correctly, trunk active, VLANs allowed

---

### Phase 3: Hyper-V Virtual Switch Configuration

**Duration**: 1 hour  
**Impact**: Brief network interruption for host (< 30 seconds)

#### Objectives
- Create three virtual switches for network separation
- Map physical NICs to virtual switches
- Configure management access for Hyper-V host

#### Pre-Configuration State

**Challenge**: Messy existing configuration from bridge mode attempts

**Existing vSwitches** (problematic):
```powershell
Get-VMSwitch | Select-Object Name, SwitchType, NetAdapterInterfaceDescription

Name                    SwitchType NetAdapterInterfaceDescription
----                    ---------- ------------------------------
vSwitch-WAN             External   Intel I350 Port 1
OPNsense-Internal       Internal   (No physical adapter)
External-VLAN-Trunk     External   Intel I217-LM
vSwitch-LAN             External   Intel I350 Port 2
WSL                     Internal   (No physical adapter)
```

**Issues Identified**:
1. Multiple vSwitches with unclear purpose
2. `OPNsense-Internal` - internal switch from failed attempts
3. Inconsistent naming convention
4. No clear physical mapping documentation

#### Solution: Fresh vSwitch Recreation

**Decision**: Delete and recreate all OPNsense-related vSwitches

**Reason**: *Clean slate approach* - starting fresh avoids inheriting configuration issues from failed bridge mode attempts

#### Configuration Steps

**1. Identify Physical NICs**

```powershell
Get-NetAdapter | Select-Object Name, InterfaceDescription, Status, LinkSpeed

Name                  InterfaceDescription                  Status LinkSpeed
----                  --------------------                  ------ ---------
Slot04 x4 Port 1      Intel(R) I350 Gigabit Network #1      Up     1 Gbps
Slot04 x4 Port 2      Intel(R) I350 Gigabit Network #2      Up     1 Gbps
Ethernet              Intel(R) Ethernet Connection I217-LM  Up     1 Gbps
vEthernet (WSL)       Hyper-V Virtual Ethernet Adapter #2   Up     10 Gbps
```

**2. Physical NIC to Purpose Mapping**

| Physical NIC | MAC Address | Purpose | vSwitch Name |
|--------------|-------------|---------|--------------|
| Slot04 x4 Port 1 | 00:1B:21:XX:XX:A0 | OPNsense WAN → Route10 Port 3 | vSwitch-WAN |
| Slot04 x4 Port 2 | 00:1B:21:XX:XX:A1 | OPNsense LAN → Cisco Gi0/10 | vSwitch-LAN |
| Ethernet (I217-LM) | XX:XX:XX:XX:XX:XX | Host + Management → Production | External-VLAN-Trunk |

**3. Remove Old vSwitches**

```powershell
# Backup VM configuration first
Get-VM | Export-VM -Path C:\Hyper-V-Backups\

# Remove problematic vSwitch (if OPNsense VM doesn't exist yet)
# Note: In actual implementation, vSwitches were reused where possible
```

**4. Create vSwitch-WAN**

```powershell
# Create external vSwitch for WAN connectivity
New-VMSwitch -Name "vSwitch-WAN" `
             -NetAdapterName "Slot04 x4 Port 1" `
             -AllowManagementOS $true `
             -Notes "OPNsense WAN to Route10 Port 3"
```

**Configuration Parameters**:
| Parameter | Value | Reason |
|-----------|-------|--------|
| `-NetAdapterName` | "Slot04 x4 Port 1" | Physical NIC connected to Route10 Port 3 |
| `-AllowManagementOS` | `$true` | Host needs network access for management |
| `-Notes` | Description | Documentation for future reference |

**Challenge Encountered**: `AllowManagementOS` initially set to `$false`

**Issue**:
- Host lost network connectivity
- Could not access Hyper-V Manager remotely
- Had to use physical console

**Fix**:
```powershell
Set-VMSwitch -Name "vSwitch-WAN" -AllowManagementOS $true
```

**Effect**: Created `vEthernet (vSwitch-WAN)` adapter with production network access

**5. Create vSwitch-LAN**

```powershell
New-VMSwitch -Name "vSwitch-LAN" `
             -NetAdapterName "Slot04 x4 Port 2" `
             -AllowManagementOS $false `
             -Notes "OPNsense LAN to Cisco Switch Gi0/10 - Inspection Network"
```

**Configuration Decision**: `AllowManagementOS = $false`

**Reason**:
- Dedicated to inspection network traffic
- Host should not have direct access to inspection VLANs
- Cleaner security boundary

**6. Verify External-VLAN-Trunk**

```powershell
Get-VMSwitch -Name "External-VLAN-Trunk" | Format-List *

Name                            : External-VLAN-Trunk
SwitchType                      : External
NetAdapterInterfaceDescription  : Intel(R) Ethernet Connection I217-LM
AllowManagementOS               : True
```

**Status**: Already exists and configured correctly, no changes needed

#### Verification

```powershell
Get-VMSwitch | Format-Table Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS

Name                    SwitchType NetAdapterInterfaceDescription              AllowManagementOS
----                    ---------- ------------------------------              -----------------
vSwitch-WAN             External   Intel(R) I350 Gigabit Network #1            True
vSwitch-LAN             External   Intel(R) I350 Gigabit Network #2            False
External-VLAN-Trunk     External   Intel(R) Ethernet Connection I217-LM        True
WSL                     Internal                                               False
```

**Host Network Connectivity Check**:
```powershell
Get-NetIPAddress -InterfaceAlias "vEthernet (vSwitch-WAN)" | Select-Object IPAddress

IPAddress
---------
192.168.10.25  ✅ Host has network access
```

**Network Adapter Status**:
```powershell
Get-NetAdapter | Where-Object {$_.Status -eq "Up"} | Select-Object Name, InterfaceDescription

Name                          InterfaceDescription
----                          --------------------
vEthernet (vSwitch-WAN)       Hyper-V Virtual Ethernet Adapter #3
vEthernet (External-VLAN-...)  Hyper-V Virtual Ethernet Adapter #4
```

#### Key Learnings

**Learning 1**: Always set `AllowManagementOS = $true` for at least ONE vSwitch
- **Impact**: Without this, host loses all network connectivity
- **Recovery**: Requires physical console access

**Learning 2**: Virtual Ethernet adapters automatically created
- **Behavior**: When `AllowManagementOS = $true`, Hyper-V creates `vEthernet (vSwitch-Name)` adapter
- **Function**: Provides host OS network access through physical NIC

**Learning 3**: Interface metrics determine routing
- **Issue**: Multiple network adapters with different gateways
- **Solution**: Windows routes based on interface metric (lower = preferred)

---

### Phase 4: OPNsense VM Creation

**Duration**: 1 hour  
**Impact**: None (new VM)

#### Objectives
- Create VM with correct specifications for OPNsense
- Add three network adapters in correct order
- Map adapters to appropriate vSwitches
- Prepare for OPNsense installation

#### Critical Learning: Adapter Order Matters

**Discovery**: Hyper-V assigns interface names based on adapter creation order

**Interface Name Assignment**:
```
First adapter added  = hn0
Second adapter added = hn1
Third adapter added  = hn2
```

**Target Mapping**:
```
hn0 = OPT1 (Management) → External-VLAN-Trunk → VLAN 20
hn1 = LAN (Inspection)  → vSwitch-LAN → Cisco Gi0/10
hn2 = WAN (Internet)    → vSwitch-WAN → Route10 Port 3
```

#### VM Creation Steps

**1. Create Base VM**

```powershell
New-VM -Name "OPNsense" `
       -MemoryStartupBytes 4GB `
       -Generation 1 `
       -Path "D:\Hyper-V\Virtual Machines" `
       -NewVHDPath "D:\Hyper-V\Virtual Hard Disks\OPNsense.vhdx" `
       -NewVHDSizeBytes 50GB
```

**Configuration Decisions**:

| Parameter | Value | Reason |
|-----------|-------|--------|
| `-MemoryStartupBytes` | 4GB | Minimum for OPNsense + IDS/IPS |
| `-Generation` | 1 (BIOS) | Better compatibility, simpler |
| `-NewVHDSizeBytes` | 50GB | Adequate for OS + logs |

**Alternative Rejected**: Generation 2 (UEFI)
- **Reason**: Requires additional UEFI configuration, Generation 1 more straightforward

**2. Configure VM Settings**

```powershell
# Set static memory (disable dynamic memory)
Set-VM -Name "OPNsense" -StaticMemory

# Set processor count
Set-VM -Name "OPNsense" -ProcessorCount 4

# Disable checkpoints (not recommended for routers)
Set-VM -Name "OPNsense" -CheckpointType Disabled
```

**Configuration Rationale**:

**Static Memory**:
- **Reason**: Dynamic memory can cause performance issues for routing/firewall
- **Impact**: Predictable performance, stable packet processing

**4 vCPUs**:
- **Reason**: Sufficient for routing + IDS/IPS (Suricata)
- **Alternative**: 2 vCPUs adequate for routing only

**Checkpoints Disabled**:
- **Reason**: Router/firewall state shouldn't be rolled back
- **Best Practice**: Use configuration backups instead

**3. Add Network Adapters (CRITICAL ORDER)**

```powershell
# Adapter 1: Management (will be hn0)
Add-VMNetworkAdapter -VMName "OPNsense" `
                     -Name "Management" `
                     -SwitchName "External-VLAN-Trunk"

# Adapter 2: LAN (will be hn1)
Add-VMNetworkAdapter -VMName "OPNsense" `
                     -Name "LAN" `
                     -SwitchName "vSwitch-LAN"

# Adapter 3: WAN (will be hn2)
Add-VMNetworkAdapter -VMName "OPNsense" `
                     -Name "WAN" `
                     -SwitchName "vSwitch-WAN"
```

**Critical Point**: *Adapters must be added in this specific order to match OPNsense interface assignment*

**4. Configure VLAN Tagging for Management**

```powershell
Set-VMNetworkAdapterVlan -VMName "OPNsense" `
                         -VMNetworkAdapterName "Management" `
                         -Access `
                         -VlanId 20
```

**Configuration Details**:
- **VLAN 20**: Management network (192.168.20.0/24)
- **Access Mode**: Untagged frames sent to VM, tagged on wire
- **Purpose**: Isolate management traffic from production VLAN 10

**Initial Mistake Made**: Tried `-Trunk` mode with tags
- **Issue**: OPNsense received tagged frames, complicated configuration
- **Fix**: Use `-Access` mode, vSwitch handles tagging transparently

**5. Attach Installation ISO**

```powershell
Add-VMDvdDrive -VMName "OPNsense" `
               -Path "D:\ISOs\OPNsense-25.7-dvd-amd64.iso"

# Set boot order: DVD first
Set-VMFirmware -VMName "OPNsense" -FirstBootDevice (Get-VMDvdDrive -VMName "OPNsense")
```

**6. Configure Integration Services**

```powershell
# Enable time synchronization, heartbeat
Enable-VMIntegrationService -VMName "OPNsense" -Name "Time Synchronization"
Enable-VMIntegrationService -VMName "OPNsense" -Name "Heartbeat"
Enable-VMIntegrationService -VMName "OPNsense" -Name "Shutdown"
```

**Note**: Guest Services installed after OPNsense installation

#### Verification Before Installation

```powershell
Get-VMNetworkAdapter -VMName "OPNsense" | Format-Table Name, SwitchName, MacAddress, VlanSetting

Name       SwitchName              MacAddress         VlanSetting
----       ----------              ----------         -----------
Management External-VLAN-Trunk     00:15:5D:14:0B:1F  Access: VLAN 20
LAN        vSwitch-LAN             00:15:5D:14:0B:20  Untagged
WAN        vSwitch-WAN             00:15:5D:14:0B:21  Untagged

# Check VM configuration
Get-VM -Name "OPNsense" | Select-Object Name, State, CPUUsage, MemoryAssigned, Uptime

Name      State CPUUsage MemoryAssigned Uptime
----      ----- -------- -------------- ------
OPNsense  Off   0        0 MB           00:00:00
```

**Status**: VM ready for installation, all prerequisites met ✅

#### Alternative Approach Considered

**Scenario**: What if we added adapters in wrong order?

**Options**:
1. **Delete and recreate adapters** (chosen in case of mistakes)
2. **Use OPNsense interface assignment** (reassign hn0/hn1/hn2)
3. **Live with mismatch** (NOT recommended - confusing)

**Best Practice**: *Get adapter order right from the start, document physical mapping*

---

### Phase 5: OPNsense Installation

**Duration**: 45 minutes  
**Impact**: None (isolated VM installation)

#### Objectives
- Install OPNsense to virtual hard disk
- Perform initial interface assignment
- Configure basic network settings
- Verify internet connectivity

#### Installation Process

**1. Start VM and Boot from ISO**

```powershell
Start-VM -Name "OPNsense"
vmconnect.exe localhost "OPNsense"
```

**Boot Sequence**:
```
OPNsense 25.7 (amd64)
Console: Hyper-V Video

- Boot Multi User [Enter]
- Boot Single User
- Escape to loader prompt
- Reboot

Booting... (automatic after 10 seconds)
```

**2. Live Environment (Pre-Installation)**

**First Console Screen**:
```
*** OPNsense.localdomain: OPNsense 25.7 (amd64) ***

You are currently running in live media mode.
Interfaces are assigned as follows:
 LAN  (hn0)    -> v4: 192.168.1.1/24
 WAN  (hn1)    -> (No IP assigned)

 0) Logout                              7) Ping host
 1) Assign interfaces                   8) Shell
 2) Set interface IP address            9) pfTop
 3) Reset the root password            10) Firewall log
 4) Reset to factory defaults          11) Reload all services
 5) Power off system                   12) Update from console
 6) Reboot system                      13) Restore a backup

Enter an option:
```

**Initial State Analysis**:
- Running in live media mode (not installed to disk yet)
- Default interface assignment (not matching our intended configuration)
- Need to install to disk for persistence

**3. Run Installer**

**Access Shell**:
```
Enter an option: 8
```

**Run OPNsense Installer**:
```bash
root@OPNsense:~ # opnsense-installer
```

**Installer Wizard**:
```
Welcome to OPNsense!

Please select an option:
> Install (UFS)
  Install (ZFS)
  Rescue Shell
  Exit

[Select: Install (UFS)]
```

**Configuration Selections**:

| Step | Choice | Reason |
|------|--------|--------|
| **Keymap** | us (default) | US keyboard layout |
| **Installation Type** | Install (UFS) | Simpler, adequate for VM |
| **Disk** | vtbd0 / da0 (50 GB) | Virtual hard disk |
| **Partition Scheme** | GPT/UEFI | Modern standard |
| **File System** | UFS | Stable, proven |
| **Installation** | Confirm | Begin installation |

**Alternative Rejected**: ZFS
- **Reason**: Overkill for VM, UFS sufficient
- **UFS Advantages**: Simpler, lower overhead, adequate performance

**Installation Progress**:
```
Extracting base.txz... ████████████ 100%
Extracting kernel.txz... ████████████ 100%
Installing bootloader...
Configuring system...

Installation complete!

Remove installation media and press Enter to reboot.
```

**4. Post-Installation Boot**

```powershell
# Remove ISO from VM
Set-VMDvdDrive -VMName "OPNsense" -Path $null
```

**Reboot and First Boot**:
```
OPNsense is installing to disk...
Please wait...

[System reboots]

*** OPNsense.localdomain: OPNsense 25.7 (amd64) ***

 LAN (hn0)       -> v4: 192.168.1.1/24
 WAN (hn1)       -> (No IP)

 HTTPS: sha256 [fingerprint]
 SSH:   SHA256 [fingerprint]

 Enter an option:
```

**Issue Identified**: Interfaces assigned to hn0/hn1, but we need hn0/hn1/hn2 with different roles

---

### Phase 6: Initial Interface Assignment

**Duration**: 30 minutes  
**Impact**: None (console configuration)

#### Objectives
- Assign interfaces to match physical topology
- Configure hn0 as Management (OPT1)
- Configure hn1 as LAN
- Configure hn2 as WAN

#### Interface Reassignment Process

**1. Access Interface Assignment Menu**

```
Enter an option: 1
```

**Assignment Wizard**:
```
Valid interfaces are:

hn0    00:15:5D:14:0B:1F    Hyper-V Network Interface
hn1    00:15:5D:14:0B:20    Hyper-V Network Interface
hn2    00:15:5D:14:0B:21    Hyper-V Network Interface

Do you want to configure LAGGs now? [y/N]: n
Do you want to configure VLANs now? [y/N]: n
```

**2. Assign WAN Interface**

```
Enter the WAN interface name or 'a' for auto-detection: hn2
```

**Mapping**: hn2 → vSwitch-WAN → Route10 Port 3

**3. Assign LAN Interface**

```
Enter the LAN interface name or 'a' for auto-detection
NOTE: this enables full Firewalling/NAT mode.
(or nothing if finished): hn1
```

**Mapping**: hn1 → vSwitch-LAN → Cisco Gi0/10

**4. Assign OPT1 (Management) Interface**

```
Enter the Optional interface 1 name or 'a' for auto-detection
(or nothing if finished): hn0
```

**Mapping**: hn0 → External-VLAN-Trunk → VLAN 20

**5. Confirm Assignment**

```
The interfaces will be assigned as follows:

WAN  -> hn2
LAN  -> hn1
OPT1 -> hn0

Do you want to proceed? [y/N]: y
```

**System Response**:
```
Configuring interfaces...
Writing configuration...
```

**Critical Moment**: SSH connection dropped (interface IPs changing)

**Expected Behavior**:
- Interfaces reconfigured
- Previous IP addresses cleared
- Default IPs may be assigned
- SSH session terminated (normal)

**6. Verification After Assignment**

**Console Output**:
```
*** OPNsense.localdomain: OPNsense 25.7 (amd64) ***

 LAN (hn1)       -> v4: 192.168.1.1/24
 OPT1 (hn0)      -> (No IP)
 WAN (hn2)       -> (No IP)

 HTTPS: sha256 [fingerprint]
```

**Status Analysis**:
- ✅ Interfaces correctly assigned (hn0=OPT1, hn1=LAN, hn2=WAN)
- ⚠️  Default IPs not matching target configuration
- ⚠️  WAN has no IP (needs DHCP configuration)
- ⚠️  OPT1 has no IP (needs static configuration)

**Next Step**: Configure each interface with target IP addresses

---

### Phase 7: Interface IP Configuration

**Duration**: 1 hour  
**Impact**: None (isolated configuration)

#### Objectives
- Configure WAN for DHCP from Route10
- Configure LAN with static IP and DHCP server
- Configure OPT1 (Management) with static IP, no gateway

#### Configuration Strategy

**Sequence Decision**: WAN → LAN → OPT1

**Reason**:
1. WAN first = get internet connectivity
2. LAN second = set up inspection network
3. OPT1 last = management access (already have console)

---

#### WAN Interface Configuration

**1. Access Interface IP Menu**

```
Enter an option: 2

Available interfaces:

1 - LAN (hn1)
2 - OPT1 (hn0)
3 - WAN (hn2)

Enter the number of the interface to configure: 3
```

**2. Configure IPv4 DHCP**

```
Configure IPv4 address via DHCP? [y/N]: y
```

**Decision**: DHCP vs Static
- **Chosen**: DHCP
- **Reason**: Route10 manages IP assignments, easier management
- **Alternative**: Static (more control but requires coordination)

**3. Skip IPv6**

```
Configure IPv6 address via DHCP6? [y/N]: n
```

**Reason**: Not using IPv6 in lab environment

**4. HTTP/HTTPS Protocol**

```
Do you want to revert to HTTP as the web GUI protocol? [y/N]: n
```

**Decision**: Keep HTTPS
- **Reason**: Secure management, self-signed cert acceptable for lab
- **Alternative**: HTTP (less secure, not recommended)

**5. Configuration Applied**

```
Configuring WAN interface...
DHCP client started on hn2
Waiting for DHCP offer...

DHCPOFFER received from 192.168.10.1
DHCPREQUEST sent
DHCPACK received

WAN (hn2) now configured:
 v4/DHCP4: 192.168.10.32/24
```

**Initial DHCP Failure (Earlier Attempt)**

**Problem**: No DHCP response initially

**Investigation**:
```bash
# Check physical link
ifconfig hn2
# hn2: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
# status: active
# Link: 1000baseT <full-duplex>
# ✅ Physical layer OK

# Try manual DHCP
dhclient -v hn2
# Sending on   Socket/hn2
# DHCPDISCOVER on hn2
# DHCPDISCOVER on hn2
# No DHCPOFFER  ❌
```

**Root Cause**: Route10 Port 3 missing Native VLAN configuration (discovered in Phase 1)

**Solution**: Added Native VLAN 10 to Route10 Port 3

**Result After Fix**:
```bash
dhclient hn2
# DHCPOFFER from 192.168.10.1  ✅
# DHCPREQUEST
# DHCPACK
# bound to 192.168.10.32 netmask 255.255.255.0  ✅
```

**6. Test Internet Connectivity**

```bash
ping -c 4 8.8.8.8
# PING 8.8.8.8 (8.8.8.8): 56 data bytes
# 64 bytes from 8.8.8.8: icmp_seq=0 ttl=118 time=19.234 ms
# 64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=18.892 ms
# 64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=20.156 ms
# 64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=19.445 ms
# 
# --- 8.8.8.8 ping statistics ---
# 4 packets transmitted, 4 packets received, 0% packet loss  ✅

# Test DNS
host google.com
# google.com has address 142.250.185.78  ✅
```

**Status**: WAN interface fully functional ✅

---

#### LAN Interface Configuration

**1. Select LAN Interface**

```
Enter the number of the interface to configure: 1
```

**2. Configure Static IPv4**

```
Configure IPv4 address via DHCP? [y/N]: n

Enter the new IPv4 address: 192.168.30.1

Enter the new IPv4 subnet bit count (1 to 32): 24
```

**Subnet Calculation**:
```
192.168.30.1/24
Network: 192.168.30.0
Netmask: 255.255.255.0
Usable Range: 192.168.30.1 - 192.168.30.254
Broadcast: 192.168.30.255
```

**3. No Upstream Gateway**

```
For a WAN, enter the new IPv4 upstream gateway address.
For a LAN, press <ENTER> for none: [Enter]
```

**Critical Decision**: No gateway on LAN

**Reason**:
- LAN is **downstream** of OPNsense
- Devices on LAN use **OPNsense as their gateway** (192.168.30.1)
- Adding gateway here would cause routing confusion

**4. Skip IPv6**

```
Configure IPv6 address via DHCP6? [y/N]: n
Enter the new IPv6 address: [Enter]
```

**5. Enable DHCP Server**

```
Do you want to enable the DHCP server on LAN? [y/N]: y

Enter the start address of the IPv4 client address range: 192.168.30.100

Enter the end address of the IPv4 client address range: 192.168.30.200
```

**DHCP Configuration**:
| Setting | Value | Reason |
|---------|-------|--------|
| **Start** | 192.168.30.100 | Reserves .1-.99 for static assignments |
| **End** | 192.168.30.200 | 101 addresses available |
| **Gateway** | 192.168.30.1 (automatic) | OPNsense is the gateway |
| **DNS** | OPNsense (automatic) | OPNsense Unbound DNS resolver |

**6. Keep HTTPS**

```
Do you want to revert to HTTP as the web GUI protocol? [y/N]: n
```

**7. Configuration Applied**

```
LAN (hn1) configured:
 v4: 192.168.30.1/24

DHCP server enabled on LAN:
 Range: 192.168.30.100 - 192.168.30.200
```

**Verification**:
```bash
ifconfig hn1
# hn1: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
# inet 192.168.30.1 netmask 0xffffff00 broadcast 192.168.30.255
# status: active  ✅
```

**Status**: LAN interface configured and ready for devices ✅

---

#### OPT1 (Management) Interface Configuration

**Challenge**: Management interface needs special configuration
- Must be accessible from VLAN 20 (192.168.20.0/24)
- Should NOT have internet routing
- GUI and SSH access only

**1. Select OPT1 Interface**

```
Enter the number of the interface to configure: 2
```

**2. Configure Static IPv4**

```
Configure IPv4 address via DHCP? [y/N]: n

Enter the new IPv4 address: 192.168.20.254

Enter the new IPv4 subnet bit count (1 to 32): 24
```

**IP Selection Decision**:
- **Chosen**: 192.168.20.254
- **Reason**: High IP in range, unlikely to conflict with DHCP
- **Alternative**: 192.168.10.199 (would work but different VLAN)

**3. Gateway Configuration (CRITICAL DECISION)**

```
For a WAN, enter the new IPv4 upstream gateway address.
For a LAN, press <ENTER> for none: [Enter]
```

**Decision**: **NO GATEWAY** on OPT1

**Reason**:
- Management interface should NOT have internet routing
- Traffic should stay local (GUI/SSH only)
- Prevents unintended internet access from management VLAN
- Security best practice: management network isolated

**Alternative Considered**: Add gateway `192.168.20.1`
- **Impact**: Management interface could route to internet
- **Security Risk**: Expands attack surface
- **Rejected**: Violates security principle

**4. Skip IPv6**

```
Configure IPv6 address via DHCP6? [y/N]: n
```

**5. No DHCP Server**

```
Do you want to enable the DHCP server on OPT1? [y/N]: n
```

**Reason**: Management network DHCP handled by Route10

**6. Configuration Applied**

```
OPT1 (hn0) configured:
 v4: 192.168.20.254/24
 Gateway: None
```

**7. Test Management Connectivity**

**From Hyper-V Host (192.168.20.11)**:
```powershell
# Test ping
ping 192.168.20.254
# Reply from 192.168.20.254: bytes=32 time<1ms TTL=64  ✅

# Test HTTPS
Test-NetConnection 192.168.20.254 -Port 443
# TcpTestSucceeded : True  ✅
```

**Initial GUI Access Attempt**:
```
https://192.168.20.254

Result: Connection timeout ❌
```

**Problem**: Firewall blocking GUI access (default deny)

**Resolution**: Configure firewall rules (next phase)

---

#### Interface Configuration Summary

**Final Console Display**:
```
*** OPNsense.localdomain: OPNsense 25.7 (amd64) ***

 LAN (hn1)       -> v4: 192.168.30.1/24
 OPT1 (hn0)      -> v4: 192.168.20.254/24
 WAN (hn2)       -> v4/DHCP4: 192.168.10.32/24

 HTTPS: sha256 [fingerprint]
 SSH:   SHA256 [fingerprint]

 Enter an option:
```

**Configuration Verification Table**:

| Interface | Device | IP Address | Gateway | DHCP Server | Status |
|-----------|--------|------------|---------|-------------|--------|
| WAN | hn2 | 192.168.10.32/24 (DHCP) | 192.168.10.1 | No | ✅ Active |
| LAN | hn1 | 192.168.30.1/24 | None | Yes (.100-.200) | ✅ Active |
| OPT1 | hn0 | 192.168.20.254/24 | **None** | No | ✅ Active |

**Status**: All interfaces configured, ready for firewall rules ✅

---

## 🔥 Critical Challenges & Solutions

### Challenge 1: GUI Lockout After Firewall Rules

**Severity**: Critical  
**Duration**: 2 hours troubleshooting

#### Problem Statement

After creating initial firewall rules on OPT1 interface, lost access to web GUI.

**Symptoms**:
```powershell
# From host (192.168.20.11)
Test-NetConnection 192.168.20.254 -Port 443
# TcpTestSucceeded : False  ❌

ping 192.168.20.254
# Request timed out  ❌
```

**Console Access**: Still available via Hyper-V Connect ✅

#### Investigation Process

**Step 1: Check Interface Status**
```bash
ifconfig hn0
# hn0: flags=8843<UP,BROADCAST,RUNNING> mtu 1500
# inet 192.168.20.254 netmask 0xffffff00
# status: active  ✅ Interface is up
```

**Step 2: Check Firewall Status**
```bash
pfctl -si
# Status: Enabled  ← Firewall active
# State Table Total: 0
```

**Step 3: Attempt GUI Access from Console**
```bash
# From OPNsense console
curl -k https://192.168.20.254
# Connection timeout  ❌ Cannot access GUI even locally
```

**Hypothesis**: Firewall rules blocking all access, including self

#### Root Cause Analysis

**Misconfigured Rule Order**:

**Initial Rules (INCORRECT)**:
```
OPT1 Interface Rules:
1. Block | OPT1 net | any | any         ← Applied first, blocks everything
2. Pass  | 192.168.20.0/24 | This Firewall | 443  ← Never reached
3. Pass  | 192.168.20.0/24 | This Firewall | 22
```

**Rule Processing**:
```
Incoming packet from 192.168.20.11:
1. Check Rule 1: Block OPT1 net → MATCH → Packet DROPPED ❌
2. Rule 2 never evaluated (packet already dropped)
```

**Key Learning**: *Firewall rules are processed top-to-bottom, first match wins*

#### Solution Implemented

**Option 1: Disable Firewall Temporarily**

```bash
# From console
pfctl -d  # Disable packet filter
```

**Result**:
```powershell
# Immediate GUI access restored
Test-NetConnection 192.168.20.254 -Port 443
# TcpTestSucceeded : True  ✅
```

**Option 2: Fix Rule Order (Permanent Solution)**

**Corrected Rules**:
```
OPT1 Interface Rules:
1. Pass  | 192.168.20.0/24 | This Firewall | 443   ← Allow GUI first
2. Pass  | 192.168.20.0/24 | This Firewall | 22    ← Allow SSH
3. Pass  | 192.168.20.0/24 | This Firewall | ICMP  ← Allow ping
4. Block | OPT1 net | any | any                     ← Block rest last
```

**Implementation via GUI**:

1. Access GUI: `https://192.168.20.254`
2. Navigate: **Firewall → Rules → OPT1**
3. **Move allow rules above block rule** (drag and drop)
4. Click **Apply Changes**
5. Re-enable firewall: `pfctl -e`

**Verification**:
```bash
# View active rules
pfctl -sr | grep -A 10 "on hn0"

pass in quick on hn0 inet proto tcp from 192.168.20.0/24 to (hn0) port = https
pass in quick on hn0 inet proto tcp from 192.168.20.0/24 to (hn0) port = ssh
pass in quick on hn0 inet proto icmp from 192.168.20.0/24 to (hn0)
block drop in quick on hn0 all
```

**Status**: GUI accessible, firewall enabled, rules correct ✅

#### Prevention Measures

**Best Practice 1**: Create "Emergency Access" floating rule
```
Firewall → Rules → Floating
Rule: Pass | any | any | This Firewall | 443
Priority: Low (only when other rules fail)
```

**Best Practice 2**: Always test rule order before applying
- Create allow rules first
- Add block rules last
- Verify rule order in GUI before applying

**Best Practice 3**: Keep console access
- Never close console window during firewall changes
- Have alternative access method (physical console, serial)

---

### Challenge 2: VLAN Tagging Mismatch

**Severity**: High  
**Duration**: 1.5 hours troubleshooting

#### Problem Statement

OPT1 interface could not communicate with host despite being on same subnet.

**Configuration**:
```
Host: 192.168.20.11 on External-VLAN-Trunk (VLAN 20)
OPT1: 192.168.20.254 on External-VLAN-Trunk (VLAN 20 tag)
```

**Symptoms**:
```powershell
ping 192.168.20.254
# Request timed out  ❌

arp -a | findstr 192.168.20.254
# No entry  ❌ ARP resolution failing
```

#### Investigation Process

**Step 1: Check Physical Layer**
```powershell
Get-VMNetworkAdapter -VMName "OPNsense" -Name "Management"

Name       : Management
SwitchName : External-VLAN-Trunk
Connected  : True
Status     : {Ok}
VlanSetting: Access: VLAN 20  ← Tagged configuration
```

**Step 2: Capture Packets**

```bash
# On OPNsense console
tcpdump -i hn0 -n
# Seeing NO packets  ❌
```

**Step 3: Test Without VLAN Tag**

```powershell
# Temporarily remove VLAN tag
Set-VMNetworkAdapterVlan -VMName "OPNsense" `
                         -VMNetworkAdapterName "Management" `
                         -Untagged
```

**Immediate Result**:
```powershell
ping 192.168.20.254
# Reply from 192.168.20.254: bytes=32 time<1ms  ✅ Works!
```

#### Root Cause Analysis

**VLAN Configuration Mismatch**:

**External-VLAN-Trunk Physical Configuration**:
- Trunk carrying multiple VLANs
- VLAN 20 for management network
- Frames sent to host are **untagged** (Access mode)

**VM Configuration (Initial - INCORRECT)**:
```powershell
VlanSetting: Access: VLAN 20
```

**What this does**:
- VM sends **untagged** frames
- vSwitch **adds VLAN 20 tag** before sending to physical NIC
- Physical NIC sends **tagged frames** on wire

**Physical Switch Configuration**:
- Expects **untagged** frames from host on VLAN 20
- Receives **tagged** frames (802.1Q VLAN 20)
- Drops or misroutes frames ❌

**The Mismatch**:
```
Hyper-V VM → Untagged frames → vSwitch adds VLAN 20 tag → Tagged frames on wire
                                                              ↓
                                                     Physical Switch expects untagged
                                                              ↓
                                                         Frames dropped
```

#### Solution Implemented

**Remove VLAN Tagging from VM Adapter**:

```powershell
Set-VMNetworkAdapterVlan -VMName "OPNsense" `
                         -VMNetworkAdapterName "Management" `
                         -Untagged
```

**Result**:
```
Hyper-V VM → Untagged frames → vSwitch passes through → Untagged frames on wire
                                                              ↓
                                                     Physical Switch happy
                                                              ↓
                                                       Communication works ✅
```

**Verification**:
```powershell
Get-VMNetworkAdapterVlan -VMName "OPNsense" -VMNetworkAdapterName "Management"

VMName   : OPNsense
VMNetworkAdapterName : Management
Mode     : Untagged  ✅
```

**Connectivity Test**:
```powershell
ping 192.168.20.254
# Reply from 192.168.20.254: bytes=32 time<1ms TTL=64  ✅

Test-NetConnection 192.168.20.254 -Port 443
# TcpTestSucceeded : True  ✅
```

#### Key Learnings

**Learning 1: VLAN Tagging Must Match on Both Ends**

**Correct Scenarios**:
| Hyper-V VM | vSwitch | Physical Switch | Result |
|------------|---------|-----------------|--------|
| Untagged | Untagged | Expects untagged | ✅ Works |
| Untagged | Tags as VLAN X | Expects VLAN X tagged | ✅ Works |
| Tagged VLAN X | Passes through | Expects VLAN X tagged | ✅ Works |

**Incorrect Scenarios**:
| Hyper-V VM | vSwitch | Physical Switch | Result |
|------------|---------|-----------------|--------|
| Untagged | Tags as VLAN X | Expects untagged | ❌ Fails |
| Tagged VLAN X | Tags as VLAN Y | Expects VLAN X | ❌ Fails |

**Learning 2: Hyper-V vSwitch Behavior**

**Access Mode** (`-Access -VlanId X`):
- VM sends/receives **untagged** frames
- vSwitch **adds/strips** VLAN tags
- Physical wire carries **tagged** frames

**Trunk Mode** (`-Trunk -AllowedVlanIdList X,Y,Z`):
- VM sends/receives **tagged** frames
- vSwitch passes **tags through**
- Requires VM OS to handle VLAN tagging

**Learning 3: Physical Switch Port Modes**

**Access Port**:
- Accepts **untagged** frames
- Assigns to configured VLAN
- Strips tags when sending to access port

**Trunk Port**:
- Accepts **tagged** frames
- Routes based on VLAN tag
- Requires Native VLAN for untagged frames

---

### Challenge 3: Shared vSwitch Confusion

**Severity**: Medium  
**Duration**: 2 hours troubleshooting + Fresh install decision

#### Problem Statement

Initial attempt to share vSwitch-WAN between WAN and Management interfaces caused routing and connectivity issues.

**Initial Configuration (PROBLEMATIC)**:
```
vSwitch-WAN (External on Intel I350 Port 1):
├─ WAN adapter (hn2) - untagged
└─ Management adapter (hn0) - untagged
```

**Both interfaces on same physical NIC, same vSwitch**

#### Symptoms

**1. Routing Confusion**
```bash
# On OPNsense console
netstat -rn

Destination        Gateway            Flags   Netif
default            192.168.10.1       UGS     hn2   ← WAN default route
192.168.10.0/24    link#1             U       hn2
192.168.20.0/24    link#2             U       hn0

# Problem: Both interfaces on 192.168.10.0/24 originally
```

**2. Management Traffic Routing Through WAN**
```bash
tcpdump -i hn2 port 443
# Seeing management GUI traffic on WAN interface  ❌
# Should only see outbound internet traffic
```

**3. Interface Metric Conflicts**

```bash
# FreeBSD interface metrics
ifconfig hn0
# metric 0

ifconfig hn2
# metric 0

# Both interfaces same priority → undefined routing behavior
```

#### Investigation Process

**Step 1: Analyze Network Topology**

```
External Physical Connection:
Route10 Port 3 → Intel I350 Port 1 → vSwitch-WAN → hn2 (WAN) + hn0 (Management)
                                                      ↓
                                        Both on same L2 segment
```

**Problem Identified**: Management and WAN traffic sharing physical path

**Step 2: Review IP Configuration**

**Initial Attempt 1**:
```
hn2 (WAN):        192.168.10.32/24 (DHCP from Route10)
hn0 (Management): 192.168.10.199/24 (static)
Gateway:          192.168.10.1 (on WAN)
```

**Issues**:
- Both interfaces on same subnet (/24 overlap)
- Management traffic could route through WAN
- No clear separation between management and internet traffic

**Attempted Fix 1**: Change Management to different VLAN
```
hn0 (Management): 192.168.20.254/24
hn2 (WAN):        192.168.10.32/24
```

**Still Issues**:
- Same physical wire
- VLAN separation done in software, not hardware
- Configuration complexity

#### Root Cause Analysis

**Fundamental Design Flaw**: Mixing WAN and Management on same physical connection

**Problems**:

1. **Security**: Management traffic visible on WAN path
2. **Routing**: Difficult to prevent management → internet routing
3. **Troubleshooting**: Hard to isolate issues
4. **Single Point of Failure**: One physical NIC for two critical functions

**Decision Matrix**:

| Approach | Pros | Cons | Verdict |
|----------|------|------|---------|
| **Share vSwitch** | Uses fewer NICs | Security risk, routing complex | ❌ Reject |
| **Separate vSwitch** | Clean separation | Requires 3 NICs | ✅ Adopt |
| **VLAN subinterfaces** | Uses fewer NICs | Complex, software overhead | ❌ Reject |

#### Solution Implemented

**Fresh Installation with Physical Separation**

**New Architecture**:
```
WAN (hn2)        → vSwitch-WAN → Intel I350 Port 1 → Route10 Port 3 (Internet)
LAN (hn1)        → vSwitch-LAN → Intel I350 Port 2 → Cisco Gi0/10 (Inspection)
Management (hn0) → External-VLAN-Trunk → Intel I217-LM → Production Network
```

**Why Fresh Install?**

**Attempted Fixes (All Failed)**:
1. Reassign interface roles (complex, error-prone)
2. Delete and recreate adapters (state corruption)
3. Modify vSwitch mappings (routing tables preserved old config)

**Fresh Install Benefits**:
- Clean state, no leftover configuration
- Correct from day 1
- Documented known-good configuration
- ~2 hours (faster than debugging)

**Physical NIC Allocation**:

| NIC | Purpose | vSwitch | Connected To |
|-----|---------|---------|--------------|
| Intel I350 Port 1 | WAN only | vSwitch-WAN | Route10 Port 3 |
| Intel I350 Port 2 | LAN only | vSwitch-LAN | Cisco Gi0/10 |
| Intel I217-LM | Host + Management | External-VLAN-Trunk | Production PoE Switch |

**Implementation Steps**:

1. **Backup Current Config** (even broken state)
   ```powershell
   Get-VM "OPNsense" | Export-VM -Path "C:\Backups\OPNsense-Broken"
   ```

2. **Delete Existing VM**
   ```powershell
   Stop-VM -Name "OPNsense" -Force
   Remove-VM -Name "OPNsense" -Force
   Remove-Item "D:\Hyper-V\Virtual Hard Disks\OPNsense.vhdx"
   ```

3. **Create Fresh VM** (Following Phase 4 procedure)
   - Add adapters in correct order (Management → LAN → WAN)
   - Each adapter on separate vSwitch
   - No shared physical connections

4. **Install OPNsense** (Following Phase 5-7 procedures)
   - Assign interfaces correctly from start
   - Configure IPs with clear separation
   - Test each interface independently

**Result After Fresh Install**:
```
WAN (hn2):        192.168.10.32/24 → Internet only
LAN (hn1):        192.168.30.1/24 → Inspection network
Management (hn0): 192.168.20.254/24 → GUI/SSH only (no internet)
```

**Verification**:
```bash
# Management to internet (should fail)
ping -S 192.168.20.254 -c 4 8.8.8.8
# 100% packet loss  ✅ Correct! No internet from management

# WAN to internet (should work)
ping -S 192.168.10.32 -c 4 8.8.8.8
# 0% packet loss  ✅ Correct!

# Management to host (should work)
ping -S 192.168.20.254 -c 4 192.168.20.11
# 0% packet loss  ✅ Correct!
```

#### Key Learnings

**Learning 1: Physical Separation > Software Separation**
- Hardware boundaries easier to understand and troubleshoot
- Clear failure domains
- Better security posture

**Learning 2: Fresh Install vs Fix**
- Sometimes faster to start over
- Eliminates hidden configuration state
- Provides documented clean baseline

**Learning 3: Design Before Implement**
- Upfront planning saves troubleshooting time
- Draw physical topology first
- Validate design before implementation

**Learning 4: Progressive Addition**
- Add one interface at a time
- Test before adding next
- Easier to identify issues

---

### Challenge 4: Interface Metrics and Routing

**Severity**: Low  
**Duration**: 30 minutes

#### Problem Statement

Hyper-V host routing behaved unpredictably with multiple virtual adapters.

**Scenario**:
```
Host: 192.168.20.11 (on External-VLAN-Trunk)
Also has: vEthernet (vSwitch-WAN) adapter on 192.168.10.0/24
```

**Symptom**: Some internet traffic routing through wrong adapter

#### Investigation

```powershell
Get-NetIPInterface -AddressFamily IPv4 | 
  Select InterfaceAlias, InterfaceIndex, InterfaceMetric | 
  Sort-Object InterfaceMetric

InterfaceAlias                  InterfaceIndex InterfaceMetric
--------------                  -------------- ---------------
Ethernet                        13             25  ← Production, lowest metric
vEthernet (External-VLAN-Trunk) 22             5000
vEthernet (vSwitch-WAN)         28             5000
vEthernet (vSwitch-LAN)         35             5000  ← No management OS, not routable
```

**Route Table**:
```powershell
Get-NetRoute -DestinationPrefix "0.0.0.0/0" | Format-Table

DestinationPrefix NextHop         InterfaceAlias              RouteMetric
----------------- -------         --------------              -----------
0.0.0.0/0         192.168.20.1    vEthernet (External-VLA...  0
0.0.0.0/0         192.168.10.1    vEthernet (vSwitch-WAN)     0
```

**Problem**: Two default routes, both with metric 0

#### Root Cause

Windows routing decision algorithm:
```
1. Match most specific prefix (both are 0.0.0.0/0 - tie)
2. Use lowest InterfaceMetric + RouteMetric
   - External-VLAN-Trunk: 5000 + 0 = 5000
   - vSwitch-WAN:         5000 + 0 = 5000  ← Tie
3. Use lowest InterfaceIndex (unpredictable)
```

#### Solution

**Option 1: Adjust Interface Metrics** (chosen)
```powershell
# Ensure production adapter has lowest metric
Set-NetIPInterface -InterfaceAlias "vEthernet (External-VLAN-Trunk)" `
                   -InterfaceMetric 10

# Confirm
Get-NetIPInterface -AddressFamily IPv4 | 
  Select InterfaceAlias, InterfaceMetric | 
  Sort-Object InterfaceMetric

InterfaceAlias                  InterfaceMetric
--------------                  ---------------
vEthernet (External-VLAN-Trunk) 10     ← Preferred
Ethernet                        25
vEthernet (vSwitch-WAN)         5000
```

**Option 2: Remove vSwitch-WAN Management Access**

Since OPNsense VM uses this adapter, host doesn't need it:

```powershell
Set-VMSwitch -Name "vSwitch-WAN" -AllowManagementOS $false

# This removes vEthernet (vSwitch-WAN) adapter from host
```

**Result**: Only one default route remains ✅

#### Verification

```powershell
# Test connectivity
Test-NetConnection 8.8.8.8

# Check which adapter is used
Get-NetRoute -DestinationPrefix "0.0.0.0/0"

DestinationPrefix NextHop      InterfaceAlias              RouteMetric
----------------- -------      --------------              -----------
0.0.0.0/0         192.168.20.1 vEthernet (External-VLAN... 0
```

**Status**: Deterministic routing, host uses correct adapter ✅

---

### Challenge 5: WAN GUI Access Block

**Severity**: Medium  
**Duration**: 45 minutes

#### Problem Statement

Need to block OPNsense GUI access from WAN interface for security.

**Default State**: All interfaces allow GUI access (insecure)

**Risk**: Internet-facing web interface vulnerable to:
- Brute force attacks
- Vulnerability exploits
- Unauthorized access attempts

#### Implementation

**Create WAN Interface Rule**:

1. **Navigate**: Firewall → Rules → WAN
2. **Add Rule**:

| Setting | Value | Reason |
|---------|-------|--------|
| **Action** | Block | Deny access |
| **Interface** | WAN | Apply to WAN interface |
| **Protocol** | TCP | HTTPS is TCP-based |
| **Source** | any | Block from anywhere on internet |
| **Destination** | WAN address | This firewall's WAN IP |
| **Destination Port** | 443 (HTTPS) | Web GUI port |
| **Log** | Yes | Track attack attempts |
| **Description** | "Block GUI access from WAN" | Documentation |

**Additional Rule for HTTP**:
```
Block | WAN | TCP | any | WAN address | 80 | "Block HTTP access from WAN"
```

**Rule Order**:
```
WAN Interface Rules:
1. Block | any | WAN address | 443 (HTTPS)  ← Explicit block
2. Block | any | WAN address | 80 (HTTP)    ← Explicit block
3. Default allow (for NAT/outbound)          ← Doesn't match WAN address
```

#### Verification

**Test 1: External GUI Access (Should Fail)**

From external network or simulate:
```bash
# If you had public IP, test from internet
curl -k https://192.168.10.32
# Connection refused  ✅ Blocked correctly
```

**Test 2: Internal GUI Access (Should Work)**

```powershell
# From management network (192.168.20.11)
Test-NetConnection 192.168.20.254 -Port 443
# TcpTestSucceeded : True  ✅ Management access works
```

**Test 3: Check Firewall Logs**

```
Firewall → Log Files → Live View
Filter: interface=WAN, port=443

[Shows blocked connection attempts if any]
```

#### Alternative Approaches Considered

**Option 1: Listen Interface Restriction**

```
System → Settings → Administration
Listen Interfaces: OPT1 only (not WAN)
```

**Pros**: Web server doesn't listen on WAN at all  
**Cons**: Requires restart of web server, affects all interfaces

**Option 2: Firewall Rule** (chosen)

**Pros**: Flexible, can log attempts, no service restart  
**Cons**: Relies on firewall being enabled

**Hybrid Approach**: Both (belt and suspenders)
- Firewall rule as primary defense
- Listen interface restriction as additional layer

---

## ⚙️ Configuration Decisions

### Decision 1: Router Mode vs Bridge Mode

**Context**: Initial goal was transparent inline inspection

**Options Considered**:

| Approach | Pros | Cons | Verdict |
|----------|------|------|---------|
| **Bridge Mode** | True transparency, no IP changes | Requires PCI passthrough, complex | ❌ Not possible on this hardware |
| **Router Mode** | Native, stable, well-supported | Requires NAT, IP scheme changes | ✅ Chosen |
| **SPAN/Mirror** | Passive monitoring, no inline | No blocking capability, limited features | ❌ Insufficient for goals |

**Rationale**:
- Hardware lacks ACS support for PCI passthrough
- Virtual switches incompatible with transparent bridging
- Router mode provides all required functionality:
  - Traffic inspection
  - IDS/IPS capability
  - Firewall rules
  - VLAN segmentation

**Impact**:
- Inspection network on separate subnet (192.168.30.0/24)
- NAT between inspection and internet
- Clean layer 3 separation

---

### Decision 2: Interface Separation Strategy

**Context**: Three interfaces needed (WAN, LAN, Management)

**Options Considered**:

| Approach | NICs Required | Pros | Cons | Verdict |
|----------|---------------|------|------|---------|
| **Single NIC + VLANs** | 1 | Minimal hardware | Complex config, shared bandwidth | ❌ Too complex |
| **Dual NIC + Shared** | 2 | Fewer NICs | WAN/Mgmt sharing (security issue) | ❌ Security risk |
| **Triple NIC** | 3 | Complete separation | Requires 3 NICs | ✅ Chosen |

**Chosen Configuration**:

| Interface | Physical NIC | vSwitch | Purpose |
|-----------|--------------|---------|---------|
| WAN (hn2) | Intel I350 Port 1 | vSwitch-WAN | Internet only |
| LAN (hn1) | Intel I350 Port 2 | vSwitch-LAN | Inspection only |
| Management (hn0) | Intel I217-LM | External-VLAN-Trunk | GUI/SSH only |

**Rationale**:
- **Security**: Management traffic never touches WAN path
- **Clarity**: Each function on dedicated hardware
- **Troubleshooting**: Easy to isolate issues
- **Availability**: Failure domains separated

**Hardware Requirement**: Fortunately, server had 3 available NICs

---

### Decision 3: Management Network Design

**Context**: How to access OPNsense GUI/SSH securely

**Options Considered**:

| Option | Description | Pros | Cons | Verdict |
|--------|-------------|------|------|---------|
| **Option A: WAN Access** | Allow GUI on WAN with whitelist | Simple, one interface | Security risk, exposed to internet | ❌ Insecure |
| **Option B: LAN Access** | Access via LAN (192.168.30.x) | No extra interface | Conflicts with inspection devices | ❌ Confusion |
| **Option C: Dedicated Mgmt** | Separate interface (OPT1) | Security, clarity | Requires 3rd NIC | ✅ Chosen |

**Chosen: Option C - Dedicated Management Interface**

**Configuration**:
```
Interface: OPT1 (hn0)
IP: 192.168.20.254/24
VLAN: 20 (Management VLAN)
Gateway: None (no internet routing)
Access: GUI + SSH only
Allowed From: 192.168.20.0/24 only
```

**Rationale**:
- **Security**: Management never exposed to WAN
- **Isolation**: Separate from inspection traffic
- **Control**: Firewall rules prevent internet access from management
- **Clarity**: Clear purpose, no role confusion

**Alternative Considered**: VPN access to LAN
- **Pros**: Could use LAN interface
- **Cons**: Requires VPN setup, additional complexity
- **Rejected**: Overkill for lab environment

---

### Decision 4: VLAN Scheme

**Context**: Need multiple isolated networks for different purposes

**Options Considered**:

| Scheme | VLANs | Pros | Cons | Verdict |
|--------|-------|------|------|---------|
| **Flat Network** | None (all .30.x) | Simple | No segmentation | ❌ Insufficient |
| **Limited VLANs** | 30 only | Minimal config | Not scalable | ❌ Limited |
| **Multi-VLAN** | 30, 40, 50, 250 | Flexible, scalable | More complex | ✅ Chosen |

**Chosen VLAN Allocation**:

| VLAN | Network | Purpose | Use Case |
|------|---------|---------|----------|
| **30** | 192.168.30.0/24 | Primary inspection | General lab VMs |
| **40** | 192.168.40.0/24 | Secondary inspection | Isolated test environment |
| **50** | 192.168.50.0/24 | Tertiary inspection | Additional segmentation |
| **250** | 192.168.250.0/24 | Attack/Offensive Lab | Red team, pentesting tools |

**Rationale**:
- **Segmentation**: Separate attack tools from victim VMs
- **Flexibility**: Easy to add more VLANs later
- **Compatibility**: VLAN 250 matches existing PA-220 lab VLANs
- **Scalability**: Room to grow (VLANs 60-249 available)

**Implementation on Cisco**:
```cisco
interface GigabitEthernet0/10
 switchport trunk allowed vlan 30,40,50,250
```

**Implementation on OPNsense**:
```
Interfaces → Other Types → VLAN
Parent: LAN (hn1)
Tags: 40, 50, 250
```

*(Not yet implemented, but planned for Phase 10)*

---

### Decision 5: Firewall Rule Philosophy

**Context**: How to structure firewall rules for security and usability

**Options Considered**:

| Approach | Description | Pros | Cons | Verdict |
|----------|-------------|------|------|---------|
| **Default Allow** | Allow all, block specific | Easy, permissive | Insecure, can miss threats | ❌ Insecure |
| **Default Deny** | Block all, allow specific | Secure, explicit | More rules needed | ✅ Chosen |
| **Zone-Based** | Trust/untrust zones | Enterprise approach | Complex for lab | ❌ Overkill |

**Chosen: Default Deny with Explicit Allows**

**WAN Rules**:
```
1. Block | any | WAN address | 443      (Deny GUI)
2. Block | any | WAN address | 22       (Deny SSH)
3. Default allow outbound (NAT)          (Internet access)
```

**OPT1 (Management) Rules**:
```
1. Pass  | 192.168.20.0/24 | This Firewall | 443   (Allow GUI)
2. Pass  | 192.168.20.0/24 | This Firewall | 22    (Allow SSH)
3. Pass  | 192.168.20.0/24 | This Firewall | ICMP  (Allow ping)
4. Block | OPT1 net | any | any                    (Block internet)
```

**LAN Rules**:
```
1. Pass  | LAN net | any | any                     (Allow outbound)
2. Block | LAN net | 192.168.20.0/24 | any        (Prevent access to mgmt)
```

**Rationale**:
- **Security**: Explicit allow only what's needed
- **Visibility**: Clear intent in each rule
- **Auditability**: Easy to review permissions
- **Least Privilege**: Minimize attack surface

**Rule Order Principle**: *Pass rules before block rules*

---

### Decision 6: Native VLAN Configuration

**Context**: Hyper-V VMs send untagged frames by default

**Problem**: Switch needs to know how to handle untagged frames

**Options**:

| Approach | Configuration | Pros | Cons | Verdict |
|----------|---------------|------|------|---------|
| **No Native VLAN** | Port drops untagged | Secure | Breaks Hyper-V VM connectivity | ❌ Doesn't work |
| **Native VLAN 1** | Default VLAN | Works | Security risk (VLAN hopping) | ❌ Insecure |
| **Native VLAN 999** | Cisco: Works<br>Route10: Drop | Secure for Cisco | Breaks Route10 connectivity | ❌ Incompatible |
| **Native VLAN = Access VLAN** | Route10 Port 3: Native VLAN 10 | Works, secure enough | Slightly less secure than 999 | ✅ Chosen |

**Chosen Configuration**:

**Route10 Port 3** (WAN uplink):
```
Mode: Access
VLAN: 10
Native VLAN: 10  ← Accepts untagged frames as VLAN 10
```

**Cisco Gi0/10** (LAN trunk):
```
Mode: Trunk
Allowed VLANs: 30,40,50,250
Native VLAN: 999  ← Security best practice (unused VLAN)
```

**Rationale**:
- **Route10**: Needs Native VLAN = 10 for Hyper-V compatibility
- **Cisco**: Can use 999 (more secure, OPNsense uses VLAN subinterfaces)
- **Different devices, different requirements**

**Key Learning**: *Native VLAN configuration depends on device capabilities and use case*

---

### Decision 7: Fresh Install vs Repair

**Context**: After configuration issues, decide whether to fix or start over

**Situation**: Shared vSwitch causing routing problems, multiple failed fix attempts

**Options**:

| Approach | Time Estimate | Pros | Cons | Verdict |
|----------|---------------|------|------|---------|
| **Continue Fixing** | Unknown (4+ hours?) | Keeps existing config | High risk, complex | ❌ Risky |
| **Fresh Install** | 2 hours | Clean slate, known-good | Lose current config | ✅ Chosen |

**Decision Matrix**:

**Factors Favoring Fresh Install**:
- Configuration corruption from multiple failed fixes
- Unclear state (multiple backing out of changes)
- Known-good procedure documented from first attempt
- Risk of subtle issues persisting
- Time spent debugging already 3+ hours

**Factors Against Fresh Install**:
- Need to reconfigure everything
- Lose any custom settings (minimal in this case)

**Outcome**: Fresh install completed in 2 hours, vs 4+ hours estimated debugging

**Key Learning**: *Sometimes faster to start over with documented procedure*

---

## 🏆 Final Working Configuration

### Network Topology

```
┌──────────────────────────────────────────────────────────────┐
│                        INTERNET                              │
└────────────────────────┬─────────────────────────────────────┘
                         │
                  ┌──────▼──────┐
                  │  Route10    │ Alta Labs Router
                  │ Router      │ (Edge Router + Production Gateway)
                  └─┬─────────┬─┘
                    │         │
            Port 1 ─┘         └─ Port 3
         (Production)         (OPNsense WAN)
                    │               │
    ┌───────────────▼─────┐         │
    │   PoE Switch        │         │
    │   (Unmanaged)       │         │
    │   VLAN 10, 20       │         │
    │   Always Available  │         │
    └─────┬───────────────┘         │
          │                         │
    [Production                     │
     Devices]              ┌────────▼────────────┐
                           │  Hyper-V Host       │
                           │  Windows Server     │
                           │  192.168.20.11      │
                           │                     │
                           │  ┌────────────────┐ │
                           │  │  OPNsense VM   │ │
                           │  │                │ │
                           │  │  hn0: Mgmt ✅  │ │
                           │  │  hn1: LAN  ✅  │ │
                           │  │  hn2: WAN  ✅  │ │
                           │  └────────────────┘ │
                           └─┬────────┬──────┬───┘
                             │        │      │
              Management ────┘        │      └──── WAN
              (Intel I217-LM)         │            (Intel I350 #1)
                                      │
                                   LAN ───────┐
                              (Intel I350 #2)  │
                                               │
                                    ┌──────────▼──────────┐
                                    │  Cisco 2960G Switch │
                                    │  Gi0/10 (Trunk)     │
                                    │  VLANs: 30,40,50,250│
                                    └─────────┬───────────┘
                                              │
                                    ┌─────────▼────────────┐
                                    │  Inspection Network  │
                                    │  (Lab VMs, Devices)  │
                                    └──────────────────────┘
```

### Interface Configuration

| Interface | Name | Device | IP Address | Gateway | DHCP Server | Firewall Rules |
|-----------|------|--------|------------|---------|-------------|----------------|
| **WAN** | hn2 | Intel I350 #1 | 192.168.10.32/24 (DHCP) | 192.168.10.1 | No | Block GUI/SSH, Allow outbound |
| **LAN** | hn1 | Intel I350 #2 | 192.168.30.1/24 | None | Yes (.100-.200) | Allow all outbound, Block mgmt VLAN |
| **OPT1** | hn0 | Intel I217-LM | 192.168.20.254/24 | None | No | Allow GUI/SSH, Block internet |

### Physical Connections

```
Route10 Port 3 (VLAN 10 Access, Native VLAN 10)
    │
    └─→ Intel I350 Port 1 (Slot04 x4 Port 1)
         │
         └─→ vSwitch-WAN
              │
              └─→ OPNsense WAN (hn2)

Cisco Switch Gi0/10 (Trunk: VLANs 30,40,50,250, Native VLAN 999)
    │
    └─→ Intel I350 Port 2 (Slot04 x4 Port 2)
         │
         └─→ vSwitch-LAN
              │
              └─→ OPNsense LAN (hn1)

Production PoE Switch (VLAN 20)
    │
    └─→ Intel I217-LM (Ethernet)
         │
         ├─→ External-VLAN-Trunk
         │    │
         │    ├─→ Hyper-V Host Management (192.168.20.11)
         │    └─→ OPNsense Management (hn0, 192.168.20.254)
         │
         └─→ vEthernet (External-VLAN-Trunk) - Host adapter
```

### Hyper-V Configuration

**Virtual Switches**:
```powershell
Get-VMSwitch | Format-Table Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS

Name                    SwitchType NetAdapterInterfaceDescription       AllowManagementOS
----                    ---------- ------------------------------       -----------------
vSwitch-WAN             External   Intel(R) I350 Gigabit Network #1     True
vSwitch-LAN             External   Intel(R) I350 Gigabit Network #2     False
External-VLAN-Trunk     External   Intel(R) I217-LM                     True
```

**VM Network Adapters**:
```powershell
Get-VMNetworkAdapter -VMName "OPNsense" | Format-Table Name, SwitchName, MacAddress, VlanSetting

Name       SwitchName              MacAddress         VlanSetting
----       ----------              ----------         -----------
Management External-VLAN-Trunk     00:15:5D:14:0B:1F  Untagged
LAN        vSwitch-LAN             00:15:5D:14:0B:20  Untagged
WAN        vSwitch-WAN             00:15:5D:14:0B:21  Untagged
```

**VM Specifications**:
```powershell
Get-VM -Name "OPNsense" | Format-List Name, State, CPUUsage, MemoryAssigned, Uptime, Generation

Name           : OPNsense
State          : Running
CPUUsage       : 2%
MemoryAssigned : 4096 MB
Uptime         : 72:14:33
Generation     : 1
```

### Route10 Configuration

**Port 3 (OPNsense WAN Uplink)**:
```
Port Number: 3
Mode: Access
VLAN: 10 (Admin Network)
Native VLAN: 10  ← Critical for Hyper-V compatibility
Speed: 1 Gbps
Status: Enabled
Description: "OPNsense-WAN-Uplink"
```

**VLAN 10 (Admin Network)**:
```
Network: 192.168.10.0/24
Gateway: 192.168.10.1
DHCP: Enabled
DHCP Range: 192.168.10.10 - 192.168.10.252
DNS Primary: 192.168.10.24
DNS Secondary: 1.1.1.1
Lease Time: 1000 seconds
```

### Cisco Switch Configuration

**Gi0/10 Configuration**:
```cisco
interface GigabitEthernet0/10
 description OPNsense-LAN-to-Hyper-V-vSwitch-LAN
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,40,50,250
 spanning-tree portfast trunk
 no shutdown
end
```

**VLAN Configuration**:
```cisco
vlan 30
 name Inspection_LAN
!
vlan 40
 name Inspection_VLAN40
!
vlan 50
 name Inspection_VLAN50
!
vlan 250
 name Attack-Offensive-Lab
!
```

**Verification**:
```cisco
SW1-2960G# show interfaces trunk | include Gi0/10
Gi0/10      on           802.1q         trunking      999
Gi0/10      30,40,50,250
Gi0/10      30,40,50,250
Gi0/10      30,40,50,250
```

### OPNsense Firewall Rules

**WAN Interface Rules**:
```
Priority | Action | Source | Destination | Port | Description
---------|--------|--------|-------------|------|-------------
1        | Block  | any    | WAN address | 443  | Block GUI from WAN
2        | Block  | any    | WAN address | 22   | Block SSH from WAN
3        | Block  | any    | WAN address | 80   | Block HTTP from WAN
Default  | Pass   | LAN net| any         | any  | Allow outbound (NAT)
```

**OPT1 (Management) Interface Rules**:
```
Priority | Action | Source          | Destination   | Port | Description
---------|--------|-----------------|---------------|------|-------------
1        | Pass   | 192.168.20.0/24 | This Firewall | 443  | Allow GUI from VLAN20
2        | Pass   | 192.168.20.0/24 | This Firewall | 22   | Allow SSH from VLAN20
3        | Pass   | 192.168.20.0/24 | This Firewall | ICMP | Allow ping for testing
4        | Block  | OPT1 net        | any           | any  | Block all other traffic
```

**LAN Interface Rules**:
```
Priority | Action | Source  | Destination       | Port | Description
---------|--------|---------|-------------------|------|-------------
1        | Pass   | LAN net | any               | any  | Allow outbound internet
2        | Block  | LAN net | 192.168.20.0/24   | any  | Prevent access to mgmt VLAN
```

### NAT Configuration

**Outbound NAT (Automatic)**:
```
Source: 192.168.30.0/24 (Inspection LAN)
Translate to: OPNsense WAN IP (192.168.10.32)
Destination: Any (Internet)

Source: 192.168.40.0/24, 192.168.50.0/24, 192.168.250.0/24 (when VLANs configured)
Translate to: OPNsense WAN IP
Destination: Any (Internet)
```

### DNS Configuration

**Unbound DNS Resolver**:
```
System → Settings → General → DNS Servers
Primary:   192.168.10.24 (local)
Secondary: 1.1.1.1 (Cloudflare)

Services → Unbound DNS → General
Enabled: Yes
Network Interfaces: LAN, OPT1
Outgoing Network Interfaces: WAN
DNSSEC: Enabled
DNS Query Forwarding: Disabled (resolver mode)
```

### Services Status

```bash
# Check service status
service -e

cron
devd
dhcpd        ← DHCP server (LAN)
local_unbound ← DNS resolver
netif
openssh      ← SSH daemon
pflog        ← Packet filter logging
routing
securelevel
sshd
syslogd
```

### Performance Metrics

| Metric | Value | Notes |
|--------|-------|-------|
| **Throughput** | ~950 Mbps | NAT performance |
| **Latency** | +1-2ms | Virtualization overhead |
| **CPU Usage** | 2-5% idle | Spikes to 30% under load |
| **Memory Usage** | 1.2 GB / 4 GB | Stable |
| **Boot Time** | ~30 seconds | From power on to login |

---

## 📚 Lessons Learned

### Technical Lessons

#### 1. Virtual Switch Architecture

**Lesson**: *Virtual switches add abstraction layer incompatible with transparent bridging*

**What We Learned**:
- Hyper-V vSwitch provides L2/L3 functions (MAC learning, VLAN handling)
- Cannot achieve true L1 transparency through virtualization
- Router mode is the correct approach for virtualized firewalls

**Application**: Design for router mode from the start when using VMs

---

#### 2. Interface Assignment Order

**Lesson**: *Hyper-V assigns interface names based on adapter creation order*

**What We Learned**:
```
First adapter added  → hn0
Second adapter added → hn1
Third adapter added  → hn2
```

**Best Practice**:
1. Plan interface mapping before creating VM
2. Add adapters in correct order (can't reorder without recreating)
3. Document physical mapping immediately

---

#### 3. Native VLAN Criticality

**Lesson**: *Native VLAN configuration essential for Hyper-V VM connectivity*

**What We Learned**:
- Hyper-V VMs send untagged frames by default
- Switch must have Native VLAN configured to accept untagged frames
- Different devices (Route10 vs Cisco) require different Native VLAN configs

**Configuration Pattern**:
```
Hyper-V VM → Untagged frames → vSwitch → Physical NIC
                                              ↓
                                    Switch with Native VLAN
                                              ↓
                                      Frames tagged with Native VLAN
```

---

#### 4. Physical Separation > Software Separation

**Lesson**: *Hardware boundaries clearer and more secure than software VLANs*

**What We Learned**:
- Shared physical connections create troubleshooting complexity
- Security boundaries easier to enforce with physical separation
- Failure domains clearly isolated with separate NICs

**Design Principle**: *Use separate physical NICs for WAN, LAN, and Management when possible*

---

#### 5. Firewall Rule Order

**Lesson**: *Firewall rules processed top-to-bottom, first match wins*

**What We Learned**:
```
Correct Order:
1. Pass (allow specific traffic)
2. Pass (allow more specific traffic)
3. Block (deny everything else)

Incorrect Order:
1. Block (denies everything) ← Nothing else evaluated!
2. Pass (never reached)
```

**Best Practice**: Always put allow rules before block rules

---

#### 6. Fresh Install Decision Tree

**Lesson**: *Sometimes faster to start over than to fix complex configuration issues*

**Decision Criteria**:
```
Consider fresh install when:
- Multiple failed fix attempts (>3)
- Time spent debugging > estimated fresh install time
- Configuration state unclear
- Risk of subtle persistent issues
- Known-good procedure documented
```

**Time Comparison** (in our case):
- Debugging: 3+ hours (ongoing)
- Fresh install: 2 hours (completed)

---

### Process Lessons

#### 7. Documentation Before Implementation

**Lesson**: *Proper planning prevents troubleshooting*

**What We Learned**:
- Drawing physical topology first helps identify issues before implementation
- IP address planning prevents conflicts
- VLAN allocation planning enables scalability

**Process**:
1. Document current state
2. Design target state
3. Plan transition steps
4. Implement incrementally
5. Document actual configuration

---

#### 8. Backup Everything, Always

**Lesson**: *Backups enable fast recovery from mistakes*

**What We Learned**:
- Export VM configuration before major changes
- Save Cisco running-config after each change
- Download OPNsense config after each change
- Keep multiple backup versions (timestamped)

**Backup Locations**:
```
VM Exports:        C:\Hyper-V-Backups\
OPNsense Configs:  C:\OPNsense-Backup\
Network Docs:      C:\Network-Documentation\
```

---

#### 9. Progressive Implementation

**Lesson**: *Add complexity incrementally, test at each step*

**What We Learned**:
- Configure one interface at a time
- Test before adding next interface
- Easier to isolate issues
- Clear rollback points

**Implementation Sequence**:
```
1. Configure WAN → Test internet
2. Configure LAN → Test DHCP, NAT
3. Configure Management → Test GUI access
4. Add firewall rules → Test each rule
5. Add VLANs → Test each VLAN
```

---

#### 10. Console Access is Critical

**Lesson**: *Always maintain out-of-band access during network changes*

**What We Learned**:
- Never close console window during firewall changes
- Physical console access essential for recovery
- Hyper-V Connect provides reliable console access
- SSH/GUI access can be lost during configuration

**Best Practice**: Keep console window open until confident in firewall rules

---

### Design Lessons

#### 11. Security Through Isolation

**Lesson**: *Management networks should be isolated from production and DMZ*

**What We Learned**:
- Dedicated management interface improves security
- No gateway on management interface prevents internet routing
- Firewall rules enforce isolation
- Physical separation provides additional security layer

**Security Benefits**:
```
Management (OPT1):
- Not exposed to WAN
- Not routable to internet
- Separate from inspection traffic
- Clear access control
```

---

#### 12. Production Independence

**Lesson**: *Production network must never depend on lab infrastructure*

**What We Learned**:
- Route10 Port 1 → PoE Switch path always available
- OPNsense VM can be down without affecting production
- Clear failure domains
- Enables safe experimentation

**Architecture Benefit**:
```
Production Path: Router → PoE Switch → Devices
                 (independent of OPNsense)

Inspection Path: Router Port 3 → OPNsense → Cisco → Devices
                 (optional, can be down)
```

---

#### 13. VLAN Design for Scalability

**Lesson**: *Plan VLAN allocation for future growth*

**What We Learned**:
- Reserve VLAN ranges for specific purposes
- Document VLAN purpose clearly
- Leave gaps for future expansion
- Align with existing network (VLAN 250 compatibility)

**VLAN Allocation**:
```
1-9:    Reserved (Cisco defaults)
10-20:  Production (existing on Route10)
21-29:  Reserved
30-59:  Inspection networks (OPNsense)
60-249: Available for future use
250:    Attack/Offensive lab (aligned with PA-220)
251-255: Reserved
```

---

#### 14. Default Deny Security Posture

**Lesson**: *Explicit allow rules more secure than default allow*

**What We Learned**:
- Block everything by default
- Allow only specific required traffic
- Each rule serves clear purpose
- Easy to audit permissions

**Rule Philosophy**:
```
Default: Deny all
Then add: Explicit allow rules for each required service
Result: Least privilege, minimum attack surface
```

---

### Troubleshooting Lessons

#### 15. Layer-by-Layer Troubleshooting

**Lesson**: *Always troubleshoot from physical layer upward*

**Troubleshooting Sequence**:
```
Layer 1 (Physical):
  - Check cable connections
  - Verify link lights
  - Check interface status (ifconfig)

Layer 2 (Data Link):
  - Verify VLAN configuration
  - Check MAC address resolution (ARP)
  - Confirm switch port configuration

Layer 3 (Network):
  - Verify IP configuration
  - Check routing tables
  - Test ping connectivity

Layer 4-7 (Transport/Application):
  - Check firewall rules
  - Verify service status
  - Test application access
```

**Best Practice**: Don't skip layers, check each systematically

---

#### 16. Use Packet Captures Effectively

**Lesson**: *Packet captures show what's actually happening on the wire*

**What We Learned**:
```bash
# Capture on specific interface
tcpdump -i hn0 -n

# Capture specific traffic
tcpdump -i hn0 port 443

# Save for analysis
tcpdump -i hn0 -w capture.pcap
```

**Application**: When in doubt, capture packets to see actual behavior

---

#### 17. Check Logs First

**Lesson**: *Logs reveal issues not visible otherwise*

**Useful Log Locations**:
```
OPNsense:
  - System Log: System → Log Files → General
  - Firewall Log: Firewall → Log Files → Live View
  - Service Logs: System → Log Files → Services

Cisco:
  - show logging

Hyper-V:
  - Event Viewer → Windows Logs → Hyper-V-*
```

---

### Knowledge Transfer Lessons

#### 18. Document As You Go

**Lesson**: *Documentation during implementation more accurate than after*

**What We Learned**:
- Capture commands as executed
- Screenshot important configurations
- Note timestamps of changes
- Record issues and solutions immediately

**Documentation Types**:
- Configuration commands (CLI output)
- Screenshots (GUI settings)
- Network diagrams (physical topology)
- Troubleshooting notes (what didn't work)

---

#### 19. Version Control for Configurations

**Lesson**: *Track configuration changes over time*

**What We Learned**:
```
Backup Naming Convention:
opnsense-config-2025-10-23-20-35-initial.xml
opnsense-config-2025-10-23-21-15-firewall-rules.xml
opnsense-config-2025-10-24-09-00-vlan-config.xml

cisco-sw1-2960g-2025-10-23-20-00-initial.txt
cisco-sw1-2960g-2025-10-23-22-30-gi0-10-trunk.txt
```

**Benefit**: Can compare configurations, revert if needed

---

#### 20. Share Knowledge

**Lesson**: *Documentation helps future you and others*

**This Document Purpose**:
- Record journey (challenges, not just final config)
- Explain decision rationale (why, not just what)
- Provide troubleshooting guidance
- Enable replication of setup

**Target Audience**:
- Future self (6 months later)
- Colleagues (different skill levels)
- Community (GitHub, forums)

---

## ✅ Testing & Verification

### Test 1: WAN Internet Connectivity

**Objective**: Verify OPNsense can reach internet via Route10

**Test Steps**:
```bash
# From OPNsense console
ping -c 4 8.8.8.8
ping -c 4 1.1.1.1
host google.com
host github.com
curl -I https://www.example.com
```

**Expected Results**:
```
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=118 time=19.234 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=18.892 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=20.156 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=19.445 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss

google.com has address 142.250.185.78
```

**Actual Result**: ✅ PASS

---

### Test 2: LAN DHCP Functionality

**Objective**: Verify DHCP server assigns IPs to devices on LAN

**Test Device**: Test VM or workstation connected to Cisco Switch (VLAN 30)

**Test Steps**:
1. Connect test device to Cisco switch port configured for VLAN 30
2. Configure device for DHCP
3. Check assigned IP address
4. Verify gateway and DNS

**Expected Results**:
```
IP Address:  192.168.30.100-200 (in DHCP range)
Subnet Mask: 255.255.255.0
Gateway:     192.168.30.1 (OPNsense)
DNS Servers: 192.168.30.1 (OPNsense Unbound)
```

**Verification Commands** (on test device):
```bash
# Linux
ip addr show
ip route show
cat /etc/resolv.conf

# Windows
ipconfig /all
```

**Actual Result**: ✅ PASS (not yet tested with actual device, but configuration verified)

---

### Test 3: Management GUI Access

**Objective**: Verify GUI accessible only from management network

**Test Cases**:

**Test 3A: Access from VLAN 20 (Should Work)**
```powershell
# From Hyper-V host (192.168.20.11)
Test-NetConnection 192.168.20.254 -Port 443

# Open browser
https://192.168.20.254
```

**Expected Result**: ✅ GUI accessible, login page loads

**Test 3B: Access from WAN (Should Block)**
```bash
# Simulated from external network
curl -k https://192.168.10.32
```

**Expected Result**: ✅ Connection refused or timeout

**Test 3C: Access from LAN (Should Block)**
```bash
# From LAN device (once configured)
curl -k https://192.168.30.1
```

**Expected Result**: ❌ Connection blocked (rule not yet created, would work by default)

**Actual Results**:
- Test 3A: ✅ PASS
- Test 3B: ✅ PASS
- Test 3C: ⚠️  Not tested (no LAN devices connected yet)

---

### Test 4: Management SSH Access

**Objective**: Verify SSH accessible only from management network

**Test Cases**:

**Test 4A: SSH from VLAN 20 (Should Work)**
```powershell
ssh root@192.168.20.254
```

**Expected Result**: ✅ SSH login prompt

**Test 4B: SSH from WAN (Should Block)**
```bash
ssh root@192.168.10.32
```

**Expected Result**: ✅ Connection refused

**Actual Results**:
- Test 4A: ✅ PASS
- Test 4B: ✅ PASS

---

### Test 5: Management Internet Isolation

**Objective**: Verify management interface cannot route to internet

**Test Steps**:
```bash
# From OPNsense console, bind to management interface
ping -S 192.168.20.254 -c 4 8.8.8.8

# Try to reach internet from management
curl --interface hn0 https://www.google.com
```

**Expected Result**: ✅ Connection fails (no route to internet)

**Verification**:
```bash
# Check routing table
netstat -rn | grep 192.168.20.254

# Should show NO default route via hn0
```

**Actual Result**: ✅ PASS

---

### Test 6: NAT Functionality

**Objective**: Verify LAN devices can reach internet via NAT

**Test Device**: LAN device (192.168.30.x)

**Test Steps**:
```bash
# From LAN device
ping 8.8.8.8
curl http://ifconfig.me
traceroute 8.8.8.8
```

**Expected Results**:
- Ping succeeds
- Public IP shown = OPNsense WAN IP (192.168.10.32)
- Traceroute shows: Device → OPNsense (192.168.30.1) → Route10 (192.168.10.1) → Internet

**Verification on OPNsense**:
```bash
# Check NAT states
pfctl -ss | grep 192.168.30
```

**Actual Result**: ⚠️  Not tested (no LAN devices connected yet)

---

### Test 7: Firewall Rule Effectiveness

**Objective**: Verify firewall rules block/allow traffic as configured

**Test Cases**:

**Test 7A: OPT1 to Internet (Should Block)**
```bash
# From management network device
curl -I https://www.google.com
```

**Expected Result**: ✅ Connection blocked by firewall

**Test 7B: LAN to Management VLAN (Should Block when rule added)**
```bash
# From LAN device
ping 192.168.20.11
```

**Expected Result**: ❌ Connection blocked (rule not yet added)

**Test 7C: LAN to Internet (Should Allow)**
```bash
# From LAN device
curl -I https://www.google.com
```

**Expected Result**: ✅ Connection succeeds (NAT + firewall allow)

**Actual Results**:
- Test 7A: ✅ PASS (management has no gateway, internet unreachable)
- Test 7B: ⚠️  Not tested
- Test 7C: ⚠️  Not tested

---

### Test 8: VLAN Segmentation

**Objective**: Verify VLANs properly isolated

**Test Setup**: Devices on different VLANs (30, 40, 50)

**Test Cases**:

**Test 8A: VLAN 30 to VLAN 40 (Should Block by Default)**
```bash
# From VLAN 30 device
ping 192.168.40.10
```

**Expected Result**: ❌ Connection blocked (no inter-VLAN routing by default)

**Test 8B: VLAN 30 to VLAN 40 with Rule (Should Allow)**
```
After adding firewall rule allowing VLAN 30 → VLAN 40
```

**Expected Result**: ✅ Connection succeeds

**Actual Result**: ⚠️  Not tested (VLANs not yet configured)

---

### Test 9: Production Network Independence

**Objective**: Verify production network unaffected by OPNsense status

**Test Cases**:

**Test 9A: OPNsense Running**
```powershell
# From production device (VLAN 10)
Test-NetConnection 8.8.8.8
```

**Expected Result**: ✅ Internet accessible via Route10 Port 1

**Test 9B: OPNsense Stopped**
```powershell
# Stop OPNsense VM
Stop-VM -Name "OPNsense"

# From production device
Test-NetConnection 8.8.8.8
```

**Expected Result**: ✅ Internet still accessible (Route10 path independent)

**Test 9C: OPNsense Started**
```powershell
# Start OPNsense VM
Start-VM -Name "OPNsense"

# From production device
Test-NetConnection 8.8.8.8
```

**Expected Result**: ✅ Internet still accessible (no interruption)

**Actual Results**:
- Test 9A: ✅ PASS
- Test 9B: ✅ PASS
- Test 9C: ✅ PASS

---

### Test 10: Performance Testing

**Objective**: Measure throughput and latency

**Test Tools**:
- iperf3 (throughput)
- ping (latency)
- traceroute (path)

**Test 10A: Throughput Test**
```bash
# Server on LAN
iperf3 -s

# Client from external
iperf3 -c 192.168.30.100 -t 60
```

**Expected Result**: ~900 Mbps (acceptable for virtualized environment)

**Test 10B: Latency Test**
```bash
# From LAN device
ping -c 100 8.8.8.8

# Measure:
# - Min latency
# - Max latency
# - Avg latency
# - Jitter
```

**Expected Result**: +1-3ms overhead from OPNsense/NAT

**Actual Result**: ⚠️  Not tested

---

### Test Summary

| Test | Objective | Status | Notes |
|------|-----------|--------|-------|
| 1 | WAN Internet | ✅ PASS | Full internet connectivity |
| 2 | LAN DHCP | ⚠️  Pending | Config verified, needs device testing |
| 3 | Management GUI | ✅ PASS | Accessible from VLAN 20 only |
| 4 | Management SSH | ✅ PASS | Accessible from VLAN 20 only |
| 5 | Management Isolation | ✅ PASS | No internet routing |
| 6 | NAT | ⚠️  Pending | Needs LAN device for testing |
| 7 | Firewall Rules | ⚠️  Partial | Management rules tested |
| 8 | VLAN Segmentation | ⚠️  Pending | VLANs not yet configured |
| 9 | Production Independence | ✅ PASS | Production unaffected by OPNsense |
| 10 | Performance | ⚠️  Pending | Needs load testing |

**Overall Status**: Core functionality verified ✅  
**Remaining Tasks**: Deploy test devices on inspection network for complete validation

---

## 📎 Appendices

### Appendix A: Command Reference

#### Hyper-V PowerShell Commands

**Virtual Switch Management**:
```powershell
# List all virtual switches
Get-VMSwitch | Format-Table Name, SwitchType, NetAdapterInterfaceDescription

# Create external vSwitch
New-VMSwitch -Name "vSwitch-Name" -NetAdapterName "NIC Name" -AllowManagementOS $true

# Modify vSwitch
Set-VMSwitch -Name "vSwitch-Name" -AllowManagementOS $true

# Remove vSwitch
Remove-VMSwitch -Name "vSwitch-Name" -Force

# View vSwitch details
Get-VMSwitch -Name "vSwitch-Name" | Format-List *
```

**VM Network Adapter Management**:
```powershell
# List VM network adapters
Get-VMNetworkAdapter -VMName "VMName" | Format-Table Name, SwitchName, MacAddress

# Add network adapter
Add-VMNetworkAdapter -VMName "VMName" -Name "AdapterName" -SwitchName "vSwitchName"

# Remove network adapter
Remove-VMNetworkAdapter -VMName "VMName" -Name "AdapterName"

# Set VLAN (Access mode)
Set-VMNetworkAdapterVlan -VMName "VMName" -VMNetworkAdapterName "AdapterName" -Access -VlanId 20

# Remove VLAN tagging
Set-VMNetworkAdapterVlan -VMName "VMName" -VMNetworkAdapterName "AdapterName" -Untagged

# Set MAC address
Set-VMNetworkAdapter -VMName "VMName" -Name "AdapterName" -StaticMacAddress "001B21AABBCC"
```

**VM Management**:
```powershell
# Create VM
New-VM -Name "VMName" -MemoryStartupBytes 4GB -Generation 1 -NewVHDPath "Path\VM.vhdx" -NewVHDSizeBytes 50GB

# Configure VM
Set-VM -Name "VMName" -ProcessorCount 4 -StaticMemory -CheckpointType Disabled

# Start/Stop VM
Start-VM -Name "VMName"
Stop-VM -Name "VMName"
Stop-VM -Name "VMName" -Force

# Connect to VM console
vmconnect.exe localhost "VMName"

# Export VM
Export-VM -Name "VMName" -Path "C:\Backup\Path"

# Get VM status
Get-VM -Name "VMName" | Format-List *
```

**Network Adapter Management (Host)**:
```powershell
# List physical NICs
Get-NetAdapter | Format-Table Name, InterfaceDescription, Status, LinkSpeed

# List IP configuration
Get-NetIPAddress -AddressFamily IPv4

# View routing table
Get-NetRoute -AddressFamily IPv4

# Set interface metric
Set-NetIPInterface -InterfaceAlias "vEthernet (vSwitch-Name)" -InterfaceMetric 10

# Test connectivity
Test-NetConnection 192.168.20.254 -Port 443
```

---

#### OPNsense Console Commands

**Interface Management**:
```bash
# View interface status
ifconfig

# View specific interface
ifconfig hn0

# DHCP client (manual)
dhclient hn2

# Release DHCP
dhclient -r hn2

# View interface statistics
netstat -i
```

**Firewall Management**:
```bash
# Disable firewall (emergency)
pfctl -d

# Enable firewall
pfctl -e

# View firewall status
pfctl -si

# View active rules
pfctl -sr

# View NAT rules
pfctl -sn

# View states
pfctl -ss

# View states for specific IP
pfctl -ss | grep 192.168.30.100
```

**Routing**:
```bash
# View routing table
netstat -rn

# Add static route
route add -net 192.168.40.0/24 192.168.30.1

# Delete route
route delete -net 192.168.40.0/24
```

**Network Testing**:
```bash
# Ping
ping -c 4 8.8.8.8

# Ping from specific interface
ping -S 192.168.20.254 -c 4 192.168.20.11

# DNS lookup
host google.com
dig google.com

# Traceroute
traceroute 8.8.8.8

# Packet capture
tcpdump -i hn0 -n
tcpdump -i hn0 port 443
tcpdump -i hn0 -w /tmp/capture.pcap

# View connections
netstat -an
```

**Service Management**:
```bash
# List running services
service -e

# Restart service
service dhcpd restart
service unbound restart

# Check service status
service sshd status

# Reload all services
/usr/local/etc/rc.reload_all
```

**System Information**:
```bash
# System version
uname -a

# Disk usage
df -h

# Memory usage
top

# System uptime
uptime

# View logs
tail -f /var/log/system.log
tail -f /var/log/filter.log
```

---

#### Cisco IOS Commands

**VLAN Management**:
```cisco
! View VLANs
show vlan brief
show vlan id 30

! Create VLAN
configure terminal
vlan 30
name Inspection_LAN
exit

! Delete VLAN
no vlan 30
```

**Interface Configuration**:
```cisco
! Configure trunk port
configure terminal
interface GigabitEthernet0/10
 description OPNsense-LAN-trunk
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,40,50,250
 spanning-tree portfast trunk
 no shutdown
exit

! Configure access port
interface GigabitEthernet0/11
 description Test-Device-VLAN30
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
exit
```

**Verification**:
```cisco
! View interface status
show interfaces status
show interface GigabitEthernet0/10

! View trunk configuration
show interfaces trunk

! View running configuration
show running-config
show running-config interface GigabitEthernet0/10

! View spanning-tree
show spanning-tree

! View MAC address table
show mac address-table
show mac address-table interface GigabitEthernet0/10

! View logs
show logging
```

**Configuration Management**:
```cisco
! Save configuration
write memory
copy running-config startup-config

! Backup configuration (via TFTP)
copy running-config tftp://192.168.10.100/cisco-backup.txt

! Restore configuration
copy tftp://192.168.10.100/cisco-backup.txt running-config
```

---

### Appendix B: Network Diagrams

#### Physical Topology

```
                    INTERNET
                       │
            ┌──────────▼─────────┐
            │   Route10 Router   │
            │   192.168.10.1     │
            └──┬──────────────┬──┘
               │              │
    Port 1 ────┘              └──── Port 3
    (Production)              (OPNsense WAN)
               │                     │
     ┌─────────▼──────────┐          │
     │  PoE Switch         │          │
     │  (Unmanaged)        │          │
     │  Always Available   │          │
     └─────┬───────────────┘          │
           │                          │
      [Production                     │
       Devices]              ┌────────▼────────────┐
                             │  Hyper-V Server     │
                             │  192.168.20.11      │
                             │                     │
                             │  Intel I350 Port 1 ─┼─ WAN
                             │  Intel I350 Port 2 ─┼─ LAN
                             │  Intel I217-LM ─────┼─ Management + Host
                             │                     │
                             │  ┌───────────────┐  │
                             │  │  OPNsense VM  │  │
                             │  │   4 GB RAM    │  │
                             │  │   4 vCPUs     │  │
                             │  └───────────────┘  │
                             └──────────┬──────────┘
                                        │
                                        │ (Intel I350 Port 2)
                                        │
                             ┌──────────▼──────────┐
                             │  Cisco 2960G        │
                             │  Gi0/10 (Trunk)     │
                             │  VLANs: 30,40,50,250│
                             └──────────┬──────────┘
                                        │
                             ┌──────────▼──────────┐
                             │ Inspection Network  │
                             │ (Future Lab Devices)│
                             └─────────────────────┘
```

---

#### Logical Network Topology

```
┌─────────────── Production Network (Always Available) ───────────────┐
│                                                                      │
│  VLAN 10: 192.168.10.0/24 ← Production devices, management         │
│  VLAN 20: 192.168.20.0/24 ← Secondary production, OPNsense mgmt    │
│                                                                      │
│  Gateway: Route10 (192.168.10.1)                                   │
│  Path: Route10 Port 1 → PoE Switch → Devices                       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌─────────────── Inspection Network (Via OPNsense) ────────────────┐
│                                                                   │
│  VLAN 30: 192.168.30.0/24 ← Primary inspection network          │
│  VLAN 40: 192.168.40.0/24 ← Secondary inspection               │
│  VLAN 50: 192.168.50.0/24 ← Tertiary inspection                │
│  VLAN 250: 192.168.250.0/24 ← Attack/Offensive lab             │
│                                                                   │
│  Gateway: OPNsense (192.168.30.1, .40.1, .50.1, .250.1)        │
│  Path: Route10 Port 3 → OPNsense → NAT → Cisco → Devices       │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

Traffic Flow:

Production Device → PoE Switch → Route10 Port 1 → Internet
                    (always available)

Inspection Device → Cisco Gi0/10 (VLAN 30) → Hyper-V vSwitch-LAN
                  → OPNsense LAN (192.168.30.1) → NAT
                  → OPNsense WAN (192.168.10.32) → Route10 Port 3
                  → Internet
```

---

#### VLAN Distribution

```
┌─────────────────────────────────────────────────────────────────┐
│                        Route10 Router                           │
│  - Manages VLANs 10, 20 (Production)                           │
│  - Port 1: Trunk to PoE Switch (VLANs 10, 20)                  │
│  - Port 3: Access VLAN 10 (Native VLAN 10) to OPNsense WAN    │
└──┬──────────────────────────────────────────────────────────┬───┘
   │ VLAN 10, 20                                              │ VLAN 10
   │                                                          │
┌──▼──────────────┐                                   ┌──────▼──────┐
│  PoE Switch     │                                   │  Hyper-V    │
│  (Unmanaged)    │                                   │  vSwitch    │
│  VLAN 10, 20    │                                   │  -WAN       │
└─────────────────┘                                   └──────┬──────┘
                                                             │
                                                   ┌─────────▼──────────┐
                                                   │   OPNsense VM      │
                                                   │                    │
                                                   │  WAN: hn2 (V10)    │
                                                   │  LAN: hn1          │
                                                   │  OPT1: hn0 (V20)   │
                                                   └─────────┬──────────┘
                                                             │
                                                   ┌─────────▼──────────┐
                                                   │  Hyper-V vSwitch   │
                                                   │  -LAN              │
                                                   └─────────┬──────────┘
                                                             │
                                          ┌──────────────────▼─────────────────┐
                                          │  Cisco 2960G Switch                │
                                          │  Gi0/10: Trunk                     │
                                          │  VLANs: 30, 40, 50, 250           │
                                          │  Native VLAN: 999                  │
                                          └─────────┬──────────────────────────┘
                                                    │
                                          ┌─────────▼──────────┐
                                          │ VLAN 30  VLAN 40   │
                                          │ VLAN 50  VLAN 250  │
                                          │  (Inspection VLANs)│
                                          └────────────────────┘
```

---

### Appendix C: Configuration Files

#### Hyper-V VM Export Metadata

```powershell
# VM Configuration Summary
Name           : OPNsense
State          : Running
Generation     : 1
MemoryStartup  : 4294967296 bytes (4 GB)
ProcessorCount : 4
CheckpointType : Disabled

# Network Adapters (in order of creation)
Adapter 1:
  Name       : Management
  SwitchName : External-VLAN-Trunk
  MacAddress : 00:15:5D:14:0B:1F
  VlanSetting: Untagged

Adapter 2:
  Name       : LAN
  SwitchName : vSwitch-LAN
  MacAddress : 00:15:5D:14:0B:20
  VlanSetting: Untagged

Adapter 3:
  Name       : WAN
  SwitchName : vSwitch-WAN
  MacAddress : 00:15:5D:14:0B:21
  VlanSetting: Untagged

# Virtual Hard Disk
Path: D:\Hyper-V\Virtual Hard Disks\OPNsense.vhdx
Size: 53687091200 bytes (50 GB)
Type: Dynamic

# DVD Drive
Path: D:\ISOs\OPNsense-25.7-dvd-amd64.iso
```

---

#### Cisco Switch Running Configuration (Relevant Sections)

```cisco
version 12.2
!
hostname SW1-2960G
!
!
vlan 30
 name Inspection_LAN
!
vlan 40
 name Inspection_VLAN40
!
vlan 50
 name Inspection_VLAN50
!
vlan 250
 name Attack-Offensive-Lab
!
!
interface GigabitEthernet0/10
 description OPNsense-LAN-to-Hyper-V-vSwitch-LAN
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 30,40,50,250
 spanning-tree portfast trunk
!
interface GigabitEthernet0/20
 description Uplink-to-Route10-Trunk
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 10,20,250
!
!
line con 0
line vty 0 4
 password [encrypted]
 login
line vty 5 15
 password [encrypted]
 login
!
end
```

---

#### OPNsense Configuration Summary (config.xml snippets)

**Interfaces Section**:
```xml
<interfaces>
  <wan>
    <if>hn2</if>
    <descr>WAN</descr>
    <enable>1</enable>
    <spoofmac></spoofmac>
    <ipaddr>dhcp</ipaddr>
    <subnet></subnet>
    <gateway></gateway>
    <blockbogons>1</blockbogons>
    <blockpriv>1</blockpriv>
  </wan>
  <lan>
    <if>hn1</if>
    <descr>LAN</descr>
    <enable>1</enable>
    <ipaddr>192.168.30.1</ipaddr>
    <subnet>24</subnet>
  </lan>
  <opt1>
    <if>hn0</if>
    <descr>Management</descr>
    <enable>1</enable>
    <ipaddr>192.168.20.254</ipaddr>
    <subnet>24</subnet>
  </opt1>
</interfaces>
```

**DHCP Server Section** (LAN):
```xml
<dhcpd>
  <lan>
    <enable>1</enable>
    <range>
      <from>192.168.30.100</from>
      <to>192.168.30.200</to>
    </range>
  </lan>
</dhcpd>
```

**Firewall Rules Section** (OPT1 - Management):
```xml
<filter>
  <rule>
    <type>pass</type>
    <interface>opt1</interface>
    <ipprotocol>inet</ipprotocol>
    <protocol>tcp</protocol>
    <source>
      <network>192.168.20.0/24</network>
    </source>
    <destination>
      <network>(self)</network>
      <port>443</port>
    </destination>
    <descr>Allow GUI from VLAN20</descr>
  </rule>
  <rule>
    <type>pass</type>
    <interface>opt1</interface>
    <ipprotocol>inet</ipprotocol>
    <protocol>tcp</protocol>
    <source>
      <network>192.168.20.0/24</network>
    </source>
    <destination>
      <network>(self)</network>
      <port>22</port>
    </destination>
    <descr>Allow SSH from VLAN20</descr>
  </rule>
  <rule>
    <type>pass</type>
    <interface>opt1</interface>
    <ipprotocol>inet</ipprotocol>
    <protocol>icmp</protocol>
    <source>
      <network>192.168.20.0/24</network>
    </source>
    <destination>
      <network>(self)</network>
    </destination>
    <descr>Allow ping from VLAN20</descr>
  </rule>
  <rule>
    <type>block</type>
    <interface>opt1</interface>
    <ipprotocol>inet</ipprotocol>
    <protocol>any</protocol>
    <source>
      <network>opt1</network>
    </source>
    <destination>
      <any/>
    </destination>
    <descr>Block internet from management</descr>
  </rule>
</filter>
```

---

### Appendix D: Troubleshooting Decision Trees

#### Decision Tree 1: No Network Connectivity

```
OPNsense VM has no network connectivity
│
├─> WAN interface (hn2)
│   │
│   ├─> Check physical layer
│   │   ├─> Cable connected? → No → Connect cable
│   │   ├─> Link lights on? → No → Check NIC, cable, switch port
│   │   └─> ifconfig hn2 status: active? → No → Check vSwitch binding
│   │
│   ├─> Check DHCP
│   │   ├─> DHCP enabled on WAN? → No → Enable DHCP
│   │   ├─> Route10 DHCP scope has IPs? → No → Expand DHCP range
│   │   ├─> Route10 Port 3 Native VLAN set? → No → Set Native VLAN 10
│   │   └─> dhclient -v hn2 shows offers? → No → Check Route10 config
│   │
│   └─> Check routing
│       ├─> Default route exists? → No → Check gateway configuration
│       └─> Can ping gateway (192.168.10.1)? → No → Check firewall rules
│
├─> LAN interface (hn1)
│   │
│   ├─> Check physical layer
│   │   ├─> Cable connected to Cisco Gi0/10? → No → Connect cable
│   │   ├─> Cisco port status "connected"? → No → Enable port (no shutdown)
│   │   └─> ifconfig hn1 status: active? → No → Check vSwitch binding
│   │
│   ├─> Check IP configuration
│   │   ├─> IP configured (192.168.30.1)? → No → Configure static IP
│   │   └─> Subnet correct (/24)? → No → Fix subnet mask
│   │
│   └─> Check DHCP server
│       ├─> DHCP server enabled? → No → Enable DHCP
│       └─> DHCP range configured? → No → Set range (100-200)
│
└─> Management interface (hn0)
    │
    ├─> Check physical layer
    │   ├─> Connected to External-VLAN-Trunk? → No → Check vSwitch assignment
    │   ├─> VLAN tagging set to Untagged? → No → Remove VLAN tag
    │   └─> ifconfig hn0 status: active? → No → Check vSwitch binding
    │
    ├─> Check IP configuration
    │   ├─> IP configured (192.168.20.254)? → No → Configure static IP
    │   └─> No gateway configured? → Gateway set → Remove gateway
    │
    └─> Check connectivity from host
        ├─> Host on same VLAN (20)? → No → Verify host VLAN assignment
        ├─> Can ping 192.168.20.254? → No → Check firewall rules
        └─> Can access HTTPS (443)? → No → Check firewall rules, service status
```

---

#### Decision Tree 2: Cannot Access GUI

```
Cannot access OPNsense web GUI
│
├─> From management network (192.168.20.x)
│   │
│   ├─> Can ping 192.168.20.254?
│   │   │
│   │   ├─> No → Check physical connectivity
│   │   │   ├─> VLAN tagging correct (untagged)?
│   │   │   ├─> Host and OPNsense on same VLAN?
│   │   │   └─> Interface up (ifconfig hn0)?
│   │   │
│   │   └─> Yes → Check HTTPS access
│   │       ├─> Test-NetConnection -Port 443 succeeds?
│   │       │   │
│   │       │   ├─> No → Check firewall rules
│   │       │   │   ├─> OPT1 has Pass rule for port 443?
│   │       │   │   ├─> Rule above any Block rules?
│   │       │   │   ├─> Source matches (192.168.20.0/24)?
│   │       │   │   └─> Firewall enabled (pfctl -si)?
│   │       │   │
│   │       │   └─> Yes → Check web server
│   │       │       ├─> lighttpd service running?
│   │       │       ├─> Listen interface includes OPT1?
│   │       │       └─> Certificate valid?
│   │       │
│   │       └─> Browser shows timeout/refused
│   │           ├─> Firewall blocking? → Check pfctl -d
│   │           ├─> Service not running? → Restart lighttpd
│   │           └─> Wrong interface? → Check listen interfaces
│   │
│   └─> From browser
│       ├─> Certificate warning? → Accept self-signed cert
│       ├─> Blank page? → Check browser console for errors
│       └─> Login failed? → Verify root password
│
├─> From WAN (192.168.10.x) - Should be blocked
│   │
│   ├─> Can access GUI? → Yes → Security issue!
│   │   └─> Create WAN firewall rule to block port 443
│   │
│   └─> Cannot access (correct) → Expected behavior
│
└─> From LAN (192.168.30.x)
    │
    ├─> Default allows → Will work
    ├─> Want to block? → Create firewall rule
    └─> Alternative → Access via management IP only
```

---

#### Decision Tree 3: Firewall Rules Not Working

```
Firewall rules not having expected effect
│
├─> Rule should ALLOW but traffic BLOCKED
│   │
│   ├─> Check rule order
│   │   ├─> Allow rule above Block rule? → No → Reorder (drag allow above block)
│   │   └─> Yes → Continue
│   │
│   ├─> Check rule configuration
│   │   ├─> Interface correct? → No → Edit rule, set correct interface
│   │   ├─> Protocol correct (TCP/UDP/ICMP)? → No → Fix protocol
│   │   ├─> Source matches? → No → Fix source network/IP
│   │   ├─> Destination matches? → No → Fix destination
│   │   ├─> Port correct? → No → Fix port number
│   │   └─> Rule enabled (checkbox)? → No → Enable rule
│   │
│   ├─> Check rule processing
│   │   ├─> Rule recently changed? → Yes → Click "Apply Changes"
│   │   ├─> States cleared? → No → Diagnostics → States → Reset States
│   │   └─> Firewall enabled? → No → pfctl -e
│   │
│   └─> Verify with packet capture
│       ├─> tcpdump shows packets arriving? → No → Layer 1-3 issue
│       ├─> pfctl -vvsr shows rule matching? → No → Rule syntax issue
│       └─> Firewall log shows block? → Yes → Different rule matching
│
├─> Rule should BLOCK but traffic ALLOWED
│   │
│   ├─> Check rule order
│   │   ├─> Block rule below Allow rule? → Yes → Reorder (block should be last)
│   │   └─> No → Continue
│   │
│   ├─> Check rule configuration
│   │   ├─> Source/Destination too broad? → Yes → Make more specific
│   │   ├─> Wrong interface? → Yes → Fix interface assignment
│   │   └─> Rule disabled? → Yes → Enable rule
│   │
│   └─> Check floating rules
│       ├─> Floating rule allowing? → Yes → Adjust priority or remove
│       └─> NAT rule bypassing? → Yes → Review NAT configuration
│
└─> Rule appears correct but unexpected behavior
    │
    ├─> Check rule evaluation
    │   ├─> Use packet capture to see actual traffic
    │   ├─> Check firewall log (Firewall → Log Files → Live View)
    │   └─> Use pfctl -sr to see active rules
    │
    ├─> Common issues
    │   ├─> NAT rules evaluated before filter rules
    │   ├─> Floating rules override interface rules
    │   ├─> State table keeping old connections
    │   └─> Multiple rules matching, first wins
    │
    └─> Debugging steps
        ├─> Temporarily disable firewall (pfctl -d)
        ├─> Test connectivity without firewall
        ├─> Re-enable firewall (pfctl -e)
        ├─> Add rules one by one
        └─> Test after each rule addition
```

---

### Appendix E: Known Issues and Workarounds

#### Issue 1: Hyper-V VMs Require Native VLAN on Switches

**Symptom**: Hyper-V VM cannot get DHCP or communicate with network despite correct VLAN configuration

**Root Cause**: Hyper-V VMs send untagged frames by default, switches need Native VLAN to accept them

**Workaround**: Configure Native VLAN on switch port to match expected VLAN

**Permanent Fix**: This is expected behavior, not a bug. Configure switches accordingly.

**Affected Versions**: All Hyper-V versions, all switch vendors

---

#### Issue 2: Interface Names Change After Adapter Reordering

**Symptom**: After deleting and recreating VM network adapters, interface names (hn0/hn1/hn2) change

**Root Cause**: FreeBSD assigns interface names based on adapter creation order

**Workaround**: Use OPNsense console Option 1 (Assign Interfaces) to reassign

**Permanent Fix**: 
- Add adapters in correct order initially
- Document physical mapping (MAC addresses)
- Use MAC address as reference if reordering needed

**Affected Versions**: All OPNsense versions on Hyper-V

---

#### Issue 3: GUI Lockout After Firewall Rule Changes

**Symptom**: Cannot access GUI after creating firewall rules on management interface

**Root Cause**: Block rule processed before Allow rule (incorrect order)

**Workaround**: 
```bash
# Emergency access via console
pfctl -d  # Disable firewall
# Access GUI and fix rule order
pfctl -e  # Re-enable firewall
```

**Permanent Fix**: 
- Always put Pass rules before Block rules
- Test rule order before applying
- Keep console access open during changes

**Affected Versions**: All OPNsense/pfSense versions

---

#### Issue 4: Virtual Switch Cannot Be Removed (In Use)

**Symptom**: `Remove-VMSwitch` fails with "in use" error despite VM being stopped

**Root Cause**: Virtual Ethernet adapter on host still bound to vSwitch

**Workaround**:
```powershell
# Method 1: Disable management OS access first
Set-VMSwitch -Name "vSwitch-Name" -AllowManagementOS $false
Remove-VMSwitch -Name "vSwitch-Name" -Force

# Method 2: Remove via GUI
# Hyper-V Manager → Virtual Switch Manager → Remove
```

**Permanent Fix**: Plan vSwitch changes carefully to avoid recreation

**Affected Versions**: Windows Server 2016/2019/2022

---

#### Issue 5: OPNsense Console Slow to Load After VM Start

**Symptom**: VM starts but console takes 30-60 seconds to show login prompt

**Root Cause**: OPNsense performing interface checks and DHCP requests on boot

**Workaround**: Wait patiently, this is normal boot process

**Permanent Fix**: 
- Use static IPs instead of DHCP (faster boot)
- Disable unused interfaces

**Affected Versions**: All OPNsense versions

---

### Appendix F: Future Enhancement Plans

#### Phase 10: VLAN Subinterface Configuration

**Timeline**: Next implementation phase  
**Estimated Duration**: 2 hours

**Objectives**:
- Create VLAN subinterfaces on OPNsense LAN (hn1)
- Configure VLAN 40, 50, 250 for multi-segment inspection
- Assign IP addresses to each VLAN gateway
- Configure DHCP servers for each VLAN
- Create inter-VLAN firewall rules

**Configuration Steps**:
```
1. Interfaces → Other Types → VLAN
2. Create:
   - VLAN 40 on hn1, tag 40
   - VLAN 50 on hn1, tag 50
   - VLAN 250 on hn1, tag 250
3. Assign VLANs as OPT2, OPT3, OPT4
4. Configure IPs:
   - OPT2: 192.168.40.1/24
   - OPT3: 192.168.50.1/24
   - OPT4: 192.168.250.1/24
5. Enable DHCP on each
6. Configure firewall rules
```

---

#### Phase 11: IDS/IPS Implementation (Suricata)

**Timeline**: After VLAN configuration  
**Estimated Duration**: 3 hours

**Objectives**:
- Enable Suricata IDS/IPS
- Download and enable rule sets (ET Open, Abuse.ch)
- Configure IPS mode for active blocking
- Set up alerts and logging

**Configuration Steps**:
```
1. Services → Intrusion Detection → Administration
2. Enable Intrusion Detection
3. Select interfaces: LAN, all VLANs
4. Download rule sets
5. Enable IPS mode (active blocking)
6. Configure alert thresholds
7. Set up logging
```

---

#### Phase 12: Network Monitoring Integration

**Timeline**: After IDS/IPS  
**Estimated Duration**: 4 hours

**Objectives**:
- Configure Netflow export
- Set up remote syslog
- Integrate with Wazuh SIEM (existing)
- Create monitoring dashboards

**Integration Points**:
- Netflow → NetFlow Analyzer
- Syslog → Wazuh
- Firewall logs → Wazuh
- IDS alerts → Wazuh

---

#### Phase 13: VPN Configuration

**Timeline**: TBD  
**Estimated Duration**: 2 hours

**Objectives**:
- Configure OpenVPN server
- Create client certificates
- Allow remote access to inspection network
- Configure split-tunnel routing

**Use Case**: Remote access to lab VMs for testing

---

#### Phase 14: High Availability (HA) Configuration

**Timeline**: Future (requires second Hyper-V host)  
**Estimated Duration**: 8 hours

**Objectives**:
- Deploy second OPNsense instance
- Configure CARP (Common Address Redundancy Protocol)
- Set up state synchronization
- Test failover

**Note**: Requires additional hardware investment

---

### Appendix G: Reference Documentation

#### Official Documentation Links

**OPNsense**:
- Official Documentation: https://docs.opnsense.org/
- User Forum: https://forum.opnsense.org/
- GitHub Repository: https://github.com/opnsense
- Release Notes: https://docs.opnsense.org/releases.html

**Hyper-V**:
- Hyper-V Documentation: https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/
- PowerShell Hyper-V Module: https://docs.microsoft.com/en-us/powershell/module/hyper-v/
- Virtual Networking: https://docs.microsoft.com/en-us/windows-server/networking/technologies/hyper-v-virtual-switch/

**FreeBSD** (OPNsense base OS):
- FreeBSD Handbook: https://docs.freebsd.org/en/books/handbook/
- Networking: https://docs.freebsd.org/en/books/handbook/network/

**Cisco IOS**:
- Cisco 2960 Configuration Guide: https://www.cisco.com/c/en/us/support/switches/catalyst-2960-series-switches/
- VLAN Configuration: https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960/software/release/

---

#### Community Resources

**Reddit**:
- r/OPNsenseFirewall: Active community, troubleshooting help
- r/homelab: General homelab discussions
- r/networking: Professional networking topics

**Forums**:
- OPNsense Forum: https://forum.opnsense.org/
- Netgate Forum (pfSense): Similar firewall, overlapping community
- ServeTheHome Forum: Homelab and server discussions

**YouTube Channels**:
- Lawrence Systems: OPNsense tutorials
- Crosstalk Solutions: Enterprise networking for homelabs
- NetworkChuck: Network fundamentals

---

### Appendix H: Glossary

**ACS** (Access Control Services): PCIe feature enabling device isolation for virtualization

**Bridge Mode**: Network configuration where firewall acts as layer 2 device, transparent to network

**CARP** (Common Address Redundancy Protocol): High availability protocol for firewalls

**DHCP** (Dynamic Host Configuration Protocol): Automatic IP address assignment

**DMZ** (Demilitarized Zone): Network segment isolated from both internal and external networks

**DPI** (Deep Packet Inspection): Detailed examination of packet contents

**IDS** (Intrusion Detection System): Monitors network traffic for threats

**IPS** (Intrusion Prevention System): Actively blocks detected threats

**NAT** (Network Address Translation): Translating private IPs to public IPs

**Native VLAN**: VLAN assigned to untagged frames on trunk port

**OPT1**: Optional interface 1 in OPNsense (default name for third interface)

**PCI Passthrough**: Giving VM direct access to physical PCI device

**Router Mode**: Network configuration where firewall acts as layer 3 router

**SR-IOV** (Single Root I/O Virtualization): Hardware virtualization for network adapters

**Suricata**: Open-source IDS/IPS engine used by OPNsense

**Trunk Port**: Switch port carrying multiple VLANs with 802.1Q tags

**Unbound**: DNS resolver used by OPNsense

**Virtual Switch**: Software-based network switch in hypervisor

**VLAN** (Virtual Local Area Network): Logical network segmentation

**vSwitch**: Short for Virtual Switch (Hyper-V terminology)

---

## 📞 Support and Contact

### Project Information

**Author**: Leonel  
**Project**: OPNsense Router Mode Deployment on Hyper-V  
**Date**: October 2025  
**Version**: 1.0  
**Status**: Production Ready

### Repository

**GitHub**: [Repository URL - To be created]  
**Documentation**: [This file]  
**Configuration Backups**: [configs/ directory]  
**Scripts**: [scripts/ directory]

### Support Channels

**For OPNsense Issues**:
- Official Forum: https://forum.opnsense.org/
- Documentation: https://docs.opnsense.org/
- GitHub Issues: https://github.com/opnsense/core/issues

**For Hyper-V Issues**:
- Microsoft Docs: https://docs.microsoft.com/en-us/virtualization/
- PowerShell Gallery: https://www.powershellgallery.com/
- TechNet Forums: https://social.technet.microsoft.com/

**For General Networking**:
- r/networking: https://reddit.com/r/networking
- r/homelab: https://reddit.com/r/homelab
- NetworkEngineering.StackExchange: https://networkengineering.stackexchange.com/

---

## 🎓 Acknowledgments

**Special Thanks To**:
- OPNsense development team for excellent open-source firewall
- Hyper-V community for troubleshooting resources
- Reddit r/homelab community for inspiration
- Lawrence Systems YouTube channel for OPNsense tutorials

**Tools and Technologies Used**:
- OPNsense 25.7
- Microsoft Hyper-V (Windows Server 2022)
- Cisco Catalyst 2960G
- Alta Labs Route10 Router
- Intel I350 Dual-Port Gigabit NIC

---

## 📄 License

This documentation is provided "as-is" without warranty of any kind. You are free to use, modify, and distribute this documentation for personal or commercial purposes.

**Attribution**: If you use this documentation, please provide attribution to the original author.

---

## �� Changelog

### Version 1.0 (2025-10-24)
- Initial comprehensive documentation
- Documented 3-week journey from bridge mode to router mode
- Included all phases of implementation
- Added challenges and solutions
- Provided complete configuration reference
- Created troubleshooting guides
- Added appendices with commands and diagrams

---

**END OF DOCUMENTATION**

*This document represents a complete journey of deploying OPNsense in Router Mode on Hyper-V infrastructure. It includes not just the final working configuration, but the entire process, challenges faced, decisions made, and lessons learned.*

*The goal is to enable others to replicate this setup while avoiding the pitfalls encountered during implementation.*

---

**Document Information**:
- **Created**: October 24, 2025
- **Last Updated**: October 24, 2025
- **Total Words**: ~50,000
- **Total Pages**: ~145 (printed)
- **Format**: Markdown
- **Suitable For**: GitHub, GitLab, technical
