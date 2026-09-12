> Shared AGENTS and its Git/GitHub rule supersede legacy agent-only push
> roles and automatic commit examples below. Publish the authorized scope with
> either operating agent; live execution restrictions remain unchanged.

# WORKFLOW.md — OPNsense Family Workflow

## Trigger Phrase

Say **`opnsense project`** to work in this repo.

When triggered, the active assistant should:

1. Read `AGENTS.md`.
2. Before reporting status or resuming/selecting work, use the canonical family
   `goal` skill and its checkout/default-branch freshness, dependency and WIP checks.
3. Search related OPEN items and the latest related log entry.
4. Read the selected owner project README and dated closeout.
5. Report the verified active item and next safe step; do not choose from stale
   local status alone. A named-file task follows AGENTS task routing instead.

## Trigger Phrases

| Say this | Means |
|----------|-------|
| `opnsense project` | Switch context to this repo |
| `start opnsense project 03` | Begin the Suricata IDS/IPS project |
| `opnsense next phase` | Continue the current project phase |
| `check what Codex did` | Read and summarize `CODEX-LOG.md` |
| `check open items` | Read and summarize `CLAUDE-REVIEW.md` |
| `opnsense screenshot phase` | Use the GUI-first steps and capture evidence |
| `opnsense break fix` | Run the planned troubleshooting exercise |

## Standard Project Cycle

```text
1. Audit current state
2. Plan the change and rollback
3. Build using GUI or reviewed commands
4. Verify with tests, logs, and screenshots
5. Break/fix exercise
6. Document evidence
7. Push to GitHub when approved
```

## Standard Project Folder

```text
projects/project-name/
|-- README.md
|-- phases/
|   |-- phase-1-audit.md
|   |-- phase-2-build.md
|   |-- phase-3-verify.md
|   |-- phase-4-breakfix.md
|   `-- phase-5-document.md
|-- configs/
|-- verification/
`-- troubleshooting/
```

## Ownership Model

| Task | Owner |
|------|-------|
| OPNsense GUI clicks and screenshots | Leonel |
| Live OPNsense SSH/API execution | Claude, after approval |
| Command drafting and troubleshooting plans | Codex |
| Sanitized config snippets and scripts | Codex |
| Final review and GitHub push | Either operating agent for the user-authorized package |
| `CLAUDE-REVIEW.md` | Claude opens items, Codex resolves or comments |
| `CODEX-LOG.md` | Codex logs its work |

## Evidence Standard

Every finished phase should have at least one of:

- screenshot from OPNsense GUI;
- command output showing state before/after;
- packet capture or log entry;
- firewall rule, DHCP lease, route, IDS alert, or VPN session evidence.

## Safety Gates

Stop and ask Leonel before:

- enabling IPS blocking mode;
- changing WAN or management rules;
- modifying production VLAN 10 or VLAN 20;
- adding port forwards from the internet;
- changing DHCP on an active segment;
- importing or restoring full OPNsense XML config;
- applying VPN or RADIUS shared secrets.

## Cross-Family Switching

| Phrase | Family |
|--------|--------|
| `homelab expansion` | Homelab_CCNA physical/CML bridge family |
| `windows server project` | Windows Server Business Admin family |
| `proxmox project` | Homelab management / Proxmox execution |
| `SNML` | Secure Network Management Labs on VirtualBox |
| `CML project` | Enterprise Network Labs in Cisco CML |

## OPNsense Access Reference

| Method | Details |
|--------|---------|
| Web UI | `https://192.168.20.254` |
| SSH | `ssh root@192.168.20.254` |
| Hyper-V Console | Hyper-V Manager on `WIN-PRQD8TJG04M` / `192.168.20.11` |

Do not commit credentials or unredacted config exports.
