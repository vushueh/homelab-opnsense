# Phase 5 — Break/Fix Exercises

## Goal

Intentionally cause DNS and DHCP failures, diagnose from symptoms, and restore service. Builds real troubleshooting skill for the most common network service failures.

---

## Break Scenario A — DNS Upstream Failure

**Break:** Remove all DoT upstream servers from Unbound query forwarding

**Symptom:**
- External DNS resolution fails for all VLANs
- Internal hostnames may still resolve (Unbound cache)
- Ping 8.8.8.8 works (routing OK) but `ping google.com` fails (DNS broken)

**Diagnose:**
```sh
# On OPNsense
dig @127.0.0.1 google.com    # SERVFAIL or timeout
cat /var/unbound/unbound.conf | grep forward   # no forwarders listed
```

**Fix:** Re-add DoT upstream servers in Unbound DNS over TLS section, apply, test.

---

## Break Scenario B — DHCP Scope Exhaustion

**Break:** Shrink VLAN 30 DHCP range to only 2 addresses (e.g. .100–.101), then spin up 3 clients

**Symptom:**
- Third client gets APIPA address (169.254.x.x) or no IP
- First two clients unaffected
- OPNsense DHCP leases table shows pool exhausted

**Diagnose:**
```sh
# On OPNsense
grep "no free leases" /var/log/system/latest.log
# OR check GUI: Services → DHCPv4 → Leases — all addresses in use
```

**Fix:** Expand DHCP range back to original, apply. Client renews and gets valid IP.

---

## Break Scenario C — DNS Rebind Block (False Positive)

**Break:** Enable DNS rebind protection, then configure an internal hostname that returns an RFC1918 address (simulating a split-horizon DNS setup)

**Symptom:**
- Internal resource unreachable by hostname but reachable by direct IP
- Unbound logs show rebind protection blocked the response

**Diagnose:**
```sh
dig @127.0.0.1 internal.lab.local
# Returns REFUSED or empty answer due to rebind protection
tail -f /var/log/system/latest.log | grep rebind
```

**Fix:** Add the internal domain to Unbound's rebind exception list in Advanced settings.

---

## Break/Fix Log

Save to `troubleshooting/break-fix-log.md`:

```
## Scenario A — DNS Upstream Failure — YYYY-MM-DD
Symptom: External DNS fails, pings to IPs work
Root cause: DoT upstream removed from Unbound
Fix: Re-added Cloudflare 1.1.1.1:853 and Google 8.8.8.8:853
Verified: dig google.com returns NOERROR
```

---

## Documentation Checklist

- [ ] All 3 scenarios completed
- [ ] Each symptom, diagnosis, and fix documented
- [ ] break-fix-log.md committed to troubleshooting/
