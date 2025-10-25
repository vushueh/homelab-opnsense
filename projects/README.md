# OPNsense Projects

This directory contains individual projects built on the OPNsense infrastructure.

## Completed Projects

### [01. Baseline Deployment](01-baseline-deployment/)
**Status**: ✅ Complete  
**Duration**: 3 weeks  
**Phases**: 1-10

Complete journey from bridge mode failures to production-ready router mode deployment on Hyper-V.

**Key Achievements**:
- Router mode implementation
- 3 physical interfaces + 4 VLANs
- Production network isolation
- 24/7 uptime capability

[View Documentation →](01-baseline-deployment/README.md)

---

### [02. VLAN Segmentation](02-vlan-segmentation/)
**Status**: ✅ Complete  
**Duration**: Part of baseline deployment  
**Phases**: 8-10

Comprehensive VLAN configuration for network segmentation.

**Networks Created**:
- VLAN 30: Lab/Test Devices (192.168.30.0/24)
- VLAN 40: Additional Segment (192.168.40.0/24)
- VLAN 50: Reserved (192.168.50.0/24)
- VLAN 250: Attack Lab (192.168.250.0/24)

[View Documentation →](02-vlan-segmentation/README.md)

---

## Planned Projects

### [03. IDS/IPS with Suricata](03-ids-ips-suricata/)
**Status**: 📋 Planned  
**Estimated Duration**: 3 hours  
**Phase**: 11

Deploy Suricata intrusion detection/prevention system.

**Objectives**:
- Enable IDS/IPS on all interfaces
- Configure rule sets (ET Open, Abuse.ch)
- Set up active blocking (IPS mode)
- Configure alerting and logging

[View Planning →](03-ids-ips-suricata/README.md)

---

### [04. VPN Remote Access](04-vpn-remote-access/)
**Status**: 📋 Planned  
**Estimated Duration**: 2 hours  
**Phase**: 13

Configure OpenVPN for remote access to inspection network.

**Objectives**:
- OpenVPN server configuration
- Client certificate generation
- Split-tunnel routing
- Access to lab VMs

[View Planning →](04-vpn-remote-access/README.md)

---

### [05. Monitoring Integration](05-monitoring-integration/)
**Status**: 📋 Planned  
**Estimated Duration**: 4 hours  
**Phase**: 12

Integrate OPNsense with monitoring and SIEM systems.

**Objectives**:
- Netflow export configuration
- Remote syslog setup
- Wazuh SIEM integration
- Dashboard creation

[View Planning →](05-monitoring-integration/README.md)

---

### [06. High Availability](06-high-availability/)
**Status**: 💡 Future Concept  
**Estimated Duration**: 8 hours  
**Phase**: 14

Deploy second OPNsense instance for HA configuration.

**Requirements**:
- Second Hyper-V host
- CARP configuration
- State synchronization
- Failover testing

**Note**: Requires additional hardware investment

[View Concept →](06-high-availability/README.md)

---

## Project Status Legend

- ✅ **Complete**: Project finished and documented
- 🔄 **In Progress**: Actively working on project
- 📋 **Planned**: Documented plan, ready to start
- 💡 **Future**: Concept phase, requires prerequisites

---

## Contributing to Projects

Each project follows this structure:
```
project-name/
├── README.md           # Main documentation
├── planning.md         # Planning and requirements
├── implementation.md   # Step-by-step guide
├── testing.md          # Testing procedures
├── lessons-learned.md  # Retrospective
└── configs/            # Configuration files
```

Feel free to suggest new projects or improvements!
