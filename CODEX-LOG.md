# CODEX-LOG.md — Codex Action Log

**How this works:** Codex appends a summary after every session. Claude reads this to stay in sync.

---

## Sessions

## Session — 2026-06-06
### Project worked on
- Repo-level OPNsense family structure setup

### What I did
- Reviewed the existing `homelab-opnsense` repo.
- Confirmed Projects 01 and 02 already preserve valuable completed work.
- Identified missing structure: planned README links pointed to folders that did not exist, and there was no `skills/` folder.
- Reworked the root README into a family-project overview.
- Updated `AGENTS.md` and `WORKFLOW.md` to match the stronger family model used by Homelab_CCNA and Windows Server Business Admin.
- Added a cross-family integration document.
- Added the OPNsense family skill.
- Created missing project folders for Projects 03 through 06.
- Gave Project 03 the most detail because Suricata IDS/IPS is the recommended next active OPNsense project.
- Added a Claude review item for scaffold review and local skill sync.

### Files created/modified
- `README.md`
- `AGENTS.md`
- `WORKFLOW.md`
- `projects/README.md`
- `docs/cross-family-integration.md`
- `skills/README.md`
- `skills/opnsense-family.md`
- `projects/03-ids-ips-suricata/README.md`
- `projects/03-ids-ips-suricata/phases/phase-1-audit-and-safety.md`
- `projects/03-ids-ips-suricata/phases/phase-2-enable-ids-mode.md`
- `projects/03-ids-ips-suricata/configs/.gitkeep`
- `projects/03-ids-ips-suricata/verification/.gitkeep`
- `projects/03-ids-ips-suricata/troubleshooting/README.md`
- `projects/04-vpn-remote-access/README.md`
- `projects/04-vpn-remote-access/phases/README.md`
- `projects/04-vpn-remote-access/configs/.gitkeep`
- `projects/04-vpn-remote-access/verification/.gitkeep`
- `projects/04-vpn-remote-access/troubleshooting/README.md`
- `projects/05-monitoring-integration/README.md`
- `projects/05-monitoring-integration/phases/README.md`
- `projects/05-monitoring-integration/configs/.gitkeep`
- `projects/05-monitoring-integration/verification/.gitkeep`
- `projects/05-monitoring-integration/troubleshooting/README.md`
- `projects/06-high-availability/README.md`
- `projects/06-high-availability/phases/README.md`
- `projects/06-high-availability/configs/.gitkeep`
- `projects/06-high-availability/verification/.gitkeep`
- `projects/06-high-availability/troubleshooting/README.md`
- `CLAUDE-REVIEW.md`
- `CODEX-LOG.md`

### Decisions made
- Preserve Project 01 as the deployment journey and bridge-mode lesson learned.
- Preserve Project 02 as VLAN segmentation history.
- Recommend Project 03 Suricata as the next real OPNsense project because it turns the firewall into a security sensor.
- Recommend Project 05 Monitoring before Project 04 VPN so remote access is built after logging visibility exists.
- Keep Project 06 High Availability as future/design-only until hardware prerequisites exist.
- Treat OPNsense as a cross-family firewall and telemetry platform, not a standalone archive.

### Open questions for Claude
- Review Item OPN-001 in `CLAUDE-REVIEW.md`.
- Decide whether to install `skills/opnsense-family.md` into local assistant skill folders.
- Decide whether Project 03 needs a dedicated local skill before Leonel starts Suricata work.

---

### Pre-framework work summary (Codex)
- Project 01 (Baseline Deployment): Complete — Router Mode on Hyper-V, 3-week journey documented
- Project 02 (VLAN Segmentation): Complete — VLANs 30, 40, 50, 250 configured
