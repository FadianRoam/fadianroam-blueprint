# Federation Relay

Federation Relay 是 FadianRoam 中唯一的共享基础设施，负责在成员之间代理 RADIUS 认证请求。

## 角色

Relay 充当中心 RADIUS 代理：

```mermaid
graph LR
    RA[Member A<br/>RADIUS] -->|user@realm.b| RELAY[Federation Relay<br/>RADIUS Proxy]
    RELAY -->|forward| RB[Member B<br/>RADIUS]
    RB -->|Access-Accept| RELAY
    RELAY -->|Access-Accept| RA
```

- 通过 MGMT VPN 接收来自成员的 RADIUS 请求
- 查找 realm 以确定目标成员
- 将请求转发至正确成员的 RADIUS
- 返回响应

Relay **不会**：

- 存储任何用户凭据
- 检查内层 EAP 负载（在 TLS 隧道中加密）
- 参与用户数据流量
- 运行 IDP

## 架构

### 组件

| 组件 | 用途 |
|------|------|
| RADIUS proxy | RADIUS 代理引擎 |
| WireGuard | MGMT VPN 中心节点（星形拓扑） |
| 联盟配置 | Realm → 成员 IP 映射 |

### 网络

Relay 是 MGMT VPN 星形拓扑的中心节点：

- IP：`172.172.10.1`
- 在 MGMT 子网内监听 RADIUS 端口 1812/1813
- 每个成员在 Relay 上都有一个 WireGuard peer 条目

## RADIUS 代理配置

### Realm 定义

Relay 为每个成员维护一个 realm 条目，由联盟注册表生成：

```
# proxy.conf on Federation Relay

# Member A
realm roam.member-a.net {
    type = radius
    authhost = 172.172.10.10:1812
    accthost = 172.172.10.10:1813
    secret = <member-a-shared-secret>
    nostrip
}

# Member B
realm roam.member-b.org {
    type = radius
    authhost = 172.172.10.11:1812
    accthost = 172.172.10.11:1813
    secret = <member-b-shared-secret>
    nostrip
}

# Reject unknown realms
realm DEFAULT {
    reject = yes
}
```

**关键设置：**

- `nostrip`：保留完整的 `user@realm` 格式，使目标 RADIUS 可以正确处理
- `DEFAULT` realm 拒绝未知 realm（仅代理已注册的成员）
- 每个成员拥有独立的共享密钥

### 客户端定义

每个成员被定义为 RADIUS 客户端：

```
# clients.conf on Federation Relay

client member-a {
    ipaddr = 172.172.10.10
    secret = <member-a-shared-secret>
    shortname = member-a
}

client member-b {
    ipaddr = 172.172.10.11
    secret = <member-b-shared-secret>
    shortname = member-b
}
```

### 虚拟服务器

Relay 运行一个最小化的虚拟服务器 — 仅负责代理，不进行本地认证：

```
# sites-enabled/default on Relay

authorize {
    preprocess
    suffix
    # No local auth — all requests are proxied via realm routing
}

authenticate {
    # Empty — proxy handles authentication
}
```

## WireGuard 中心节点配置

```ini
# /etc/wireguard/fadianroam-mgmt.conf on Relay

[Interface]
Address = 172.172.10.1/24
PrivateKey = <relay-private-key>
ListenPort = 51820

[Peer]
# Member A
PublicKey = <member-a-pubkey>
AllowedIPs = 172.172.10.10/32

[Peer]
# Member B
PublicKey = <member-b-pubkey>
AllowedIPs = 172.172.10.11/32

# ... one [Peer] per member
```

## 配置管理

Relay 的配置来源于联盟注册表（`members/*.yml` 文件）：

1. 成员提交包含 `members/<realm>.yml` 的 PR
2. PR 经审核并批准
3. 合并后，Relay 配置自动重新生成：
    - `proxy.conf`：新增 realm 条目
    - `clients.conf`：新增客户端条目
    - WireGuard：新增 peer 条目
4. 服务重新加载

可以通过 CI/CD 实现自动化：

```bash
# Example: regenerate and reload
./scripts/generate-relay-config.sh
# reload RADIUS proxy service
wg syncconf fadianroam-mgmt <(wg-quick strip fadianroam-mgmt)
```

## 部署建议

| 方面 | 建议 |
|------|------|
| 服务器 | 专用 VPS（独立于任何成员的基础设施） |
| 位置 | 对大多数成员延迟最低的区域 |
| 配置 | 1 vCPU，1 GB RAM（轻量代理负载） |
| 操作系统 | Linux |
| 冗余 | 未来计划：多个 Relay 共享 realm 配置 |

## 监控

### RADIUS

```bash
# Check RADIUS proxy is running
# Verify proxy decisions in logs:
# - Realm lookup successful
# - Request proxied to correct member
# - Response forwarded back
```

### WireGuard

```bash
# All peer handshakes
wg show fadianroam-mgmt latest-handshakes

# Identify members with stale handshakes (>2 minutes)
wg show fadianroam-mgmt latest-handshakes | awk '$2 > 120 {print $1}'
```

### 健康检查

```bash
# Ping all members
for ip in $(wg show fadianroam-mgmt allowed-ips | awk '{print $2}' | cut -d/ -f1); do
    ping -c 1 -W 2 $ip && echo "$ip OK" || echo "$ip UNREACHABLE"
done
```
