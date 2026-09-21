# Q027 Evidence Plan — Proof Inventory Per Phase

Status: in-progress planning. No evidence has been captured yet; all rows below are prospective (proof artifact chosen before action, per AC-05). No screenshots exist for Q027 yet (the Codex-verified planning brief (2026-09-20)).

Related documents: [Execution Runbook](q027-execution-runbook.md) · [Procedure Corrections](q027-procedure-corrections.md) · [Acceptance Checklist](q027-acceptance-checklist.md)

## Evidence classes (used in the "Proof type" column)

- **Local-alert** — alert visible in OPNsense's own EVE/log viewer.
- **Transport-receipt** — receiver-side confirmation of the test log/syslog message (does not prove decoding/indexing).
- **Decoded/indexed/dashboard** — confirmed present, decoded, and searchable in the Wazuh dashboard.
- **Missing** — expected evidence not obtainable at this time; recorded as a gap, not fabricated.

## Screenshot conventions

Standard path: `evidence/screenshots/q027/<phase>-<short-label>.png`. Each screenshot reviewed for sensitive content before saving (no credentials, no external IPs, no raw config/EVE text visible). Embed width=900, maximum 2 images per phase in any README that references this plan. All screenshots are captured by the user (GUI operator); Claude has no live GUI tools in this assignment; Codex may read the open tab, while the user operates the GUI.

## Evidence matrix

| Phase | Proof artifact (chosen in advance) | Expected result | Negative control | Screenshot name/path | Privacy check | Proof type |
|---|---|---|---|---|---|---|
| 0 — Precheck | Backup filename/timestamp record; console-access confirmation note; interface list | Backup exists created today before the change, with the clock checked; console access confirmed; interfaces confirmed lab-only | N/A (precheck, no trigger) | `evidence/screenshots/q027/phase0-backup-list.png` | No config content visible, filename/date only | Local-alert N/A — administrative confirmation only |
| 0 — Checkpoint | Screenshot of Intrusion Detection settings page (view-only) | Shows service state, interface(s), ruleset(s), policy as currently configured | N/A | `evidence/screenshots/q027/phase0-ids-settings-view.png` | Crop/redact any unrelated interface IPs not in scope | Local view (no alert claim) |
| 1 — Enable/confirm detect-only | Screenshot of IDS status page post-Save/Apply | Service running, correct interface(s) bound, alert (not drop) policy, ruleset enabled | N/A (state confirmation, not a trigger test) | `evidence/screenshots/q027/phase1-ids-status-post-apply.png` | Redact any non-lab interface names/IPs | Local view (service state) |
| 2 — Signature identification | Screenshot of rule detail view showing chosen SID, message, match tuple | Exact SID/tuple recorded, matches what Phase 3 trigger will target | N/A | `evidence/screenshots/q027/phase2-signature-detail.png` | No unrelated rule content captured | Local view |
| 3 — Positive control trigger | Screenshot of EVE/log viewer entry for the matching SID/tuple/timestamp | Alert appears in local log viewer within recorded timing window | See same-row negative control | `evidence/screenshots/q027/phase3-positive-alert.png` | Redact source/destination IPs outside documented lab scope if UI shows extra columns; no raw EVE JSON pasted | Local-alert |
| 3 — Negative control trigger | Screenshot of EVE/log viewer showing absence of matching alert for altered tuple, same window | No alert for the deliberately mismatched tuple | (this row is the control for the row above) | `evidence/screenshots/q027/phase3-negative-control.png` | Same redaction rule | Local-alert (absence) |
| 3 — Wazuh correlation (if receiver available) | Screenshot of Wazuh dashboard search filtered to SID/timestamp | Same SID, tuple, and timestamp appear decoded/indexed | Same negative-control tuple confirmed absent in Wazuh dashboard | `evidence/screenshots/q027/phase3-wazuh-dashboard.png` | No credentials/tokens visible in dashboard chrome | Decoded/indexed/dashboard |
| 3 — Wazuh correlation (if receiver NOT available) | N/A | N/A | N/A | N/A | N/A | Missing — recorded as external live prerequisite gap, not fabricated |
| 4 — Rule disable confirm | Screenshot of ruleset view showing test SID disabled | Rule shown disabled, Save/Apply confirmed | N/A | `evidence/screenshots/q027/phase4-rule-disabled.png` | No unrelated rule state captured | Local view |
| 4 — Alert loss confirm | Screenshot of EVE/log viewer for repeat positive-control trigger post-disable | No alert for previously-matching tuple | Uses Phase 3 negative-control tuple as secondary confirmation of quiet baseline | `evidence/screenshots/q027/phase4-alert-loss.png` | Same redaction rule | Local-alert (absence) |
| 4 — Rule re-enable confirm | Screenshot of ruleset view showing test SID re-enabled | Rule shown enabled again, Save/Apply confirmed | N/A | `evidence/screenshots/q027/phase4-rule-reenabled.png` | Same | Local view |
| 4 — Alert restoration confirm | Screenshot of EVE/log viewer for repeat positive-control trigger post-re-enable | Alert reappears, matching SID/tuple, new timestamp in new window | Phase 3 negative-control tuple re-run, still absent | `evidence/screenshots/q027/phase4-alert-restored.png` | Same redaction rule | Local-alert |
| 5 — Health check | Screenshot/note of GUI reachability and unchanged out-of-scope settings | GUI reachable, no unintended interface/rule/service change vs Phase 0/1 baseline | N/A | `evidence/screenshots/q027/phase5-health-check.png` | No config content beyond confirmed scope | Local view |

## Distinguishing evidence tiers (per AC-05)

- A **local-alert** entry alone proves the sensor fired; it does **not** prove Wazuh correlation.
- A **transport-receipt** (e.g., receiver-side syslog receipt) alone does **not** prove the Wazuh dashboard decoded or indexed the event — this distinction is required per the Codex-verified planning brief (2026-09-20)and must not be conflated in any acceptance claim.
- Only a **decoded/indexed/dashboard** screenshot showing the matching SID/tuple/timestamp counts as Wazuh correlation evidence.
- Any row marked **Missing** in this table is an explicit gap (e.g., Wazuh receiver not yet available) and must not be presented as passed.

## No completed-project claim

This evidence plan is prospective. Completion of Q027 requires actual retained evidence captured during live execution, not this planning document (the Codex-verified planning brief (2026-09-20)). No row in this table currently has captured evidence attached.


## Independent control requirement

Every quiet/negative window needs proof the test traffic arrived, plus a separate known-positive control rule that still alerts. Use fresh timestamps/markers to exclude stale alerts and record loss/latency boundaries. Tuning evidence must compare equivalent before/after traffic and counts, not merely a disabled SID. Wazuh correlation is mandatory for current Q027 acceptance; an unavailable receiver is a gap. Phase labels here describe runbook steps; they do not claim completion of the six repository phases.
