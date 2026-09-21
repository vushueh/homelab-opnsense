# Q027 — Detect-only Suricata closeout

Completed2026-09-20 (final checks September21UTC). Owner: homelab-opnsense. Status: Complete. GitHub PR records own publication status. Q028 remains planned and unstarted.

## Outcome and acceptance

The existing PCAP sensor detected a bounded benign lab signature, forwarded EVE to Wazuh, and yielded a decoded/indexed event correlated to local evidence. Separate local tuning and fault/restoration tests passed with an independent control. Cleanup removed the disposable rule and final controls passed both locally and in the index. IPS remains deferred.

[Acceptance](q027-acceptance-checklist.md) maps every live requirement. [Final indexed evidence](evidence/q027-final-indexed-validation.json) records16 surrounding positives,0 quiet-window events,8 later positives and8 final cleanup controls. [Cleanup](evidence/q027-final-cleanup.json) confirms no user rule/rendered test rule remains, unchanged PID/interface scope, successful traffic and local control8. Seven screenshots are retained and reviewed.

## Scope and review disposition

Leonel approved source sharing with Claude, the scoped Wazuh repair, bounded traffic, and a temporary Alert-only lab-pair rule. He replaced broad service/interface failure exercises with a disposable-rule fault test and later authorized Codex to finish GUI work. Claude supplied four planning documents; Codex independently reviewed them, corrected procedures and implemented verification. Invalid reload-overlap and reused-flow attempts remain in the evidence rather than being counted as passes.

The temporary rule generated an IP-only signature. Independent break/fix used fresh TCP handshakes to the owned target's known port22 without authentication and a separate inert ICMP control. Baseline/disabled/restored temporary counts1/0/1; control8 in each state. No IPS, routing, interface, service restart, restore or clock changes were performed during these rule tests.

## Operational limitations and follow-up

- OPNsense clock is roughly127seconds ahead and NTP reports alarm/spike. Owner: Leonel/OPNsense operations; diagnose before time-sensitive correlation or any IPS proposal. No repair is claimed.
- Final controls prove ingestion for those events, not zero transport loss or global backlog clearance. Owner: Wazuh operations; monitor current Filebeat/index errors separately from historical faults.
- Backup file/checksum and encrypted wrapper verified; restore not executed. Private XML and credentials remain outside Git.
- Synthetic noise tuning does not establish production threat-rule false-positive tolerance. Existing WAN monitoring was preserved, which is another reason IPS is deferred.

## Durable sources and publication

Wazuh decoder and rule100086 source live in homelab-management/wazuh/local-decoders/q027-suricata-eve.xml and local-rules/q027-suricata-rules.xml with reproduction/rollback notes. Source hashes match the tested installer payload. The owning README, acceptance, evidence and family trackers record this closeout. Publication is authorized; GitHub PR records own final merge outcomes. Unrelated dirty files remain untouched.

Default branch freshness was checked read-only: family tracker main0b67773c08330c3838efbe4eb75ddfbcbc9e0066; owner maineaf7317be1ab9c9e4e6fd851320bf0c567ce0560, each matching live origin/main at closeout. Q028 — OPNsense Monitoring and SIEM Integration is the next queue item but is not started by this closeout.
