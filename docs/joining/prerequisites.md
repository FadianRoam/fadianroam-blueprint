# Prerequisites

Before joining FadianRoam, ensure you meet the following requirements.

## Required Infrastructure

### 1. Wi-Fi Access Point with 802.1X

You need at least one Wi-Fi access point that supports **WPA2/WPA3-Enterprise** (802.1X):

- Must support RADIUS authentication
- Must be configurable to point to your local RADIUS server
- Consumer-grade routers generally do **not** support this
- Recommended: Ubiquiti UniFi, Mikrotik, OpenWrt-based APs

### 2. RADIUS Server

A FreeRADIUS server (or compatible) configured for:

- EAP-TTLS/PAP authentication
- Realm-based proxying (forward non-local realms to Federation Relay)
- REST module for Keycloak ROPC integration

Recommended: **FreeRADIUS 3.x** on Debian/Ubuntu.

### 3. Identity Provider (IDP)

An OpenID Connect-capable IDP that supports the **ROPC grant**:

- Recommended: **Keycloak** (open-source, self-hosted)
- Must support Resource Owner Password Credentials grant type
- A dedicated realm for roaming users is recommended

### 4. Server / VPS

A server to run RADIUS and Keycloak:

- Linux server (Debian 12+ recommended)
- Public IP address (recommended for BGP members, optional for VPN-only)
- Minimum: 2 vCPU, 2 GB RAM, 20 GB disk
- Ports: WireGuard (UDP, configurable), RADIUS (UDP 1812/1813)

### 5. TLS Certificate

A valid TLS certificate for your RADIUS server's EAP tunnel:

- From a publicly trusted CA (Let's Encrypt, ZeroSSL)
- Auto-renewal configured
- Self-signed certificates will cause client connection failures

### 6. Domain Name

A domain or subdomain for your realm identifier:

- Example: `roam.example.net`
- Used as the RADIUS realm suffix (e.g., `user@roam.example.net`)
- DNS A record pointing to your server (for federation coordination)

## Optional (BGP Members)

If you want to participate in FadianNet BGP:

| Requirement | Details |
|-------------|---------|
| Own ASN | Public or private ASN |
| BGP daemon | BIRD, FRRouting, or similar |
| IP prefix | At least one routable prefix to announce |
| Public IP | Required for BGP peering |

## Network Requirements

| Port | Protocol | Direction | Purpose |
|------|----------|-----------|---------|
| WireGuard (e.g., 51820) | UDP | Outbound | MGMT VPN to Federation Relay |
| 1812 | UDP | Inbound | RADIUS authentication |
| 1813 | UDP | Inbound | RADIUS accounting |

!!! tip "Firewall"
    At minimum, your RADIUS ports (1812/1813 UDP) must be reachable from the Federation Relay's MGMT VPN IP. If you are a BGP member, additional ports may be needed for the FadianNet VPN.

## Skill Requirements

You should be comfortable with:

- Linux server administration
- Basic networking concepts (IP addressing, routing, firewalls)
- WireGuard VPN setup
- RADIUS concepts (realms, proxying, EAP)

FadianRoam provides setup guides, but members are expected to maintain their own infrastructure.
