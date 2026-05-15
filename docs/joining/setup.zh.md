# 配置指南

申请获批后，按以下步骤配置您的 FadianRoam 成员基础设施。

## 步骤 1：MGMT VPN (WireGuard)

获批后，您将收到：

- **中继服务器公钥**: 联邦中继服务器的 WireGuard 公钥
- **中继服务器端点**: 中继服务器的公网 IP 和端口
- **您的 MGMT IP**: 在 `172.172.10.0/24` 网段中分配给您的 IP

### 生成密钥

```bash
wg genkey | tee /etc/wireguard/mgmt-privatekey | wg pubkey > /etc/wireguard/mgmt-publickey
```

### 配置 WireGuard

创建 `/etc/wireguard/fadianroam-mgmt.conf`：

```ini
[Interface]
Address = 172.172.10.XX/24          # Your assigned MGMT IP
PrivateKey = <your-private-key>
ListenPort = 51820                 # Or any available port

[Peer]
PublicKey = <relay-public-key>
Endpoint = <relay-endpoint>:51820
AllowedIPs = 172.172.10.0/24
PersistentKeepalive = 25
```

### 启动并启用

```bash
systemctl enable --now wg-quick@fadianroam-mgmt
```

### 验证

```bash
wg show fadianroam-mgmt
ping 172.172.10.1  # Federation Relay
```

## 步骤 2：身份提供者

部署一个可供您的 RADIUS 服务器进行认证的身份提供者。您的 IDP 必须能够代表 RADIUS 服务器验证用户凭据（例如通过 ROPC、LDAP、REST API 或您的 RADIUS 实现所支持的任何方式）。

创建一组测试用户用于联邦验证。

## 步骤 3：RADIUS 服务器

部署一台具备以下能力的 RADIUS 服务器：

1. **EAP-TTLS/PAP** — 接受来自 AP 的 802.1X 认证
2. **IDP 集成** — 根据您的 IDP 验证本地用户凭据
3. **基于 realm 的代理转发** — 将非本地 realm 的请求转发至联邦中继服务器

### 关键配置要点

**EAP**: 将 EAP-TTLS 配置为默认方法。使用来自公开受信任 CA 的有效 TLS 证书。

**IDP 集成**: 配置您的 RADIUS 服务器根据 IDP 验证凭据。应使用 `Stripped-User-Name`（去除 realm 后的部分）作为用户名。

**Realm 代理转发**: 配置您的 RADIUS 服务器：

- 在本地处理您自己的 realm
- 将所有未知 realm（`DEFAULT`）代理转发至联邦中继服务器 `172.172.10.1:1812`，使用批准时提供的共享密钥

**联邦中继客户端**: 允许联邦中继服务器（`172.172.10.1`）作为 RADIUS 客户端，以便其将漫游请求转发给您。

### 验证

```bash
# Test local authentication
radtest user@your-realm.example.net PASSWORD localhost 0 testing123
```

## 步骤 4：Wi-Fi AP 配置

将您的接入点配置为 WPA2/WPA3-Enterprise：

| 设置项 | 值 |
|--------|------|
| 安全方式 | WPA2-Enterprise (802.1X) |
| RADIUS 服务器 | 您的 RADIUS 服务器 IP |
| RADIUS 端口 | 1812 |
| RADIUS 密钥 | 您的本地 RADIUS 客户端密钥 |
| SSID | `FadianRoam`（推荐） |

!!! tip "SSID 命名约定"
    在所有成员站点统一使用 SSID `FadianRoam`，可使设备在漫游时自动连接，类似于 `eduroam` 的 SSID。

## 步骤 5：联邦验证

完成配置后，通知联邦维护者执行跨站点认证测试：

1. 来自其他成员站点的测试用户尝试在您的 AP 进行认证
2. 验证 RADIUS 代理链端到端正常工作
3. 确认用户在认证后获得网络访问权限

验证通过后，您在联邦注册表中的成员状态将更新为**活跃**。
