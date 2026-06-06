# CLAUDE-REVIEW.md — Open Items for Codex

**How this works:** Claude writes items here with status OPEN. Codex resolves them before doing new work. Claude marks CLOSED after verifying.

---

## Active Items

*None. Ready to start Project 03.*

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
