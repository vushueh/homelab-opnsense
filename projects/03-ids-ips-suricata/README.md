# Project 03 — IDS/IPS with Suricata

## Status

Planned. Recommended next active OPNsense project.

## Purpose

Turn OPNsense into a real IDS/IPS sensor for lab traffic. Start in IDS alert-only mode, generate controlled test traffic from Kali/VLAN 40 toward vulnerable systems on VLAN 250, then evaluate whether IPS blocking mode is safe.

## Skills Practiced

- OPNsense plugin/service management
- Suricata IDS vs IPS behavior
- interface selection and rule scope
- ET Open / abuse.ch rule tuning
- alert triage
- firewall log correlation
- safe blocking-mode rollout
- screenshot and evidence capture

## Cross-Family Links

| Family | Link |
|--------|------|
| Homelab_CCNA | Compare ACL/firewall filtering to real IDS visibility |
| SOC / Blue Team | Feed alerts to Wazuh or Security Onion in Project 05 |
| Kali / Attack Lab | Generate controlled scans and test events |
| Windows Server | Later correlate Windows events with firewall/IDS events |

## Phase Index

| Phase | File | Goal |
|-------|------|------|
| 1 | [Audit and Safety Plan](phases/phase-1-audit-and-safety.md) | Confirm interfaces, backups, rules, and logging baseline |
| 2 | [Enable IDS Mode](phases/phase-2-enable-ids-mode.md) | Enable Suricata in alert-only mode |
| 3 | [Verify Alerts](phases/phase-3-verify-alerts.md) | Generate safe test traffic and confirm alerts |
| 4 | [Tune Rules](phases/phase-4-tune-rules.md) | Reduce noise and document chosen rule sets |
| 5 | [Break/Fix](phases/phase-5-break-fix.md) | Mis-scope an interface or disable a rule, diagnose, restore |
| 6 | [IPS Decision](phases/phase-6-ips-decision.md) | Decide whether blocking mode is appropriate |

## Safety Gates

Stop and ask before:

- enabling IPS blocking mode;
- applying rules to WAN or management without review;
- enabling automatic blocking;
- changing production VLANs;
- exposing logs or config exports with secrets.

## Completion Evidence

- screenshot of Suricata enabled in IDS mode;
- screenshot of selected interfaces and rule sets;
- at least one controlled alert from lab traffic;
- alert interpretation notes;
- rollback note;
- decision on whether IPS blocking mode should be deferred or enabled.