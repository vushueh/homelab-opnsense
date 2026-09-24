# Project Closeout — Q028: Monitoring and SIEM Integration

**Repo:** homelab-opnsense | **Closed:** 2026-09-24 | **Closed by:** Codex under Leonel's autonomous-completion approval

I closed the accepted firewall monitoring and SIEM recovery scope after comparing exact event identities at the manager and index. The [acceptance checklist](q028-acceptance-checklist.md) records all pass/deferred decisions. No controlled VoIP-loss or successful dashboard-login claim is hidden in the completion status.

## Closeout Checks

- [x] Five phase rows and first-person narratives match the accepted scope; explicit deferred triggers are retained.
- [x] Current source, manager, index, collector, health and cleanup evidence retained; all seven manager/indexed identity populations match exactly.
- [x] Screenshot evidence and documented exceptions are listed in the [evidence inventory](evidence/README.md); no recreated dashboard images.
- [x] Public package excludes private addresses, credentials and raw protected configuration; private manifests preserve exact provenance.
- [x] Temporary extension, emitters, watcher, state, staging and replay input removed; protected rollback copies retained. Sudoers validated; a new passwordless sudo attempt was denied.
- [x] Permanent auth target, lab flow capture and scoped Wazuh rules retained. Narrow workstation audit exclusion verified without changing Windows audit policy.
- [x] Owner review/log, root/project indexes and canonical state/queue updated; Q029 remains planned and unstarted.
- [x] STAR and five-part collaboration record are in the [README](README.md#portfolio-summary).
- [x] Re-verification and rollback boundaries are in the runbook and private recovery record.
- [x] Publication is authorized for this Q028 package; GitHub PR records own the actual merge outcome.
- [x] Vault synchronization omitted because it was not requested. Withheld Q018 and unrelated changes are excluded.

## Measured Outcome

The controlled forwarding test logged 1,260 packets locally; 863 were received outside the intentional outage, with 397 absent only while forwarding was disabled. Adding the 60 baseline markers gives 923 expected indexed events, each present once. The stale alert fired after 67 seconds. Three benign allowed connections produced six firewall observations; the IDS control produced eight positive and zero negative observations. Ten authentication events, four pipeline-state events, 645 PBX-state events and the three-event recovery canary all matched manager identities exactly.

U1 reduced root usage from 99% to 67%, leaving 57 GB free. Two open-deleted raw files were preserved before Filebeat restarted. Four closed daily files were copied, checksum-compared, gzip-tested and only then removed from root. Three replay files delivered 66,199 valid non-noise records from acknowledged offsets or an initially empty daily index. All replay bytes were acknowledged before the temporary input was removed. The registry was never reset. Twenty malformed historical fragments remain in preserved raw sources; complete historical recovery is not claimed.

The indexer's 90-day retention policy is active on the current daily index. That policy does not rotate raw daily alert files. Watermark protection remained enabled. The single-node cluster remains yellow because of unassigned replicas; all 198 active primaries were available, with zero initializing/relocating shards or pending tasks at verification.

## Carried-Forward Items

| Item | Trigger and location | Owner |
|---|---|---|
| Controlled V3 healthy/lost/recovered and emitter-down sequence | A dedicated test softphone stays registered; rebuild only disposable components from the private runbook in a new bounded window | Leonel, PBX owner |
| Dashboard saved-query import and screenshot | An authenticated Wazuh GUI session is available; [portable definitions](verification/q028-saved-queries.ndjson) and [query guide](verification/siem-queries.md) are ready | Leonel, SIEM owner |
| Handset audio verification | Next normal attended family call; current acceptance covers unchanged configuration and registered 200/201, not media | Leonel |
| Raw-log retention, compression and disk/backlog alerts | Private recovery follow-up; review capacity before the next scheduled snapshot, then automate under a separate bounded plan | Leonel, SIEM owner |
| Sensor clock skew | Before latency-sensitive correlation or IPS; existing Q027 follow-up | OPNsense owner |

## Cross-Repo Updates

| Repository | Owning record |
|---|---|
| homelab-opnsense | This closeout, evidence, phase story, review/log, project indexes and predecessor handoff |
| homelab-management | Private RESUME, filtered manager/index evidence, archive manifests, retained Wazuh source and U1 resolution |
| family-projects-ai-playbook | docs/state.yaml and docs/homelab-goals.yaml; Q028 complete with named deferrals, Q029 not started |

## What Happens Next

Q029 — CCNA P02 Wireshark Labs is the immediate successor. The [canonical queue](https://github.com/vushueh/family-projects-ai-playbook/blob/main/docs/homelab-goals.yaml) selects its entry gate; this closeout does not start it.
