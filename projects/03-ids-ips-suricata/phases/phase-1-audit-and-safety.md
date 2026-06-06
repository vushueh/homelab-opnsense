# Phase 1 — Audit and Safety Plan

## Goal

Confirm current OPNsense state before enabling Suricata. This phase is read-only except for creating backups or screenshots.

## Track A — GUI Steps

[LEONEL]

1. Log into OPNsense web UI.
2. Go to **System → Configuration → Backups**.
3. Download a config backup.
4. Go to **Interfaces → Assignments** and capture the interface list.
5. Go to **Firewall → Rules** and capture rules for VLAN 30, VLAN 40, VLAN 250, and MGMT.
6. Go to **Services** and confirm whether IDS/IPS or Suricata is already installed/enabled.

## Track B — Verification Commands

[CLAUDE/CODEX]

Use only after approval or from a read-only SSH session:

```sh
configctl interface list
sockstat -4 -l
sysctl net.inet.ip.forwarding
```

## What Good Looks Like

- backup exists before changes;
- interfaces are clearly mapped;
- production VLANs are not part of the first IDS test;
- test scope is limited to lab VLANs;
- MGMT access is preserved.

## Screenshots to Capture

- Configuration backup page after backup is downloaded;
- interface assignment page;
- VLAN firewall rule pages;
- current IDS/IPS service page.

## Safety Notes

Do not enable IPS blocking mode in this phase. Do not change WAN or MGMT rules.

## Documentation Checklist

- [ ] Backup captured and stored outside GitHub
- [ ] Interface map documented
- [ ] Firewall rules reviewed
- [ ] Lab test VLANs selected
- [ ] Production VLANs excluded from first test
