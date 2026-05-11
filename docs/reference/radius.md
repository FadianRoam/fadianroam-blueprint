# RADIUS Configuration

Reference for member RADIUS server requirements in a FadianRoam deployment.

## Overview

Each member runs a RADIUS server that:

1. Accepts 802.1X authentication from local APs
2. Validates local users against the member's Identity Provider
3. Proxies non-local realm requests to the Federation Relay

## Functional Requirements

### EAP

- **EAP-TTLS/PAP** must be supported as the default method
- TLS 1.2 minimum for the outer EAP tunnel
- Certificate must be from a publicly trusted CA

### IDP Integration

The RADIUS server must validate credentials against the member's IDP. The specific integration method is up to each member. The key behavior:

- After realm stripping, the RADIUS server receives a bare username and password
- These credentials are forwarded to the IDP for validation
- IDP returns success or failure
- RADIUS responds with Access-Accept or Access-Reject

### Realm Handling

The `suffix` module (or equivalent) must be configured for realm-based routing:

- Parse the realm from `user@realm.example.net`
- Strip the realm suffix for local authentication
- Route non-local realms to the Federation Relay via proxy

## Proxy Configuration

### Local Realm

Your own realm should be handled locally:

```
realm your-realm.example.net {
    # Handled locally
}
```

### Federation Proxy

All unknown realms must be forwarded to the Federation Relay:

```
realm DEFAULT {
    type = radius
    authhost = 172.172.10.1:1812
    accthost = 172.172.10.1:1813
    secret = <federation-shared-secret>
    nostrip
}
```

`nostrip` preserves the full `user@realm` so the Relay can route to the correct member.

## Client Configuration

### Local AP Client

Your APs must be registered as RADIUS clients:

```
client wifi-ap {
    ipaddr = <your-ap-subnet>
    secret = <ap-radius-secret>
}
```

### Federation Relay Client

The Federation Relay must be allowed to send requests to your RADIUS (for roaming users whose home realm is yours):

```
client federation-relay {
    ipaddr = 172.172.10.1
    secret = <federation-shared-secret>
}
```

## Testing

### Local Authentication

```bash
radtest user@your-realm.example.net PASSWORD localhost 0 testing123
```

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| `TLS Alert: unknown CA` | Self-signed or missing CA cert | Use a publicly trusted certificate |
| Realm not proxied | Realm stripping not configured | Ensure suffix module (or equivalent) is enabled |
| Relay unreachable | MGMT VPN down | Check WireGuard with `wg show` |
| IDP validation fails | Wrong credentials or IDP config | Check IDP integration independently |
