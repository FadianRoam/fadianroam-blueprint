# Setup Guide

Step-by-step instructions for configuring your FadianRoam member infrastructure after approval.

## Step 1: MGMT VPN (WireGuard)

After approval, you will receive:

- **Relay public key**: The Federation Relay's WireGuard public key
- **Relay endpoint**: The Relay's public IP and port
- **Your MGMT IP**: Your assigned IP in `172.172.10.0/24`

### Generate Keys

```bash
wg genkey | tee /etc/wireguard/mgmt-privatekey | wg pubkey > /etc/wireguard/mgmt-publickey
```

### Configure WireGuard

Create `/etc/wireguard/fadianroam-mgmt.conf`:

```ini
[Interface]
Address = 172.172.10.XX/24          # Your assigned MGMT IP
PrivateKey = <your-private-key>
ListenPort = 51820                 # Or any available port

[Peer]
PublicKey = <relay-public-key>
Endpoint = <relay-endpoint>:51820
AllowedIPs = 172.172.10.0/24
PersistentKeepalive = 25
```

### Start and Enable

```bash
systemctl enable --now wg-quick@fadianroam-mgmt
```

### Verify

```bash
wg show fadianroam-mgmt
ping 172.172.10.1  # Federation Relay
```

## Step 2: Identity Provider

Deploy an Identity Provider that your RADIUS server can authenticate against. Your IDP must be able to verify user credentials on behalf of the RADIUS server (e.g., via ROPC, LDAP, REST API, or any method your RADIUS implementation supports).

Create a set of test users for federation verification.

## Step 3: RADIUS Server

Deploy a RADIUS server capable of:

1. **EAP-TTLS/PAP** — accepting 802.1X authentication from APs
2. **IDP integration** — validating local user credentials against your IDP
3. **Realm-based proxying** — forwarding non-local realm requests to the Federation Relay

### Key Configuration Points

**EAP**: Configure EAP-TTLS as the default method. Use a valid TLS certificate from a publicly trusted CA.

**IDP integration**: Configure your RADIUS server to validate credentials against your IDP. The `Stripped-User-Name` (after realm stripping) should be used as the username.

**Realm proxying**: Configure your RADIUS to:

- Handle your own realm locally
- Proxy all unknown realms (`DEFAULT`) to the Federation Relay at `172.172.10.1:1812` using the shared secret provided on approval

**Federation Relay client**: Allow the Federation Relay (`172.172.10.1`) as a RADIUS client so it can forward roaming requests to you.

### Verify

```bash
# Test local authentication
radtest user@your-realm.example.net PASSWORD localhost 0 testing123
```

## Step 4: Wi-Fi AP Configuration

Configure your access point for WPA2/WPA3-Enterprise:

| Setting | Value |
|---------|-------|
| Security | WPA2-Enterprise (802.1X) |
| RADIUS Server | Your RADIUS server IP |
| RADIUS Port | 1812 |
| RADIUS Secret | Your local RADIUS client secret |
| SSID | `FadianRoam` (recommended) |

!!! tip "SSID Convention"
    Using the SSID `FadianRoam` across all member sites allows devices to automatically connect when roaming, similar to the `eduroam` SSID.

## Step 5: Verify Federation

After completing setup, notify the federation maintainers to run a cross-site authentication test:

1. A test user from another member site attempts to authenticate at your AP
2. Verify the RADIUS proxy chain works end-to-end
3. Confirm user receives network access after authentication

Once verified, your member status is updated to **Active** in the federation registry.
