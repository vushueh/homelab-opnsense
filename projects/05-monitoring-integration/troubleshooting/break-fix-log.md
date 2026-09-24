# Q028 break/fix record — September 23–24, 2026

## Forwarding interruption

Leonel disabled only the existing Wazuh syslog target while the marker streams continued. Local logging retained all 1,260 packets. Wazuh received 863 outside the outage and the stale detector alerted after 67 seconds. The missing 397 source ports form the recorded interruption interval. PBX receipt stayed healthy as a control. Re-enabling the target restored the exact generated configuration hash, and receipt resumed. The indexed stage is tracked separately in the acceptance checklist.

## Empty dashboard searches

The manager had received the events, but the indexer had made indices read-only because root storage reached 99%. Workstation firewall-audit events 5152/5157 dominated alert growth. The approved recovery preserved two deleted-but-open JSON files before any restart, backed up configuration and the Filebeat registry, and removed only those two event IDs from that workstation's existing Security query. Readback confirmed one subscription and preserved unrelated logon events.

Inactive daily files are being copied to a separate disk, checksum-verified, compressed and gzip-tested before unlinking their original single-link names. Crossing below the high watermark automatically released index blocks; disk protections remained enabled. A historical backlog still delays current indexed proof. No disk expansion was needed so far.

## Test execution without an unattended Kali login

The bounded alternative used a disposable source on the same lab VLAN. Generic Claude review identified bridge MAC/MTU and global DHCP-default risks; the script addressed them before execution. A DHCP timeout produced no persistent changes. The later out-of-pool source passed duplicate-address detection and gateway verification. Three connections and the positive/negative IDS echoes succeeded, followed by exact host-state cleanup checks.

## Retained limitations

Controlled PBX registration loss/recovery and the emitter-down test remain Deferred until a dedicated test softphone stays registered throughout the sequence. A manager-stage unplanned loss alert is not a substitute for that test. Sensor time skew, UDP outage loss, historical compression failures and delayed indexing remain explicit rather than being called successful recovery.


## Final recovery — 2026-09-24 08:30–08:33 UTC

Root is 67% used with 57 GB free; the archive disk has 67 GB free. All index blocks released automatically, all four central services are active and Filebeat TLS output passed. The current index contains the exact 923 marker identities and all three recovery canaries. All historical replay inputs were fully acknowledged and removed from the live configuration. The specific workstation audit filter remains; no registry reset or disk expansion occurred.

Twenty malformed historical fragments and partial daily coverage remain explicit evidence limits. The 90-day index policy is active but raw-log retention/compression and capacity alerting require follow-up. The PBX cleanup wrapper's prefix check was corrected after independently verifying successful deletion and unchanged family configuration; no deletion was repeated. Temporary sudo access was removed last and passwordless escalation was independently denied.
