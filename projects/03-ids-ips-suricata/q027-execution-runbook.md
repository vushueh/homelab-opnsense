# Q027 Execution Runbook — Detect-Only IDS Sensor (GUI-First)

Status: in-progress planning. All live gates below are **pending**. Only the dashboard was read on 2026-09-20: OPNsense 26.1.11_10-amd64, FreeBSD 14.3, last configuration change September 3. The Intrusion Detection label alone does not establish enabled state or capture mode. None of the live steps below has been executed. All actions in Phase 1+ are performed by the user in the OPNsense GUI; Codex/Claude do not execute live commands (owner rule, the Codex-verified planning brief (2026-09-20)).

Related documents: [Procedure Corrections](q027-procedure-corrections.md) · [Evidence Plan](q027-evidence-plan.md) · [Acceptance Checklist](q027-acceptance-checklist.md)

## Roles

- **User (operator):** performs all GUI clicks, console access, backup export/import, and any Save/Apply action. Only the user executes live changes.
- **Claude (this document):** drafts steps, evidence expectations, rollback text. No execution.
- **Codex:** independent source/technical review of this package; does not execute live infrastructure commands.

## Global stop conditions (apply to every phase)

Stop and do not proceed if any of the following occurs:
1. No current configuration backup exists or cannot be confirmed exported.
2. Console/out-of-band access to the firewall is not confirmed available before a change.
3. The interface(s) in scope cannot be confirmed as lab-only (no WAN, no production/management traffic).
4. A step would disrupt existing monitoring beyond the exact approved delta. Whole-service stop or all-rule disable is outside this proposed narrow exercise, not a blanket prohibition in the owner rules.
5. Any required fact (interface name, rule/SID, EVE transport, Wazuh receiver, target IP) is unverified and cannot be confirmed live in that moment — record as a checkpoint, do not substitute a guess.
6. The user does not explicitly grant the Approval Gate for that phase.

## Phase 0 — Precheck (must complete before any Save/Apply anywhere in this plan)

0.1. User confirms a current OPNsense configuration backup exists (Config > Backups, or manual export) created today before the change, with the clock checked. Record filename/timestamp in evidence log — no raw config content captured.
0.2. User confirms console or out-of-band management access to the firewall is available (e.g., physical/serial/hypervisor console), independent of the web GUI, in case a change needs manual recovery.
0.3. User confirms which interface(s) are in scope and that none carry WAN or production/management traffic. Record interface names as observed live (do not reuse any name from legacy docs without live confirmation).
0.4. User confirms the rules/ruleset state currently in effect for Intrusion Detection is viewable (Services > Intrusion Detection, or current equivalent menu per installed 26.1.11 GUI) — **view only, no Save/Apply**.
0.5. User confirms Wazuh receiver reachability/existence, if a receiver is intended for this phase, or records correlation as an unresolved gate. Q027 cannot close successfully without this required proof unless Leonel explicitly accepts a revised scope.
0.6. Record Phase 0 results in the evidence log per [Evidence Plan](q027-evidence-plan.md) Phase 0 row.

**Checkpoint (first actionable step, no Save/Apply):** View current Intrusion Detection settings page in the GUI and record: service enabled state, interface(s) assigned, ruleset(s) enabled, policy (alert/drop/disabled) if shown. This is read-only reconnaissance — continue read-only inspection as needed; stop before a configuration change until its exact scope is approved. Read-only confirmation of an already-enabled service needs no extra approval.

## Approval Gate 1 — Enable/confirm detect-only IDS

Proposed approval summary for the named change before any Save/Apply in Phase 1:
> "I approve enabling (or confirming already-enabled) Suricata Intrusion Detection in alert-only policy on interface(s) [named after 0.3], with backup [filename from 0.1] confirmed, and I will not enable drop/IPS mode."

Only after approval covering these named changes is recorded may Phase 1 proceed.

## Phase 1 — Audit and enable detect-only (alert policy)

1.1. If IDS is not yet enabled: user enables the Intrusion Detection service, selects only the lab interface(s) confirmed in 0.3, selects PCAP/IDS capture mode (or verified equivalent on 26.1), with IPS/Netmap/Divert disabled, and alert action on the specific test rules (not drop), and does not disable any pre-existing baseline monitoring already active on other interfaces.
1.2. If IDS is already enabled (per 0.4 finding): per brief instruction, do not force a recreate — record current state as baseline and proceed to verification without re-saving unrelated settings.
1.3. User enables a small, named ruleset/category (exact name recorded live from GUI, not assumed) sufficient to support the Phase 3 trigger.
1.4. Save/Apply (only this specific change).
1.5. **Verification:** user reloads the Intrusion Detection status page and confirms service running state, interface binding, and ruleset enabled state match what was intended in 1.1–1.3. Screenshot per Evidence Plan Phase 1.
1.6. **Rollback:** if verification fails or unintended interfaces/services are affected, user reverses only the approved setting delta to the recorded baseline and verifies management and lab connectivity. Stop if that fails; do not import the full configuration, reboot, or reset. Full restoration needs its own concrete approved recovery plan.

## Approval Gate 2 — Deterministic benign trigger test

Proposed approval summary (equivalent clear authorization is sufficient):
> "I approve running the benign trigger traffic described in the Evidence Plan Phase 3 row, from VLAN40 lab source to owned VLAN250 target, within the recorded timing window, with no Internet-facing target."

## Phase 2 — Identify verified signature (checkpoint, view-only)

2.1. User locates one specific signature/SID currently enabled in the ruleset from 1.3, records its SID, message text, and documented match tuple exactly as shown in the GUI/rule detail view.
2.2. This SID becomes the basis for the Phase 3 trigger construction. If no suitable signature is found, stop and record as a gap — do not invent a SID.

## Phase 3 — Deterministic benign trigger + EVE/Wazuh correlation

3.1. User (or a lab-scoped test host under user control) sends traffic matching the exact tuple/protocol/content documented for the SID from 2.1, sourced from lab VLAN40 to owned VLAN250 target only.
3.2. **Positive control:** confirm local EVE log (via GUI log viewer, not raw file export) shows an alert with the matching SID, tuple, and timestamp within the recorded window.
3.3. **Negative control:** Record the exact command/traffic count, fresh marker and timestamp. Establish traffic delivery through receiver observation or scoped packet capture; quiet logs alone are not a negative-control PASS. Then run the same trigger mechanism with one tuple element altered (a rule-dependent nonmatching payload; changing a port is valid only if that port is part of the signature condition) and confirm no matching alert appears in the same window.
3.4. Required completion gate, once the Wazuh receiver is qualified (per 0.5): confirm the same SID/tuple/timestamp appears decoded and indexed in the Wazuh dashboard — syslog transport receipt alone is not sufficient evidence of correlation (see Procedure Corrections and the Codex-verified planning brief (2026-09-20)).
3.5. Record results per Evidence Plan Phase 3 row. No raw EVE/log content is copied into any document — only field values (SID, tuple, timestamp) and a redacted screenshot reference.

## Approval Gate 3 — Bounded tuning / break-fix test

Proposed approval summary (equivalent clear authorization is sufficient):
> "I approve disabling exactly one named test rule (SID recorded), re-running the positive-control trigger to confirm alert loss, then re-enabling that same rule and confirming alert restoration. I will not disable the whole ruleset or stop the IDS service."

## Phase 4 — Tuning and bounded break/fix

4.1. User disables the single test-rule SID identified in 2.1 (or a second dedicated test SID if 2.1's rule is needed as a control).
4.2. Save/Apply (only this change).
4.3. Re-run the Phase 3.1 trigger; confirm no alert appears (expected: alert loss confirms rule was the detection source).
4.4. Re-enable the same SID; Save/Apply.
4.5. Re-run the Phase 3.1 trigger; confirm alert reappears with matching SID/tuple.
4.6. **Rollback:** if re-enabling does not restore the alert, stop and diagnose capture/rule loading/traffic rather than assuming a full restore is necessary. Reverse the recorded test-rule delta if possible; a full configuration restore is not authorized by this runbook.
4.7. Record all four sub-results (disable-confirm, loss-confirm, re-enable-confirm, restoration-confirm) per Evidence Plan Phase 4 row.

## Phase 5 — Management/traffic health check (post-change)

5.1. User confirms firewall GUI/console remains reachable and responsive.
5.2. User confirms no unintended interface, rule, or service state changed outside the scope of Phases 1 and 4 (compare against 0.4 baseline and 1.5/4.7 records).
5.3. Record health-check pass/fail in evidence log.

## Phase 6 — IPS-deferred decision (documentation only, no action)

6.1. This plan explicitly defers any move to drop/IPS (blocking) mode. No step in this document enables IPS/drop policy.
6.2. Decision to consider IPS mode in a future project step requires a separate future planning pass and its own approval gate; it is out of scope here (see project-level future options below).

## Project-level future options (not part of this checkpoint)

- Evaluating IPS/drop mode after sustained detect-only stability — future, unapproved, no timeline implied.
- Expanding ruleset coverage beyond the small initial set — future.
- Formal Wazuh rule/dashboard tuning beyond basic correlation confirmation — future.
- os-suricata plugin evaluation if base IDS proves insufficient — future, pending 0.4/Checkpoint findings (see Procedure Corrections §1).

These are explicitly separated from the first actionable checkpoint (Phase 0 view-only Intrusion Detection settings review) and are not authorized by this document.


## Required qualification record before implementation

Record: current enabled state and capture mode; assigned interface labels mapped to VLANs and both flow directions; existing rule/policy actions; backup filename, time and verified nonempty file kept outside Git; working hypervisor console; owned source/target addresses and route; test signature conditions and expected count; EVE forwarding settings; Wazuh receiver/protocol/port/source filter and current ingestion health. Do not weaken HOME_NET or change firewall rules merely to make a test pass. If existing settings include production monitoring, preserve them and stop to review the conflict.

Before approval, replace every unresolved target with verified values in a small change record listing exact before/after settings, operator, test commands, observation windows, stop triggers and delta rollback. The generic summaries above are not executable approval requests yet.

## EVE/Wazuh and tuning details

Identify the existing collection route first. If a new EVE/syslog route is necessary, prepare a separate exact change: firewall application/level filter, transport/receiver, receiver source restriction and log collector/decoder. Never install a Linux agent on OPNsense by copying an Ubuntu example. Verify receipt, JSON field decoding, alert creation, index visibility and dashboard search separately. Correlate SID, source/destination, protocol, fresh test timestamp and flow/marker where available, accounting for clock offset. A sanitized representative event may be used in an approved offline decoder test; do not dump raw logs.

Wazuh's [JSON decoder documentation](https://documentation.wazuh.com/current/user-manual/ruleset/decoders/json-decoder.html) shows Suricata JSON field extraction. This is a reference, not proof that this deployment's syslog framing decodes correctly.

Tuning is distinct from break/fix: baseline a chosen rule's useful and unwanted alerts over a defined traffic set/window; propose one narrow policy change if justified; repeat equivalent traffic and retain counts plus a still-working control SID. If no noisy rule exists, document the observation and use an approved dedicated test rule for a tuning demonstration rather than suppressing useful security detections. For break/fix, keep a second independent control rule enabled and trigger it during the quiet phase, proving collection is still working. Disable/re-enable only an isolated temporary test rule with the approved scope; do not assume an existing threat rule is disposable.

If no installed signature yields a benign deterministic trigger, record that gap. A dedicated local marker signature and owned test endpoint can be proposed only after verifying the installed UI supports the intended rule and validating syntax without changing generated files. Do not guess rule IDs or issue traffic until exact endpoints, payload, bounded count and cleanup are reviewed.

The runbook's step numbering is execution sequencing, not a replacement for the repository's six phase statuses. Only Phase 1 audit/planning is active. Legacy break/fix requirements remain pending until the scope decision is accepted; no old checkbox is silently counted as passed.
