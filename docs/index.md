# FadianRoam

**An eduroam-like roaming Wi-Fi federation for independent networks.**

FadianRoam enables participants to deploy local wireless access points that authenticate users from any member site. A user with credentials at one member institution can seamlessly connect at any other member's location — just like eduroam, but open to independent network operators.

## How It Works

1. **Layer 1 — MGMT VPN**: Each member's RADIUS server connects to the Federation Relay for roaming authentication
2. **Layer 2 — FadianNet**: BGP Sites (with own ASN) form a data backbone, announcing a shared /24 prefix with RPKI
3. **Layer 3 — Access**: Sites dial PPPoE over VPN to get a /32 IP, NAT AP users behind it
4. **FadianLink**: BGP Sites extend connectivity to Access Members without ASN — no BGP knowledge needed to join

## FadianNet vs FadianRoam

- **FadianNet** — The BGP data backbone, collectively maintained by BGP Sites as a **public good**
- **FadianRoam** — The Wi-Fi roaming federation (authentication, SSID, user management), running **on top of** FadianNet

BGP Sites interconnect on a mutual-aid basis. FadianRoam Sites operate under [fair-use policies](governance.md#usage-policy-fair-use) with abuse prevention and optional commercial plans.

## Governance

Membership is managed through **democratic federation voting**:

- New members submit a PR → existing members vote → **>50% approval** required
- Each Site autonomously manages its own users and registration policy
- As the federation grows, a [Governance Committee](governance.md#fadianroam-governance-committee) (发电委员会) will be formed by core members

## Join the Project

FadianRoam is a non-profit, community-driven project. Whether you're a BGP operator or just want to deploy an AP, you're welcome to join.

[Join via Telegram :fontawesome-brands-telegram:](https://t.me/+WLLU-KOXcQFiMTg1){ .md-button .md-button--primary }
[Submit Application :fontawesome-solid-paper-plane:](joining/apply.md){ .md-button }

## Quick Links

- [Architecture Overview](architecture/overview.md) — System design and network layers
- [Governance](governance.md) — Federation model, voting, fair-use policy
- [Join FadianRoam](joining/prerequisites.md) — What you need to participate
- [Current Members](members.md) — Active federation members
- [Roadmap](roadmap.md) — Development phases and future plans

## Project

FadianRoam is an open, non-profit federation. All configuration and membership is managed transparently via GitHub.

| | |
|---|---|
| **GitHub** | [github.com/FadianRoam](https://github.com/FadianRoam) |
| **Federation Repo** | [FadianRoam/fadianroam-blueprint](https://github.com/FadianRoam/fadianroam-blueprint) |
| **Support** | [YunZheng HelpCentre](https://helpdesk.yunzheng.space) |
| **Contact** | [edward.sun@as204921.net](mailto:edward.sun@as204921.net) |
| **Telegram** | [Join Group](https://t.me/+WLLU-KOXcQFiMTg1) |
| **Status** | Early Development |
| **Docs** | [fadianroam.yunzheng.space](https://fadianroam.yunzheng.space) |
