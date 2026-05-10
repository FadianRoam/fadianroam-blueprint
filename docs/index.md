# FadianRoam

**An eduroam-like roaming Wi-Fi federation for independent networks.**

FadianRoam enables participants to deploy local wireless access points that authenticate users from any member site. A user with credentials at one member institution can seamlessly connect at any other member's location — just like eduroam, but open to independent network operators.

## How It Works

1. **Layer 1 — MGMT VPN**: Each member's RADIUS server connects to the Federation Relay for roaming authentication
2. **Layer 2 — FadianNet**: BGP Sites (with own ASN) form a data backbone, announcing a shared /24 prefix with RPKI
3. **Layer 3 — Access**: Sites dial PPPoE over VPN to get a /32 IP, NAT AP users behind it
4. **FadianLink**: BGP Sites extend connectivity to Access Members without ASN — no BGP knowledge needed to join

## Join the Project

FadianRoam is a non-profit, community-driven project. Whether you're a BGP operator or just want to deploy an AP, you're welcome to join.

[Join via Telegram :fontawesome-brands-telegram:](https://t.me/+WLLU-KOXcQFiMTg1){ .md-button .md-button--primary }

## Quick Links

- [Architecture Overview](architecture/overview.md) — Understand the system design
- [Join FadianRoam](joining/prerequisites.md) — What you need to participate
- [Submit Application](joining/apply.md) — Apply via GitHub Pull Request
- [Current Members](members.md) — Active federation members

## Project

FadianRoam is an open, non-profit federation. All configuration and membership is managed transparently via GitHub.

| | |
|---|---|
| **GitHub** | [github.com/FadianRoam](https://github.com/FadianRoam) |
| **Federation Repo** | [FadianRoam/fadianroam-blueprint](https://github.com/FadianRoam/fadianroam-blueprint) |
| **Contact** | [edward.sun@as204921.net](mailto:edward.sun@as204921.net) |
| **Telegram** | [Join Group](https://t.me/+WLLU-KOXcQFiMTg1) |
| **Status** | Early Development |
| **Docs** | [fadianroam.yunzheng.space](https://fadianroam.yunzheng.space) |
