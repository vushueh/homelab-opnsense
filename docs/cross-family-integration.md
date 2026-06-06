# OPNsense Cross-Family Integration

OPNsense is the firewall and routing family that connects multiple homelab tracks. This document defines what OPNsense provides to each family and what must stay separate.

## Integration Summary

| Family | What OPNsense Provides | Example Project Link |
|--------|-------------------------|----------------------|
| Homelab_CCNA | Real VLAN routing, NAT, ACL/firewall comparisons, syslog/NetFlow | Physical VLAN and monitoring projects |
| CML Enterprise Labs | Firewall comparison, external routing/NAT target, hybrid routing future | CML Project 05/06/09/11/13 extensions |
| Windows Server Business Admin | Future RADIUS/NPS admin auth, DNS tests, event forwarding | Windows Project 13 identity capstone |
| Proxmox / Hyper-V | Firewall-controlled VM segments and management reachability | VM access and VLAN placement work |
| SOC / Blue Team | Suricata alerts, firewall logs, DHCP/DNS telemetry, NetFlow | Wazuh/Security Onion/TheHive/MISP stack |
| SNML VirtualBox | Concept mapping only; keep VirtualBox implementation separate | SNML firewall, IDS/IPS, VPN, monitoring projects |

## Non-Negotiable Boundaries

- Route10 remains the production router unless a future migration project says otherwise.
- OPNsense lab segments should not break home internet or production VLANs.
- Do not merge SNML VirtualBox implementation into this repo. Link concepts, not platforms.
- Do not publish secrets from VPN, RADIUS, API, or XML exports.

## VLAN Role Map

| VLAN | Subnet | Primary Use | Cross-Family Consumers |
|------|--------|-------------|------------------------|
| 10 | 192.168.10.0/24 | Production / WAN side | Route10, Hyper-V WAN-side VMs |
| 20 | 192.168.20.0/24 | Management | Windows Server, OPNsense management, Hyper-V host |
| 30 | 192.168.30.0/24 | Blue-team/services | Security Onion, TheHive, MISP, OpenCTI candidates |
| 40 | 192.168.40.0/24 | Attack/admin tooling | Kali, Windows attack/tooling VMs |
| 50 | 192.168.50.0/24 | Reserved lab segment | Future project use |
| 250 | 192.168.250.0/24 | Vulnerable systems | DVWA, Metasploitable, vulnerable Windows labs |

## Future Capstone Ideas

### Capstone A — AD-backed Firewall Administration

Goal: use Windows Server NPS/RADIUS to authenticate OPNsense admin access where supported, while keeping local emergency admin fallback.

Skills covered:

- AD security groups;
- NPS RADIUS clients;
- firewall admin authorization;
- audit logging;
- local break-glass access.

### Capstone B — SOC Visibility Through OPNsense

Goal: send OPNsense logs and Suricata alerts to the SOC stack, then generate controlled attacks from Kali/VLAN 40 against vulnerable systems on VLAN 250.

Skills covered:

- Suricata alert triage;
- firewall log interpretation;
- Wazuh/Security Onion ingestion;
- incident timeline documentation.

### Capstone C — Hybrid CML and Physical Firewall Behavior

Goal: compare CML firewall/ACL behavior against real OPNsense policy behavior using the same traffic intent.

Skills covered:

- ACL design;
- NAT policy comparison;
- packet captures;
- route and firewall troubleshooting.

## Documentation Requirement

Every cross-family project must document:

- source family and target family;
- networks involved;
- firewall rules involved;
- identity/authentication source if used;
- telemetry/log destination;
- rollback path.
