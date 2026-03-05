# Linux TUN 模式科学上网

在 Linux 服务器上配置 Mihomo (Clash Meta) TUN 模式透明代理的 OpenClaw Skill。

## 功能

- 一键在 Linux 服务器上安装配置 Mihomo 内核
- 配置 TUN 模式实现系统全局透明代理
- 配置 systemd 守护进程实现开机自启
- 可选安装 Metacubexd Web 管理面板

## 使用方法

1. 确保你有一台 Linux 云服务器（Ubuntu/Debian）
2. 准备好以下信息：
   - 云主机 IP 地址
   - SSH 用户名
   - SSH 密码（或 SSH 密钥）
   - 机场订阅地址

3. 通过 OpenClaw 调用此 skill，按照指引完成配置

## 需要提供的信息

| 信息 | 说明 | 示例 |
|------|------|------|
| 云主机 IP | 服务器的公网 IP 地址 | 192.168.1.100 |
| SSH 用户名 | 登录服务器的用户名 | root 或 ubuntu |
| SSH 密码 | 用户密码 | your_password |
| 机场订阅地址 | 机场的 Clash 订阅链接 | https://example.com/sub?token=xxx |

## 注意事项

- 仅支持 Ubuntu/Debian 系统
- 需要服务器有公网 IP
- TUN 模式需要服务器开启 IP 转发
- 建议使用密钥认证而非密码认证
