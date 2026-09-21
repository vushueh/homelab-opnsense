# Q027 Procedure Corrections — IDS Detect-Only Sensor (OPNsense Suricata)

Status: in-progress planning. No live configuration or command has been executed for Q027. All items below are corrections to legacy phase1–phase6 documentation, not new observed facts.

Related documents: [Execution Runbook](q027-execution-runbook.md) · [Evidence Plan](q027-evidence-plan.md) · [Acceptance Checklist](q027-acceptance-checklist.md)

## Why this correction pass exists

Legacy Q027 predecessor docs (see the Codex-verified planning brief (2026-09-20)) described phase1–phase6 with three destructive break/fix examples and unverified plugin/category assumptions. Per owner rules, this planning pass corrects those defects before any GUI action is authorized, rather than repeating them.

## 1. Plugin/category assumptions

**Legacy claim:** Recommended installing an `os-suricata` plugin and enabling an "ET Open test category" ruleset.
**Correction:** The current OPNsense dashboard already shows an "Intrusion Detection" service label (observed Sept 20), so a separate plugin install is not established as necessary — it may already be built into base OPNsense IDS/IPS functionality. Whether an "ET Open test category" exists in the installed 26.1.11 ruleset UI is **unverified**. This runbook treats plugin presence/absence and exact category names as a first-checkpoint verification item (view-only), not a prerequisite to prescribe or install. Source: the Codex-verified planning brief (2026-09-20).

## 2. Dual-interface visibility

**Legacy gap:** Prior phases discussed removing/disabling ingress capture on one interface without accounting for egress visibility.
**Correction:** Removing capture on an ingress interface does not guarantee traffic is invisible to the sensor if a second (egress) interface also carries the flow. Any interface-scope change must document both interfaces in scope and confirm which interface(s) the assigned test rule/category is bound to before concluding "no detection" or "capture removed." Source: the Codex-verified planning brief (2026-09-20); https://docs.opnsense.org/manual/ips.html (interface selection affects capture).

## 3. Deterministic trigger requirement

**Legacy gap:** Prior "safe traffic alert" phase did not specify a repeatable, deterministic trigger tied to a verified installed signature.
**Correction:** A generic scan (e.g., nmap) is not guaranteed to fire any given signature. AC-03 requires: (a) confirm the exact signature/SID is installed and enabled in the current ruleset before testing, (b) construct a traffic tuple (source/destination/port/protocol/content) that matches that signature's documented match conditions, (c) run a positive control (expected alert) and a negative control (expected no alert) with recorded timing windows. No signature ID is asserted here; it must be read from the live ruleset during the checkpoint step.

## 4. Overly broad break/fix scope

**Legacy defect:** Phase5 legacy examples were "wrong scope," "stopped service," and "all rules disabled" — each a broad, hard-to-bound change to existing monitoring.
**Correction:** This plan proposes replacing broad break/fix with a single narrowly scoped test: disable one specific, non-baseline test rule, confirm loss of alert on the positive-control trigger, then re-enable and confirm alert restoration — never a full ruleset disable or service stop, and never applied to any interface already carrying production/management traffic. Existing baseline detection (if already enabled) must remain untouched throughout. Source: the Codex-verified planning brief (2026-09-20).

## 5. Raw EVE disclosure

**Legacy risk:** Prior phases implied pasting raw EVE JSON log lines into shared docs/evidence for review.
**Correction:** Raw EVE log content and raw configuration values must not be placed in Git, delivery documents, or shared evidence artifacts (owner rule, the Codex-verified planning brief (2026-09-20)). Evidence capture instead records: field names present, SID, tuple, timestamp, and a screenshot of the dashboard/log viewer with sensitive values (source IPs outside the lab scope, credentials, tokens) redacted or cropped out. No raw log excerpt is reproduced in any delivery file.

## 6. Older UI wording

**Legacy defect:** Predecessor docs used older Suricata/OPNsense UI wording (e.g., menu labels, "IPS" vs "Intrusion Detection" naming) that may not match the installed 26.1.11_10-amd64 GUI.
**Correction:** All GUI step references in the runbook use the observed current label "Intrusion Detection" (as seen on the dashboard Sept 20) and defer exact submenu/button wording to the live-verification checkpoint rather than asserting older wording is still accurate. https://docs.opnsense.org/manual/ips.html is cited as the primary reference, with an explicit note that installed-version wording may differ and must be confirmed live before any step is executed.

## Scope decision recorded

Per legacy item 4, the break/fix scope choice (single test-rule disable/re-enable with control rule, described in §4 above) is a proposed scope decision requiring explicit acceptance before it replaces the legacy three-scenario checklist. It does not carry forward or reinstate the legacy three-checkbox claim, and does not itself constitute execution or approval — approval is a separate live gate (see Execution Runbook §Approval Gate).
