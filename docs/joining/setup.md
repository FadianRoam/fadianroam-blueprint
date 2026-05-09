# Setup Guide

Step-by-step instructions for configuring your FadianRoam member infrastructure after approval.

## Step 1: MGMT VPN (WireGuard)

After approval, you will receive:

- **Relay public key**: The Federation Relay's WireGuard public key
- **Relay endpoint**: The Relay's public IP and port
- **Your MGMT IP**: Your assigned IP in `10.250.0.0/24`

### Generate Keys

```bash
wg genkey | tee /etc/wireguard/mgmt-privatekey | wg pubkey > /etc/wireguard/mgmt-publickey
```

### Configure WireGuard

Create `/etc/wireguard/fadianroam-mgmt.conf`:

```ini
[Interface]
Address = 10.250.0.XX/24          # Your assigned MGMT IP
PrivateKey = <your-private-key>
ListenPort = 51820                 # Or any available port

[Peer]
PublicKey = <relay-public-key>
Endpoint = <relay-endpoint>:51820
AllowedIPs = 10.250.0.0/24
PersistentKeepalive = 25
```

### Start and Enable

```bash
systemctl enable --now wg-quick@fadianroam-mgmt
```

### Verify

```bash
wg show fadianroam-mgmt
ping 10.250.0.1  # Federation Relay
```

## Step 2: Keycloak (IDP)

### Install Keycloak

Follow the [official Keycloak guide](https://www.keycloak.org/guides) for your platform. Recommended: Keycloak 24+ with PostgreSQL.

### Create Roaming Realm

Create a dedicated realm for roaming users (e.g., `roam`):

1. Log in to Keycloak Admin Console
2. Create new realm named `roam`
3. Disable user self-registration (members manage their own users)

### Create FreeRADIUS Client

In the `roam` realm:

1. Go to **Clients** → **Create client**
2. Client ID: `freeradius`
3. Client authentication: **On**
4. Authentication flow: Enable **Direct access grants** (ROPC) and **Service accounts roles**
5. Save and copy the **Client secret**

### Create Test User

1. Go to **Users** → **Add user**
2. Set username, email
3. Go to **Credentials** → Set password (temporary: off)

## Step 3: FreeRADIUS

### Install

```bash
apt update && apt install -y freeradius freeradius-rest
```

### TLS Certificate

Obtain a certificate for your RADIUS domain:

```bash
# Using acme.sh with DNS-01 (example with Cloudflare)
acme.sh --issue --dns dns_cf -d radius.example.net \
  --keylength ec-256 \
  --install-cert \
  -d radius.example.net \
  --key-file /etc/freeradius/3.0/certs/radius.key \
  --fullchain-file /etc/freeradius/3.0/certs/radius.cer \
  --ca-file /etc/freeradius/3.0/certs/ca.cer \
  --reloadcmd "systemctl restart freeradius"
```

Set permissions:

```bash
chown freerad:freerad /etc/freeradius/3.0/certs/radius.*
chmod 640 /etc/freeradius/3.0/certs/radius.*
```

### Configure EAP

Edit `/etc/freeradius/3.0/mods-enabled/eap`:

```
eap {
    default_eap_type = ttls

    tls-config tls-common {
        private_key_file = /etc/freeradius/3.0/certs/radius.key
        certificate_file = /etc/freeradius/3.0/certs/radius.cer
        ca_file = /etc/freeradius/3.0/certs/ca.cer
        tls_min_version = "1.2"
        tls_max_version = "1.3"
    }

    ttls {
        tls = tls-common
        default_eap_type = md5
        copy_request_to_tunnel = yes
        use_tunneled_reply = yes
        virtual_server = "inner-tunnel"
    }
}
```

### Configure REST Module

Edit `/etc/freeradius/3.0/mods-enabled/rest`:

```
rest {
    connect_uri = "http://127.0.0.1:8080"

    authorize {
        uri = "${..connect_uri}/realms/roam/protocol/openid-connect/token"
        method = 'post'
        body = 'post'
        data = "client_id=freeradius&client_secret=<YOUR_SECRET>&grant_type=password&username=%{%{Stripped-User-Name}:-%{User-Name}}&password=%{User-Password}&scope=openid"
        force_to = 'plain'
    }

    authenticate {
        uri = "${..connect_uri}/realms/roam/protocol/openid-connect/token"
        method = 'post'
        body = 'post'
        data = "client_id=freeradius&client_secret=<YOUR_SECRET>&grant_type=password&username=%{%{Stripped-User-Name}:-%{User-Name}}&password=%{User-Password}&scope=openid"
        force_to = 'plain'
    }
}
```

### Configure Realm Proxying

Edit `/etc/freeradius/3.0/proxy.conf`:

```
# Your local realm — handle locally
realm your-realm.example.net {
}

# Federation — proxy all unknown realms to the Relay
realm DEFAULT {
    type = radius
    authhost = 10.250.0.1:1812
    accthost = 10.250.0.1:1813
    secret = <shared-secret-from-federation>
}
```

### Configure Default Site

In `/etc/freeradius/3.0/sites-enabled/default`, ensure:

- **authorize** section: `suffix` module is enabled (for realm stripping)
- **authorize** section: Set `Auth-Type := PAP` when `User-Password` is present
- **authenticate** section: `Auth-Type PAP { rest }`

### Configure Inner Tunnel

In `/etc/freeradius/3.0/sites-enabled/inner-tunnel`:

- **authenticate** section: `Auth-Type PAP { rest }`

### Test

```bash
# Debug mode
freeradius -X

# In another terminal
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
