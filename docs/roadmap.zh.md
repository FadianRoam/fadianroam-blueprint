# 路线图

FadianRoam 的开发按阶段组织，从基础联盟逐步发展为具有完善治理、计费和商业支持的成熟网络。

## 第一阶段 — 基础建设（当前）

**状态**: 进行中

- [x] Federation Relay（RADIUS 代理）已部署
- [x] 管理 VPN（WireGuard 星型拓扑）已运行
- [x] 首个成员站点（YunZheng Lab）已上线
- [x] EAP-TTLS/PAP 认证与 Keycloak ROPC 已对接
- [x] Blueprint 文档已发布
- [ ] 第二个成员站点引导入驻
- [ ] 跨站点漫游端到端验证

**治理**: 维护者主导。成员通过 GitHub PR 投票（>50% 门槛）审批加入。

**成本模式**: 免费。所有基础设施费用由创始成员承担。

## 第二阶段 — 增长

- [ ] 3 个以上活跃成员站点
- [ ] 启用 RADIUS Accounting（UDP 1813）进行流量统计
- [ ] 各站点使用量仪表盘
- [ ] 定义合理使用基线（带宽、用户数量阈值）
- [ ] 自动化成员引导（PR 合并后 CI/CD 自动生成 Relay 配置）
- [ ] FadianLink 运营（接入成员通过 BGP 站点赞助接入）

**治理**: 正式投票流程。发电委员会开始组建。

**成本模式**: 免费层级附带合理使用限制。超额使用请求通过 [YunZheng HelpCentre](https://helpdesk.yunzheng.space) 处理。

## 第三阶段 — 成熟

- [ ] 联盟计费系统，用于跨站点漫游核算
- [ ] 站间流量结算
- [ ] 商业使用框架与许可
- [ ] 多个 Federation Relay 实现冗余
- [ ] 部署区域 Route Reflector
- [ ] IPv6 支持
- [ ] 共享 /24 前缀与 RPKI（赞助）
- [ ] 联盟基础设施的运行监控与 SLA

**治理**: 发电委员会（FadianRoam Governance Committee）全面运作。核心成员制定政策、解决争端、管理商业协议。

**成本模式**: 分层。社区使用免费层级，商业或高流量站点付费方案。收入用于 FadianNet 骨干网维护。
