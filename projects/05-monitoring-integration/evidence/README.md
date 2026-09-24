# Q028 evidence boundary

The [acceptance checklist](../q028-acceptance-checklist.md) owns measured results. Detailed internal tuples, JSON identities, manifests and backups are retained in the private management project.

| Phase | Screenshot or exception |
|---|---|
| 1 | DOC-only baseline evidence: private configurations and source logs are represented by sanitized readbacks; raw configuration contains secrets. |
| 2 | [Enabled authentication target](screenshots/p05-phase2-auth-forwarding.png), captured 2026-09-24. Hostname column hidden using the actual UI; cropped before saving; no image content synthesized. |
| 3 | Screenshot-unsafe raw collector/IDS output contains internal addressing; sanitized tuple counts and hashes are retained as text. |
| 4 | Index API evidence is retained. The Wazuh browser login expired before saved-query creation, so no successful dashboard screenshot or UI import is claimed. |
| 5 | DOC-only cleanup hashes, service readbacks and exact file inventories. Protected rollback backups are not copied into public evidence. |

The phase 2 screenshot proves the enabled target, not end-to-end delivery. The portable query file is prepared for import; it is not evidence that queries were saved in the expired dashboard session.
