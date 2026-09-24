# Q028 monitoring baseline — 2026-09-24

The September 23 audit found an existing UDP 514 Wazuh syslog target and the Q027 Suricata forwarding/decoder path. Two Security Onion destinations were retained. Wazuh alert JSON was enabled and full archive logging disabled, so ordinary non-alerting events could not support loss accounting.

SSH authentication used the auth/authpriv facilities. Application, level and facility filters are ANDed, so a separate auth-only destination was added instead of narrowing the existing target. NetFlow v9 already fed a dedicated collector; only the two lab capture interfaces were added.

Readback on September 24 matched the saved post-change generated syslog SHA-256 `55737f06e1bd5e8f7492bf99382592fbd894f39d02450ab6aa00ef34be294c7c`. The original LAN and both added lab NetFlow nodes remained active. No raw firewall configuration was read during the resume.

Private addressing, protected backup locations and full checksums are in the owning management project's handoff and resume record. Backups exist; a restore was not performed.
