# OPNsense Homelab Firewall

> Production-ready OPNsense deployment on Hyper-V with comprehensive documentation

[![Status](https://img.shields.io/badge/Status-Production-success)]()
[![Platform](https://img.shields.io/badge/Platform-Hyper--V-blue)]()
[![OPNsense](https://img.shields.io/badge/OPNsense-25.7-orange)]()

## 🎯 Quick Links

- [Deployment Journey](projects/01-baseline-deployment/README.md) - Complete 3-week journey from bridge mode failures to router mode success
- [Network Topology](diagrams/) - Visual architecture diagrams
- [Configuration Guide](docs/01-initial-setup/) - Step-by-step setup instructions
- [Troubleshooting](troubleshooting/) - Common issues and solutions

## 📊 Project Status

| Project | Status | Phase | Documentation |
|---------|--------|-------|---------------|
| Baseline Deployment | ✅ Complete | Phase 1-10 | [View](projects/01-baseline-deployment/) |
| VLAN Segmentation | ✅ Complete | Phase 8-10 | [View](projects/02-vlan-segmentation/) |
| IDS/IPS (Suricata) | 📋 Planned | Phase 11 | [View](projects/03-ids-ips-suricata/) |
| VPN Remote Access | 📋 Planned | Phase 13 | [View](projects/04-vpn-remote-access/) |
| Monitoring Integration | 📋 Planned | Phase 12 | [View](projects/05-monitoring-integration/) |
| High Availability | 💡 Future | Phase 14 | [View](projects/06-high-availability/) |

**Legend**: ✅ Complete | 🔄 In Progress | 📋 Planned | 💡 Future

## 🏗️ Current Architecture
```
Internet
    ↓
Route10 Router (Alta Labs)
    ├── Port 1 → Production Network (VLANs 10, 20) ← Always Available
    └── Port 3 → OPNsense WAN (VLAN 10)
         ↓
    OPNsense VM (Router Mode - Hyper-V)
         ├── WAN: 192.168.10.x (DHCP from Route10)
         ├── LAN: 192.168.30.1/24 (VLAN 30)
         ├── OPT1: 192.168.40.1/24 (VLAN 40)
         ├── OPT2: 192.168.50.1/24 (VLAN 50)
         ├── OPT3: 192.168.250.1/24 (VLAN 250 - Attack Lab)
         └── MGMT: 192.168.20.254 (Management)
         ↓
    Cisco Catalyst 2960G (VLAN Distribution)
         ↓
    Inspection Network Devices
```

## 🔧 Environment Specifications

**Hypervisor**
- Platform: Microsoft Hyper-V
- Host OS: Windows Server 2022
- RAM Allocated: 4GB (OPNsense VM)
- Storage: 500GB SSD

**Network Hardware**
- Router: Alta Labs Route10
- Core Switch: Cisco Catalyst 2960G
- NICs: Intel I350 Dual-Port Gigabit + Intel I217-LM

**OPNsense Configuration**
- Version: 25.7
- Mode: Router Mode (not bridge)
- Interfaces: 3 physical + 4 VLANs
- Uptime: 24/7 capable

## 🎓 Key Learnings

> **Bridge Mode Failed**: Virtual switches (Hyper-V, Proxmox) cannot provide true transparent layer 2 bridging. Hardware limitations (no PCI passthrough/ACS support) made this impossible.

> **Router Mode Succeeded**: Embracing layer 3 routing architecture provided clean segmentation, full functionality, and excellent stability.

> **Production Isolation Critical**: Keeping production network (VLANs 10, 20) completely separate from inspection network (VLANs 30, 40, 50, 250) ensures zero production impact during testing.

## 📚 Documentation Structure
```
docs/
├── 01-initial-setup/         # Hardware, Hyper-V, network planning
├── 02-network-configuration/ # Interfaces, VLANs, firewall rules, NAT
├── 03-security/              # IDS/IPS, hardening, threat detection
├── 04-services/              # DHCP, DNS, VPN
├── 05-monitoring/            # Netflow, syslog, SIEM integration
└── 06-advanced/              # HA, reverse proxy, load balancing
```

## 🚀 Quick Start

### Prerequisites
- Hyper-V enabled Windows Server 2022 (or Windows 10/11 Pro)
- Minimum 3 network adapters (physical or virtual)
- 4GB RAM for OPNsense VM
- 20GB storage for OPNsense VM

### Installation Steps

1. **Read the deployment journey**
   - [Complete 3-week documentation](projects/01-baseline-deployment/README.md)
   - Learn from failures (bridge mode) and success (router mode)

2. **Plan your network**
   - [Hardware requirements](docs/01-initial-setup/hardware-requirements.md)
   - [IP address planning](docs/01-initial-setup/network-planning.md)

3. **Deploy OPNsense**
   - [Hyper-V VM configuration](docs/01-initial-setup/hyper-v-configuration.md)
   - [Router mode setup](docs/01-initial-setup/router-mode-setup.md)

4. **Configure networking**
   - [Interface assignment](docs/02-network-configuration/interfaces.md)
   - [VLAN configuration](docs/02-network-configuration/vlans.md)
   - [Firewall rules](docs/02-network-configuration/firewall-rules.md)

## 🛠️ Available Scripts
```bash
# Backup automation
./scripts/backup/backup-config.sh           # Manual config backup
./scripts/backup/automated-backup.ps1       # Scheduled backups (Windows)

# Deployment helpers
./scripts/deployment/initial-setup.sh       # Post-installation setup
./scripts/deployment/vlan-creator.sh        # Bulk VLAN creation

# Monitoring
./scripts/monitoring/health-check.sh        # System health check
./scripts/monitoring/traffic-stats.sh       # Real-time traffic stats

# Maintenance
./scripts/maintenance/update-opnsense.sh    # Safe update procedure
./scripts/maintenance/config-diff.sh        # Compare configurations
```

## 📁 Repository Structure
```
homelab-opnsense/
├── docs/              # Reference documentation
├── projects/          # Implementation projects with status
├── configs/           # Configuration files and backups
├── scripts/           # Automation and maintenance scripts
├── diagrams/          # Network topology diagrams
├── troubleshooting/   # Common issues and solutions
└── references/        # Cheat sheets and links
```

## 🐛 Troubleshooting

Common issues and solutions:

- [Bridge mode failures](troubleshooting/bridge-mode-failures.md) - Why it doesn't work in virtualization
- [Hyper-V specific issues](troubleshooting/hyper-v-specific.md) - vSwitch, MAC spoofing, VLAN trunk
- [Network connectivity](troubleshooting/network-connectivity.md) - Routing, DHCP, DNS problems
- [Performance tuning](troubleshooting/performance-tuning.md) - Optimize throughput

## 🤝 Contributing

This is a personal homelab project, but suggestions and improvements are welcome!

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with clear documentation

## 📄 License

MIT License - Free to use and modify with attribution

## 📞 Support

**For OPNsense Issues**:
- Official Documentation: https://docs.opnsense.org/
- Community Forum: https://forum.opnsense.org/

**For Hyper-V Issues**:
- Microsoft Docs: https://docs.microsoft.com/en-us/virtualization/

**General Networking**:
- r/homelab: https://reddit.com/r/homelab
- r/OPNsenseFirewall: https://reddit.com/r/OPNsenseFirewall

## 🎯 Roadmap

- [x] Phase 1-10: Baseline deployment (Router Mode) ✅
- [ ] Phase 11: IDS/IPS with Suricata
- [ ] Phase 12: Monitoring integration (Netflow, Wazuh)
- [ ] Phase 13: VPN remote access
- [ ] Phase 14: High availability configuration

## 🙏 Acknowledgments

- **OPNsense Team** - Excellent open-source firewall
- **r/homelab community** - Inspiration and troubleshooting help
- **Lawrence Systems** - OPNsense tutorials on YouTube

---

**Author**: Leonel Chongong  
**Created**: October 2025  
**Status**: Production Ready  
**Last Updated**: 2025-10-24
