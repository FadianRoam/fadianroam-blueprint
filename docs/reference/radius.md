# RADIUS Configuration

Detailed reference for FreeRADIUS configuration in a FadianRoam deployment.

## Overview

Each member runs a FreeRADIUS instance that:

1. Accepts 802.1X authentication from local APs
2. Validates local users against Keycloak via ROPC
3. Proxies non-local realm requests to the Federation Relay

## Module Configuration

### EAP Module (`mods-enabled/eap`)

```
eap {
    default_eap_type = ttls
    ignore_unknown_eap_types = no
    cisco_accounting_username_bug = no
    max_sessions = ${max_requests}

    tls-config tls-common {
        private_key_file = /etc/freeradius/3.0/certs/<your-domain>.key
        certificate_file = /etc/freeradius/3.0/certs/<your-domain>.cer
        ca_file = /etc/freeradius/3.0/certs/<your-domain>.ca.cer
        tls_min_version = "1.2"
        tls_max_version = "1.3"
        cipher_list = "DEFAULT"
        ecdh_curve = "prime256v1"
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

**Key points:**

- Use EAP-TTLS as default — it creates a TLS tunnel for inner PAP authentication
- TLS 1.2 minimum — TLS 1.0/1.1 are deprecated
- Certificate must be from a publicly trusted CA

### REST Module (`mods-enabled/rest`)

```
rest {
    connect_uri = "http://127.0.0.1:8080"
    connect_timeout = 5.0

    authorize {
        uri = "${..connect_uri}/realms/<realm>/protocol/openid-connect/token"
        method = 'post'
        body = 'post'
        data = "client_id=freeradius&client_secret=<SECRET>&grant_type=password&username=%{%{Stripped-User-Name}:-%{User-Name}}&password=%{User-Password}&scope=openid"
        force_to = 'plain'
        tls = {}
    }

    authenticate {
        uri = "${..connect_uri}/realms/<realm>/protocol/openid-connect/token"
        method = 'post'
        body = 'post'
        data = "client_id=freeradius&client_secret=<SECRET>&grant_type=password&username=%{%{Stripped-User-Name}:-%{User-Name}}&password=%{User-Password}&scope=openid"
        force_to = 'plain'
        tls = {}
    }
}
```

**Key points:**

- `Stripped-User-Name` is used when available (after realm stripping), falls back to `User-Name`
- Keycloak must have a client with **Direct Access Grants** enabled
- `connect_uri` points to Keycloak's local address (not through reverse proxy)

### Suffix Module (`mods-enabled/suffix`)

The default `suffix` module handles realm stripping. No changes needed:

```
realm suffix {
    format = suffix
    delimiter = "@"
}
```

This strips `@realm.example.net` from `user@realm.example.net`, setting `Stripped-User-Name = user`.

## Virtual Server Configuration

### Default Site (`sites-enabled/default`)

Critical sections:

```
authorize {
    filter_username
    preprocess
    suffix          # strips realm, enables proxy routing

    # Set Auth-Type for PAP when password is present
    if (&User-Password) {
        update control {
            Auth-Type := PAP
        }
    }

    eap {
        ok = return
    }
}

authenticate {
    Auth-Type PAP {
        rest        # validate via Keycloak ROPC
    }

    eap
}
```

!!! warning "Do not use the `pap` module"
    The default `pap` module expects a locally stored password hash. Since passwords are validated by Keycloak, use the `rest` module directly in `Auth-Type PAP`.

### Inner Tunnel (`sites-enabled/inner-tunnel`)

For EAP-TTLS inner authentication:

```
authorize {
    filter_username
    suffix

    if (&User-Password) {
        update control {
            Auth-Type := PAP
        }
    }

    eap {
        ok = return
    }
}

authenticate {
    Auth-Type PAP {
        rest
    }

    eap
}
```

## Proxy Configuration (`proxy.conf`)

### Local Realm

```
realm your-realm.example.net {
    # Empty — handled locally
}
```

### Federation Proxy

```
realm DEFAULT {
    type = radius
    authhost = 10.250.0.1:1812
    accthost = 10.250.0.1:1813
    secret = <federation-shared-secret>
    nostrip
}
```

The `DEFAULT` realm catches all non-local realms and forwards them to the Federation Relay over the MGMT VPN.

`nostrip` preserves the full `user@realm` so the Relay can route to the correct member.

## Client Configuration (`clients.conf`)

### Local AP Client

```
client wifi-ap {
    ipaddr = 192.168.1.0/24   # Your AP subnet
    secret = <ap-radius-secret>
    shortname = local-ap
}
```

### Federation Relay Client

```
client federation-relay {
    ipaddr = 10.250.0.1
    secret = <federation-shared-secret>
    shortname = fadianroam-relay
}
```

## Testing

### Local Authentication

```bash
radtest user@your-realm.example.net PASSWORD localhost 0 testing123
```

### Debug Mode

```bash
# Stop the service first
systemctl stop freeradius

# Run in debug mode
freeradius -X

# Watch the output for:
# - Realm stripping (suffix module)
# - Auth-Type selection
# - REST module HTTP response
# - Final Accept/Reject
```

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| `No "known good" password` | `pap` module in authorize | Replace with `if (&User-Password)` block |
| `rest: 401 Unauthorized` | Wrong client secret or user credentials | Check Keycloak client config |
| `TLS Alert: unknown CA` | Self-signed or missing CA cert | Use a publicly trusted certificate |
| Realm not proxied | Missing `suffix` in authorize | Enable `suffix` module |
| Relay unreachable | MGMT VPN down | Check WireGuard with `wg show` |
