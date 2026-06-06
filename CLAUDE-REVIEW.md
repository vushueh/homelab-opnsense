# CLAUDE-REVIEW.md — Open Items for Codex

**How this works:** Claude writes items here with status OPEN. Codex resolves them before doing new work. Claude marks CLOSED after verifying.

---

## Active Items

### 🔴 OPEN — Item OPN-001: Review family scaffold and local skill sync

**Context:** Codex restructured the repo into the same family-project model used by the other homelab repos.

**Claude should review:**

- `README.md`
- `AGENTS.md`
- `WORKFLOW.md`
- `projects/README.md`
- `docs/cross-family-integration.md`
- `skills/opnsense-family.md`
- `projects/03-ids-ips-suricata/`
- `projects/04-vpn-remote-access/`
- `projects/05-monitoring-integration/`
- `projects/06-high-availability/`

**Questions for Claude:**

1. Does the project order make sense: P03 Suricata → P05 Monitoring → P04 VPN → P06 HA design?
2. Should `skills/opnsense-family.md` be copied into local skill folders?
3. Should Project 03 get a dedicated local skill before Leonel starts it?
4. Are there any live OPNsense safety gates missing before Suricata work begins?

---

## Closed Items

*None yet.*
