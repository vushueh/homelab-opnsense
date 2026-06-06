# Phase 3 — Verify Alerts with Controlled Test Traffic

## Goal

Generate safe, controlled traffic from Kali (VLAN 40) toward VLAN 250 targets and confirm Suricata produces alerts. Do not touch production VLANs.

---

## Track A — GUI Steps

[LEONEL]

1. Log into OPNsense → **Services → Intrusion Detection → Alerts**
2. Confirm the alerts tab is visible and timestamps are current
3. Open a second tab: **Services → Intrusion Detection → Administration** — confirm IDS still running, IPS disabled
4. From a Kali VM on VLAN 40, run the following safe test commands:

```bash
# Simple port scan — should trigger scan-related ET rules
nmap -sV 192.168.250.0/24

# HTTP request with suspicious User-Agent
curl -A "sqlmap/1.0" http://192.168.250.1

# Test known signature string (EICAR-like, network variant)
curl http://192.168.250.1/test.php?id=1+OR+1=1
```

5. Return to OPNsense → **Services → Intrusion Detection → Alerts**
6. Wait up to 60 seconds — refresh the page
7. Confirm at least one alert entry appears with:
   - Source IP from VLAN 40 range (192.168.40.x)
   - Destination IP from VLAN 250 range (192.168.250.x)
   - Rule name or SID visible

---

## Track B — CLI Verification

[CODEX/CLAUDE — after approval]

```sh
# Confirm Suricata log has entries
grep '"event_type":"alert"' /var/log/suricata/eve.json | tail -20

# Check alert count
grep -c '"event_type":"alert"' /var/log/suricata/eve.json

# View most recent alert source/dest
grep '"event_type":"alert"' /var/log/suricata/eve.json | tail -5 | \
  python3 -c "import sys,json; [print(json.loads(l).get('src_ip'), '->', json.loads(l).get('dest_ip'), json.loads(l).get('alert',{}).get('signature','')) for l in sys.stdin]"
```

---

## What Good Looks Like

- At least one Suricata alert triggered by Kali traffic
- Alert source = VLAN 40 IP, destination = VLAN 250 IP
- Alert visible in both GUI and eve.json log
- No unexpected alerts from production VLANs (10, 20)
- IPS blocking mode still disabled

---

## Troubleshooting

| Symptom | Check |
|---------|-------|
| No alerts appear | Confirm VLAN 40/250 interfaces selected in IDS config |
| Alert source wrong VLAN | Confirm Kali VM is actually on VLAN 40 interface |
| Suricata stopped | `ps aux \| grep suricata` — restart from GUI if stopped |
| eve.json empty | Confirm Suricata logging is enabled and check `/var/log/suricata/` disk space |

---

## Screenshots to Capture

- Alerts tab with at least one entry from VLAN 40 → VLAN 250
- Alert detail showing rule name / SID and timestamp
- Kali terminal showing nmap or curl command that triggered the alert

## Documentation Checklist

- [ ] Controlled test traffic generated from Kali VLAN 40
- [ ] At least one alert confirmed in GUI and eve.json
- [ ] Alert source/dest IPs verified correct
- [ ] No production VLAN impact observed
- [ ] Screenshots captured
