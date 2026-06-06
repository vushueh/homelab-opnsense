# Project 08 — Policy Automation

## Status

Planned. Requires Projects 03–05 complete.

## Purpose

Automate OPNsense configuration management using the OPNsense API and export workflows.
Build repeatable change-control processes: configuration backups, rule exports,
and scripted policy pushes — reducing manual GUI effort and documenting lab state in git.

## Skills Practiced

- OPNsense REST API authentication and usage
- Automated configuration backup via API
- Firewall rule export and diff comparison
- Python/curl scripting against OPNsense API
- Configuration version control (git-backed backups)
- Change-control workflow: backup → change → verify → commit

## Tools

| Tool | Purpose |
|------|---------|
| OPNsense REST API | Read/write firewall rules, aliases, DHCP, services |
| curl / Python requests | API client scripts |
| git | Version-controlled config backups |
| Ansible (optional) | Idempotent policy push using OPNsense Ansible collection |

## Cross-Family Links

| Family | Link |
|--------|------|
| Homelab_CCNA P08 | Compare OPNsense API automation to Cisco Netmiko/Ansible |
| Windows Server P09 | Compare OPNsense backup scripts to PowerShell admin platform |

## Phase Index

| Phase | Goal |
|-------|------|
| [1](phases/phase-1-api-setup.md) | Enable OPNsense API, create API key, test with curl |
| [2](phases/phase-2-config-backup.md) | Script automated config backup via API to local git repo |
| [3](phases/phase-3-rule-export.md) | Export firewall rules to JSON, diff against previous export |
| [4](phases/phase-4-scripted-push.md) | Push a simple alias or rule change via API script |
| [5](phases/phase-5-break-fix.md) | Break/fix: wrong API key, revoked key, malformed payload |
| [6](phases/phase-6-document-and-close.md) | Document API workflow, sanitize keys, close project |

## Safety Gates

Stop before:
- pushing any rule that touches WAN or VLAN 10/20 via API without Claude review;
- committing API keys or session tokens to git — always use environment variables;
- running destructive API calls (reset, factory defaults) without explicit approval.

## Completion Evidence

- working API backup script saving config to git on a schedule;
- firewall rule export showing diff between two snapshots;
- successful scripted alias push verified in GUI;
- break/fix log documented;
- STAR summary complete.
