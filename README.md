# vcontrol 管理脚本

这是一个用于 Linux 服务器的 `vcontrol` 安装与管理脚本仓库。脚本负责安装上游 `V2Ray Core`、生成配置、维护 `systemd` 服务，并在需要时自动配置 Caddy 以启用 TLS。

## 功能概览

- 安装 `vcontrol`、上游 `V2Ray Core` 和基础依赖
- 管理多份入站配置，支持添加、修改、查看、删除
- 支持以下协议和传输方式
- `VMess`: `tcp`、`kcp`、`quic`、`ws`、`h2`、`grpc`
- `VMess` 动态端口: `tcpd`、`kcpd`、`quicd`
- `VLESS`: `vws`、`vh2`、`vgrpc`
- `Trojan`: `tws`、`th2`、`tgrpc`
- 其他: `ss`、`socks`、`door`、`http`
- 支持自动 TLS、手动 TLS、DNS 设置、BBR、日志查看、二维码和 URL 输出

## 目录结构

- `install.sh`: 安装入口
- `vcontrol.sh`: 命令入口
- `src/init.sh`: 运行环境初始化
- `src/core.sh`: 主要命令逻辑
- `src/caddy.sh`: Caddy 配置生成
- `src/download.sh`: Core、脚本、数据文件下载逻辑
- `src/systemd.sh`: `systemd` 服务生成
- `src/dns.sh`: DNS 设置
- `src/log.sh`: 日志级别与日志查看
- `src/bbr.sh`: BBR 启用逻辑

## 环境要求

- Linux，支持 `x86_64`、`arm64`、`armv7`、`armv6`、`armv5`
- `Ubuntu`、`Debian` 或 `CentOS`
- 使用 `systemd`
- 需要 `root` 权限执行安装
- 使用 `ws`、`h2`、`grpc` 这类 TLS 协议时，建议提前准备好域名并完成 DNS 解析

说明：

- `arm64` 使用 `v2ray-linux-arm64-v8a.zip`
- `armv7` 使用 `v2ray-linux-arm32-v7a.zip`
- `armv6` 使用 `v2ray-linux-arm32-v6.zip`
- `armv5` 使用 `v2ray-linux-arm32-v5.zip`

## 安装教程

### 安装当前仓库代码

如果你正在修改这个仓库，或者就是想安装当前工作区里的代码，请使用本地安装模式：

```bash
git clone https://github.com/Linorman/v2ray.git
cd v2ray
sudo bash install.sh -l
```

说明：

- `-l` 会直接复制当前目录下的脚本到 `/etc/vcontrol/sh`
- 这是测试本仓库改动时应使用的方式

### 安装发布版脚本

如果只是想部署脚本默认使用的发布版，可以直接运行：

```bash
sudo bash install.sh
```

说明：

- 不带 `-l` 时，安装脚本会从内置的 GitHub Release 源下载脚本压缩包和上游 `V2Ray Core`
- 这种方式不会使用你当前目录中的未发布改动

### 安装参数

- `-l`, `--local-install`: 使用当前目录代码安装
- `-f`, `--core-file <path>`: 使用本地 V2Ray Core 压缩包
- `-v`, `--core-version <ver>`: 指定 Core 版本
- `-p`, `--proxy <addr>`: 下载时使用代理
- `-h`, `--help`: 查看安装帮助

## 首次安装后会发生什么

- 安装目录创建在 `/etc/vcontrol`
- 命令入口软链接到 `/usr/local/bin/vcontrol`
- 自动生成 `/etc/vcontrol/config.json`
- 自动创建一个默认的 `VMess TCP` 配置
- 创建 `vcontrol.service`
- 如果使用 TLS 协议且未启用 `no-auto-tls`，脚本会自动安装并配置 Caddy

## 基本使用

### 打开交互菜单

```bash
vcontrol
```

脚本无参数运行时会进入交互菜单，适合首次使用。

### 查看帮助

```bash
vcontrol help
```

### 查看版本和状态

```bash
vcontrol version
vcontrol status
```

## 添加配置

### 通用方式

```bash
vcontrol add
```

不带协议参数时，脚本会弹出协议选择菜单。

### 常用示例

```bash
vcontrol add tcp
vcontrol add kcp
vcontrol add quic
vcontrol add ws example.com auto /ws remark=hk-main sniffing=on
vcontrol add vws example.com auto /vless listen=127.0.0.1
vcontrol add tws example.com auto /trojan
vcontrol add ss 8443 strong-password aes-128-gcm
vcontrol add socks 1080 user strong-password
```

说明：

- `tcp`、`kcp`、`quic` 这类协议可以不提供域名
- `ws`、`h2`、`grpc` 以及 `vws`、`vh2`、`vgrpc`、`tws`、`th2`、`tgrpc` 需要域名
- 参数写成 `auto` 时，脚本会自动生成 UUID、路径或端口等值
- 现在可以额外追加 `key=value` 高级字段，例如 `remark=hk-main`、`listen=0.0.0.0`、`sniffing=off`、`seed=my-kcp-seed`
- 交互式创建流程也会提示是否继续自定义这些高级字段

### 动态端口示例

```bash
vcontrol add tcpd
vcontrol add kcpd
vcontrol add quicd
```

## 查看和导出配置

```bash
vcontrol info
vcontrol url
vcontrol qr
```

说明：

- 不带配置名时，脚本会先让你选择配置
- `url` 输出订阅/导入链接
- 对 `VMess` 配置，`info` 和 `url` 会同时给出域名、公网 IPv4/IPv6、局域网地址等不同地址版本的 VMess 链接
- 每条 VMess 链接都会明确标注当前使用的是哪个地址，便于区分公网导入和局域网直连
- `qr` 输出二维码；如果机器没装 `qrencode`，先执行包管理器安装即可

## 修改配置

### 交互方式

```bash
vcontrol change
```

### 直接修改单项参数

```bash
vcontrol port <name> 443
vcontrol host <name> example.com
vcontrol path <name> /new-path
vcontrol id <name> auto
vcontrol passwd <name> new-password
vcontrol method <name> aes-256-gcm
vcontrol type <name> none
vcontrol remark <name> hk-main
vcontrol listen <name> 127.0.0.1
vcontrol sniff <name> off
vcontrol web <name> example.org
vcontrol dp <name> 20000 30000
```

说明：

- `<name>` 为配置文件名，通常可通过 `vcontrol info` 或交互选择看到
- `full` 可以一次性重建当前配置的多个参数
- `new` 可以把现有配置切换为另一种协议

## 启动、停止和测试

```bash
vcontrol start
vcontrol stop
vcontrol restart
vcontrol test
vcontrol log
vcontrol logerr
```

如果 Caddy 已安装，也可以：

```bash
vcontrol restart caddy
```

## 更新和维护

```bash
vcontrol update core
vcontrol update sh
vcontrol update dat
vcontrol dns
vcontrol bbr
vcontrol reinstall
vcontrol uninstall
```

说明：

- `update core` 更新 V2Ray Core
- `update sh` 更新脚本本体
- `update dat` 更新 `geoip.dat` 和 `geosite.dat`
- `reinstall` 使用当前已安装在 `/etc/vcontrol/sh` 下的脚本重新安装

## 手动 TLS 模式

如果你不希望脚本自动安装 Caddy，可以使用：

```bash
vcontrol no-auto-tls ws example.com auto /ws
```

这类模式下，脚本会输出端口和路径，你需要自行完成：

- TLS 证书签发
- 反向代理配置
- 443 端口接入

## 重要路径

- `/etc/vcontrol/bin`: `vcontrol` 运行文件
- `/etc/vcontrol/conf`: 各配置文件
- `/etc/vcontrol/config.json`: 主配置
- `/etc/vcontrol/sh`: 脚本文件
- `/var/log/vcontrol`: 访问日志和错误日志
- `/etc/caddy/Caddyfile`: Caddy 主配置

## 默认行为说明

首次生成的 `config.json` 会包含以下默认行为：

- 创建一个仅监听本机的 API 入站
- 阻断 `bittorrent`
- 阻断 `geoip:cn`
- 阻断私有地址流量
- 将 `openai.com` 相关域名走 `direct`

如果这些规则不符合你的环境，请按需修改配置。

## 注意事项

- `del` 和 `ddel` 会直接删除配置，不会二次确认
- 自动分配端口时，脚本依赖 `netstat` 或 `ss` 检测占用情况
- 如果 `80` 或 `443` 已被占用，自动 TLS 模式会让 Caddy 改用其他端口
- 需要测试当前仓库改动时，一定要使用 `install.sh -l`
