# Federation Governance

FadianRoam is governed by its member sites through a democratic federation model.

## Terminology

| Term | Chinese | Definition |
|------|---------|------------|
| **FadianRoam** | 发电漫游 | The roaming authentication federation — handles 802.1X, RADIUS proxy, SSID branding, and user management |
| **FadianNet** | 发电网络 | The BGP data backbone that provides network transport for FadianRoam |
| **Site** | 站点 | A complete deployment unit at a single location, consisting of the network endpoint (RADIUS server, gateway/router), local infrastructure (APs, switches, VLANs), and the user devices connecting through it |
| **Governance Committee** | 发电委员会 | The committee responsible for federation operations, strategic decisions, dispute resolution, and policy |

## Principles

- **FadianNet is a public good** — BGP Sites collectively maintain the backbone infrastructure on a non-profit, mutual-aid basis
- **FadianRoam is the business layer** — Wi-Fi authentication, SSID branding, and user management operate on top of FadianNet
- **Each Site is autonomous** — members control their own users, registration policy, and local infrastructure
- **Decisions are collective** — membership changes require majority approval from existing members

## Membership Admission

### Application

A prospective member submits their application by opening a Pull Request to the [federation repository](https://github.com/FadianRoam/fadianroam-blueprint) with their `members/<realm>.yml` file. The application must include:

- Organization or individual name
- RADIUS realm and server details
- Network type (BGP Site or Access Member)
- Contact information
- Intended use (community, educational, commercial, etc.)

### Voting

All existing FadianRoam Site representatives vote on the application:

| Step | Action |
|------|--------|
| 1 | Applicant submits PR with member YAML |
| 2 | Existing members review the application |
| 3 | Each member votes via GitHub PR review: **Approve** or **Request Changes** |
| 4 | Voting period: 3 days (or until threshold is reached) |
| 5 | If **>50%** of existing members approve → application is accepted |
| 6 | Maintainers merge the PR and begin onboarding |

!!! info "Early Stage"
    The >50% threshold applies regardless of federation size. With 2 members, both must approve. With 3 members, at least 2 must approve.

### Rejection

If an application does not reach the approval threshold:

- The applicant is notified with reasons
- The applicant may revise and resubmit after 30 days
- Repeated rejections may be escalated to the Governance Committee

## FadianRoam Governance Committee

As the federation grows, core members will form the **FadianRoam Governance Committee** (发电委员会):

- **Composition**: Founding members and long-standing active contributors
- **Role**: Federation operations, strategic decisions, dispute resolution, commercial policy, billing rules
- **Formation**: Established once the federation reaches a critical mass of active members

## FadianNet vs FadianRoam

FadianNet and FadianRoam are conceptually separate:

| | FadianNet | FadianRoam |
|---|---|---|
| **Purpose** | BGP data backbone | Wi-Fi roaming federation |
| **Participants** | BGP Sites (with ASN) | All Sites (BGP + Access) |
| **Nature** | Public good, mutual-aid | Business layer with usage policies |
| **Maintenance** | Collectively by BGP Sites | Governed by federation rules |
| **Traffic** | User data transport | 802.1X authentication + SSID |
| **Cost model** | Shared equally among BGP Sites | Usage-based, with fair-use limits |

FadianRoam runs **on top of** FadianNet — the SSID, authentication, and project identity belong to FadianRoam, while the actual data transport is carried by FadianNet.

## Usage Policy & Fair Use

The spirit of FadianRoam is **Fadian (发电)** — powering your own use and sharing with each other. The federation billing system is **not about real charges** — it defines a reasonable hobby-use baseline and ensures the sustainability of shared infrastructure.

### Billing System

Once the Governance Committee is established, a federation billing system will be introduced:

- The billing system **tracks usage**, it does not directly charge fees
- The federation defines a **reasonable hobby-use baseline** for each Site (bandwidth, user count, traffic, etc.)
- Usage within the baseline is completely free, covered by the collective FadianNet backbone
- Billing data is used for fair-use monitoring and federation transparency

### Exceeding the Baseline

If a Site's usage exceeds the hobby-use baseline, the following process applies:

| Step | Action |
|------|--------|
| 1 | The Site receives an over-baseline notification |
| 2 | Site submits an explanation ticket to [YunZheng HelpCentre](https://helpdesk.yunzheng.space) |
| 3 | **Explain the reason**: temporary spike or sustained need |
| 4 | Governance Committee (or maintainers in early stage) reviews |
| 5 | Approved or adjustment requested based on the situation |

### Special Scenario Approval

The following scenarios can be approved for temporary over-baseline usage at no additional cost:

- **FadianRoam's own offline meetups** (member meetups, technical exchange events, etc.)
- **Offline events in collaboration with FadianRoam** (exhibitions, demos, community events, etc.)
- **Temporary testing and debugging** (new Site onboarding, stress testing, etc.)

To apply: submit a ticket to [YunZheng HelpCentre](https://helpdesk.yunzheng.space) in advance with event details and estimated usage.

### Commercial Use

If a Site needs to use FadianRoam for commercial purposes (sustained high usage, paid services, etc.):

1. Disclose commercial intent in the membership application
2. Obtain Governance Committee approval
3. Bear the corresponding network costs
4. Comply with branding and service-level requirements

## Support & Tickets

All federation support requests, applications, and escalations are handled through:

| Channel | Purpose |
|---------|---------|
| [YunZheng HelpCentre](https://helpdesk.yunzheng.space) | Tickets: membership applications, elevated use requests, technical support, disputes |
| [GitHub Issues](https://github.com/FadianRoam/fadianroam-blueprint/issues) | Bug reports, feature requests, documentation improvements |
| [Telegram Group](https://t.me/+WLLU-KOXcQFiMTg1) | Informal discussion, quick questions, community chat |

### Ticket Workflow

```
Applicant / Member
       │
       ▼
  Submit ticket to YunZheng HelpCentre
       │
       ├── Membership application → Voting process (GitHub PR)
       ├── Elevated use request → Council review → Approval / Commercial plan
       ├── Technical support → Maintainer assistance
       └── Dispute / Escalation → Governance Committee
```
