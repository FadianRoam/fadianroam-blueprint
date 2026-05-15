# 联盟治理

FadianRoam 由其成员站点通过民主联盟模式进行治理。

## 原则

- **FadianNet 是公共基础设施** — BGP 站点以非营利、互助方式共同维护骨干基础设施
- **FadianRoam 是业务层** — Wi-Fi 认证、SSID 品牌和用户管理运行在 FadianNet 之上
- **每个站点自治** — 成员控制自己的用户、注册政策和本地基础设施
- **集体决策** — 成员变更需要现有成员多数批准

## 成员准入

### 申请

申请者通过向[联盟仓库](https://github.com/FadianRoam/fadianroam-blueprint)提交 Pull Request，提交其 `members/<realm>.yml` 文件来申请加入。申请必须包含：

- 组织或个人名称
- RADIUS realm 和服务器信息
- 网络类型（BGP 站点或接入成员）
- 联系方式
- 用途说明（社区、教育、商业等）

### 投票

所有现有 FadianRoam 站点代表对申请进行投票：

| 步骤 | 操作 |
|------|------|
| 1 | 申请者提交包含成员 YAML 的 PR |
| 2 | 现有成员审核申请 |
| 3 | 每位成员通过 GitHub PR review 投票：**Approve** 或 **Request Changes** |
| 4 | 投票期限：7 天（或达到门槛时结束） |
| 5 | 如果现有成员中 **>50%** 批准 → 申请通过 |
| 6 | 维护者合并 PR 并开始引导入驻 |

!!! info "早期阶段"
    无论联盟规模大小，均适用 >50% 门槛。2 个成员时，双方都须批准。3 个成员时，至少 2 个须批准。

### 驳回

如果申请未达到批准门槛：

- 申请者将收到附带原因的通知
- 申请者可在 30 天后修改并重新提交
- 多次被驳回的申请可升级至治理委员会处理

## FadianRoam 治理委员会 { #fadianroam-governance-council }

随着联盟发展，核心成员将组建 **FadianRoam 治理委员会**（发电组织）：

- **组成**: 创始成员和长期活跃贡献者
- **职责**: 战略决策、争端解决、商业政策、计费规则
- **成立条件**: 联盟达到一定规模的活跃成员时成立

## FadianNet 与 FadianRoam

FadianNet 和 FadianRoam 在概念上是分离的：

| | FadianNet | FadianRoam |
|---|---|---|
| **用途** | BGP 数据骨干网 | Wi-Fi 漫游联盟 |
| **参与者** | BGP 站点（拥有 ASN） | 所有站点（BGP + 接入） |
| **性质** | 公共基础设施，互助 | 业务层，含使用政策 |
| **维护** | BGP 站点共同维护 | 由联盟规则治理 |
| **流量** | 用户数据传输 | 802.1X 认证 + SSID |
| **成本模式** | BGP 站点平均分摊 | 基于用量，含合理使用限制 |

FadianRoam 运行在 FadianNet **之上** — SSID、认证和项目标识属于 FadianRoam，而实际的数据传输由 FadianNet 承载。

## 使用政策与合理使用 { #usage-policy-fair-use }

FadianRoam 站点默认在**合理使用模式**下运营：

### 默认层级（免费）

- 每个站点的标准带宽分配
- 合理的用户数量和流量额度
- 非商业用途
- 无额外费用 — 由 FadianNet 骨干网集体承担

### 超额使用

如果 FadianRoam 站点需要超出默认层级的资源——由于用户数量多、流量大或商业运营——适用以下流程：

| 步骤 | 操作 |
|------|------|
| 1 | 站点向 [YunZheng HelpCentre](https://helpdesk.yunzheng.space) 提交请求工单 |
| 2 | 请求由治理委员会（或早期阶段的维护者）审核 |
| 3 | 委员会评估对 FadianNet 基础设施的影响 |
| 4 | 如获批准，商定商业方案或成本分摊安排 |
| 5 | 站点在批准条款下运营 |

超额使用的示例：

- 大量注册用户（例如超过 100 名活跃用户）
- 持续的高带宽消耗
- 基于 FadianRoam 的商业 Wi-Fi 服务
- 转售或分许可接入

### 商业使用

以商业目的运营 FadianRoam 的站点必须：

1. 在成员申请中披露商业意图
2. 获得治理委员会批准
3. 同意成本分摊或许可安排
4. 遵守品牌和服务级别要求

## 支持与工单

所有联盟支持请求、申请和升级事项通过以下渠道处理：

| 渠道 | 用途 |
|------|------|
| [YunZheng HelpCentre](https://helpdesk.yunzheng.space) | 工单：成员申请、超额使用请求、技术支持、争议处理 |
| [GitHub Issues](https://github.com/FadianRoam/fadianroam-blueprint/issues) | Bug 报告、功能请求、文档改进 |
| [Telegram 群组](https://t.me/+WLLU-KOXcQFiMTg1) | 非正式讨论、快速提问、社区交流 |

### 工单流程

```
申请者 / 成员
       │
       ▼
  向 YunZheng HelpCentre 提交工单
       │
       ├── 成员申请 → 投票流程（GitHub PR）
       ├── 超额使用请求 → 委员会审核 → 批准 / 商业方案
       ├── 技术支持 → 维护者协助
       └── 争议 / 升级 → 治理委员会
```
