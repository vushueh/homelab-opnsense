# Q028 — Test visibility and failure

Work date: 2026-09-24. Phase status is owned by the [project phase table](../README.md#phase-status).

I tested the approved forwarding target outage and then diagnosed a real indexing failure. The controlled test produced 1,260 local markers, 863 manager receipts outside the outage and 397 deliberately unforwarded events. The stale alert fired in 67 seconds. Disk recovery preserved evidence before removing closed originals and left watermark protection enabled. Controlled phone loss/recovery and emitter-down testing remain deferred until a stable disposable softphone is available.

## Evidence And Re-Verification

See the [dated supporting record](../troubleshooting/break-fix-log.md) and [current runbook](../q028-execution-runbook.md). Historical planned procedures were superseded by the approved Q028 gates and actual readbacks.

Screenshots are used only where a credential-free final state can be captured safely. Internal configuration, raw logs and protected backups stay private; sanitized text carries the detailed operational evidence.
