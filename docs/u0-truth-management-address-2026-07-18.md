# U0-TRUTH OPNsense Management-Address Reconciliation — 2026-07-18

- **Status:** Named unknown; no live setting changed
- **Question:** Is the current VLAN 20 OPNsense address `.253`, `.254`, both,
  or neither?
- **Scope:** Current owner statements and retained cross-family evidence

## Conflicting Sources

| Source | Claim | Evidence class |
|---|---|---|
| `CLAUDE.md`, updated 2026-07-04 | `192.168.20.253` on `hn0` | Current owner assertion, but not freshly re-read during U0 |
| Route10 P02 retained evidence | `.253` is the OPNsense VLAN 20 interface | Dated supporting evidence |
| Route10 P02 retained evidence | `.254` is a Cisco R1 VLAN 20 path | Dated supporting evidence that `.254` was misattributed in OPNsense docs |
| OPNsense `AGENTS.md` and root `README.md` before U0 | `.254` is OPNsense management | Conflicting current/legacy assertion without fresh proof |
| Historical OPNsense project pages | `.254` is OPNsense management | Historical observation; not silently rewritten |

## U0 Read-Only Attempt

The repository assigns live SSH coordination to Claude and forbids Codex from
executing live infrastructure commands. Codex bounded Claude to read only the
OPNsense Hyper-V VM state/network-adapter report plus TCP/443 and neighbor
corroboration for `.253` and `.254` on the documented Windows host. The bridge
returned no evidence before the stop timeout and was terminated. No direct
OPNsense access, configuration read, or live change occurred.

Leonel then approved trying the documented Route10-facing OPNsense address
`192.168.10.32`. Claude confirmed TCP/22 and TCP/443 are open and attempted one
non-interactive key-only `root` query for `ifconfig hn0`. Authentication failed;
BatchMode prevented a prompt, no password was used, and Claude stopped without
retrying.

## Disposition

The current address remains **unknown** under U0's evidence standard. `.253` is
the best-supported retained value, and `.254` is separately documented as a
Cisco R1 path, but neither fact was freshly authoritative during this run.
Current owner pages therefore must not present either value as verified.
The reachable `.32` services strengthen the host-identity path but do not reveal
the `hn0` assignment.

## Exact Resolution Trigger

At a later approved read-only window, Leonel should open the existing OPNsense
Hyper-V console and run:

```text
ifconfig hn0
```

Capture only the `hn0` interface name and private IPv4/prefix line. Do not show
the login prompt, username, configuration export, public WAN data, certificates,
keys, or other interfaces. A screenshot is optional; sanitized text is enough.
If `hn0` is no longer the VLAN 20 interface, stop and record the current
interface mapping without changing it.

## Owner And Follow-On

- **Owner:** `homelab-opnsense`
- **Trigger:** Manual OPNsense console readback by Leonel in an approved
  read-only window
- **Downstream:** Q018 owns full IPAM reconciliation; U0 closes this conflict as
  a named unknown and does not start Q018.
