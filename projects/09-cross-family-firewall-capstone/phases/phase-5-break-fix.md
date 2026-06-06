# Phase 5 — Break/Fix: Cross-Family Failures

## Goal

Test resilience of the cross-family integrations. What happens when RADIUS is unreachable? When Wazuh stops receiving logs? When VPN bypasses a firewall rule?

---

## Scenario A — RADIUS Unreachable

**Break:** Change NPS server IP in OPNsense RADIUS config to wrong address.

**Symptom:** AD login to OPNsense fails; local admin fallback activates.

**Diagnose:**
- OPNsense → System → Access → Servers — ping the RADIUS server manually
- Check NPS logs on WIN-PRQD8TJG04M: Event Viewer → Custom Views → Network Policy and Access Services

**Fix:** Restore correct NPS IP, re-test AD login.

**Lesson:** RADIUS fallback to local is critical — never remove local admin until RADIUS is proven stable.

---

## Scenario B — Wazuh Log Gap

**Break:** Change syslog remote target port to wrong value (from P05 break/fix pattern).

**Symptom:** OPNsense firewall events stop arriving in Wazuh. Suricata alerts also stop.

**Diagnose:**
```sh
grep -R "<wazuh-ip>" /usr/local/etc/syslog-ng.conf.d/
# Shows wrong port
```

**Fix:** Correct syslog target port, apply. Verify events resume in Wazuh within 60 seconds.

---

## Scenario C — VPN Policy Bypass Attempt

**Break:** Temporarily remove the VPN_WG block rule for VLAN 10.

**Symptom:** VPN client can now ping 192.168.10.x — production breach!

**Diagnose:**
- OPNsense → Firewall → Rules → VPN_WG — block rule missing
- OPNsense → Firewall → Log Files — VPN traffic to VLAN 10 showing as passed

**Fix:** Re-add block rule for VPN → VLAN 10, confirm ping from VPN client fails again.

**Lesson:** VPN interface rules must mirror LAN rules — VPN clients get the same restrictions as on-VLAN hosts.

---

## Break/Fix Log

Save to `troubleshooting/break-fix-log.md` — all 3 scenarios with symptom, root cause, fix, and verification.

---

## Documentation Checklist

- [ ] All 3 cross-family break/fix scenarios completed
- [ ] break-fix-log.md updated with capstone scenarios
- [ ] Each fix verified with a specific test
