# OPNsense Lab Network Documentation - README

## 📚 Overview

This repository contains professional technical documentation for a complete OPNsense-based network inspection lab spanning multiple hypervisors (Hyper-V and Proxmox VE). The documentation was created by systematically analyzing and consolidating information from multiple troubleshooting sessions and configuration activities conducted between October 2-26, 2025.

## 📂 Repository Structure

```
homelab-docs/
├── README.md (this file)
├── OPNsense-Lab-Network-Documentation.md (main documentation)
└── configs/ (optional - for configuration backups)
```

## 🎯 What's Included

The main documentation (`OPNsense-Lab-Network-Documentation.md`) contains:

### ✅ **Complete Network Architecture**
- High-level topology diagrams
- Physical to virtual interface mappings
- VLAN design and implementation
- Traffic flow analysis for various scenarios

### ✅ **8 Major Challenges Documented**
Each challenge includes:
- Problem statement and symptoms
- Root cause analysis
- Step-by-step solution
- Verification commands
- Lessons learned

Key challenges covered:
1. VLAN interface creation (CLI vs GUI)
2. Static route configuration on Route10
3. Firewall rules blocking inter-VLAN traffic
4. DHCP server configuration for multiple VLANs
5. Proxmox VM network connectivity
6. Hyper-V VM network isolation
7. Managing OPNsense via different interfaces
8. VLAN 250 gateway conflict resolution

### ✅ **Complete Configuration Details**
- OPNsense interface setup (WAN, LAN, Management, VLANs)
- Cisco 2960G switch configuration (trunk ports, VLANs)
- Route10 static routing
- Hyper-V virtual switch configuration
- Proxmox VLAN-aware bridge setup
- Firewall rules for all interfaces
- DHCP server configurations
- NAT setup

### ✅ **Troubleshooting Guide**
Common issues with:
- Diagnosis steps
- Root cause analysis
- Solutions
- Verification commands

### ✅ **Traffic Flow Analysis**
Detailed packet flows for:
- Internet access from VLAN 250 VM
- Attack from VLAN 20 to VLAN 250
- VM communication across hypervisors
- Management access to OPNsense

### ✅ **Verification & Testing**
Complete test procedures for:
- DHCP functionality
- Gateway reachability
- Inter-VLAN routing
- Internet connectivity
- Cross-hypervisor communication
- Firewall rule enforcement
- IDS/IPS detection
- Performance testing

## 🔍 How to Use This Documentation

### For Initial Setup
1. **Start with "Network Architecture"** section to understand the topology
2. **Review "VLAN Design"** to understand the IP addressing scheme
3. **Follow "Step-by-Step Configuration"** in order:
   - Phase 1: OPNsense setup
   - Phase 2: VLAN configuration
   - Phase 3: Cisco switch configuration
   - Phase 4: Route10 configuration

### For Troubleshooting
1. **Check "Troubleshooting Guide"** for common issues
2. **Review "Challenges & Solutions"** for similar problems encountered
3. **Use "Verification & Testing"** to confirm fixes
4. **Reference "Appendix C"** for useful commands

### For Understanding Traffic Flow
1. **Read "Traffic Flow Analysis"** section
2. **Follow packet paths** through the network
3. **Understand NAT and routing** decisions at each hop

### For Security Configuration
1. **Review "Firewall Rules & Security"** section
2. **Understand rule order and logic**
3. **Follow security best practices** outlined

## 📊 Quick Reference Tables

The documentation includes multiple quick reference tables:

- **IP Address Assignments** (Appendix D.1)
- **Port Mappings** (Appendix D.2)
- **VLAN Scheme** (VLAN Design section)
- **Firewall Rules Summary** (Firewall Rules section)
- **DHCP Configuration** (DHCP Configuration section)
- **Challenge Summary** (Each challenge section)

## 🔧 Technologies Covered

| Category | Technologies |
|----------|-------------|
| **Firewall/Router** | OPNsense 24.7 (FreeBSD-based) |
| **Hypervisors** | Windows Server 2022 Hyper-V, Proxmox VE 8.x |
| **Networking** | Cisco IOS 15.x (Catalyst 2960G), Alta Labs Route10 |
| **Protocols** | VLAN tagging (802.1Q), Static routing, NAT, DHCP |
| **Security** | Firewall rules, Network segmentation, IDS/IPS ready |

## 📈 Project Metrics

- **Duration**: ~3 weeks (October 2-26, 2025)
- **VLANs Configured**: 5 (VLAN 10, 20, 30, 40, 70, 250)
- **Hypervisors Integrated**: 2 (Hyper-V, Proxmox VE)
- **Network Switches**: 2 (Route10, Cisco 2960G)
- **Major Challenges Resolved**: 8
- **Success Rate**: 100% operational

## 🎓 Learning Outcomes

By following this documentation, you will understand:

1. **VLAN Implementation**
   - Creating VLANs on OPNsense (proper GUI method)
   - Trunk port configuration on Cisco switches
   - VLAN tagging in Hyper-V and Proxmox

2. **Inter-VLAN Routing**
   - Configuring OPNsense as inter-VLAN router
   - Static route configuration
   - Understanding traffic flow between VLANs

3. **Firewall Configuration**
   - Creating proper firewall rules
   - Understanding rule order and logic
   - NAT configuration for multiple VLANs

4. **Multi-Hypervisor Integration**
   - Bridging networks across Hyper-V and Proxmox
   - VLAN trunk configuration on both platforms
   - VM networking best practices

5. **Troubleshooting Skills**
   - Systematic problem diagnosis
   - Using CLI tools (tcpdump, ifconfig, etc.)
   - Reading logs and interpreting errors

## ⚠️ Important Notes

### Security Considerations
- This is a **lab environment** configuration
- Some settings prioritize ease of use over security
- **Review and harden** before production use
- Management interface has NO internet access (by design)
- Attack lab (VLAN 250) is isolated for security testing

### Prerequisites
To implement this configuration, you need:
- Basic networking knowledge (IP addressing, routing, VLANs)
- Access to hypervisor host (Hyper-V or Proxmox)
- Managed switch with VLAN support
- OPNsense installation media
- Familiarity with CLI (PowerShell, Bash, Cisco IOS)

### Hardware Requirements (Minimum)
- **Hypervisor Host**: 16GB RAM, 4+ CPU cores, 100GB storage
- **Managed Switch**: VLAN-capable (802.1Q tagging)
- **Network Cards**: Multiple NICs for interface separation

## 🚀 Next Steps

After reviewing this documentation:

1. **Export Your Configurations**
   - OPNsense: System → Configuration → Backups
   - Cisco: `show running-config`
   - Hyper-V: PowerShell export scripts (see Appendix A.4)
   - Proxmox: `/etc/network/interfaces`

2. **Implement IDS/IPS**
   - Follow "Future Enhancements" section
   - Enable Suricata on OPNsense
   - Configure rule sets

3. **Set Up Monitoring**
   - Deploy logging solution (ELK, Graylog)
   - Configure alerting
   - Create dashboards

4. **Add Automation**
   - Backup configs to GitHub automatically
   - Script common tasks
   - Implement Infrastructure as Code

## 📝 Contributing

If you're using this documentation and find:
- **Errors or omissions**: Please document them
- **Alternative solutions**: Share your approach
- **Additional challenges**: Document your findings

Consider creating your own branch or fork to track your specific modifications.

## 📧 Support

For questions or clarifications about this documentation:
1. Review the "Troubleshooting Guide" section first
2. Check "Challenges & Solutions" for similar issues
3. Verify configurations match your environment
4. Use the command reference in Appendix C for diagnostics

## 🏆 Credits

This documentation was compiled from:
- Multiple troubleshooting sessions (October 2-26, 2025)
- Real-world problem-solving experiences
- Best practices from OPNsense, Cisco, and virtualization communities
- Lessons learned from trial and error

## 📄 License

This documentation is provided as-is for educational and reference purposes.
Modify and adapt as needed for your specific environment.

---

**Document Version**: 1.0  
**Last Updated**: October 26, 2025  
**Status**: Complete and Verified  
**Verified On**: OPNsense 24.7, Windows Server 2022 Hyper-V, Proxmox VE 8.x

---

## 🔗 Quick Links Within Documentation

- [Executive Summary](#executive-summary)
- [Network Architecture](#network-architecture)
- [Challenges & Solutions](#challenges--solutions)
- [Step-by-Step Configuration](#step-by-step-configuration)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Verification & Testing](#verification--testing)
- [Quick Reference Tables](#appendix)

**Ready to get started? Open `OPNsense-Lab-Network-Documentation.md` and begin with the Executive Summary!**
