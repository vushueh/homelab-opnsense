# Phase 2 — Enable Suricata in IDS Mode

## Goal

Enable Suricata alert-only IDS mode on lab-facing interfaces. Do not enable blocking mode yet.

## Track A — GUI Steps

[LEONEL]

1. Open OPNsense web UI.
2. Go to **Services → Intrusion Detection → Administration**.
3. Enable IDS.
4. Leave IPS/blocking mode disabled.
5. Select only lab interfaces for the first test, such as VLAN 30, VLAN 40, or VLAN 250.
6. Go to **Download** or rule set section and enable a small starter set, such as ET Open informational/test rules.
7. Apply changes.
8. Start or restart the IDS service.

## Track B — Verification

[CODEX/CLAUDE]

Look for service state and alerts. Exact commands may vary by OPNsense version, so prefer GUI verification first.

Possible checks after approval:

```sh
ps aux | grep -i suricata
ls -lah /var/log/suricata/
tail -n 50 /var/log/suricata/suricata.log
```

## What Good Looks Like

- IDS enabled;
- IPS/blocking disabled;
- Suricata service running;
- selected lab interfaces only;
- no management lockout;
- no unexpected production impact.

## Screenshots to Capture

- IDS enabled with blocking disabled;
- selected interfaces;
- rule sources enabled;
- service running state.

## Rollback

Disable IDS from the same page and apply changes. If management access becomes unstable, use Hyper-V console to disable IDS from the local console or restore the pre-phase config backup.

## Documentation Checklist

- [ ] IDS enabled
- [ ] IPS/blocking disabled
- [ ] Interface scope documented
- [ ] Rule set documented
- [ ] Service state verified
- [ ] Rollback path documented
