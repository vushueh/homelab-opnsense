# Q027 acceptance checklist

Updated September20,2026 (September21UTC). Q027 is Complete for the approved detect-only scope; publication is pending. Claude's four-document drafting package was accepted after Codex corrections; planning acceptance is separate from live project acceptance.

| ID | Requirement | Current result |
|---|---|---|
| L1 | Pre-change private backup | PASS: refreshed encrypted download, size and SHA256 recorded; restore untested. |
| L2 | Console/out-of-band readiness | PASS by Leonel's confirmation; Hyper-V VM running status independently checked. Console interaction was not independently captured. |
| L3 | Owned test scope, preserve existing monitoring | PASS: exact Kali VLAN40 to Metasploitable2 VLAN250 pair; existing WAN monitoring retained. No new WAN experiment. |
| L4 | Inspect current IDS settings | PASS: GUI and configuration readback. |
| L5 | Detect-only IDS runtime | PASS: existing Suricata PCAP process and five interfaces preserved. |
| L6 | Installed signature and match tuple | PASS: actual installed SID2062640 and matching inert ICMP payload. |
| L7 | Fresh matching local alert | PASS: EVE byte-boundary windows with delivery proof. |
| L8 | Negative control | PASS locally: positive/negative/positive counts8/0/8. PASS indexed: surrounding16, quiet0, later positive8 using exact embedded timestamp windows. |
| L9 | Decoded/indexed/dashboard correlation | PASS for the previously captured positive event; exact SID, endpoints, embedded timestamp and flow match. Final cleanup event also exactly correlated; eight final controls indexed. |
| L10 | Separate scoped fault and alert loss | PASS locally: fresh TCP connection succeeded with temporary SID0 and independent control SID8 after disabling only temporary rule. |
| L11 | Restore and prove alert return | PASS locally: fresh TCP temporary SID1, independent ICMP control SID8 after re-enable/reload. |
| L12 | Cleanup, reachability, unintended-change check | PASS: user-rule list and generated rule absent, same PID58176 and interfaces, PCAP runtime retained, GUI/SSH reachable, fresh TCP succeeds and independent ICMP control produces8 alerts. No full unrelated-config diff or restore test claimed. |
| L13 | Separate synthetic tuning comparison | PASS locally: temporary alert1 before disable,0 afterward, independent control8 in both states; all traffic delivered. Not a real threat-rule false-positive suppression claim. |
| L14 | Explicit IPS decision | DEFERRED by design: preserve PCAP; production false-positive tolerance unproven, existing WAN scope, and unresolved clock skew. No IPS/drop/bypass change. |

## Corrections that matter

- Leonel approved replacing old broad interface/service fault exercises with an isolated disposable-rule disable/restore exercise.
- Repeated Apply was avoided. The observed two consecutive reloads must both complete before traffic; a prior reload-complete message is not sufficient.
- IP-only rules inspect once per flow direction. Reused ICMP flow was an invalid new baseline; fresh TCP connections to the same target's known SSH port supplied distinct flows without authentication or application commands. Failed attempts remain recorded.
- Local and indexed evidence are separate. Zero dashboard hits while Filebeat is behind do not prove a negative test.
- OPNsense time remains roughly127seconds ahead with NTP alarm/spike status. Byte boundaries avoid depending on sensor time for local windows. No clock adjustment was performed.

## Closed acceptance and operational follow-ups

Final indexed validation passed after renewed login: see [exact windows and matched final event](evidence/q027-final-indexed-validation.json). All required proof for the approved scope is present. Q028 remains unstarted. Publication authorized; GitHub PR records own merge status.

Owner operational follow-ups: diagnose the persistent sensor clock skew before time-sensitive correlation or IPS; monitor Filebeat delivery/backlog and historical indexing errors without claiming zero loss. Backup restoration remains untested by design. These do not invalidate the observed detect-only controls or authorize additional live changes.

See [detailed evidence](evidence/q027-controlled-test-20260920.md), [scope and rollback](q027-bounded-rule-test-plan.md), and [project narrative](README.md).
