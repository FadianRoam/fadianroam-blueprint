# FadianRoam

**An eduroam-like roaming Wi-Fi federation for independent networks.**

FadianRoam enables participants to deploy local wireless access points that authenticate users from any member site. A user with credentials at one member institution can seamlessly connect at any other member's location — just like eduroam, but open to independent network operators.

## How It Works

1. Each member runs their own **Identity Provider** (IDP) and **RADIUS server** with a unique realm (e.g., `@roam.example.net`)
2. A central **Federation Relay** routes RADIUS authentication requests between members
3. Members interconnect via **FadianNet VPN**, which carries both management traffic and user data
4. Optional **BGP integration** allows members to contribute transit and build a shared internet backbone

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
| **Status** | Early Development |
| **Docs** | [fadianroam.yunzheng.space](https://fadianroam.yunzheng.space) |
