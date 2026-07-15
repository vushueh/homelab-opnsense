# P02 — OPNsense VLAN Segmentation

- **Status:** ✅ Complete — 2025-10-26
- **Project ID:** `OPNsense-P02`
- **Platform:** OPNsense, Hyper-V, Cisco IOS, Proxmox VE, and Route10
- **Scope:** OPNsense-owned lab VLANs across two hypervisors
- **Parent baseline:** [P01 — OPNsense Baseline Router Deployment](../01-baseline-deployment/)

## Why This Matters

A security lab needs separate networks for trusted services, test clients, and
intentionally vulnerable systems. I built P02 so those workloads could share
physical switching without sharing one unrestricted broadcast domain.

The project also demonstrates that segmentation is an end-to-end property. An
OPNsense VLAN is not useful until the Hyper-V trunk, Cisco ports, Proxmox bridge,
Route10 return routes, DHCP, firewall policy, and NAT all agree.

## Portfolio Summary

**Situation:** P01 provided a stable router-mode firewall, but the lab workloads
still needed controlled VLANs across Hyper-V and Proxmox.

**Task:** Build OPNsense-owned VLAN gateways, carry their tags across the virtual
and physical infrastructure, route approved traffic, isolate the attack lab,
and retain enough evidence to reproduce the design.

**Action:** I created VLAN interfaces through the OPNsense GUI, configured
trunks on Hyper-V, Cisco, and Proxmox, added DHCP and firewall policy, installed
the required Route10 static routes, corrected NAT and gateway conflicts, and
verified the result with interface, route, lease, and screenshot evidence.

**Result:** PASS for the documented 2025 scope. VLANs 30, 40, 70, and 250 were
routed by OPNsense across both hypervisors; VLAN 250 remained the isolated attack
lab; and Route10 retained ownership of the production networks.

## How To Read This Project

| Reader | Start here |
|---|---|
| Hiring manager or non-technical reader | [Portfolio Summary](#portfolio-summary), [What I Proved](#what-i-proved), and [Phase 6](#phase-6--verification-and-evidence) |
| Technical reviewer | [Complete implementation guide](OPNsense-Lab-Network-Documentation.md), [quick reference](Quick-Reference-Guide.md), and [screenshot folders](images/) |
| Future operator | [Reproduce Or Re-Verify](#reproduce-or-re-verify), then the ordered configuration and troubleshooting sections in the [implementation guide](OPNsense-Lab-Network-Documentation.md) |

## My Test Boundary

| Item | Boundary |
|---|---|
| OPNsense-owned 2025 lab VLANs | 30, 40, 70, and 250 |
| Attack lab | VLAN 250; intentionally isolated |
| Hypervisors | Hyper-V and Proxmox VE |
| Physical transport | Cisco 2960G 802.1Q trunks |
| Upstream routing | Route10 static routes via OPNsense WAN `192.168.10.32` |
| Protected networks | Route10-owned production VLANs 10 and 20 |

This completion record reflects the documented 2025 build. The repository's
current architecture and ownership files are authoritative if an address,
release, or later-added VLAN has changed since that evidence was captured.

## Phase Status

| Phase | Work | Status |
|---:|---|---|
| 1 | Map the multi-hypervisor topology and VLAN ownership | Complete |
| 2 | Create OPNsense VLAN interfaces and gateways | Complete |
| 3 | Configure Hyper-V, Cisco, and Proxmox trunks | Complete |
| 4 | Add DHCP, firewall policy, and outbound NAT | Complete |
| 5 | Add Route10 return routes and resolve gateway conflicts | Complete |
| 6 | Verify leases, routes, traffic paths, and screenshots | Complete |

## Phase 1 — Topology And Ownership

I mapped every physical NIC, Hyper-V switch, Cisco trunk, Proxmox bridge, and
OPNsense interface before changing the VLAN layer. I kept Route10 responsible
for production VLANs 10 and 20 and assigned the lab networks to OPNsense. The
[implementation guide](OPNsense-Lab-Network-Documentation.md#network-architecture)
records the physical-to-virtual mapping and the [quick reference](Quick-Reference-Guide.md)
condenses the final paths. That ownership map prevented duplicate gateways when
the VLAN interfaces were created.

## Phase 2 — OPNsense VLAN Interfaces

I first created VLANs from the FreeBSD CLI, but those interfaces did not exist
in OPNsense's configuration database and therefore could not be managed safely
through the GUI. I removed the temporary interfaces and recreated each VLAN
through **Interfaces → Other Types → VLAN**, assigned it, enabled it, and added
the correct gateway address. This made the VLANs persistent and available to
DHCP, firewall, NAT, backup, and screenshot workflows.

## Phase 3 — End-To-End Trunking

I configured the Hyper-V lab adapter as a trunk, allowed the matching tags on
the Cisco ports toward Hyper-V and Proxmox, and used a VLAN-aware Proxmox bridge.
Several early failures were transport problems rather than firewall problems:
wrong allowed lists, access-versus-trunk mismatches, and VM tags at the wrong
layer. The retained [Cisco screenshots](images/cisco/), [Hyper-V screenshots](images/hyperv/),
and configuration sections in the guide show the final agreement across all
three layers.

## Phase 4 — DHCP, Firewall Policy, And NAT

I enabled a separate DHCP scope on each lab VLAN, added rules on the interface
where traffic entered OPNsense, and extended outbound NAT for the lab networks.
Rule order initially blocked inter-VLAN traffic, so I moved the specific allow
rules above broader denies and retained the attack-lab restrictions. The
[OPNsense screenshots](images/opnsense/) preserve interface assignments, rules,
leases, NAT, and routing evidence without publishing a raw configuration export.

## Phase 5 — Return Routes And Gateway Conflicts

Traffic from production networks could reach OPNsense but could not return to
an OPNsense-owned lab network until Route10 knew the next hop. I added the
required static routes through `192.168.10.32` and verified them in the
[Route10 evidence](images/route10/). I also resolved a VLAN 250 gateway conflict
by keeping a single gateway authority. These changes completed the round trip
without moving production routing away from Route10.

## Phase 6 — Verification And Evidence

I tested DHCP leases, gateway reachability, inter-VLAN paths, internet access,
cross-hypervisor communication, firewall behavior, and route selection. The
[implementation guide](OPNsense-Lab-Network-Documentation.md#verification--testing)
records the ordered checks, and the screenshot folders tie the result to
OPNsense, Route10, Cisco, and Hyper-V views. I then wrote the
[quick-reference guide](Quick-Reference-Guide.md) so future troubleshooting can
start from the final map instead of replaying the three-week discovery process.

## What I Proved

- OPNsense can own multiple isolated lab VLANs without taking over production routing.
- VLAN tags can cross Hyper-V, a Cisco trunk, and a Proxmox VLAN-aware bridge.
- GUI-created OPNsense VLANs persist and integrate with DHCP, firewall, NAT, and backup.
- Route10 return routes are required for production-to-lab round trips.
- Firewall rules must be placed on the ingress interface and evaluated in order.
- VLAN 250 can remain a distinct attack-lab boundary.
- The saved screenshots and guides support both review and reproduction.

## Technical Evidence

- [Original project navigation record](technical-details.md)
- [Complete implementation and troubleshooting guide](OPNsense-Lab-Network-Documentation.md)
- [Quick-reference guide](Quick-Reference-Guide.md)
- [OPNsense screenshots](images/opnsense/)
- [Route10 screenshots](images/route10/)
- [Cisco screenshots](images/cisco/)
- [Hyper-V screenshots](images/hyperv/)
- [Current repository architecture](../../README.md#current-architecture)

## How We Worked Together

### My Input And How I Helped

I performed the original OPNsense, Hyper-V, Cisco, Proxmox, and Route10 work,
tested the traffic paths, captured the screenshots, and consolidated the eight
major troubleshooting cases. My original commits and evidence predate the
current shared-agent workflow.

### What Codex Did And How

Codex did not claim the original 2025 implementation. During this migration,
Codex preserved the existing README as `technical-details.md`, kept the complete
implementation guide intact, reconciled current ownership notes, and wrote this
short phase-based entry page.

### What Claude Did And How

The retained 2025 project record does not document a Claude role in the original
build, so I do not assign one. For the 2026 migration, Claude was asked to review
the new summary independently for factual overstatement, link integrity, and
clear separation between historical evidence and current architecture.

### How We Communicated And Completed The Project

I completed and documented the original troubleshooting sessions. Later, I set
the portfolio documentation standard, Codex reorganized the entry page without
discarding detail, and Claude independently reviewed the migration. Readers can
move from this short story to the implementation guide when they need exact
commands or screenshots.

### Pushback And How We Resolved It

The network repeatedly challenged assumptions at different layers. CLI-created
VLANs were not manageable in the GUI, Route10 lacked return routes, firewall
rule order blocked traffic, hypervisor trunks disagreed on tags, and VLAN 250
had competing gateway assumptions. I resolved each problem at its owning layer,
verified the correction, and documented both the symptom and root cause instead
of hiding the failed attempt.

## Reproduce Or Re-Verify

1. Read the current repository ownership rules and compare them with the
   historical [quick reference](Quick-Reference-Guide.md).
2. Back up OPNsense and record Route10, Cisco, Hyper-V, and Proxmox state before
   changing a trunk or route.
3. Create VLANs through the OPNsense GUI, then assign, enable, and address each
   one before adding services.
4. Configure the same allowed tag list from the OPNsense VM adapter through the
   Cisco ports and Proxmox bridge.
5. Add DHCP, ingress firewall policy, NAT, and only the required Route10 return
   routes, then run the verification sequence in the
   [implementation guide](OPNsense-Lab-Network-Documentation.md#verification--testing).

## What Happens Next

P02 is closed. [P03](../03-ids-ips-suricata/) is the next OPNsense project and
owns the staged Suricata IDS/IPS work. This VLAN result does not authorize P03,
production-route changes, or blocking-mode deployment.
