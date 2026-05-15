# VPN 配置

FadianRoam 使用 WireGuard 作为 MGMT 和 FadianNet VPN 隧道。

## MGMT VPN

MGMT VPN 将每个成员的 RADIUS 服务器连接到 Federation Relay，**仅**承载 RADIUS 代理流量。

### 拓扑结构

星形拓扑 — 每个成员都连接到中心的 Federation Relay：

```
Member A ──── WireGuard ────┐
Member B ──── WireGuard ────┤── Federation Relay (172.172.10.1)
Member C ──── WireGuard ────┘
```

### 地址分配

| 角色 | IP 范围 |
|------|---------|
| Federation Relay | `172.172.10.1` |
| 成员 | `172.172.10.10` – `172.172.10.254` |
| 子网 | `172.172.10.0/24` |

### 成员配置

`/etc/wireguard/fadianroam-mgmt.conf`：

```ini
[Interface]
Address = 172.172.10.XX/24
PrivateKey = <member-private-key>
ListenPort = 51820

[Peer]
# Federation Relay
PublicKey = <relay-public-key>
Endpoint = <relay-public-ip>:51820
AllowedIPs = 172.172.10.0/24
PersistentKeepalive = 25
```

### Relay 配置

Relay 为每个成员设置一个 peer 条目：

```ini
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
```

### 防火墙规则

在成员端，将 MGMT 隧道限制为仅允许 RADIUS 流量：

```bash
# Allow RADIUS to/from MGMT subnet
iptables -A INPUT -i fadianroam-mgmt -p udp --dport 1812 -j ACCEPT
iptables -A INPUT -i fadianroam-mgmt -p udp --dport 1813 -j ACCEPT
iptables -A INPUT -i fadianroam-mgmt -j DROP
```

## FadianNet VPN

FadianNet 在认证完成后承载用户数据流量。成员通过 WireGuard 隧道连接到 FadianNet 骨干网。

### 点对点链路

BGP 成员之间建立点对点 WireGuard 隧道：

```
Member A ──── P2P WireGuard ──── Member B
   │                                │
   └──── P2P WireGuard ──── Member C
```

### 地址分配

| 用途 | 子网 |
|------|------|
| Loopback（路由器 ID） | `172.172.11.0/24` |
| P2P 隧道链路 | `172.172.12.0/16` |

P2P 链路使用 `172.172.12.0/16` 范围中的 /30 子网：

```
Member A ←→ Member B: 172.172.12.0/30
  A: 172.172.12.1
  B: 172.172.12.2

Member A ←→ Member C: 172.172.12.4/30
  A: 172.172.12.5
  C: 172.172.12.6
```

### BGP 成员 FadianNet 配置

`/etc/wireguard/fadiannet-peer-b.conf`：

```ini
[Interface]
Address = 172.172.12.1/30
PrivateKey = <member-a-private-key>
ListenPort = 51821

[Peer]
PublicKey = <member-b-pubkey>
Endpoint = <member-b-public-ip>:51821
AllowedIPs = 172.172.12.2/32, 172.172.11.0/24, 0.0.0.0/0
PersistentKeepalive = 25
```

### 仅 VPN 成员配置

仅 VPN 成员通过单条隧道连接到最近的 BGP 成员：

```ini
[Interface]
Address = 172.172.12.X.X/30
PrivateKey = <member-private-key>
ListenPort = 51821

# Default route through FadianNet
PostUp = ip route add default via 172.172.12.X.Y dev %i table fadiannet
PostDown = ip route del default via 172.172.12.X.Y dev %i table fadiannet

[Peer]
PublicKey = <upstream-bgp-member-pubkey>
Endpoint = <upstream-ip>:51821
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## WireGuard 操作

### 生成密钥

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

### 启动 / 停止

```bash
# Start
wg-quick up fadianroam-mgmt

# Stop
wg-quick down fadianroam-mgmt

# Enable on boot
systemctl enable wg-quick@fadianroam-mgmt
```

### 状态检查

```bash
# Show all interfaces
wg show

# Show specific interface
wg show fadianroam-mgmt

# Check handshake (should be recent)
wg show fadianroam-mgmt latest-handshakes
```

### 故障排查

| 问题 | 检查方法 |
|------|----------|
| 无握手 | 防火墙是否阻止了 UDP 端口，endpoint 是否正确 |
| 握手成功但无流量 | `AllowedIPs` 配置不匹配 |
| 间歇性断连 | 未设置 `PersistentKeepalive`，NAT 超时 |
| MTU 问题 | 在 Interface 部分设置 `MTU = 1420` |
