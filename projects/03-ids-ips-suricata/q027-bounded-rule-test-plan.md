# Q027 proposed bounded tuning and break/fix

Status: APPROVED by Leonel on September20; executed with local tuning, independent break/fix and final cleanup verified. Final indexed quiet-window and cleanup-control validation passed; see project-closeout.md. Approval includes the temporary rule, scoped tuning/disable/restore, cleanup, and replacing the older broad break/fix exercises. Existing positive end-to-end proof and local positive-negative-positive test are retained separately.

## Exact temporary rule
Use Services > Intrusion Detection > User defined. Add one enabled rule:
- Description: Q027 TEMP lab-pair alert validation
- Source: 192.168.40.158/32
- Destination: 192.168.250.172/32
- Action: Alert
- SSL fingerprint: empty
- Bypass: disabled

The installed IDS model and vendor OPNsense.rules template were read. Empty fingerprint generates an IP rule with the supplied source/destination. Enabled-rule SID derives from4294967295 minus array position; record the actual rendered SID after creation rather than assuming it. Source/destination restrict this rule to owned Kali-to-Metasploitable2 traffic. Existing interfaces, threat rules, policies, capture mode and forwarding remain unchanged.

## Prechecks and stop conditions
Before applying, verify console availability, latest backup/recovery readiness, no pending unrelated GUI changes, existing user-rule count and no matching description, IDS running and current PCAP baseline. Preserve the existing encrypted backup; take a new encrypted download if configuration has changed since it (Wazuh syslog application changed). Do not restore any backup. Save/Apply only the named rule; expected effect is a rules reload. Stop for service failure, unexpected global changes, failed control alert, or changed rule identity. No production interface selection, whole-ruleset disable, IPS/drop/pass/bypass or firewall restart.

## Separate observations
1. Tuning: enable temporary broad pair rule; send two nonmatching-marker echoes and two matching SID2062640 echoes. Record test-rule counts and existing control SID counts using fresh local log byte boundaries. The broad pair rule intentionally adds low-value alerts to harmless traffic. Disable ONLY that temporary rule; repeat equivalent traffic, prove delivery, reduced test-rule noise and unaffected2062640 control. Document this as a synthetic tuning demonstration, not evidence that a real threat rule was a false positive.
2. Break/fix: independently re-enable the temporary rule and establish a new passing baseline. Disable it, run its trigger and the independent2062640 control, diagnose missing test-rule alerts while control and delivery remain healthy. Re-enable it and verify alert restoration with fresh evidence. These are separate recorded windows, not reused tuning results.
3. Cleanup: remove ONLY the temporary rule, apply, verify its absence, current IDS running/capture scope, and one existing control alert. Preserve all pre-existing monitoring. Keep positive test traffic bounded to two echoes per pattern/window; no scanning or exploitation.

Rollback: reverse only the last temporary-rule delta. If that cannot be verified, stop and use console-assisted diagnosis; full restore/reboot is not approved. Cleanup of the named temporary test rule is part of the proposed scope.

## Approved scope replacement
Leonel accepted the isolated temporary-rule disable/restore exercise in place of the older broad interface/service fault exercises. Existing threat SID2062640 remained active. He later explicitly authorized Codex to complete the remaining scoped GUI actions while away.

## Executed test-method correction
IP-only rules inspect once per flow direction. Repeated ICMP traffic reused a flow and was unsuitable for a fresh break/fix baseline. The actual independent break/fix exercise therefore used one harmless TCP connection to the same owned target's known SSH port22, closed without login, plus two ICMP control echoes per state. Source/destination, temporary SID/action and production boundary remained unchanged. Before every traffic window, confirm the most recent Apply completed both observed reloads. Do not accept a stale completion record. Failed original attempts and reviewed results remain in evidence.

## Evidence
Retain actual SID, before/after rule state, commands/delivery, local boundary counts, control SID counts, final cleanup/health and two phase-appropriate screenshots. Wazuh indexing may lag; verify matching positive controls and do not infer a negative pass from an index that has not processed the later positive control. NTP selected a peer on latest recheck but still reports leap_alarm/spike_detect; do not declare the127-second offset repaired yet.
