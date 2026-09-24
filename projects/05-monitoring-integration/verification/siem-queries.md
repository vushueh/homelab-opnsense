# Q028 SIEM queries

Use the Wazuh alert index and an absolute interval covering September 24, 2026. Replace source aliases with private values from the management runbook when narrowing a query. Verify field mappings on the actual daily index; older indices can have incompatible mappings.

| View | DQL | Meaning |
|---|---|---|
| Firewall blocks | `rule.id:100081 and data.dstport:39028` | Narrow further to the approved marker sender and test window for loss counts. |
| Lab firewall pass | `rule.id:100285 and data.protocol:tcp` | Correlate the three approved connection tuples. |
| IDS control | `rule.id:100086 and data.alert.signature_id:2062640` | Narrow to the new test flow and its positive/negative windows. |
| SSH authentication | `rule.id:5715` | Add the private OPNsense location; this broad form also contains other senders. |
| Pipeline stale/recovery | `rule.id:(100283 or 100284)` | Correlate with the recorded forwarding interruption. |
| PBX heartbeat/loss/recovery | `rule.id:(100280 or 100281 or 100282)` | Group by `data.q028_run` and `data.q028_seq`. Controlled V3 remains deferred. |

Final API verification passed on 2026-09-24: 923 firewall markers, six pass observations, eight positive/zero negative IDS observations, ten authentication events, four pipeline and 645 PBX-state events. Six [portable saved-query definitions](q028-saved-queries.ndjson) are retained. The browser session expired before saving; GUI import and screenshot are Deferred until authenticated access is available. The accepted marker count is 923 distinct source ports: 60 baseline plus 863 received outside the intentional outage. Count duplicates separately. The 397 packets sent while forwarding was disabled were logged locally and cannot be backfilled by UDP syslog.

DHCP lease display is not DHCP-log evidence. VPN searches are excluded because that project is not built. Flow records are verified directly with the dedicated collector's `nfdump`.
