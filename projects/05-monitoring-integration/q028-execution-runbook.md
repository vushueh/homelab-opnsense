# Q028 Execution Runbook — OPNsense Monitoring and VoIP Observability

Status: resumed 2026-09-24 with Leonel's approval to complete Q028 autonomously, including U1, bounded verification, cleanup and scoped publication. The initial handoff is historical; the private `RESUME-20260924.md` records current execution. Initial gates G1/G2/G3/G5/GF remain valid. Controlled V3 remains deferred until a stable disposable softphone is available. Final indexed acceptance passed and temporary components were removed; see [closeout](project-closeout.md). The handoff and early gate statuses below are chronological records, superseded by the dated acceptance checklist.

Private addresses, receiver mappings and detailed readbacks live in the private
`homelab-management/projects/q028-monitoring-integration/` package; this public
page uses role names only.

Related: [acceptance checklist](q028-acceptance-checklist.md) ·
[procedure corrections](q028-procedure-corrections.md) ·
[P03 closeout](../03-ids-ips-suricata/project-closeout.md)

## Done when

Firewall and sanitized VoIP observability answer the named questions below,
with alert tests and a measured log-pipeline loss test.

## Named questions

| ID | Question | Source → transport | Answered by |
|---|---|---|---|
| F1 | What was blocked, where, and by which rule? | pf filter log → existing UDP syslog target | Wazuh pf decoding / existing OPNsense rule, saved search |
| F2 | Did the approved allowed lab test cross? | Logged lab pass rule → same | Verified pass-action search on the exact test tuple |
| F3 | Did IDS detect the benign marker? | Suricata EVE → same (Q027 path) | Existing Q027 decoder/rule, SID filter |
| F4 | Which admin/auth events occurred? | System/auth application → same | Existing OPNsense auth rules (to verify) |
| V1 | Which synthetic endpoint lost registration? | Isolated Asterisk contact state → sanitized JSON → syslog | New scoped decoder/rule |
| V2 | Did it recover, with no false alarm while healthy? | Same periodic samples | State/recovery rules |
| P1 | Did telemetry disappear, or did registration fail? | Sequenced samples + independent receipt check | Heartbeat/gap comparison |

Use the field names verified against the current daily index in the [query guide](verification/siem-queries.md).

## Scope

In: firewall pass/block, admin/auth, Q027 IDS regression, saved searches,
isolated V3 registration loss/recovery, measured pipeline loss.

Out: production PBX faults, trunks/DIDs, QoS/NAT, MOS or media-quality claims,
VPN events (P04 not built), IPS, clock repair without its own gate.

NetFlow correction (discovery, 2026-09-23): OPNsense already exports v9 flows
from LAN only to its own dedicated collector instance, live since 2026-06-21.
Codex's repository-only plan missed that instance and proposed deferral; live
readback overrides it. Gate G5 added both lab capture interfaces; exact TCP tuples were verified on the collector on 2026-09-24.

| ID | Question | Source → transport | Answered by |
|---|---|---|---|
| F5 | What flow crossed for the lab test, and how many bytes? | OPNsense NetFlow v9 → dedicated collector | `nfdump` on the collector for the exact tuple/window |

## Phase 0 — Read-only discovery (LIVE-RO, approved 2026-09-23)

Leonel approved read-only inspection of OPNsense, the Wazuh manager and the
FreePBX host. No writes, restarts or key/trust changes.

| Host | Reads | Status |
|---|---|---|
| Wazuh manager | service state, listeners, `<remote>` blocks, rules/decoders inventory, `wazuh-analysisd -t`, alert fields | Unprivileged reads done; privileged reads need Leonel's sudo |
| OPNsense | firmware, clock/NTP, generated remote destinations, filter/system log presence, NetFlow process | Done 2026-09-23: three UDP 514 targets (two SOC sensor, one Wazuh); filter log live with sequence IDs; system log silent since 2026-09-21; clock ~129.5 s ahead; LAN-only NetFlow live |
| FreePBX | Asterisk version/uptime, endpoint/contact counts, time sync, logger channels | Done 2026-09-23: Asterisk 22.6, NTP synced, 5 endpoints, file loggers only (no syslog channel) |

## Change gates (each approved separately)

The exact rev-2 gate package (G0–G5, GF), after Codex's FAIL review of rev 1,
lives in the private `homelab-management/projects/q028-monitoring-integration/q028-change-gates.md`.
Summary below.

Every gate needs: exact before/after values, protected backup, operator,
verification, delta-only rollback, stop triggers.

1. **G1 Logging selections** — add only missing observed filter/system/auth
   applications to the existing Wazuh target; keep Suricata and levels and
   leave both SOC-sensor targets untouched; diagnose the silent system log
   first; prefer
   logging on exact lab test rules over logging all default passes. Stop on any
   unrelated form change (Q027's `miniupnpd` removal precedent).
2. **G2 Decoding** — replay a sanitized real filter log line through
   `wazuh-logtest`; add custom decoding only if built-in pf decoding fails. Add
   VoIP JSON/state rules after rule-ID collision and wrong-source tests. Back up
   touched files; validate before any authorized manager restart.
3. **G3 V3 lab** — disposable endpoints and sanitized state emitter only.
   Proposed cadence 15 s; loss after two absent samples; telemetry stale after
   60 s — frozen after measuring expiry. Restore endpoints, remove emitter.
4. **G5 NetFlow lab capture** — add only the VLAN 40 and VLAN 250 lab
   interfaces to the existing export; keep LAN and the destination unchanged.
   Verify fresh records for the test tuple on the dedicated collector; roll
   back by removing only the two interfaces. Stop on collector errors or any
   change to the other exporter's collector.
5. **G4 Receiver** — only if needed, add the exact lab sender to syslog source
   acceptance; preserve existing entries; regression-test Q027.

## Alert and loss tests

- Firewall: logged allow and block tuples, plus Q027 marker and a
  non-matching control after current SID verification.
- VoIP: disconnect only the synthetic endpoint through measured expiry;
  require loss alert, recovery event and zero loss alerts while healthy.
- Pipeline fault: disable only the approved forwarding target briefly; local
  logs must keep growing; an independent watcher must flag stale receipt;
  restore and prove new events plus baseline controls.
- Loss: compare emitted, manager-received and indexed identity sets;
  `loss% = (emitted − indexed) / emitted`; report duplicates, late arrivals
  and stage-specific gaps separately. Absence is accepted only after later
  controls index and searches return without shard errors.

Clock: OPNsense ran ~127 s ahead at Q027. Use sequence IDs and byte
boundaries, not cross-host latency, until a separate clock gate closes it.

## Evidence layout

Public (this folder, synthetic identifiers, reviewed crops): runbook,
acceptance checklist, procedure corrections, `verification/siem-queries.md`,
`troubleshooting/break-fix-log.md`, `evidence/screenshots/p05-phaseN-*.png`,
`project-closeout.md`. Private (homelab-management): dated discovery, alert,
loss and rollback JSON/Markdown with real addressing. Secrets and raw backups
stay outside all Git.
