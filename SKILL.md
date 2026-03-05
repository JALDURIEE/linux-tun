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

## 更新机场订阅

当机场订阅地址变更或需要更新节点时，**必须**严格按照以下流程操作。

> [!IMPORTANT]
> **警告**：直接执行 `curl` 下载新配置会**彻底覆盖**现有的 `config.yaml`。
> 你**必须**先备份旧配置，并在下载后**手动恢复** TUN、DNS 和 Web UI 相关配置，否则服务将无法正常工作。

### 第一步：准备工作 (备份与停服)

1. 进入目录并**备份**当前可用的配置：
   ```bash
   cd /etc/mihomo
   sudo cp config.yaml config.yaml.bak
   ```
2. 如果当前订阅已过期导致无法联网，请**必须先停止**服务以恢复直接联网：
   ```bash
   sudo systemctl stop mihomo
   ```

### 第二步：下载新配置

使用新的订阅地址下载（替换 `{新的订阅地址URL编码}`）：
```bash
sudo curl -o config.yaml "https://api.wcc.best/sub?target=clash&url={新的订阅地址URL编码}&insert=false&emoji=true"
```

### 第三步：恢复关键配置 (强制执行)

下载的文件仅包含节点信息，你**必须**将以下内容重新添加回 `config.yaml` 的顶部：

```yaml
# 1. 基础与 Web UI 配置
external-controller: :9090
external-ui: ui
secret: '你的密码'

# 2. TUN 模式配置
tun:
  enable: true
  stack: system
  dns-hijack:
    - any:53
  auto-route: true
  auto-detect-interface: true

# 3. DNS 配置
dns:
  enable: true
  enhanced-mode: fake-ip
  listen: 0.0.0.0:1053
  nameserver:
    - 114.114.114.114
    - 223.5.5.5
```

### 第四步：重启并验证

```bash
sudo systemctl restart mihomo
sudo systemctl status mihomo
```

### 3. 重启服务使配置生效

```bash
sudo systemctl restart mihomo
sudo systemctl status mihomo
```

---

## 常用命令

- 启动: `sudo systemctl start mihomo`
- 停止: `sudo systemctl stop mihomo`
- 重启: `sudo systemctl restart mihomo`
- 状态: `sudo systemctl status mihomo`
- 查看日志: `sudo journalctl -u mihomo -f`
- 更新订阅: 参考上方的 [更新机场订阅](#更新机场订阅) 部分
