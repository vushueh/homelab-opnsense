# Phase 2 — OPNsense Admin Auth via Windows NPS/RADIUS

## Goal

Configure OPNsense to authenticate admin logins against Windows Active Directory via NPS/RADIUS. Test that AD group membership controls OPNsense access level.

---

## Architecture

```
Admin browser → OPNsense GUI (192.168.20.254)
  └── OPNsense RADIUS client → NPS on WIN-PRQD8TJG04M (192.168.20.11)
        └── NPS policy → AD group check (GG-NetAdmins)
              └── Accept/Reject → OPNsense grants or denies login
```

---

## Track A — Windows NPS Setup (coordinates with Windows Server P13)

[LEONEL — on WIN-PRQD8TJG04M]

1. Open **NPS (Network Policy Server)**
2. Add RADIUS Client:
   - Friendly name: `OPNsense-Firewall`
   - IP: `192.168.20.254` (OPNsense management IP)
   - Shared secret: `<strong-unique-secret>` — record for Phase 2 Step 2
3. Create Network Policy:
   - Condition: Windows Group = `CHONGONG\GG-NetAdmins`
   - Auth method: PAP or MS-CHAPv2
   - Grant access

---

## Track B — OPNsense RADIUS Client Config

[LEONEL]

1. OPNsense → **System → Access → Servers**
2. Click **+** to add RADIUS server:
   - **Type:** RADIUS
   - **Hostname:** `192.168.20.11` (WIN-PRQD8TJG04M)
   - **Shared secret:** same as NPS client config
   - **Port:** 1812
   - **Services offered:** Authentication
3. Save

4. OPNsense → **System → Access → Users** → **Settings** tab:
   - Set **Authentication Server** to the RADIUS server just created
   - Enable **Fallback to local** (critical — ensures admin access if RADIUS unreachable)
5. Apply

---

## Test

```
[ ] Log in to OPNsense with AD credentials (leonel / adm-leonel password)
[ ] Login succeeds → RADIUS auth working
[ ] Log in with a non-AD account that is NOT in GG-NetAdmins → should fail
[ ] Disable RADIUS server (wrong IP) → local admin fallback still works
[ ] Re-enable correct RADIUS server → AD auth resumes
```

---

## Safety Gate

**Do not remove local admin account** until RADIUS fallback is proven working. If both RADIUS and local fail simultaneously, OPNsense is locked out — access only via Hyper-V console.

---

## Documentation Checklist

- [ ] NPS RADIUS client and policy configured for OPNsense
- [ ] OPNsense RADIUS auth server configured
- [ ] AD login test successful
- [ ] Local fallback tested and confirmed
- [ ] Shared secret stored securely (NOT in repo)
