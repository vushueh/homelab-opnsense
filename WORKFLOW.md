# WORKFLOW.md — How This Family Works

## Trigger Phrase

Say **`opnsense project`** to work in this repo.

When triggered, Claude will:
1. Read this WORKFLOW.md and README.md
2. Check CLAUDE-REVIEW.md for open items
3. Check CODEX-LOG.md for latest Codex work
4. Report current project status and exact next step

---

## Trigger Phrases

| Say this | Means |
|----------|-------|
| `opnsense project` | Switch context to this repo |
| `start opnsense project [N]` | Begin project N |
| `check what Codex did` | Claude reads CODEX-LOG.md and summarises |
| `check open items` | Claude lists OPEN items in CLAUDE-REVIEW.md |
| `push to github` | Claude compiles work and pushes |

---

## Project Cycle

Every project follows this cycle — consistent across all families:

```
1. Audit current state  (show/check commands, save before-state)
2. Build                (apply config via web UI or SSH)
3. Verify              (test connectivity, check logs)
4. Break/Fix           (deliberate fault → diagnose → restore)
5. Document            (save configs + outputs)
6. Push to GitHub      (Claude does this)
```

---

## Folder Structure Per Project

```
projects/NN-name/
├── README.md            ← Objective, topology, phase list, status
├── phases/
│   ├── phase-1-name.md  ← Steps tagged [LEONEL], [CLAUDE], [CODEX]
│   └── ...
├── configs/             ← Before/after OPNsense configs (XML exports)
├── verification/        ← Screenshots, test outputs
└── troubleshooting/     ← Break/fix log
```

---

## Who Does What

| Task | Owner |
|------|-------|
| OPNsense web UI changes | Leonel |
| Review configs before applying | Claude |
| SSH to OPNsense | Claude |
| All GitHub pushes | Claude |
| Config XML generation | Codex |
| Script drafts (bash, PowerShell) | Codex |
| Phase guides and project specs | Claude |
| CODEX-LOG.md updates | Codex |
| CLAUDE-REVIEW.md | Claude |

---

## Cross-Family Context

OPNsense connects to other families:

| Project | Links to |
|---------|----------|
| 05 Monitoring Integration | Physical gear + SOC stack in Homelab_CCNA |
| OSPF (future) | R1-2900 is an OSPF neighbour in Homelab_CCNA |

Say `homelab expansion` to switch to the Homelab_CCNA family.

---

## OPNsense Access

| Method | Details |
|--------|--------|
| Web UI | https://192.168.20.254 (or OPNsense WAN IP) |
| SSH | `ssh root@192.168.20.254` |
| Hyper-V Console | Hyper-V Manager on 192.168.20.11 |
