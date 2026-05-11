# Prerequisites

Before joining FadianRoam, ensure you meet the following requirements. FadianRoam does not prescribe specific brands, software, or deployment methods — members are free to choose their own solutions as long as they meet the functional requirements below.

## Required Infrastructure

### 1. Wi-Fi Access Point with 802.1X

At least one Wi-Fi access point that supports **WPA2/WPA3-Enterprise** (802.1X):

- Must support RADIUS authentication
- Must be configurable to point to your RADIUS server

### 2. RADIUS Server

A RADIUS server capable of:

- EAP-TTLS/PAP authentication
- Realm-based proxying (forward non-local realms to Federation Relay)
- Integrating with your Identity Provider for credential verification

### 3. Identity Provider (IDP)

An identity provider that your RADIUS server can authenticate against:

- Must be able to verify user credentials on behalf of RADIUS
- How you implement this (ROPC, LDAP, local database, etc.) is up to you

### 4. Server

A server to host your RADIUS and IDP infrastructure:

- Must be able to establish a WireGuard tunnel to the Federation Relay
- Must be reachable for RADIUS traffic over the MGMT VPN

### 5. TLS Certificate

A valid TLS certificate for your RADIUS server's EAP tunnel:

- Must be from a publicly trusted CA (self-signed certificates will cause client connection failures)
- Auto-renewal is strongly recommended

### 6. Domain Name

A domain or subdomain for your realm identifier:

- Used as the RADIUS realm suffix (e.g., `user@roam.example.net`)
- DNS A record pointing to your server

## Optional (BGP Members)

If you want to participate in FadianNet BGP:

| Requirement | Details |
|-------------|---------|
| Own ASN | Public or private ASN |
| BGP daemon | Capable of eBGP peering with regional Route Reflectors |
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

Members are expected to independently deploy and maintain their own infrastructure. You should be comfortable with:

- Linux server administration
- Basic networking (IP addressing, routing, firewalls)
- WireGuard VPN
- RADIUS concepts (realms, proxying, EAP)
