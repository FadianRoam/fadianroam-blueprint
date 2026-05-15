# Federation Governance

FadianRoam is governed by its member sites through a democratic federation model.

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
| 4 | Voting period: 7 days (or until threshold is reached) |
| 5 | If **>50%** of existing members approve → application is accepted |
| 6 | Maintainers merge the PR and begin onboarding |

!!! info "Early Stage"
    The >50% threshold applies regardless of federation size. With 2 members, both must approve. With 3 members, at least 2 must approve.

### Rejection

If an application does not reach the approval threshold:

- The applicant is notified with reasons
- The applicant may revise and resubmit after 30 days
- Repeated rejections may be escalated to the Governance Council

## FadianRoam Governance Council

As the federation grows, core members will form the **FadianRoam Governance Council** (发电组织):

- **Composition**: Founding members and long-standing active contributors
- **Role**: Strategic decisions, dispute resolution, commercial policy, billing rules
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

FadianRoam Sites operate under a **fair-use model** by default:

### Default Tier (Free)

- Standard bandwidth allocation per Site
- Reasonable user count and traffic volume
- Non-commercial use
- No additional cost — covered by the collective FadianNet backbone

### Elevated Use

If a FadianRoam Site requires resources beyond the default tier — due to high user count, heavy traffic, or commercial operations — the following process applies:

| Step | Action |
|------|--------|
| 1 | Site submits a request ticket to [YunZheng HelpCentre](https://helpdesk.yunzheng.space) |
| 2 | Request is reviewed by the Governance Council (or maintainers in early stage) |
| 3 | Council assesses impact on FadianNet infrastructure |
| 4 | If approved, a commercial plan or cost-sharing arrangement is agreed |
| 5 | Site operates under the approved terms |

Examples of elevated use:

- Large number of registered users (e.g., >100 active users)
- Sustained high bandwidth consumption
- Commercial Wi-Fi service built on FadianRoam
- Reselling or sublicensing access

### Commercial Use

Sites operating FadianRoam for commercial purposes must:

1. Disclose commercial intent in their membership application
2. Obtain Governance Council approval
3. Agree to a cost-sharing or licensing arrangement
4. Comply with any branding and service-level requirements

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
       └── Dispute / Escalation → Governance Council
```
