# P01 — OPNsense Baseline Router Deployment

- **Status:** ✅ Complete — 2025-10-24
- **Project ID:** `OPNsense-P01`
- **Platform:** OPNsense on Microsoft Hyper-V
- **Scope:** Router-mode baseline with separate WAN, lab, and management paths
- **Repository:** [OPNsense Firewall Labs](../../)

## Why This Matters

A firewall cannot be a useful lab control point if its own network placement is
unstable. I needed OPNsense to inspect and route lab traffic without placing the
household production path behind an experimental VM.

P01 records the three-week transition from unsuccessful bridge-mode attempts to
a stable router-mode design. It preserves the failures as troubleshooting
evidence while making the final supported architecture easy to understand.

## Portfolio Summary

**Situation:** Transparent bridge designs on Hyper-V and Proxmox created
interface, virtual-switch, and routing problems that made the firewall hard to
manage and unsafe to place in the production path.

**Task:** Build a clean OPNsense VM with deterministic interface mapping,
separate management, working WAN connectivity, a lab gateway, and no dependency
from the household network.

**Action:** I changed the architecture to router mode, corrected the Route10
native-VLAN handoff, built dedicated Hyper-V switches and three VM adapters,
installed OPNsense, assigned interfaces deliberately, added narrowly scoped
rules, and tested management, WAN, isolation, and production independence.

**Result:** PASS for the baseline router scope. OPNsense obtained WAN service,
the management GUI and SSH path worked from the approved network, the lab
gateway was ready for downstream segments, and stopping the VM did not interrupt
the production network.

## How To Read This Project

| Reader | Start here |
|---|---|
| Hiring manager or non-technical reader | [Portfolio Summary](#portfolio-summary), [What I Proved](#what-i-proved), and [Phase 7](#phase-7--verification-and-closeout) |
| Technical reviewer | [Original technical record](technical-details.md), [Phase Status](#phase-status), and [Technical Evidence](#technical-evidence) |
| Future operator | [Reproduce Or Re-Verify](#reproduce-or-re-verify), then the command references and decision trees in [technical details](technical-details.md) |

## My Test Boundary

| Item | Boundary |
|---|---|
| Firewall role | Router mode; not transparent bridge mode |
| Hypervisor | Existing Windows Server Hyper-V host |
| WAN handoff | Route10-facing lab connection |
| Management | Separate approved management network |
| Lab side | OPNsense-owned lab gateway and later VLAN trunk |
| Protected path | Household production routing remains on Route10 |

The completed project did not make OPNsense the household edge. Future VLAN,
IDS/IPS, VPN, monitoring, and HA work remained separate projects with their own
approval and rollback requirements.

## Phase Status

| Phase | Work | Status |
|---:|---|---|
| 1 | Define the safe architecture and abandon bridge mode | Complete |
| 2 | Correct the Route10 WAN handoff and native VLAN | Complete |
| 3 | Configure the Cisco switch path | Complete |
| 4 | Build clean Hyper-V switches and ordered VM adapters | Complete |
| 5 | Install OPNsense and assign WAN, LAN, and management | Complete |
| 6 | Configure addresses, access rules, NAT, and services | Complete |
| 7 | Verify connectivity, isolation, and production independence | Complete |

## Phase 1 — Architecture And Safety Boundary

I first attempted transparent bridge designs because they promised inspection
without changing routing. Hyper-V switch behavior, interface ambiguity, and
management risk made that model unreliable in this environment. I documented
the failed bridge and Proxmox attempts in the [technical record](technical-details.md),
then chose router mode with physically and logically separate WAN, lab, and
management paths. That decision protected the Route10 production path and gave
the remaining phases a stable target.

## Phase 2 — Route10 WAN Handoff

The OPNsense WAN adapter showed a healthy physical link but received no DHCP
offer. Layer-by-layer checks traced the failure to the Route10 port's missing
native-VLAN setting, not to OPNsense. After correcting the handoff, WAN received
`192.168.10.32/24`, reached the internet, and resolved DNS. This proved the
upstream path before I added more virtual switches or firewall rules.

## Phase 3 — Cisco Switch Path

I configured and verified the Cisco switch port that carried the lab-side
traffic toward Hyper-V. VLAN and trunk output confirmed the allowed network
path and prevented a physical switching error from being misdiagnosed later as
an OPNsense problem. With the upstream and switch layers known, I could rebuild
the virtual layer cleanly.

## Phase 4 — Hyper-V Switches And VM Adapters

Earlier bridge experiments had left confusing virtual-switch state. I backed up
the VM information, created or verified dedicated external switches, and built
the OPNsense VM with three adapters in a deliberate order: management, lab, and
WAN. I used static memory and retained console access because a router must stay
recoverable when networking fails. This produced predictable FreeBSD interface
mapping for the installation phase.

## Phase 5 — Installation And Interface Assignment

I installed OPNsense, removed the installation media, and mapped the three
Hyper-V adapters to the correct FreeBSD interfaces. Adapter order mattered, so
I verified MAC addresses and link state rather than trusting interface names
alone. Once WAN, LAN, and management were distinct, I could assign addresses
without creating overlapping routes or exposing the GUI on the wrong side.

## Phase 6 — Addresses, Rules, NAT, And Services

I configured the WAN lease, lab gateway, management address, firewall access,
outbound NAT, DNS, and initial DHCP service. A rule-order mistake temporarily
blocked the GUI, and a shared-vSwitch design caused ambiguous routing. Console
access let me restore the firewall, separate the interfaces, and apply a narrow
management rule before re-enabling protection. The current repo architecture
records management at `192.168.20.254`; historical addresses and changes remain
in the technical chronology.

## Phase 7 — Verification And Closeout

I tested WAN connectivity, DNS, management GUI and SSH access, management
isolation, firewall behavior, and the production path with OPNsense running and
stopped. The core checks passed, and the shutdown test proved that household
routing did not depend on the lab firewall. I closed P01 as the router baseline
and handed the clean lab-side path to P02 for actual VLAN segmentation.

## What I Proved

- Router mode is stable on this Hyper-V host where bridge mode was not.
- The WAN path receives service from Route10 and reaches the internet.
- Management access is separated from the WAN and lab data paths.
- Console recovery can restore access after a firewall-rule mistake.
- Dedicated virtual switches and ordered adapters remove ambiguous routing.
- The production network remains operational when the OPNsense VM is stopped.
- The baseline is ready for separately controlled VLAN and security projects.

## Technical Evidence

- [Original 4,600-line implementation and troubleshooting record](technical-details.md)
- [Repository architecture and current ownership](../../README.md#current-architecture)
- [Project index and status](../README.md)
- [Cross-family integration map](../../docs/cross-family-integration.md)
- [Repository safety and role rules](../../AGENTS.md)

## How We Worked Together

### My Input And How I Helped

I built the original environment, performed the Hyper-V, Route10, Cisco, and
OPNsense actions, tested each failure and recovery, and wrote the detailed
technical chronology. I also made the decisive change from bridge mode to
router mode and kept the household production path outside the experiment.

### What Codex Did And How

Codex did not claim the original 2025 build. For this migration, Codex preserved
my full record as `technical-details.md`, reconciled it with the current repo
facts, and wrote this short phase narrative with direct evidence links.

### What Claude Did And How

The retained 2025 record does not document a Claude role in the original build,
so I do not invent one. During the 2026 documentation migration, Claude was
asked to independently review the new summary for factual claims, role accuracy,
and broken links; that review does not replace my original evidence.

### How We Communicated And Completed The Project

The original work was completed through my hands-on troubleshooting and saved
documentation. In the later migration, I defined the portfolio standard, Codex
converted the record, and Claude independently challenged the result. The long
technical history remains available when a reader needs command-level detail.

### Pushback And How We Resolved It

The strongest pushback came from the environment itself: bridge mode repeatedly
failed to provide a safe, deterministic path. I stopped treating the original
design as mandatory and chose router mode. Later DHCP, GUI, VLAN, and routing
failures were resolved one layer at a time with console access, packet checks,
and clean virtual-switch separation instead of repeated broad changes.

## Reproduce Or Re-Verify

1. Confirm the current Route10, Cisco, Hyper-V, and OPNsense ownership boundaries
   in the repository instructions.
2. Back up the OPNsense VM and configuration, preserve console access, and record
   the physical NIC-to-vSwitch mapping.
3. Build three separate VM adapters and verify their MAC addresses before
   assigning WAN, lab, and management inside OPNsense.
4. Test the Route10 WAN handoff before adding firewall or VLAN complexity.
5. Add the minimum management and lab rules, then verify WAN, GUI, SSH, NAT,
   isolation, and production independence using the procedures in
   [technical details](technical-details.md).

## What Happens Next

P01 is closed. [P02](../02-vlan-segmentation/) later extended this router
baseline with OPNsense-owned lab VLANs across Hyper-V, Cisco, and Proxmox. P01
does not authorize changes to P02 or any later firewall project.
