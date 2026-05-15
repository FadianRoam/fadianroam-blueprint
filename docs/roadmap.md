# Roadmap

FadianRoam development is organized into phases, progressing from basic federation to a fully governed network with billing and commercial support.

## Phase 1 — Foundation (Current)

**Status**: In Progress

- [x] Federation Relay (RADIUS proxy) deployed
- [x] MGMT VPN (WireGuard star topology) operational
- [x] First member site (YunZheng Lab) online
- [x] EAP-TTLS/PAP authentication with Keycloak ROPC
- [x] Blueprint documentation published
- [ ] Second member site onboarded
- [ ] Cross-site roaming verified end-to-end

**Governance**: Maintainer-driven. Membership approval by existing members via GitHub PR voting (>50% threshold).

**Cost model**: Free. All infrastructure costs absorbed by founding members.

## Phase 2 — Growth

- [ ] 3+ active member sites
- [ ] RADIUS Accounting (UDP 1813) enabled for traffic statistics
- [ ] Per-site usage dashboards
- [ ] Fair-use baseline defined (bandwidth, user count thresholds)
- [ ] Automated member onboarding (CI/CD generates Relay config on PR merge)
- [ ] FadianLink operational (Access Members via BGP Site sponsorship)

**Governance**: Formal voting process. Governance Council formation begins.

**Cost model**: Free tier with fair-use limits. Elevated use requests handled via [YunZheng HelpCentre](https://helpdesk.yunzheng.space).

## Phase 3 — Maturity

- [ ] Federation billing system for cross-site roaming accounting
- [ ] Inter-site traffic settlement
- [ ] Commercial use framework and licensing
- [ ] Multiple Federation Relays for redundancy
- [ ] Regional Route Reflectors deployed
- [ ] IPv6 support
- [ ] Shared /24 prefix with RPKI (sponsored)
- [ ] Uptime monitoring and SLA for federation infrastructure

**Governance**: Governance Council (发电组织) fully operational. Core members define policies, resolve disputes, manage commercial agreements.

**Cost model**: Tiered. Free tier for community use, paid plans for commercial or high-volume sites. Revenue funds FadianNet backbone maintenance.
