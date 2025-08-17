---
title: 'Ygg in Naive：在中美之间安全组网的实践'
published: 2025-08-17
draft: false
description: '通过 NaiveProxy 访问 Yggdrasil节点，实现中美之间更安全的组网，避免 GFW 对 TLS 的识别。'
tags: ['ygg', 'naive', 'network']
---

## 背景

由于 **Yggdrasil 的 TLS 有可能会被防火墙（GFW）识别**，在美国的 VPS 上直接运行 Ygg 并与中国节点组网可能有风险。如果改用 TCP 虽然能避免 TLS 特征，但又缺少加密保护。

为此，我们考虑将 **NaiveProxy 作为跳板机**，在 Naive 的加密隧道内连接 Ygg 节点（即 *Ygg in Naive*），从而避免特征被识别，同时确保数据安全。

---

## 服务器端配置（美国 VPS）

### 1. 安装依赖

```bash
sudo apt update
sudo apt install golang-go -y
````

### 2. 编译带插件的 Caddy
这里额外添加了 **l4 插件**，因为此 VPS 可能还需要用来转发 Minecraft 服务器的流量。

```bash
~/go/bin/xcaddy build \
  --with github.com/caddyserver/forwardproxy@caddy2=github.com/klzgrad/forwardproxy@naive \
  --with github.com/mholt/caddy-l4
```

编译完成后移动到系统路径：

```bash
sudo mv ~/go/bin/caddy /usr/bin/caddy
```

然后参考官方文档安装 Caddy 服务管理脚本：
👉 [Caddy 官方运行文档](https://caddyserver.com/docs/running#using-the-service)

---

### 3. 创建专用用户和用户组

为安全起见，不要使用 root 来运行 Caddy。

```bash
sudo groupadd --system caddy

sudo useradd --system \
    --gid caddy \
    --create-home \
    --home-dir /var/lib/caddy \
    --shell /usr/sbin/nologin \
    --comment "Caddy web server" \
    caddy
```

---

### 4. 设置权限

```bash
sudo chown root:caddy /etc/caddy/Caddyfile
sudo chmod 640 /etc/caddy/Caddyfile
```

---

### 5. 配置文件 `/etc/caddy/Caddyfile`

```caddy
:443, xxx.xxx.xxx {
    route {
        forward_proxy {
            basic_auth usr pwd
            hide_ip
            hide_via
            probe_resistance
        }
        reverse_proxy https://xxx.com {
            header_up Host {upstream_hostport}
            header_up X-Forwarded-Host {host}
        }
    }
}
```

其中：

* `usr` 和 `pwd` 替换为自定义的代理认证信息
* `xxx.xxx.xxx` 替换为 VPS 的域名或 IP
* `https://xxx.com` 替换为需要伪装的网站

---

### 6. Systemd 服务文件

`/etc/systemd/system/caddy.service`：

```ini
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

加载并启动服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable caddy
sudo systemctl start caddy
```

查看运行状态：

```bash
sudo systemctl status caddy
```

应用配置文件修改：

```bash
sudo systemctl reload caddy
```

---

## 安装 Yggdrasil

下载并安装（Debian 系统）：

```bash
wget https://github.com/yggdrasil-network/yggdrasil-go/releases/download/v0.5.12/yggdrasil-0.5.12-amd64.deb
sudo dpkg -i yggdrasil-0.5.12-amd64.deb
```

编辑配置文件 `/etc/yggdrasil/yggdrasil.conf`：

```yaml
Listen: ["tcp://[::]:8091?password="]
```

> 注意：这里用 `tcp` 而不是 `tls`，否则会出现 **TLS in TLS** 的特征，反而显眼。

---

### 防火墙限制

仅允许 Naive 本地访问 Ygg 端口：

```bash
# IPv4
sudo iptables -I INPUT -p tcp --dport 8091 -s 127.0.0.1 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8091 -j DROP

# IPv6
sudo ip6tables -I INPUT -p tcp --dport 8091 -s ::1 -j ACCEPT
sudo ip6tables -A INPUT -p tcp --dport 8091 -j DROP
```

---

## 客户端配置

### 1. 下载 NaiveProxy

👉 [NaiveProxy Releases](https://github.com/klzgrad/naiveproxy/releases)

创建 `config.json`：

```json
{
  "listen": ["socks://127.0.0.1:1080"],
  "proxy": "https://usr:pwd@xxx.xxx.xxx",
  "log": ""
}
```

### 2. Ygg 客户端配置

`yggdrasil.conf`：

```yaml
Peers: [
  "socks://127.0.0.1:1080/[::]:8091?password="
]
```

这样，客户端通过 NaiveProxy 的 socks5 转发流量，再由 Naive 安全传输到 VPS，最终进入 Ygg 网络。

---

## 总结

* **NaiveProxy 负责中美之间的加密传输**，规避 GFW 特征识别。
* **Ygg 在 Naive 内部**，保证通信安全和匿名性。
* 防火墙规则确保 Ygg 只能被本机（Naive）访问，避免暴露服务端口。

这套方案能有效提升 Ygg 在跨境场景下的可用性和安全性。
---

*本文部分内容由 GPT-5 辅助生成与整理。*