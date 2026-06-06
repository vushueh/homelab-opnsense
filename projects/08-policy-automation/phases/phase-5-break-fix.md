# Phase 5 — Break/Fix: API Failures

## Goal

Simulate common API failure scenarios and diagnose them from error responses. Builds confidence in using the API safely in production workflows.

---

## Scenario A — Wrong API Key

**Break:** Change `OPN_KEY` env variable to a wrong value, run backup script.

**Symptom:** HTTP 403 Forbidden response. Script exits with error.

**Diagnose:**
```bash
curl -k -u "wrongkey:$OPN_SECRET" https://$OPN_HOST/api/core/firmware/info
# Returns: {"message":"Authentication failed"}
```

**Fix:** Restore correct key value in environment variable. Re-run script.

**Lesson:** Always test API key with a read-only endpoint before running write operations.

---

## Scenario B — Revoked API Key

**Break:** Delete the API key in OPNsense GUI (System → Users → edit user → remove key).

**Symptom:** HTTP 403 even with previously working credentials.

**Diagnose:**
```bash
# Same 403 as wrong key — distinguish by checking GUI
# OPNsense → System → Access → Users → verify API key still listed
```

**Fix:** Generate a new API key pair, update environment variables.

**Lesson:** API key rotation needs to be tracked — document when keys were created and rotate periodically.

---

## Scenario C — Malformed Payload

**Break:** In create-alias.py, send `"type": "invalid_type"` instead of `"host"`.

**Symptom:** HTTP 200 response but with validation error in JSON body.

**Diagnose:**
```python
resp = requests.post(...)
print(resp.json())
# Returns: {"result": "failed", "validations": {"alias.type": "invalid value"}}
```

**Fix:** Check OPNsense API documentation for valid `type` values: `host`, `network`, `port`, `url`, `geoip`.

**Lesson:** OPNsense API returns 200 even for validation failures — always check the JSON body, not just the HTTP status code.

---

## Break/Fix Log

Save to `troubleshooting/break-fix-log.md`:

```
## Scenario A — Wrong API Key — YYYY-MM-DD
Symptom: HTTP 403 on all API calls
Root cause: OPN_KEY env variable contained wrong value
Fix: Corrected env variable value
Verified: curl returns valid JSON response
```

---

## Documentation Checklist

- [ ] All 3 scenarios completed and documented
- [ ] break-fix-log.md updated
- [ ] Key lesson documented per scenario
