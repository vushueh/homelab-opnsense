---
name: opnsense-evidence-documentation
description: >
  OPNsense Firewall evidence and portfolio documentation workflow.
  Trigger when Leonel says: "document phase", "save screenshots", "phase complete",
  "project complete", "update README", "log the break fix", "save the evidence",
  or after completing any OPNsense project (P01-P09).
  This skill handles PROVING the work was done — safely, with no secrets committed.
  The technical skills (homelab-opnsense-projects, opnsense-p03-suricata, etc.)
  handle HOW to do the work.
---

# OPNsense Firewall — Evidence and Portfolio Documentation

## Purpose

This skill guides you through capturing and organizing proof that each OPNsense
phase was completed — with a strict no-secrets policy throughout.

The outcome is a GitHub portfolio showing:
- Real firewall rule screenshots
- Real Suricata alert evidence
- Real VPN and monitoring verification
- No private keys, no raw config XML, no WAN IPs, no password hashes

---

## Folder Structure Per Project

```
projects/<number>-<project-name>/
├── README.md
├── phases/
├── configs/
│   ├── sanitized-rule-summary-before-pNN.md    ← rules summarized, no secrets
│   └── sanitized-rule-summary-after-pNN.md
├── verification/
│   ├── screenshots/             ← all GUI screenshots
│   └── cli-outputs/             ← CLI command outputs (sanitized)
└── troubleshooting/
    └── break-fix-log.md
```

---

## CRITICAL — What NEVER Goes in the Repo

```
❌ Full OPNsense XML config backup   ← contains ALL secrets
❌ WireGuard private keys            ← compromises VPN security
❌ WireGuard pre-shared keys
❌ RADIUS/NPS shared secrets
❌ WAN IP address                    ← exposes attack surface
❌ API key or API secret             ← full OPNsense access
❌ Password hashes or credentials
❌ Suricata eve.json raw dumps       ← may contain internal IP data
```

**Safe to commit:**
```
✅ Firewall rule screenshots (blur WAN IP if visible)
✅ Suricata alert list (blur source IPs from real attack traffic)
✅ Sanitized firewall rule summary or reviewed API export
✅ CLI outputs from read-only commands
✅ WireGuard client config TEMPLATE (placeholder values only)
✅ API script without credentials (env vars only)
```

---

## Screenshot Naming Convention

```
<project>-<phase>-<what-it-shows>.png

Examples:
  p03-ph2-suricata-ids-enabled-no-ips.png      ← IDS on, blocking off
  p03-ph3-suricata-alert-kali-to-vlan250.png   ← alert from VLAN 40 to 250
  p03-ph4-suricata-policy-sid-disabled.png     ← tuned rule in policy
  p04-ph2-wireguard-instance-created.png       ← WG instance (blur keys)
  p04-ph3-vpn-wg-firewall-rules.png            ← VPN_WG rules in order
  p05-ph2-syslog-remote-target-configured.png  ← syslog target in GUI
  p05-ph5-wazuh-opnsense-events.png            ← SIEM showing OPN events
```

**Always blur or crop:**
- WAN IP address in any screenshot
- API keys if visible in any screen
- Real client IPs from outside the lab

---

## CLI Output Naming Convention

```
<project>-<phase>-<command>.txt

Examples:
  p03-ph2-ps-aux-suricata-running.txt
  p03-ph3-eve-json-alert-count.txt
  p04-ph3-sockstat-udp-51820.txt
  p05-ph2-syslog-conf-remote-entry.txt
```

---

## Sanitized Firewall Rule Evidence

Use the P08-approved rule export method only after Project 08 creates and reviews it.
This repo does not assume `scripts/export-rules.py` already exists.

If no reviewed export script exists yet, create a manual summary instead:

Rename for the project:
```
configs/sanitized-rule-summary-before-p04.md
configs/sanitized-rule-summary-after-p04.md
```

Recommended fields:
```markdown
| Interface | Action | Source | Destination | Port | Description |
|-----------|--------|--------|-------------|------|-------------|
| VPN_WG | Pass | VPN subnet | VLAN 30 | Any | VPN to lab services |
```

Before committing any API export, review it for WAN IPs, aliases containing real public values,
VPN objects, credentials, descriptions with private notes, and any identifiers outside lab scope.

---

## Per-Phase Documentation Loop

### Step 1 — Before State

```sh
# Take a screenshot of the relevant GUI section before changes
# Example: Services → Intrusion Detection → Administration (before enabling)

# Save firewall rule before-state as a sanitized summary or reviewed P08 export
# Move output to configs/sanitized-rule-summary-before-pNN.md
```

### Step 2 — Do the Work

Follow the phase file Track A (GUI):
- Screenshot each significant configuration screen
- Name following the convention above

### Step 3 — After State

```sh
# Screenshot after changes applied
# Re-save rules for after-state comparison
# Move to configs/sanitized-rule-summary-after-pNN.md
```

### Step 4 — CLI Verification (approved commands only)

```sh
# Save outputs to verification/cli-outputs/

# P03 Suricata:
ps aux | grep suricata                          → confirm running
grep -c '"event_type":"alert"' /var/log/suricata/eve.json  → alert count

# P04 WireGuard:
sockstat -4 -l | grep 51820                     → confirm listening
ifconfig | grep -A2 '^wg'                       → confirm wgX interface (no keys)
# wg show → shows handshake info but NO private keys — safe to screenshot

# P05 Syslog:
grep -R "SIEM-IP-PLACEHOLDER" /usr/local/etc/syslog-ng.conf /usr/local/etc/syslog-ng.conf.d /var/etc 2>/dev/null   → confirm configured
tail -n 5 /var/log/system/latest.log            → confirm log generation

# P07 DNS:
dig @127.0.0.1 google.com | grep -E "status|ANSWER"    → resolver working
sockstat -4 -6 -c | grep ':853'                         → DoT connections
```

### Step 5 — Break/Fix Log

Add to `troubleshooting/break-fix-log.md`.

### Step 6 — Phase Completion Note

```markdown
### ✅ Phase N Complete — YYYY-MM-DD
- What I configured: [1-3 bullets]
- Evidence: [screenshot filename]
- Verification: [CLI command + result in one line]
- Problem/fix: [if any]
```

---

## Break/Fix Log Template

```markdown
## Break/Fix — [Short Title] — YYYY-MM-DD

**Phase:** P0X Phase Y
**What I did:** [Action that caused the issue]

**Symptom:**
[What I observed]

**Screenshot:** [filename — if captured]

**Diagnosis:**
```
[CLI commands run to investigate]
[Relevant output — sanitize IPs if needed]
```

**Root cause:**
[One sentence]

**Fix:**
[GUI steps or CLI commands]

**Verification:**
[How I confirmed it was fixed]

**Security lesson:**
[One sentence — firewall-specific lesson]
```

---

## Completed Project README Template

```markdown
---

## ✅ Project Complete — YYYY-MM-DD

### What I Built
- [e.g. Suricata IDS running in alert-only mode on VLAN 40 and 250]
- [e.g. ET Open rules tuned — 3 noisy SIDs suppressed via Policy]
- [e.g. IPS blocking mode deferred — safety gate failed (no console access)]

### Key Evidence

| IDS Enabled | First Alert | Rule Tuning |
|---|---|---|
| ![IDS On](verification/screenshots/p03-ph2-suricata-ids-enabled-no-ips.png) | ![Alert](verification/screenshots/p03-ph3-suricata-alert-kali-to-vlan250.png) | ![Policy](verification/screenshots/p03-ph4-suricata-policy-sid-disabled.png) |

### Verification Summary
```
# Suricata running
ps aux | grep suricata → /usr/local/bin/suricata -c /usr/local/etc/suricata/suricata.yaml

# Alerts generated from Kali test traffic
grep -c '"event_type":"alert"' /var/log/suricata/eve.json → 47
```

### Problems Encountered and Fixed
| Problem | Root Cause | Fix |
|---------|-----------|-----|
| [e.g. No alerts from Kali scan] | VLAN 40 not in IDS interface scope | Re-added VLAN 40, applied |

### Result
[2-3 sentences — what the firewall now detects/blocks that it didn't before]

### Links
[Phases](phases/) | [Screenshots](verification/screenshots/) | [CLI outputs](verification/cli-outputs/) | [Break/Fix](troubleshooting/break-fix-log.md)
```

---

## Main Family README Update

```markdown
| [03](projects/03-ids-ips-suricata/) | IDS/IPS with Suricata | ✅ Complete — 2026-07-20 |

## ✅ Project 03 — IDS/IPS with Suricata

**Built:** 2026-07-20 | [Full detail →](projects/03-ids-ips-suricata/)

Deployed Suricata in IDS alert-only mode on lab VLANs 40 and 250. Generated
controlled Kali attack traffic and confirmed alert pipeline. Tuned ET Open rules
to reduce noise. IPS blocking deferred pending Hyper-V console access confirmation.

[→ Screenshots](projects/03-ids-ips-suricata/verification/screenshots/) |
[→ CLI outputs](projects/03-ids-ips-suricata/verification/cli-outputs/) |
[→ Break/Fix log](projects/03-ids-ips-suricata/troubleshooting/break-fix-log.md)
```

---

## Trigger Phrases

Load this skill when Leonel says:
- "document this phase" / "phase complete" (for any OPNsense project)
- "save the screenshots" / "what do I save"
- "project complete" / "update README"
- "log the break fix"
- "is it safe to commit this" (secrets check)
- After completing any OPNsense project phase
