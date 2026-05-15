# RADIUS 配置

FadianRoam 部署中成员 RADIUS 服务器的技术参考。

## 概述

每个成员运行一个 RADIUS 服务器，该服务器需要：

1. 接受来自本地 AP 的 802.1X 认证请求
2. 通过成员的身份提供商（Identity Provider）验证本地用户
3. 将非本地 realm 的请求代理转发至 Federation Relay

## 功能要求

### EAP

- 必须支持 **EAP-TTLS/PAP** 作为默认认证方式
- 外层 EAP 隧道最低要求 TLS 1.2
- 证书必须由受公开信任的 CA 签发

### IDP 集成

RADIUS 服务器必须通过成员的 IDP 验证凭据。具体的集成方式由各成员自行决定。核心行为如下：

- 在 realm 剥离后，RADIUS 服务器收到裸用户名和密码
- 将这些凭据转发至 IDP 进行验证
- IDP 返回成功或失败结果
- RADIUS 响应 Access-Accept 或 Access-Reject

### Realm 处理

必须配置 `suffix` 模块（或等效模块）以实现基于 realm 的路由：

- 从 `user@realm.example.net` 中解析 realm
- 对本地认证剥离 realm 后缀
- 将非本地 realm 通过代理转发至 Federation Relay

## 代理配置

### 本地 Realm

你自己的 realm 应在本地处理：

```
realm your-realm.example.net {
    # Handled locally
}
```

### 联盟代理

所有未知 realm 必须转发至 Federation Relay：

```
realm DEFAULT {
    type = radius
    authhost = 172.172.10.1:1812
    accthost = 172.172.10.1:1813
    secret = <federation-shared-secret>
    nostrip
}
```

`nostrip` 保留完整的 `user@realm` 格式，使 Relay 能够路由到正确的成员。

## 客户端配置

### 本地 AP 客户端

你的 AP 必须注册为 RADIUS 客户端：

```
client wifi-ap {
    ipaddr = <your-ap-subnet>
    secret = <ap-radius-secret>
}
```

### Federation Relay 客户端

Federation Relay 必须被允许向你的 RADIUS 发送请求（用于漫游用户的 home realm 属于你的情况）：

```
client federation-relay {
    ipaddr = 172.172.10.1
    secret = <federation-shared-secret>
}
```

## 测试

### 本地认证

```bash
radtest user@your-realm.example.net PASSWORD localhost 0 testing123
```

### 常见问题

| 症状 | 原因 | 修复方法 |
|------|------|----------|
| `TLS Alert: unknown CA` | 自签名证书或缺少 CA 证书 | 使用受公开信任的证书 |
| Realm 未被代理转发 | 未配置 realm 剥离 | 确认 suffix 模块（或等效模块）已启用 |
| Relay 不可达 | MGMT VPN 断开 | 使用 `wg show` 检查 WireGuard |
| IDP 验证失败 | 凭据错误或 IDP 配置问题 | 独立检查 IDP 集成 |
