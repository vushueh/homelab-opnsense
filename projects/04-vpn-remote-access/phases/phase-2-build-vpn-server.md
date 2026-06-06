# Phase 2 — Build VPN Server (WireGuard)

## Goal

Configure a WireGuard VPN server on OPNsense. Generate server and client key pairs.
No WAN exposure yet — local testing only in this phase.

---

## Track A — GUI Steps (WireGuard)

[LEONEL]

### 2.1 Confirm WireGuard Availability

WireGuard is built into modern OPNsense releases. Do not install `os-wireguard` on OPNsense 24.x unless the GUI is missing WireGuard and Claude confirms the installed release needs a plugin.

1. OPNsense → **VPN → WireGuard**
2. Confirm the **Instances**, **Peers**, and **General** pages are visible
3. If WireGuard is missing, stop and record the OPNsense version from **System → Firmware → Status** for Claude review

### 2.2 Create Server Instance

1. OPNsense → **VPN → WireGuard → Instances**
2. Click **+** to add instance
3. Configure:
   - **Enabled:** Checked
   - **Name:** `lab-vpn`
   - **Listen Port:** `51820`
   - **Tunnel Address:** `10.200.200.1/24` (VPN subnet — does not exist elsewhere in lab)
   - **Generate Keys** → click to auto-generate server public/private key pair
   - **Save** (private key auto-stored, public key shown)

4. Record server **Public Key** — needed for client config

### 2.3 Create Client Peer Entry

1. OPNsense → **VPN → WireGuard → Peers**
2. Click **+**
3. Configure:
   - **Enabled:** Checked
   - **Name:** `leonel-laptop`
   - **Public Key:** (generate on client first — see Track B)
   - **Allowed IPs:** `10.200.200.2/32` (client VPN IP)
4. Save and apply
5. Return to **VPN → WireGuard → Instances**
6. Edit `lab-vpn`, add `leonel-laptop` in the **Peers** field, save and apply

### 2.4 Enable WireGuard

1. OPNsense → **VPN → WireGuard → General**
2. Enable WireGuard → Apply

---

## Track B — Client Key Generation

[LEONEL — on laptop/client machine]

**On Windows (PowerShell — wireguard must be installed):**
```powershell
# Generate client key pair
$privkey = & "C:\Program Files\WireGuard\wg.exe" genkey
$pubkey  = $privkey | & "C:\Program Files\WireGuard\wg.exe" pubkey
Write-Host "Private: $privkey"
Write-Host "Public:  $pubkey"
```

**On Linux/macOS:**
```bash
wg genkey | tee client-private.key | wg pubkey > client-public.key
cat client-public.key   # paste this into OPNsense peer config
```

**DO NOT commit private key to GitHub.**

---

## Client Config Template

```ini
[Interface]
PrivateKey = <client-private-key>
Address = 10.200.200.2/32
DNS = 192.168.30.1

[Peer]
PublicKey = <server-public-key>
Endpoint = <OPNsense-WAN-IP>:51820
AllowedIPs = 192.168.30.0/24, 192.168.40.0/24, 192.168.250.0/24, 10.200.200.0/24
PersistentKeepalive = 25
```

Save client config as `client-leonel-laptop.conf` — **do not commit this file to GitHub**.
Add to `.gitignore`: `*.conf` (WireGuard client configs)

---

## Local Connectivity Test (before WAN exposure)

With OPNsense and client on the same LAN (before opening WAN port):

```bash
# Change Endpoint to OPNsense LAN IP temporarily
Endpoint = 192.168.20.254:51820

# Start WireGuard tunnel on client
# Ping VPN server
ping 10.200.200.1

# Check handshake
wg show
# Expected: latest handshake a few seconds ago, transfer: X bytes received
```

---

## Screenshots to Capture

- WireGuard Instance showing server public key and tunnel address
- Peers list showing leonel-laptop peer entry
- `wg show` on client with active handshake
- Ping 10.200.200.1 success

## Documentation Checklist

- [ ] WireGuard confirmed available in OPNsense
- [ ] Server instance created (port 51820, tunnel 10.200.200.1/24)
- [ ] Client key pair generated (private key NOT in repo)
- [ ] Peer entry configured in OPNsense
- [ ] Local handshake test successful
- [ ] Client config template saved (sanitized — no real keys)
