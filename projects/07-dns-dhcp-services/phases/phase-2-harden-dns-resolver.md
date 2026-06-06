# Phase 2 — Harden Unbound DNS Resolver

## Goal

Configure OPNsense Unbound with DNS over TLS upstream, DNSSEC validation, and DNS rebind attack protection. Verify DNS resolution still works for all lab VLANs.

---

## Track A — GUI Steps

[LEONEL]

### 2.1 Enable DNS over TLS

1. OPNsense → **Services → Unbound DNS → DNS over TLS**
2. Add upstream servers:

| Server | Port | Verify CN |
|--------|------|-----------|
| 1.1.1.1 | 853 | cloudflare-dns.com |
| 8.8.8.8 | 853 | dns.google |

3. Save and apply

### 2.2 Enable DNSSEC

1. OPNsense → **Services → Unbound DNS → Advanced**
2. Enable **DNSSEC** → Save and apply
3. Test: `dig sigfail.verteiltesysteme.net` — should return SERVFAIL (DNSSEC working)

### 2.3 Enable DNS Rebind Protection

1. OPNsense → **Services → Unbound DNS → General**
2. Enable **Rebind detection** → Save
3. Test: internal hostnames still resolve correctly from each VLAN

### 2.4 Disable Plain Forwarding (if DoT is active)

1. OPNsense → **Services → Unbound DNS → Query Forwarding**
2. Confirm plain-text forwarding is OFF — DoT handles upstream queries

---

## Track B — CLI Verification

[CODEX/CLAUDE — after approval]

```sh
# Confirm DoT connections
ss -tnp | grep :853

# Test DNSSEC validation
dig @127.0.0.1 sigfail.verteiltesysteme.net
# Expected: SERVFAIL (bad signature correctly rejected)

dig @127.0.0.1 google.com
# Expected: NOERROR with ANSWER

# Test rebind protection
dig @127.0.0.1 <a-hostname-with-rfc1918-response>
```

---

## Rollback

If DNS stops resolving for lab clients:

1. OPNsense → Services → Unbound DNS → General → disable DoT temporarily
2. Re-enable plain forwarding to 8.8.8.8 as fallback
3. Investigate which setting caused the issue before re-applying

---

## Documentation Checklist

- [ ] DNS over TLS configured with Cloudflare and Google
- [ ] DNSSEC enabled and SERVFAIL test passed
- [ ] DNS rebind protection enabled
- [ ] All lab VLANs still resolve internal and external names
- [ ] Screenshots saved
