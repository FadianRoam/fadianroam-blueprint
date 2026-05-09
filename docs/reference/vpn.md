# VPN Configuration

FadianRoam uses WireGuard for both MGMT and FadianNet VPN tunnels.

## MGMT VPN

The MGMT VPN connects each member's RADIUS server to the Federation Relay. It carries **only** RADIUS proxy traffic.

### Topology

Star topology — every member connects to the central Federation Relay:

```
Member A ──── WireGuard ────┐
Member B ──── WireGuard ────┤── Federation Relay (10.250.0.1)
Member C ──── WireGuard ────┘
```

### Addressing

| Role | IP Range |
|------|----------|
| Federation Relay | `10.250.0.1` |
| Members | `10.250.0.10` – `10.250.0.254` |
| Subnet | `10.250.0.0/24` |

### Member Configuration

`/etc/wireguard/fadianroam-mgmt.conf`:

```ini
[Interface]
Address = 10.250.0.XX/24
PrivateKey = <member-private-key>
ListenPort = 51820

[Peer]
# Federation Relay
PublicKey = <relay-public-key>
Endpoint = <relay-public-ip>:51820
AllowedIPs = 10.250.0.0/24
PersistentKeepalive = 25
```

### Relay Configuration

The Relay has a peer entry for each member:

```ini
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
```

### Firewall Rules

On the member side, restrict MGMT tunnel to RADIUS traffic only:

```bash
# Allow RADIUS to/from MGMT subnet
iptables -A INPUT -i fadianroam-mgmt -p udp --dport 1812 -j ACCEPT
iptables -A INPUT -i fadianroam-mgmt -p udp --dport 1813 -j ACCEPT
iptables -A INPUT -i fadianroam-mgmt -j DROP
```

## FadianNet VPN

FadianNet carries user data traffic after authentication. Members connect via WireGuard tunnels to the FadianNet backbone.

### Point-to-Point Links

BGP members establish point-to-point WireGuard tunnels:

```
Member A ──── P2P WireGuard ──── Member B
   │                                │
   └──── P2P WireGuard ──── Member C
```

### Addressing

| Purpose | Subnet |
|---------|--------|
| Loopbacks (router IDs) | `10.251.0.0/24` |
| P2P tunnel links | `10.252.0.0/16` |

P2P links use /30 subnets from the `10.252.0.0/16` range:

```
Member A ←→ Member B: 10.252.0.0/30
  A: 10.252.0.1
  B: 10.252.0.2

Member A ←→ Member C: 10.252.0.4/30
  A: 10.252.0.5
  C: 10.252.0.6
```

### BGP Member FadianNet Config

`/etc/wireguard/fadiannet-peer-b.conf`:

```ini
[Interface]
Address = 10.252.0.1/30
PrivateKey = <member-a-private-key>
ListenPort = 51821

[Peer]
PublicKey = <member-b-pubkey>
Endpoint = <member-b-public-ip>:51821
AllowedIPs = 10.252.0.2/32, 10.251.0.0/24, 0.0.0.0/0
PersistentKeepalive = 25
```

### VPN-Only Member Config

VPN-only members get a single tunnel to the nearest BGP member:

```ini
[Interface]
Address = 10.252.X.X/30
PrivateKey = <member-private-key>
ListenPort = 51821

# Default route through FadianNet
PostUp = ip route add default via 10.252.X.Y dev %i table fadiannet
PostDown = ip route del default via 10.252.X.Y dev %i table fadiannet

[Peer]
PublicKey = <upstream-bgp-member-pubkey>
Endpoint = <upstream-ip>:51821
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## WireGuard Operations

### Generate Keys

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

### Start / Stop

```bash
# Start
wg-quick up fadianroam-mgmt

# Stop
wg-quick down fadianroam-mgmt

# Enable on boot
systemctl enable wg-quick@fadianroam-mgmt
```

### Status

```bash
# Show all interfaces
wg show

# Show specific interface
wg show fadianroam-mgmt

# Check handshake (should be recent)
wg show fadianroam-mgmt latest-handshakes
```

### Troubleshooting

| Issue | Check |
|-------|-------|
| No handshake | Firewall blocking UDP port, wrong endpoint |
| Handshake but no traffic | `AllowedIPs` mismatch |
| Intermittent drops | `PersistentKeepalive` not set, NAT timeout |
| MTU issues | Set `MTU = 1420` in Interface section |
