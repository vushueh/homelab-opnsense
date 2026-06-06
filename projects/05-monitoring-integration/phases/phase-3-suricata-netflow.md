# Phase 3 — Suricata Alert Forwarding and NetFlow

## Goal

Forward Suricata IDS alerts from Project 03 to the SIEM and enable NetFlow/flow telemetry
for traffic visibility without full packet capture.

---

## Track A — Suricata Alert Forwarding

[LEONEL]

### 3.1 Confirm Suricata EVE JSON Output

1. OPNsense → **Services → Intrusion Detection → Administration**
2. Confirm EVE JSON logging is enabled (default in modern OPNsense + Suricata)
3. Note log path: `/var/log/suricata/eve.json`

### 3.2 Forward Suricata Alerts via Syslog

OPNsense can forward Suricata alerts through syslog. Prefer the built-in EVE syslog output before adding agents or custom tail jobs.

**Option A — Built-in EVE syslog output (recommended first):**

1. OPNsense → **Services → Intrusion Detection → Administration**
2. Enable **Enable eve syslog output**
3. OPNsense → **System → Settings → Logging → Remote**
4. Add or edit the SIEM destination
5. Set **Applications** to include `suricata` if narrowing the target; otherwise leave empty during first test
6. Set **Levels** to include `info`
7. Save, apply, and trigger the Phase 3 Kali scan again

**Option B — Wazuh agent on OPNsense (only if required and supported):**
Install `os-wazuh-agent` only if it is available in the OPNsense plugin list and Claude approves the package choice. Configure it to read `/var/log/suricata/eve.json`.

**Option C — Custom syslog-ng/tail forwarder (last resort):**
```sh
# Avoid editing generated syslog-ng files directly.
# Use a reviewed template or syshook-backed method if the built-in EVE syslog output is not enough.
```

Avoid long-running background tail jobs as a permanent solution:
```sh
# Temporary diagnostic only — do not leave running after the test
tail -F /var/log/suricata/eve.json | logger -n <SIEM-IP> -P 514 -t suricata &
```

Document which option was used in `verification/suricata-forwarding-method.md`.

---

## Track B — NetFlow Configuration

[LEONEL]

NetFlow provides per-flow traffic summaries (source, dest, bytes, protocol) without full packet capture.

### 3.3 Configure Built-In NetFlow Exporter

OPNsense includes a NetFlow exporter and Insight analyzer. Do not install `os-softflowd`; it is not the current OPNsense path for this lab.

1. OPNsense → **Reporting → NetFlow**
2. Configure:
   - **Interfaces:** VLAN 30, VLAN 40, VLAN 250 (lab interfaces)
   - **Egress only:** add lab interfaces here if you want to avoid double-counting traffic to/from the firewall itself
   - **Capture local:** enable if Leonel also wants local Insight analysis
   - **Version:** NetFlow v9
   - **Destinations:** `<SIEM-IP or ntopng IP>:2055`
3. Enable and apply

---

## Verification

### Suricata in SIEM:
- Trigger Kali scan (from P03): `nmap -sV 192.168.250.0/24`
- Check SIEM for Suricata alert event with `event_type: alert`
- Confirm SID and signature name visible in SIEM

### NetFlow:
```sh
# On SIEM or NetFlow collector — confirm flows arriving
# Exact command depends on collector (ntopng, nfdump, etc.)
nfdump -R /var/cache/nfdump/ -s record/bytes | head -20
# OR if using ntopng: check flow dashboard in browser
```

---

## Screenshots to Capture

- Suricata alert visible in SIEM with source VLAN 40, dest VLAN 250
- NetFlow configuration page in OPNsense under Reporting
- NetFlow flows visible in SIEM/collector (at least one flow from lab traffic)

## Documentation Checklist

- [ ] Suricata forwarding method chosen and documented
- [ ] Suricata alerts visible in SIEM after Kali test traffic
- [ ] Built-in NetFlow exporter configured
- [ ] NetFlow flows arriving at collector
- [ ] suricata-forwarding-method.md saved to verification/
