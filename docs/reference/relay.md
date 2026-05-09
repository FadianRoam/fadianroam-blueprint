# Federation Relay

The Federation Relay is the only shared infrastructure in FadianRoam. It proxies RADIUS authentication requests between members.

## Role

The Relay acts as a central RADIUS proxy:

```mermaid
graph LR
    RA[Member A<br/>RADIUS] -->|user@realm.b| RELAY[Federation Relay<br/>FreeRADIUS Proxy]
    RELAY -->|forward| RB[Member B<br/>RADIUS]
    RB -->|Access-Accept| RELAY
    RELAY -->|Access-Accept| RA
```

- Receives RADIUS requests from members over MGMT VPN
- Looks up the realm to determine the destination member
- Forwards the request to the correct member's RADIUS
- Returns the response

The Relay does **not**:

- Store any user credentials
- Inspect inner EAP payloads (encrypted in TLS tunnel)
- Participate in user data traffic
- Run an IDP or Keycloak instance

## Architecture

### Components

| Component | Purpose |
|-----------|---------|
| FreeRADIUS | RADIUS proxy engine |
| WireGuard | MGMT VPN hub (star topology) |
| Federation config | Realm → member IP mapping |

### Network

The Relay is the hub of the MGMT VPN star:

- IP: `10.250.0.1`
- Listens on RADIUS ports 1812/1813 within the MGMT subnet
- Each member has a WireGuard peer entry on the Relay

## FreeRADIUS Proxy Configuration

### Realm Definitions

The Relay maintains a realm entry for each member, generated from the federation registry:

```
# proxy.conf on Federation Relay

# Member A
realm roam.member-a.net {
    type = radius
    authhost = 10.250.0.10:1812
    accthost = 10.250.0.10:1813
    secret = <member-a-shared-secret>
    nostrip
}

# Member B
realm roam.member-b.org {
    type = radius
    authhost = 10.250.0.11:1812
    accthost = 10.250.0.11:1813
    secret = <member-b-shared-secret>
    nostrip
}

# Reject unknown realms
realm DEFAULT {
    reject = yes
}
```

**Key settings:**

- `nostrip`: Preserves the full `user@realm` so the destination RADIUS can process it
- `DEFAULT` realm rejects unknown realms (only registered members are proxied)
- Each member has a unique shared secret

### Client Definitions

Each member is defined as a RADIUS client:

```
# clients.conf on Federation Relay

client member-a {
    ipaddr = 10.250.0.10
    secret = <member-a-shared-secret>
    shortname = member-a
}

client member-b {
    ipaddr = 10.250.0.11
    secret = <member-b-shared-secret>
    shortname = member-b
}
```

### Virtual Server

The Relay runs a minimal virtual server — it only proxies, no local authentication:

```
# sites-enabled/default on Relay

authorize {
    preprocess
    suffix
    # No local auth — all requests are proxied via realm routing
}

authenticate {
    # Empty — proxy handles authentication
}
```

## WireGuard Hub Configuration

```ini
# /etc/wireguard/fadianroam-mgmt.conf on Relay

[Interface]
Address = 10.250.0.1/24
PrivateKey = <relay-private-key>
ListenPort = 51820

[Peer]
# Member A
PublicKey = <member-a-pubkey>
AllowedIPs = 10.250.0.10/32

[Peer]
# Member B
PublicKey = <member-b-pubkey>
AllowedIPs = 10.250.0.11/32

# ... one [Peer] per member
```

## Configuration Management

The Relay's configuration is derived from the federation registry (`members/*.yml` files):

1. Member submits PR with their `members/<realm>.yml`
2. PR is reviewed and approved
3. On merge, Relay configuration is regenerated:
    - `proxy.conf`: New realm entry
    - `clients.conf`: New client entry
    - WireGuard: New peer entry
4. Services are reloaded

This can be automated with CI/CD:

```bash
# Example: regenerate and reload
./scripts/generate-relay-config.sh
systemctl reload freeradius
wg syncconf fadianroam-mgmt <(wg-quick strip fadianroam-mgmt)
```

## Deployment Recommendations

| Aspect | Recommendation |
|--------|---------------|
| Server | Dedicated VPS (separate from any member's infrastructure) |
| Location | Low-latency region for majority of members |
| Specs | 1 vCPU, 1 GB RAM (lightweight proxy workload) |
| OS | Debian 12+ |
| Redundancy | Future: multiple relays with shared realm config |

## Monitoring

### RADIUS

```bash
# Check FreeRADIUS is running
systemctl status freeradius

# Debug mode (stop service first)
freeradius -X

# Watch proxy decisions in debug output:
# - "Found realm: roam.member-b.org"
# - "Proxying to realm roam.member-b.org"
```

### WireGuard

```bash
# All peer handshakes
wg show fadianroam-mgmt latest-handshakes

# Identify members with stale handshakes (>2 minutes)
wg show fadianroam-mgmt latest-handshakes | awk '$2 > 120 {print $1}'
```

### Health Checks

```bash
# Ping all members
for ip in $(wg show fadianroam-mgmt allowed-ips | awk '{print $2}' | cut -d/ -f1); do
    ping -c 1 -W 2 $ip && echo "$ip OK" || echo "$ip UNREACHABLE"
done
```
