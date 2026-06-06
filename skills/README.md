# OPNsense Skills

Skills are short operating references for Claude and Codex. They keep the repo workflow consistent across sessions.

## Available Skills

| Skill | Purpose |
|-------|---------|
| [opnsense-family.md](opnsense-family.md) | Main family skill for all OPNsense project work |
| [homelab-opnsense-projects.md](homelab-opnsense-projects.md) | Comprehensive Projects 03-09 skill written for cross-project use |
| [opnsense-p03-suricata](opnsense-p03-suricata/SKILL.md) | Project 03 IDS/IPS with Suricata |
| [opnsense-p04-vpn](opnsense-p04-vpn/SKILL.md) | Project 04 VPN remote access |
| [opnsense-p05-monitoring](opnsense-p05-monitoring/SKILL.md) | Project 05 syslog, Suricata forwarding, NetFlow, and SIEM integration |
| [opnsense-p06-ha-design](opnsense-p06-ha-design/SKILL.md) | Project 06 OPNsense high availability design |

## Installation Note

Repo skills should be copied into local assistant skill folders only after they are reviewed.

For standard skill folders, copy the whole folder, not only `SKILL.md`:

- `C:\Users\CHONGONG\.agents\skills\`
- `C:\Users\CHONGONG\.codex\skills\`

Do not put secrets in skills.
