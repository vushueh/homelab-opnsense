# Project 06 — High Availability Design

## Status

Future concept. Design-only until a second suitable firewall instance or host exists.

## Purpose

Document what would be required to make OPNsense highly available using CARP/pfsync/XMLRPC sync, then define safe lab tests without risking the production network.

## Skills Practiced

- HA firewall architecture
- CARP VIP planning
- pfsync state synchronization
- config synchronization
- split-brain risk analysis
- failover testing
- rollback planning

## Phase Index

| Phase | Goal |
|-------|------|
| 1 | Requirements and hardware audit |
| 2 | HA topology design |
| 3 | CARP/pfsync planning |
| 4 | Build second node when hardware is available |
| 5 | Failover testing |
| 6 | Break/fix: failed sync or split brain |
| 7 | Document production readiness decision |

## Current Decision

Do not build HA yet unless hardware and network paths support it. Keep this as a planning project until prerequisites are real.
