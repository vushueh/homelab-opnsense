# Q028 — Correlate IDS and flows

Work date: 2026-09-24. Phase status is owned by the [project phase table](../README.md#phase-status).

I kept the Q027 detect-only forwarding path and verified a harmless positive/negative marker pair. The two extra lab interfaces feed the existing exporter. Three allowed TCP tuples appeared in both manager alerts and a pinned collector file: ten exported records, 24 packets and 1,192 bytes. Multiple capture points prevent treating that sum as unique wire traffic. The temporary client was removed with host routes, interface properties and VLAN memberships unchanged.

## Evidence And Re-Verification

See the [dated supporting record](../verification/suricata-forwarding-method.md) and [current runbook](../q028-execution-runbook.md). Historical planned procedures were superseded by the approved Q028 gates and actual readbacks.

Screenshots are used only where a credential-free final state can be captured safely. Internal configuration, raw logs and protected backups stay private; sanitized text carries the detailed operational evidence.
