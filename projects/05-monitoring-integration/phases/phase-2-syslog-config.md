# Q028 — Fill logging gaps

Work date: 2026-09-24. Phase status is owned by the [project phase table](../README.md#phase-status).

I retained the existing UDP 514 target and added a separate auth/authpriv destination. The original application selections and other SOC destinations were preserved. The generated post-change syslog configuration matched its retained SHA-256 on the resume. Permanent Q028 rules were validated; the temporary watcher was later removed. Receipt and indexing are separate acceptance stages.

## Evidence And Re-Verification

See the [dated supporting record](../verification/log-format-notes.md) and [current runbook](../q028-execution-runbook.md). Historical planned procedures were superseded by the approved Q028 gates and actual readbacks.

Screenshots are used only where a credential-free final state can be captured safely. Internal configuration, raw logs and protected backups stay private; sanitized text carries the detailed operational evidence.
