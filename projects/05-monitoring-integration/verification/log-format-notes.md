# Q028 log formats and counting boundaries

- The retained OPNsense target uses legacy syslog. `sequenceId` does not survive this path. The controlled firewall-block population is identified by source port, destination test port and sender, not by a presumed sequence field.
- Built-in `sshd` decoding handles `sshd-session` authentication lines: 5715 for successful authentication and 5710 for invalid-user events.
- Q027 rule 100086 retains decoded Suricata fields including `data.alert.signature_id`, `data.flow_id`, `data.timestamp` and `data.in_iface`.
- Q028 rule 100285 raises an alert for the existing logged lab-to-lab pass population. Three TCP connections produced six manager alerts with two different firewall rule tracking IDs; these are multiple logged rule observations, not six connections.
- The heartbeat decoder exposes flat fields such as `data.q028_seq`, `data.q028_run` and `data.q028_transition`. A query for `data.q028.seq` would be wrong.
- Manager receipt, index acceptance and dashboard visibility are separate stages. An enabled setting, a UDP send or a collector listener cannot prove end-to-end delivery.
- The sensor clock was about 129 seconds ahead. Match exact tuples, flow IDs and embedded timestamps; do not claim sub-minute cross-host latency.
