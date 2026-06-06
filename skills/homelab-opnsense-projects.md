---
name: homelab-opnsense-projects
description: >
  OPNsense Firewall Labs — Projects 03 through 09.
  Trigger when Leonel says: "opnsense project 03-09", "Suricata", "IDS", "IPS",
  "WireGuard", "VPN", "syslog SIEM", "NetFlow", "OPNsense DNS", "DHCP static",
  "Unbound DoT", "OPNsense API", "policy automation", "cross-family firewall",
  "RADIUS OPNsense", or any phase number within P03-P09.
  Works alongside homelab-opnsense (P01-02 complete) and windows-server (NPS/RADIUS).
---

# OPNsense Firewall Labs — Projects 03–09

## Environment Reference (all projects)

| Component | Value |
|-----------|-------|
| OPNsense VM | Hyper-V on WIN-PRQD8TJG04M |
| Management IP | 192.168.20.254 |
| WAN interface | hn2 — DHCP from production (192.168.10.x) |
| LAN / VLAN 30 | 192.168.30.1/24 |
| VLAN 40 (Kali) | 192.168.40.1/24 |
| VLAN 50 | 192.168.50.1/24 |
| VLAN 250 (Vulnerable) | 192.168.250.1/24 |
| Route10 (production) | 192.168.70.1 — NEVER modify |

**Lab VLANs:** 30, 40, 50, 250 | **Never touch:** VLAN 10, 20
**SSH to OPNsense:** via Tailscale or management VLAN 20
**GUI:** https://192.168.20.254

---

## Project 03 — IDS/IPS with Suricata

**Requires:** P02 (VLAN segmentation) complete
**Slash command:** `/opnsense-p03`

### Purpose
Deploy Suricata as IDS (alert-only) on lab VLANs. Generate controlled test traffic from Kali. Tune rules. Decide whether IPS blocking mode is safe.

### Phase 1 — Audit and Safety Plan

```
GUI: System → Firmware → Plugins → confirm os-suricata installed
GUI: Interfaces → Assignments → document all interfaces
GUI: Firewall → Rules → capture screenshots of VLAN 30/40/250 rules
GUI: Services → Intrusion Detection → confirm service state
CLI (after approval): configctl interface list | sockstat -4 -l
```
**Save config backup first:** System → Configuration → Backups → Download

### Phase 2 — Enable IDS Mode

```
GUI: Services → Intrusion Detection → Administration
  → Enable IDS: YES
  → IPS/Blocking mode: DISABLED (alert only)
  → Interfaces: select VLAN 40 and VLAN 250 ONLY — NOT WAN, NOT MGMT
GUI: Download tab → enable ET Open → category: test only (first run)
     WHY: enabling all ET Open categories generates hundreds of alerts/minute
Apply → Start service
VERIFY: ps aux | grep suricata → process running
        ls -lh /var/log/suricata/ → eve.json exists
```

### Phase 3 — Verify Alerts

```bash
! From Kali VM (VLAN 40) — safe test traffic
nmap -sV 192.168.250.0/24
curl -A "sqlmap/1.0" http://192.168.250.1
curl "http://192.168.250.1/test.php?id=1+OR+1=1"

! CLI verify (after approval)
grep '"event_type":"alert"' /var/log/suricata/eve.json | tail -5 | \
  python3 -c "import sys,json; [print(json.loads(l).get('src_ip'),'->', json.loads(l).get('dest_ip'), json.loads(l).get('alert',{}).get('signature','')) for l in sys.stdin]"
```

GUI: Services → Intrusion Detection → Alerts — confirm entries with src=VLAN40, dst=VLAN250

### Phase 4 — Tune Rules

```
GUI: Services → Intrusion Detection → Alerts → sort by count → identify noisy SIDs
GUI: Services → Intrusion Detection → Policy → create policy for noisy SID → set action to disable
GUI: Services → Intrusion Detection → Rules → search SID → manual per-SID disable
CLI: grep '"event_type":"alert"' /var/log/suricata/eve.json | \
     python3 -c "import sys,json,collections; c=collections.Counter(); [c.update([json.loads(l).get('alert',{}).get('signature_id','?')]) for l in sys.stdin]; [print(v,k) for k,v in c.most_common(20)]"
```

### Phase 5 — Break/Fix

| Scenario | Break | Symptom | Fix |
|----------|-------|---------|-----|
| Wrong interface | Remove VLAN 40 from IDS scope | Kali scan generates no alerts | Re-add VLAN 40, apply |
| Service stopped | `service suricata stop` | No alerts for any traffic | Restart from GUI |
| All rules disabled | Uncheck all rule sets | IDS running, 0 rules, no alerts | Re-enable ET Open test, sync |

### Phase 6 — IPS Decision

Pre-flight checklist before enabling IPS:
- [ ] Suppression rules tuned (Phase 4 done)
- [ ] Hyper-V console access confirmed
- [ ] Config backup from today available
- [ ] MGMT (VLAN 20) excluded from IPS scope
- [ ] WAN excluded from IPS scope

If all YES → enable IPS, test with Kali nmap, confirm `"action":"blocked"` in eve.json
If any NO → document "Deferred — reason" and close project

---

## Project 04 — VPN Remote Access (WireGuard)

**Requires:** P03 complete (monitoring before opening WAN)
**Slash command:** `/opnsense-p04`

### Purpose
Secure remote access to lab VLANs via WireGuard. Firewall rules enforce least privilege. Private keys never committed.

### Phase 1 — Audit and Design

```
GUI: Interfaces → Assignments → document all interfaces and IPs
GUI: Firewall → Rules → WAN → confirm no existing inbound rules
GUI: VPN → WireGuard → confirm no existing config
Access scope: VLAN 30/40/250 YES | VLAN 10/20 NO | Internet: NO (split tunnel)
```

### Phase 2 — Build WireGuard Server

```
WHY: WireGuard is built into OPNsense 24.x — no plugin install needed
     If VPN → WireGuard menu is missing, record OPNsense version and ask Claude

GUI: VPN → WireGuard → Instances → +
  Name: lab-vpn | Listen Port: 51820
  Tunnel Address: 10.200.200.1/24
  Generate Keys → record Public Key (needed for client config)

GUI: VPN → WireGuard → Peers → +
  Name: leonel-laptop | Public Key: <client-public-key>
  Allowed IPs: 10.200.200.2/32
  Save → edit lab-vpn instance → add leonel-laptop to Peers field

GUI: VPN → WireGuard → General → Enable → Apply
```

```bash
! Client key generation (on laptop — NOT on server)
wg genkey | tee client-private.key | wg pubkey > client-public.key
! NEVER commit client-private.key to git
```

Client config template (sanitized — no real keys):
```ini
[Interface]
PrivateKey = <client-private-key>
Address = 10.200.200.2/32
DNS = 192.168.30.1

[Peer]
PublicKey = <server-public-key>
Endpoint = <OPNsense-WAN-IP>:51820
AllowedIPs = 192.168.30.0/24, 192.168.40.0/24, 192.168.250.0/24, 10.200.200.0/24
PersistentKeepalive = 25
```

### Phase 3 — Firewall Rules

```
GUI: Interfaces → Assignments → find wgX device → assign → name VPN_WG
  Enable | IPv4/IPv6 type: None (tunnel IP stays on WireGuard Instance) | Apply
  Restart WireGuard if interface not active

GUI: Firewall → Rules → VPN_WG → add rules IN ORDER:
  1. BLOCK: src 10.200.200.0/24 → dst 192.168.10.0/24 (VLAN 10 — production)
  2. BLOCK: src 10.200.200.0/24 → dst 192.168.20.0/24 (VLAN 20 — management)
  3. PASS: src 10.200.200.0/24 → dst 192.168.30.0/24 (VLAN 30)
  4. PASS: src 10.200.200.0/24 → dst 192.168.40.0/24 (VLAN 40)
  5. PASS: src 10.200.200.0/24 → dst 192.168.250.0/24 (VLAN 250)
  WHY: OPNsense first-match — BLOCK rules MUST be above PASS rules

GUI: Firewall → Rules → WAN → add: Protocol UDP, Dest WAN address, Port 51820

CLI: sockstat -4 -l | grep 51820 → confirm listening
     ifconfig | grep -A3 '^wg' → confirm wgX interface
```

### Phase 4 — Test and Break/Fix

```bash
! Full remote test — connect from phone hotspot
wg show    ! handshake present? transfer bytes?

! Reachable (should succeed)
ping 192.168.30.1 | ping 192.168.40.1 | ping 192.168.250.1

! Blocked (should fail — timeout)
ping 192.168.10.35 | ping 192.168.20.254
```

| Scenario | Break | Fix |
|----------|-------|-----|
| Wrong endpoint | Change client config Endpoint IP | Correct IP |
| Rule order wrong | Move VLAN 250 block below allow | Move block rule above allow |
| WAN port closed | Remove UDP 51820 WAN rule | Re-add WAN rule |

---

## Project 05 — Monitoring and SIEM Integration

**Requires:** P03 (Suricata alerts) and P04 (VPN) complete
**Slash command:** `/opnsense-p05`

### Purpose
Forward OPNsense telemetry (firewall logs, Suricata alerts, NetFlow) to Wazuh/SIEM. Validate cross-family integration.

### Phase 1 — Audit

```
GUI: System → Settings → Logging → check Remote tab for existing destinations
GUI: Firewall → Log Files → Live View → confirm logs generating
GUI: Services → Intrusion Detection → Alerts → confirm Suricata alerts present from P03
Note OPNsense version: System → Firmware → Status
```

### Phase 2 — Remote Syslog

```
GUI: System → Settings → Logging → Remote → +
  Transport: UDP | Hostname: <SIEM-IP> | Port: 514 (or 1514 for Wazuh)
  Levels: Informational | Facilities: leave empty for first test
  Description: Wazuh-SIEM | Save → Apply

GUI: Firewall → Settings → Advanced → enable Log firewall default rules

CLI (verify): grep -R "<SIEM-IP>" /usr/local/etc/syslog-ng.conf /usr/local/etc/syslog-ng.conf.d/ 2>/dev/null
              tail -n 20 /var/log/filter/latest.log    ! firewall log (NOT clog — modern OPNsense)
              tail -n 20 /var/log/system/latest.log    ! system log
```

### Phase 3 — Suricata Alert Forwarding and NetFlow

```
! Suricata EVE forwarding — OPNsense native option:
GUI: Services → Intrusion Detection → Administration → Logging
  → Enable EVE syslog output if available in your OPNsense version
  → Or use: Services → Intrusion Detection → Log Forwarding (version-dependent)

! NetFlow — built-in (no plugin needed in modern OPNsense)
GUI: Reporting → NetFlow → Settings
  Interface: VLAN 30, 40, 250 | Collector: <SIEM-IP> | Port: 2055 | Version: v9
  Enable → Apply
```

### Phase 4 — Dashboards and Break/Fix

Saved SIEM queries to create:
- Firewall blocks: `filterlog` + action=block
- IDS alerts: `event_type: alert`
- DHCP leases: new device detection
- VPN connections: WireGuard handshake events

Break/Fix — logs stop arriving:
```
! Break: change syslog target port to wrong value
GUI: System → Settings → Logging → Remote → edit → wrong port
! Symptom: SIEM shows gap in OPNsense events
! Diagnose: grep wrong port in syslog config; test with nc -u <SIEM-IP> 514
! Fix: correct port, apply
```

---

## Project 07 — DNS and DHCP Services

**Requires:** P02 complete
**Slash command:** `/opnsense-p07`

### Purpose
Harden Unbound DNS (DoT, DNSSEC, rebind protection). Create DHCP static mappings. Verify per-VLAN options.

### Phase 1 — Audit

```
GUI: Services → Unbound DNS → General → document current settings
GUI: Services → Unbound DNS → DNS over TLS → document upstreams
GUI: Services → DHCPv4 → [each VLAN] → document range, DNS, gateway, domain
CLI: cat /var/unbound/unbound.conf | grep forward-addr
     cat /var/dhcpd/var/db/dhcpd.leases | head -60
```

### Phase 2 — Harden DNS Resolver

```
GUI: Services → Unbound DNS → DNS over TLS → add:
  1.1.1.1 | port 853 | verify cloudflare-dns.com
  8.8.8.8 | port 853 | verify dns.google

GUI: Services → Unbound DNS → Advanced → enable DNSSEC
GUI: Services → Unbound DNS → General → enable Rebind detection → Apply

VERIFY: dig @127.0.0.1 sigfail.verteiltesysteme.net → SERVFAIL (DNSSEC working)
        dig @127.0.0.1 google.com → NOERROR
        ss -tnp | grep :853 → DoT connections active
```

### Phase 3 — DHCP Static Mappings

```
GUI: Services → DHCPv4 → [VLAN interface] → Static Mappings → +
  MAC: from Leases table | IP: chosen static (outside dynamic range)
  Hostname: kali-vm / vuln-lab | Description: lab purpose

VERIFY: dhclient -r && dhclient eth0 on each VM → confirm correct static IP
        show ip dhcp snooping binding (on switch) → static reflected
```

### Phase 4 — Per-VLAN Options

| VLAN | Gateway | DNS | Domain |
|------|---------|-----|--------|
| 30 | 192.168.30.1 | 192.168.30.1 | lab.local |
| 40 | 192.168.40.1 | 192.168.40.1 | lab.local |
| 250 | 192.168.250.1 | 192.168.250.1 | lab.local |

Break/Fix: set wrong gateway → client gets DHCP but routing fails → `ip route show` shows bad default

---

## Project 08 — Policy Automation (OPNsense API)

**Requires:** P03-P05 complete
**Slash command:** `/opnsense-p08`

### Purpose
OPNsense REST API for config backups, rule exports, and scripted changes. Git-backed change control.

### Phase 1 — API Setup

```
GUI: System → Access → Users → admin → API keys → + → download key/secret
NEVER store key/secret in git

! Test:
export OPN_KEY="<key>" OPN_SECRET="<secret>" OPN_HOST="192.168.20.254"
curl -k -u "$OPN_KEY:$OPN_SECRET" "https://$OPN_HOST/api/core/firmware/info"
! Expected: JSON with OPNsense version
```

### Phase 2 — Config Backup Script

```python
import os, requests
from datetime import datetime
from pathlib import Path

OPN_HOST = os.environ["OPN_HOST"]
OPN_KEY  = os.environ["OPN_KEY"]
OPN_SECRET = os.environ["OPN_SECRET"]

resp = requests.post(f"https://{OPN_HOST}/api/core/backup/download",
    auth=(OPN_KEY, OPN_SECRET), verify=False, timeout=30)
Path(f"backups/opnsense-{datetime.now().strftime('%Y%m%d-%H%M')}.xml").write_bytes(resp.content)
! WARNING: XML contains firewall rules and VPN keys — PRIVATE repo only
```

### Phase 3 — Rule Export and Diff

```python
resp = requests.get(f"https://{OPN_HOST}/api/firewall/filter/searchRule",
    auth=(OPN_KEY, OPN_SECRET), verify=False)
! Save as JSON → git diff before/after GUI change → confirms only intended change
```

### Phase 4 — Scripted Alias Push

```python
alias_data = {"alias": {"enabled":"1","name":"Lab-Kali-Hosts",
    "type":"host","content":"192.168.40.10\n192.168.40.11"}}
requests.post(f"https://{OPN_HOST}/api/firewall/alias/addItem",
    auth=(OPN_KEY, OPN_SECRET), json=alias_data, verify=False)
requests.post(f"https://{OPN_HOST}/api/firewall/alias/reconfigure",
    auth=(OPN_KEY, OPN_SECRET), verify=False)
! OPNsense returns HTTP 200 even for validation failures — always check JSON body
```

### Break/Fix

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| HTTP 403 | Wrong API key | Verify env var value |
| HTTP 403 after working | Key revoked in GUI | Generate new key pair |
| HTTP 200 + failed in JSON | Invalid field value | Check API docs for valid values |

---

## Project 09 — Cross-Family Firewall Capstone

**Requires:** P01-P08 complete + Windows Server P13 (NPS/RADIUS) + Wazuh active
**Slash command:** `/opnsense-p09`

### Purpose
OPNsense as unified security enforcement for the entire homelab. AD auth, cross-family policy, capstone test.

### Phase 1 — Readiness Audit

```bash
ping 192.168.20.11    ! Windows NPS reachable
ping <wazuh-ip>       ! Wazuh reachable
ps aux | grep suricata ! IDS running
grep -R "Wazuh" /usr/local/etc/syslog-ng.conf.d/ ! syslog active
```

### Phase 2 — AD RADIUS Auth for OPNsense Admin

```
Windows NPS (WIN-PRQD8TJG04M):
  Add RADIUS Client: OPNsense-Firewall | IP: 192.168.20.254 | secret: <unique>
  Network Policy: condition Windows Group = CHONGONG\GG-NetAdmins | Grant access

OPNsense GUI: System → Access → Servers → +
  Type: RADIUS | Hostname: 192.168.20.11 | Secret: <same as NPS>
  Port: 1812 | Save

System → Access → Users → Settings → Auth Server: select RADIUS
  Enable Fallback to local: YES (critical — never disable local until RADIUS proven)

TEST: login with AD credential → success
      login with non-GG-NetAdmins user → fail
      disable RADIUS server → local admin fallback works
```

### Phase 3 — Cross-Family Policy Matrix

| Source | Destination | Rule |
|--------|-------------|------|
| VLAN 40 (Kali) | VLAN 250 | PASS |
| VLAN 40 | VLAN 10/20 | BLOCK |
| VPN 10.200.200.0/24 | VLAN 30/40/250 | PASS |
| VPN | VLAN 10/20 | BLOCK |
| VLAN 150 (CML bridge) | VLAN 30 | PASS |
| VLAN 150 | VLAN 10/20 | BLOCK |

### Phase 4 — Capstone Test

```
1. BEFORE: Kali ping 192.168.250.10 → success (screenshot)
2. Apply block rule: Firewall → Rules → OPT-40 → BLOCK src VLAN40 dst VLAN250
3. Kali ping 192.168.250.10 → fails
4. Observe simultaneously:
   OPNsense: Firewall → Log Files → Live View → block event
   Suricata: Services → IDS → Alerts → alert appears
   Wazuh: dashboard → OPNsense firewall event indexed
5. Remove block rule → ping succeeds again
6. Record timestamps for each observation
```

### Phase 5 — Break/Fix

| Scenario | Break | Fix |
|----------|-------|-----|
| RADIUS unreachable | Wrong NPS IP | Restore correct IP, test AD login |
| Wazuh log gap | Wrong syslog port | Correct port, verify events resume |
| VPN bypass | Remove VPN_WG VLAN10 block | Re-add block, test from VPN client |

---

## Skills Saved Locations

| Location | Path |
|----------|------|
| Repo | `skills/homelab-opnsense-projects.md` |
| Agents | `C:\Users\CHONGONG\.agents\skills\homelab-opnsense-projects\SKILL.md` |
| Codex | `C:\Users\CHONGONG\.codex\skills\homelab-opnsense-projects\SKILL.md` |
| Slash command | `C:\Users\CHONGONG\.claude\commands\homelab-opnsense-projects.md` |

**Trigger phrases:** "opnsense project 03-09", "Suricata", "WireGuard VPN", "OPNsense syslog",
"Unbound DoT", "DHCP static mappings", "OPNsense API", "cross-family firewall", "RADIUS OPNsense"
