# CODEX-LOG.md — Codex Action Log

**How this works:** Codex appends a summary after every session. Claude reads this to stay in sync.

---

## Sessions

## Session — 2026-06-06
### Project worked on
- OPNsense evidence-documentation skill review

### What I did
- Reviewed Claude's S02 request in `CLAUDE-REVIEW.md`.
- Reviewed `skills/opnsense-evidence-documentation/SKILL.md`.
- Patched OPNsense-specific evidence workflow issues around folder naming, rule export assumptions, syslog verification, and DNS-over-TLS verification.
- Marked S02 resolved for Claude review.

### Files created/modified
- `skills/opnsense-evidence-documentation/SKILL.md`
- `CLAUDE-REVIEW.md`
- `CODEX-LOG.md`

### Decisions made
- Use the repo's real project path pattern: `projects/<number>-<project-name>/`.
- Do not assume `scripts/export-rules.py` exists until Project 08 creates and reviews it.
- Prefer sanitized firewall rule summaries unless an API export has been reviewed for WAN IPs, aliases, VPN objects, credentials, and private notes.
- Use FreeBSD/OPNsense `sockstat -4 -6 -c` for DoT verification.

### Open questions for Claude
- Review and push S02 corrections.
- In the sibling repos, apply the evidence-skill corrections Codex could only outline because write access was not granted.

---

## Session — 2026-06-06
### Project worked on
- OPNsense project skills review and creation

### What I did
- Reviewed Claude's S01 request for `skills/homelab-opnsense-projects.md`.
- Patched technical issues in the comprehensive Projects 03-09 skill.
- Used the skill-creator workflow to scaffold standard skill folders for Projects 03-06.
- Wrote concise project-specific `SKILL.md` files for Suricata IDS/IPS, VPN, monitoring/SIEM, and HA design.
- Updated the skills index to list the comprehensive skill and the new project-specific skills.
- Attempted `quick_validate.py`; it could not run because this Python environment is missing the `yaml` module. Performed a manual structure check for required names, descriptions, `SKILL.md`, and `agents/openai.yaml`.
- Marked S01 resolved for Claude review.

### Files created/modified
- `skills/homelab-opnsense-projects.md`
- `skills/opnsense-p03-suricata/SKILL.md`
- `skills/opnsense-p03-suricata/agents/openai.yaml`
- `skills/opnsense-p04-vpn/SKILL.md`
- `skills/opnsense-p04-vpn/agents/openai.yaml`
- `skills/opnsense-p05-monitoring/SKILL.md`
- `skills/opnsense-p05-monitoring/agents/openai.yaml`
- `skills/opnsense-p06-ha-design/SKILL.md`
- `skills/opnsense-p06-ha-design/agents/openai.yaml`
- `skills/README.md`
- `CLAUDE-REVIEW.md`
- `CODEX-LOG.md`

### Decisions made
- Keep `homelab-opnsense-projects.md` as the comprehensive cross-project skill, but clarify future/proposed scope for P07-P09.
- Add separate standard skill folders for P03-P06 so Codex/Claude can load smaller project-specific guidance when Leonel names a project.
- Keep phase detail in project phase files and keep skills focused on triggers, safety gates, operating pattern, and closeout.
- Use Project 06 as design-only unless Leonel explicitly approves live HA work later.

### Open questions for Claude
- Decide whether to install all four project skill folders into local assistant skill locations, or only install P03 first.
- Decide whether `homelab-opnsense-projects.md` should remain as a broad skill alongside the narrower project-specific skills.

---

## Session — 2026-06-06
### Project worked on
- Projects 03-05 phase-file technical review

### What I did
- Reviewed Claude's D01 request in `CLAUDE-REVIEW.md`.
- Checked Project 03 Suricata phases 3-6, Project 04 VPN phases 1-5, and Project 05 monitoring phases 1-5 for OPNsense 24.x accuracy and safety gates.
- Patched stale or risky guidance in the phase files.
- Marked D01 resolved for Claude review.

### Files created/modified
- `projects/03-ids-ips-suricata/phases/phase-3-verify-alerts.md`
- `projects/03-ids-ips-suricata/phases/phase-4-tune-rules.md`
- `projects/04-vpn-remote-access/phases/phase-2-build-vpn-server.md`
- `projects/04-vpn-remote-access/phases/phase-3-firewall-rules.md`
- `projects/04-vpn-remote-access/phases/phase-4-test-and-breakfix.md`
- `projects/05-monitoring-integration/phases/phase-1-audit-logging.md`
- `projects/05-monitoring-integration/phases/phase-2-syslog-config.md`
- `projects/05-monitoring-integration/phases/phase-3-suricata-netflow.md`
- `projects/05-monitoring-integration/phases/phase-4-dashboards-and-breakfix.md`
- `projects/05-monitoring-integration/phases/phase-5-document-and-close.md`
- `CLAUDE-REVIEW.md`
- `CODEX-LOG.md`

### Decisions made
- Treat WireGuard as built into modern OPNsense; do not instruct `os-wireguard` installation unless the installed version lacks WireGuard and Claude approves.
- Use `VPN → WireGuard → Instances` terminology and explicitly attach peers back to the instance.
- Use `/32` for the road-warrior client tunnel address in the client config template, matching the OPNsense peer Allowed IP.
- Keep WireGuard interface assignment as recommended for clean rules, but leave IP configuration as `None` because the tunnel address belongs to the WireGuard instance.
- Keep VPN break/fix rule-order testing on lab VLAN 250 rather than intentionally exposing production VLAN 10 or management VLAN 20.
- Use OPNsense's built-in Suricata EVE syslog output for SIEM forwarding before any agent or custom tail/syslog-ng workaround.
- Use built-in `Reporting → NetFlow`; do not use `os-softflowd`.
- Use modern log paths with `latest.log` symlinks instead of legacy `clog /var/log/filter.log`.

### Open questions for Claude
- Confirm the live OPNsense release before Leonel starts Project 04 or 05, especially if the GUI differs from the documented 24.x/modern paths.

---

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
