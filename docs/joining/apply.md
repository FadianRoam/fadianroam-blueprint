# Submit Application

FadianRoam membership is managed through GitHub Pull Requests to the federation repository.

## Application Process

### 1. Fork the Repository

Fork [FadianRoam/fadianroam-blueprint](https://github.com/FadianRoam/fadianroam-blueprint) on GitHub.

### 2. Create Your Member File

Add a new YAML file at `members/<your-realm>.yml`:

```yaml
member:
  name: "Your Organization Name"
  realm: "roam.example.net"
  contact: "admin@example.net"
  joined: null  # filled by maintainers on approval

network:
  type: "vpn-only"  # or "bgp"
  radius_ip: null    # assigned on approval
  mgmt_ip: null      # assigned on approval

  # BGP members only:
  # asn: 65000
  # prefixes:
  #   - "203.0.113.0/24"

radius:
  endpoint: "your-server.example.net"
  port: 1812
  eap_methods:
    - "EAP-TTLS/PAP"

location:
  city: "Your City"
  country: "XX"
  coordinates: [0.0, 0.0]  # optional
```

### 3. Submit Pull Request

Create a PR with:

- **Title**: `[Join] roam.example.net`
- **Description**: Brief introduction of your organization and intended use

### 4. Review

Existing members and maintainers will review your application:

- Verify your infrastructure meets [prerequisites](prerequisites.md)
- Check realm name uniqueness
- Assign MGMT VPN IP and RADIUS shared secret
- Test connectivity after VPN setup

### 5. Approval & Onboarding

Once approved:

1. Maintainers merge your PR with assigned IPs and secrets
2. You receive WireGuard configuration for the MGMT VPN
3. You configure your RADIUS server to proxy through the Relay
4. A connectivity test is performed
5. Your member entry is updated with `joined` date

## Member File Reference

| Field | Required | Description |
|-------|----------|-------------|
| `member.name` | Yes | Organization or individual name |
| `member.realm` | Yes | Your unique RADIUS realm |
| `member.contact` | Yes | Admin contact email |
| `network.type` | Yes | `bgp` or `vpn-only` |
| `radius.endpoint` | Yes | RADIUS server hostname or IP |
| `radius.port` | Yes | RADIUS port (usually 1812) |
| `radius.eap_methods` | Yes | Supported EAP methods |
| `location.city` | Yes | City name |
| `location.country` | Yes | ISO 3166-1 alpha-2 country code |
| `network.asn` | BGP only | Your AS number |
| `network.prefixes` | BGP only | IP prefixes you will announce |

## After Approval

See the [Setup Guide](setup.md) for step-by-step configuration instructions.
