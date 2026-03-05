---
name: linux-tun
description: 在 Linux 服务器上配置 Mihomo TUN 模式科学上网
emoji: 🐳
metadata:
  {
    "openclaw":
      {
        "requires":
          { "ssh": "需要能 SSH 到目标 Linux 服务器" },
      },
  }
---

# Linux TUN 模式科学上网

基于 Mihomo (Clash Meta) 在 Linux 服务器上配置 TUN 模式透明代理。

## 快速开始

### 1. 安装 Mihomo 内核

SSH 到服务器后执行：

```bash
sudo mkdir -p /etc/mihomo
cd /etc/mihomo
sudo wget https://github.com/MetaCubeX/mihomo/releases/download/v1.18.3/mihomo-linux-amd64-v1.18.3.gz
sudo gzip -d mihomo-linux-amd64-v1.18.3.gz
sudo mv mihomo-linux-amd64-v1.18.3 /usr/local/bin/mihomo
sudo chmod +x /usr/local/bin/mihomo
```

### 2. 下载订阅配置

```bash
curl -o config.yaml "https://api.wcc.best/sub?target=clash&url={你的机场订阅地址URL编码}&insert=false&emoji=true"
```

### 3. 配置 TUN 模式

**开启 IP 转发：**
```bash
sudo vim /etc/sysctl.conf
# 添加: net.ipv4.ip_forward=1
sudo sysctl -p
```

**修改 config.yaml，添加：**
```yaml
external-controller: :9090
secret: '你的密码'

tun:
  enable: true
  stack: system
  dns-hijack:
    - any:53
    - tcp://any:53
  auto-route: true
  auto-redirect: true
  auto-detect-interface: true

dns:
  enable: true
  listen: 0.0.0.0:1053
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - 114.114.114.114
    - 223.5.5.5
  fallback:
    - 8.8.8.8
    - 1.1.1.1
```

### 4. 配置 systemd 守护进程

```bash
sudo vim /etc/systemd/system/mihomo.service
```

写入：
```ini
[Unit]
Description=mihomo Daemon, Another Clash Kernel.
After=network.target NetworkManager.service systemd-networkd.service iwd.service

[Service]
Type=simple
LimitNPROC=500
LimitNOFILE=1000000
Restart=always
ExecStartPre=/usr/bin/sleep 1s
ExecStart=/usr/local/bin/mihomo -d /etc/mihomo
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

启动服务：
```bash
sudo systemctl daemon-reload
sudo systemctl start mihomo
sudo systemctl enable mihomo
sudo systemctl status mihomo
```

### 5. 验证

```bash
ip a        # 查看虚拟网卡
ping www.google.com  # 测试连通性
```

### 6. 安装 Web 面板 (可选)

```bash
cd /etc/mihomo
sudo wget -O metacubexd.zip https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip
sudo unzip metacubexd.zip
sudo mv metacubexd-gh-pages ui
sudo rm metacubexd.zip
```

修改 config.yaml 添加：
```yaml
external-ui: ui
```

重启服务：
```bash
sudo systemctl restart mihomo
```

访问面板：`http://服务器IP:9090/ui/`

---

## 常用命令

- 启动: `sudo systemctl start mihomo`
- 停止: `sudo systemctl stop mihomo`
- 重启: `sudo systemctl restart mihomo`
- 状态: `sudo systemctl status mihomo`
- 查看日志: `sudo journalctl -u mihomo -f`
- 更新订阅: 重新执行 curl 下载命令
