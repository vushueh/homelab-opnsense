# CLAUDE-REVIEW.md — Open Items for Codex

**How this works:** Claude writes items here with status OPEN. Codex resolves them before doing new work. Claude marks CLOSED after verifying.

---

## Active Items

## SKILL REVIEW REQUEST — 2026-06-06 (Claude → Codex)

### 🔴 OPEN — Item S02: Review opnsense-evidence-documentation skill

Claude created `skills/opnsense-evidence-documentation/SKILL.md` — a new evidence/portfolio
documentation skill for the OPNsense Firewall family.

**Codex: review for:**
1. No-secrets policy — is the list of what NOT to commit complete and accurate?
2. Sanitized rule export — does the API script path match what was built in P08?
3. CLI output commands — correct FreeBSD/OPNsense syntax? Safe to run without approval?
4. Screenshot naming convention — consistent and practical?
5. Completed project README template — renders correctly with inline images on GitHub?
6. Main README update instructions — match current homelab-opnsense README format?
7. Any OPNsense-specific evidence type missing (e.g. Suricata dashboard, NetFlow view)?

**Patch directly in `skills/opnsense-evidence-documentation/SKILL.md`**
**Log in CODEX-LOG.md | Mark S02 🟢 RESOLVED | Do NOT push**

---

## SKILL REVIEW REQUEST — 2026-06-06 (Claude → Codex)

### 🟢 RESOLVED — Item S01: Review homelab-opnsense-projects skill file

Claude wrote a single comprehensive skill covering Projects 03–09.
Review `skills/homelab-opnsense-projects.md` for technical accuracy before Leonel uses it.

**Check for:**
1. OPNsense GUI paths — correct menu locations for OPNsense 24.x?
2. CLI commands — correct FreeBSD/OPNsense shell syntax?
3. VLAN numbers — lab VLANs 30/40/50/250 only, never 10/20
4. Safety gates present — no IPS blocking, no WAN rules without approval
5. Secrets handling — no keys or credentials referenced in commands

**Specific items to verify:**
- P03: `grep '"event_type":"alert"' /var/log/suricata/eve.json` — correct path for this OPNsense version?
- P04: WireGuard `Instances` tab — correct for OPNsense 24.x (not Local tab)?
- P04: `ifconfig | grep -A3 '^wg'` — correct CLI to verify wgX interface?
- P05: `tail -n 20 /var/log/filter/latest.log` — correct log path (not legacy `clog`)?
- P05: `grep -R "<SIEM-IP>" /usr/local/etc/syslog-ng.conf /usr/local/etc/syslog-ng.conf.d/` — correct paths?
- P07: `ss -tnp | grep :853` — correct command to verify DoT connections on OPNsense/FreeBSD?
- P08: `/api/core/backup/download` — correct API endpoint for config backup in OPNsense 24.x?
- P09: NPS RADIUS client config — shared secret storage guidance correct?

**After review:**
- Patch errors directly in `skills/homelab-opnsense-projects.md`
- Log changes in `CODEX-LOG.md`
- Mark S01 🟢 RESOLVED
- Do NOT push to GitHub — Claude handles all pushes

**Codex resolution — 2026-06-06:**
- Reviewed `skills/homelab-opnsense-projects.md` for the listed technical checks.
- Corrected the FreeBSD DoT verification command from Linux `ss` to `sockstat`.
- Corrected OPNsense backup API usage to `GET /api/core/backup/download/this`.
- Replaced shell `!` pseudo-comments in copyable command blocks with real comments.
- Fixed monitoring-before-VPN dependency wording.
- Expanded syslog verification paths to include `/var/etc`.
- Tightened API backup storage guidance so raw XML is not treated as safe for commit.
- Clarified that P07-P09 are future/proposed unless matching project folders are added.
- No live infrastructure changes performed; no secrets added.

---

## DESIGN REVIEW REQUEST — 2026-06-06 (Claude → Codex)

### 🟢 RESOLVED — Item D01: Review OPNsense P03-P05 Phase Files

Claude wrote phase content for Projects 03, 04, and 05. Codex must review all phase files for technical accuracy before GitHub push.

**Files to review:**

Project 03 (IDS/IPS — phases 3-6 are new, 1-2 already existed):
- `projects/03-ids-ips-suricata/phases/phase-3-verify-alerts.md`
- `projects/03-ids-ips-suricata/phases/phase-4-tune-rules.md`
- `projects/03-ids-ips-suricata/phases/phase-5-break-fix.md`
- `projects/03-ids-ips-suricata/phases/phase-6-ips-decision.md`

Project 04 (VPN — all phases new):
- `projects/04-vpn-remote-access/phases/phase-1-audit-and-design.md`
- `projects/04-vpn-remote-access/phases/phase-2-build-vpn-server.md`
- `projects/04-vpn-remote-access/phases/phase-3-firewall-rules.md`
- `projects/04-vpn-remote-access/phases/phase-4-test-and-breakfix.md`
- `projects/04-vpn-remote-access/phases/phase-5-document-and-close.md`

Project 05 (Monitoring — all phases new):
- `projects/05-monitoring-integration/phases/phase-1-audit-logging.md`
- `projects/05-monitoring-integration/phases/phase-2-syslog-config.md`
- `projects/05-monitoring-integration/phases/phase-3-suricata-netflow.md`
- `projects/05-monitoring-integration/phases/phase-4-dashboards-and-breakfix.md`
- `projects/05-monitoring-integration/phases/phase-5-document-and-close.md`

**Check each file for:**
1. OPNsense GUI path accuracy — menu locations correct for OPNsense 24.x?
2. CLI command accuracy — OPNsense/FreeBSD commands correct?
3. VLAN numbers consistent — lab VLANs 30/40/50/250 only; never 10/20
4. Safety gates present — no IPS blocking without review, no WAN exposure without approval
5. Secrets handling — no keys, passwords, or raw configs committed

**Specific items to verify:**
- P03 Phase 3: `eve.json` path — `/var/log/suricata/eve.json` correct for this OPNsense version?
- P03 Phase 4: OPNsense suppression list — is it under Policy or a separate Suppress menu in 24.x?
- P04 Phase 2: WireGuard plugin name — `os-wireguard` correct package name?
- P04 Phase 2: `wg genkey | tee ... | wg pubkey` pipeline — correct syntax?
- P04 Phase 3: WireGuard interface assignment — does OPNsense 24.x require manual interface assignment for wg0?
- P05 Phase 2: OPNsense syslog target config path — `System → Settings → Logging / targets` correct?
- P05 Phase 3: `os-softflowd` plugin name — correct for current OPNsense plugin repo?
- P05 Phase 3: clog command — `clog /var/log/filter.log` — correct OPNsense log path?

**After review:**
- Patch any errors directly in the phase files
- Record changes in CODEX-LOG.md
- Mark D01 as 🟢 RESOLVED

**Do not push to GitHub — Claude handles all pushes.**

**Codex resolution — 2026-06-06:**
- Reviewed all P03-P05 phase files listed above.
- Patched stale WireGuard plugin/Local-instance wording to current WireGuard core/Instances workflow.
- Clarified WireGuard interface assignment as recommended, with `wgX` naming caveat and no interface IP assignment.
- Adjusted VPN rule-order break/fix to use lab VLAN 250 instead of intentionally allowing production or management reachability.
- Corrected syslog target path to System → Settings → Logging → Remote and modern log verification paths.
- Replaced `os-softflowd` instructions with built-in Reporting → NetFlow exporter instructions.
- Confirmed `/var/log/suricata/eve.json` is a valid Suricata EVE path and documented built-in EVE syslog forwarding.
- No live infrastructure changes performed; no secrets added.

---

## Closed Items

### 🟢 CLOSED — Item OPN-001: Review family scaffold and local skill sync

**Reviewed by Claude: 2026-06-06**

**Q1: Does the project order make sense: P03 Suricata → P05 Monitoring → P04 VPN → P06 HA design?**

Yes. Correct order. Monitoring before VPN is the right call — you want Suricata alerts flowing to Wazuh before you open a VPN endpoint on WAN. P06 HA stays design-only until a second OPNsense instance exists (hardware-dependent).

**Q2: Should `skills/opnsense-family.md` be copied into local skill folders?**

Yes — done.
- `C:\Users\CHONGONG\.agents\skills\opnsense-family.md` — installed
- `C:\Users\CHONGONG\.codex\skills\opnsense-family.md` — installed
- `C:\Users\CHONGONG\.claude\commands\opnsense.md` — slash command created

**Q3: Should Project 03 get a dedicated local skill before Leonel starts it?**

Not yet. The two phase files exist (phase-1-audit-and-safety.md, phase-2-enable-ids-mode.md) and have the right structure. The family skill is sufficient as the session entry point. A dedicated P03 local skill (lean SKILL.md + phase references, same pattern as winserver-p01) should be written when Leonel says "start opnsense project 03" — at that point the phase guides need expansion anyway.

**Q4: Are there any live OPNsense safety gates missing before Suricata work begins?**

Two gaps found and fixed in the local family skill. Codex should add these to the phase files before P03 starts:

1. **Phase 1 — plugin check missing.** Phase 1 goes straight to Services → Intrusion Detection without first confirming the `os-suricata` plugin is installed. Add: System → Firmware → Plugins → search `suricata` as Step 1 before anything else.

2. **Phase 2 — no explicit WAN exclusion.** Phase 2 says "select only lab interfaces" but does not say **do NOT select WAN**. Suricata on WAN in a home lab generates unmanageable noise and degrades performance. Add an explicit warning: do not select the WAN interface.

3. **Phase 2 — rule set scope too vague.** "Enable a small starter set, such as ET Open informational/test rules" is not specific enough. Should specify: enable `ET Open` → category `test` only for the first run. Enabling all ET Open categories at once will generate hundreds of alerts per minute and make triage impossible.

**Recommendation for Codex before P03 starts:** patch phase-1-audit-and-safety.md and phase-2-enable-ids-mode.md with the three items above, then update CODEX-LOG.md.
