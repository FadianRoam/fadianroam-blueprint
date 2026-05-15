# 加入前提条件

在加入 FadianRoam 之前，请确保您满足以下要求。FadianRoam 不限定特定品牌、软件或部署方式——成员可以自由选择自己的解决方案，只要满足以下功能要求即可。

## 必需基础设施

### 1. 支持 802.1X 的 Wi-Fi 接入点

至少一个支持 **WPA2/WPA3-Enterprise** (802.1X) 的 Wi-Fi 接入点：

- 必须支持 RADIUS 认证
- 必须能够配置指向您的 RADIUS 服务器

### 2. RADIUS 服务器

一台具备以下能力的 RADIUS 服务器：

- EAP-TTLS/PAP 认证
- 基于 realm 的代理转发（将非本地 realm 转发至联邦中继服务器）
- 与您的身份提供者集成以验证用户凭据

### 3. 身份提供者 (IDP)

一个可供您的 RADIUS 服务器进行认证的身份提供者：

- 必须能够代表 RADIUS 验证用户凭据
- 具体实现方式（ROPC、LDAP、本地数据库等）由您自行决定

### 4. 服务器

一台用于托管 RADIUS 和 IDP 基础设施的服务器：

- 必须能够与联邦中继服务器建立 WireGuard 隧道
- 必须能够通过 MGMT VPN 接收 RADIUS 流量

### 5. TLS 证书

用于 RADIUS 服务器 EAP 隧道的有效 TLS 证书：

- 必须来自公开受信任的 CA（自签名证书会导致客户端连接失败）
- 强烈建议启用自动续期

### 6. 域名

用作 realm 标识符的域名或子域名：

- 用作 RADIUS realm 后缀（例如 `user@roam.example.net`）
- DNS A 记录需指向您的服务器

## 可选项（BGP 成员）

如果您希望参与 FadianNet BGP：

| 要求 | 详情 |
|------|------|
| 自有 ASN | 公有或私有 ASN |
| BGP 守护进程 | 能够与区域 Route Reflector 建立 eBGP 对等连接 |
| IP 前缀 | 至少一个可路由的前缀用于宣告 |
| 公网 IP | BGP 对等连接所需 |

## 网络要求

| 端口 | 协议 | 方向 | 用途 |
|------|------|------|------|
| WireGuard（例如 51820） | UDP | 出站 | 通过 MGMT VPN 连接联邦中继服务器 |
| 1812 | UDP | 入站 | RADIUS 认证 |
| 1813 | UDP | 入站 | RADIUS 计费 |

!!! tip "防火墙"
    至少需要确保您的 RADIUS 端口（1812/1813 UDP）可从联邦中继服务器的 MGMT VPN IP 访问。如果您是 BGP 成员，FadianNet VPN 可能还需要开放额外端口。

## 技能要求

成员应具备独立部署和维护自身基础设施的能力。您应熟悉以下内容：

- Linux 服务器管理
- 基础网络知识（IP 地址、路由、防火墙）
- WireGuard VPN
- RADIUS 概念（realm、代理转发、EAP）
