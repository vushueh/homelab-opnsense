# Q028 acceptance checklist

Verified 2026-09-24 08:30–08:33 UTC. Q028 closes in the accepted monitoring/recovery scope with the explicit deferrals below. Counts are tied to exact test identities on the September 24 daily index; every query returned without timeout or failed shards. The [sanitized summary](evidence/q028-acceptance-summary.json) and private exact-identity proof support this table.

| ID | Requirement | Final result |
|---|---|---|
| A1 | Fresh discovery of OPNsense, Wazuh and PBX | PASS; initial audit plus current readbacks |
| A2 | Protected pre-change backup | PASS; firewall/PBX backups retained; Wazuh configs and registry preserved before changes |
| A3 | Firewall block events decoded and indexed (F1) | PASS; 60 baseline plus 863 GF receipts = 923 distinct source ports; exact manager/indexed IDs; zero duplicates |
| A4 | Approved allowed lab tuples indexed (F2) | PASS; three TCP connections, six observations across two logged rules; exact manager/indexed IDs |
| A5 | IDS positive and nonmatching control (F3) | PASS; eight indexed observations for two positive ICMP packets; zero negative-control detections; later canary indexed |
| A6 | Authentication events indexed (F4) | PASS; ten source-scoped successful SSH authentication events, exact manager/indexed IDs |
| A7 | Reusable searches with verified fields | PASS for API predicates and six portable query definitions; dashboard import/screenshot DEFERRED until an authenticated GUI session is available |
| A8 | Controlled V3 loss/recovery; family unaffected | DEFERRED by Leonel; revive with a dedicated softphone that remains registered. 645 indexed state events (638 samples, four unplanned losses, three recoveries) are observational, not controlled acceptance |
| A9 | No false loss during controlled healthy samples | DEFERRED with A8; not inferred from unplanned phone behavior |
| A10 | Pipeline fault and emitter-down distinction | PASS GF: local 1260/1260, receipt stale in 67 s, byte-identical restore. Controlled emitter-down experiment DEFERRED with V3 |
| A11 | Stage-specific loss and duplicates | PASS; 397 packets were intentionally not forwarded; manager→index 923/923, zero duplicates. UDP does not backfill the outage |
| A12 | Cleanup and family boundary | PASS; 9028 absent, family 200/201 registered and configuration hashes unchanged; heartbeat/watcher/staging/replay input/sudo removed. Handset audio test DEFERRED to next attended family call |
| A13 | Exact test flow on dedicated collector (F5) | PASS; three connection tuples, ten exported records, 24 packets and 1,192 bytes; not unique wire volume |
| A14 | Public/private evidence boundary | PASS; public evidence uses roles, source-port identities and aggregate counts; protected backups and internal addressing stay private |
| U1 | Indexing recovery | PASS; root 67% with 57 GB free, no blocked indices, four services active, TLS output passed, current canary 3/3; no disk growth or disabled watermarks |

The indexed pipeline records are the GF stale event, recovery event, post-stream stale event and a PBX stale event during cleanup. The actual indexed recovery alert timestamp is 06:31:16 UTC; the earlier handoff's 06:31:44 records a later observation/readback. Neither the cleanup stale event nor unplanned phone transitions are substituted for a controlled V3 experiment.
