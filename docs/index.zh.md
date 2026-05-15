# FadianRoam

**一个类似 eduroam 的漫游 Wi-Fi 联盟，面向独立网络运营者。**

FadianRoam 使参与者能够部署本地无线接入点，通过任意成员站点对用户进行认证。持有某一成员机构凭证的用户可以在其他任何成员的站点无缝连接——就像 eduroam，但向独立网络运营者开放。

## 工作原理

1. **第一层 — 管理 VPN**: 每个成员的 RADIUS 服务器通过 Federation Relay 连接，实现漫游认证
2. **第二层 — FadianNet**: BGP 站点（拥有自己的 ASN）构成数据骨干网，通过 RPKI 公告共享的 /24 前缀
3. **第三层 — 接入**: 站点通过 VPN 拨号 PPPoE 获取 /32 IP，AP 用户通过 NAT 上网
4. **FadianLink**: BGP 站点为没有 ASN 的接入成员提供连接——无需 BGP 知识即可加入

## FadianNet 与 FadianRoam

- **FadianNet** — BGP 数据骨干网，由 BGP 站点作为**公共基础设施**共同维护
- **FadianRoam** — Wi-Fi 漫游联盟（认证、SSID、用户管理），运行在 FadianNet **之上**

BGP 站点以互助方式互联。FadianRoam 站点在[合理使用政策](governance.zh.md#usage-policy-fair-use)下运营，配备滥用防范机制和可选的商业方案。

## 治理

成员管理通过**民主联盟投票**进行：

- 新成员提交 PR → 现有成员投票 → 需要**超过 50% 批准**
- 每个站点自主管理自己的用户和注册政策
- 随着联盟发展，核心成员将组建[治理委员会](governance.zh.md#fadianroam-governance-council)（发电组织）

## 加入项目

FadianRoam 是一个非营利、社区驱动的项目。无论你是 BGP 运营者还是只想部署一个 AP，都欢迎加入。

[通过 Telegram 加入 :fontawesome-brands-telegram:](https://t.me/+WLLU-KOXcQFiMTg1){ .md-button .md-button--primary }
[提交申请 :fontawesome-solid-paper-plane:](joining/apply.md){ .md-button }

## 快速链接

- [架构概览](architecture/overview.md) — 系统设计与网络层次
- [治理](governance.zh.md) — 联盟模式、投票、合理使用政策
- [加入 FadianRoam](joining/prerequisites.md) — 参与所需条件
- [当前成员](members.zh.md) — 活跃的联盟成员
- [路线图](roadmap.zh.md) — 开发阶段与未来规划

## 项目信息

FadianRoam 是一个开放的非营利联盟。所有配置和成员管理均通过 GitHub 透明运作。

| | |
|---|---|
| **GitHub** | [github.com/FadianRoam](https://github.com/FadianRoam) |
| **联盟仓库** | [FadianRoam/fadianroam-blueprint](https://github.com/FadianRoam/fadianroam-blueprint) |
| **技术支持** | [YunZheng HelpCentre](https://helpdesk.yunzheng.space) |
| **联系方式** | [edward.sun@as204921.net](mailto:edward.sun@as204921.net) |
| **Telegram** | [加入群组](https://t.me/+WLLU-KOXcQFiMTg1) |
| **状态** | 早期开发阶段 |
| **文档** | [fadianroam.yunzheng.space](https://fadianroam.yunzheng.space) |
