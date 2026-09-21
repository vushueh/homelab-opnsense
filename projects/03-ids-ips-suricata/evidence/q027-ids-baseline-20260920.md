# Q027 IDS settings baseline — 2026-09-20

Leonel opened the current IDS Administration Settings page. Codex read rendered controls and captured an inspected screenshot without clicking or changing any control.

| Setting | Observed form state |
|---|---|
| Enabled | Checked |
| Capture mode | PCAP live mode (IDS) |
| Promiscuous mode | Checked |
| Interfaces | AttackLabVLAN250, LAN_VLAN30, VLAN40, VLAN200DMZ, WAN |
| Pattern matcher | Hyperscan |
| Syslog alerts | Checked |
| EVE syslog output | Checked |
| Rotation / logs retained | Weekly / 4 |

<img src="screenshots/q027/phase1-ids-baseline.png" alt="Existing IDS configuration before Q027 changes" width="900">

The screenshot truncates the interface text; the full selected list above was verified through the rendered select control. These are displayed settings, not proof of running service, loaded signatures, packet capture, remote log receipt or indexed Wazuh alerts. No Apply, service start/stop or interface selection was performed.

WAN is already selected. This conflicts with the proposed lab-only experiment boundary and must be explicitly reconciled without silently removing existing monitoring. The next audit step is a current backup, followed by interface/rule/policy and service/log verification. Preserve the entire observed baseline until exact changes are reviewed and approved. Q027 phase 1 remains In Progress.
