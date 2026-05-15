# 提交申请

FadianRoam 的成员管理通过向联邦仓库提交 GitHub Pull Request 进行。

## 申请流程

### 1. Fork 仓库

在 GitHub 上 Fork [FadianRoam/fadianroam-blueprint](https://github.com/FadianRoam/fadianroam-blueprint)。

### 2. 创建您的成员文件

在 `members/<your-realm>.yml` 路径下添加一个新的 YAML 文件：

```yaml
member:
  name: "Your Organization Name"
  realm: "roam.example.net"
  contact: "admin@example.net"
  joined: null  # filled by maintainers on approval

network:
  type: "vpn-only"  # or "bgp"
  radius_ip: null    # assigned on approval
  mgmt_ip: null      # assigned on approval

  # BGP members only:
  # asn: 65000
  # prefixes:
  #   - "203.0.113.0/24"

radius:
  endpoint: "your-server.example.net"
  port: 1812
  eap_methods:
    - "EAP-TTLS/PAP"

location:
  city: "Your City"
  country: "XX"
  coordinates: [0.0, 0.0]  # optional
```

### 3. 提交 Pull Request

创建 PR 时请包含：

- **标题**: `[Join] roam.example.net`
- **描述**: 简要介绍您的组织及预期用途

### 4. 联邦投票

所有现有 FadianRoam 站点代表通过 GitHub PR 审核对您的申请进行投票：

- 每位成员在 PR 上投 **Approve**（批准）或 **Request Changes**（请求修改）
- 投票期限：**3 天**（或提前达到阈值时截止）
- 批准阈值：**超过 50%** 的现有成员必须批准
- 如获批准，维护者将进行后续入网操作
- 如被拒绝，您将收到说明原因的通知，并可在 30 天后重新提交

!!! info "投票阈值"
    无论联邦规模大小，均适用超过 50% 的规则。2 个成员时，两者都必须批准；3 个成员时，至少 2 个必须批准。

如有疑问或希望在提交前讨论您的申请，请在 [YunZheng HelpCentre](https://helpdesk.yunzheng.space) 提交工单，或加入 [Telegram 群组](https://t.me/+WLLU-KOXcQFiMTg1)。

### 5. 批准与入网

获得批准后：

1. 维护者合并您的 PR，并分配 IP 地址和密钥
2. 您将收到 MGMT VPN 的 WireGuard 配置信息
3. 您需配置 RADIUS 服务器通过中继服务器进行代理转发
4. 执行连通性测试
5. 您的成员记录将更新 `joined` 日期

## 成员文件参考

| 字段 | 必填 | 描述 |
|------|------|------|
| `member.name` | 是 | 组织或个人名称 |
| `member.realm` | 是 | 您的唯一 RADIUS realm |
| `member.contact` | 是 | 管理员联系邮箱 |
| `network.type` | 是 | `bgp` 或 `vpn-only` |
| `radius.endpoint` | 是 | RADIUS 服务器主机名或 IP |
| `radius.port` | 是 | RADIUS 端口（通常为 1812） |
| `radius.eap_methods` | 是 | 支持的 EAP 方法 |
| `location.city` | 是 | 城市名称 |
| `location.country` | 是 | ISO 3166-1 alpha-2 国家代码 |
| `network.asn` | 仅 BGP | 您的 AS 号码 |
| `network.prefixes` | 仅 BGP | 您将宣告的 IP 前缀 |

## 需要帮助？

- **申请前**: 加入 [Telegram 群组](https://t.me/+WLLU-KOXcQFiMTg1) 讨论您的计划
- **审核期间**: 在 [YunZheng HelpCentre](https://helpdesk.yunzheng.space) 提交工单获取技术支持
- **批准后**: 参阅[配置指南](setup.zh.md)获取分步配置说明
