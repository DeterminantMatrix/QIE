# QIE

`qie` 是一个用于管理和切换落地机出口的命令行脚本。原机器运行 sing-box 服务端入站，落地机使用 Hysteria2 承接出口流量。

它不主动 SSH 到落地机。正确流程是：

1. 在落地机 SSH 中执行导出命令，生成 `QIE_NODE_BEGIN` / `QIE_NODE_END` 数据块。
2. 回到原机器运行 `sudo qie` 或 `sudo qie add`。
3. 粘贴落地机生成的数据块。
4. `qie` 自动保存该落地机出口配置和测速协议，之后可以在菜单中切换并查看延迟。

## 安装 qie

Debian / Ubuntu：

```bash
sudo apt update && sudo apt install -y curl python3 && tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

Alpine：

```sh
apk add --no-cache curl python3 && tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

安装后首次运行：

```bash
sudo qie
```

第一次使用时，`qie` 默认没有任何落地机配置，会提示粘贴落地机导出的数据块。

## 在落地机生成导入数据

在落地机 SSH 中执行：

Debian / Ubuntu：

```bash
sudo apt update && sudo apt install -y curl python3 && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sudo sh
```

Alpine：

```sh
apk add --no-cache curl python3 && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh
```

如果没有用 `-n` 指定名称，脚本会要求你为这台落地机起名，并把这个名字写入 `QIE_NODE` 数据块。也可以直接指定：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node
```

导出脚本会先显示检测结果：

- 是否已有 hy2
- 是否已有 sing-box
- 当前 BBR 状态
- 自动探测到的落地机连接地址

然后逐项询问：

- 是否安装/启用 BBR
- 如果没有 hy2，是否安装独立 Hysteria2
- 是否导出 `QIE_NODE_BEGIN` 数据块

导出脚本会在落地机上执行这些动作：

1. 检测 `python3`、`curl`、`openssl`。
2. 如果缺少基础依赖，会用当前系统的包管理器安装。
3. 检测 BBR 当前状态；如果你确认安装/启用 BBR，会调用成熟的 `vps-tcp-tune` 脚本：

```bash
bash <(curl -fsSL "https://raw.githubusercontent.com/Eric86777/vps-tcp-tune/main/install-alias.sh?$(date +%s)")
bash <(curl -fsSL "https://raw.githubusercontent.com/Eric86777/vps-tcp-tune/main/net-tcp-tune.sh?$(date +%s)")
```

第一行会安装以后手动使用的 `bbr` 快捷命令；第二行会在本次脚本中直接运行 BBR 工具，避免 `curl | sh` 环境里 alias 不生效。

4. 优先检测已有 Hysteria2 独立服务端配置。
5. 如果没有 Hysteria2 配置，再检测已有 sing-box 配置中的 `hysteria2` / `hy2` inbound。
6. 如果检测到已有 hy2，会直接生成导入数据，不安装新的 Hysteria2。
7. 如果没有任何 hy2，并且你确认安装，才从 Hysteria GitHub Release 下载二进制，自动生成自签 TLS 证书、随机密码和 Hysteria2 服务端配置。
8. 启动或重启 Hysteria2 服务。
9. 输出原机器可粘贴导入的 `QIE_NODE_BEGIN` 数据块。这个数据块里是原机器 sing-box 使用的 hysteria2 outbound 配置。
10. 清理本脚本的临时文件；在 Alpine 上，如果 `curl`、`python3`、`openssl` 是本脚本临时安装的，会在导出完成后尝试移除。

Hysteria2 二进制、服务文件、配置和证书是落地机运行所必需的，不会删除。

如果希望保留本脚本安装的辅助依赖：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node --keep-deps
```

如果需要非交互模式，自动确认安装缺失 hy2 并导出。BBR 不会在 `-y` 下自动启用，仍需交互确认：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node -y
```

命令会输出类似下面的数据块：

```text
QIE_NODE_BEGIN
...
QIE_NODE_END
```

导出数据会写入 `test_protocol`。Hysteria2 / TUIC 这类 UDP 协议会按 UDP 节点处理，避免在 `qie` 菜单中被错误显示为 TCP 超时。

如果落地机已经安装了第三方 Hysteria2，`luodi` 会从现有配置或证书中提取端口、密码和 SNI。比如 Alpine 脚本安装的 hy2 使用 `www.bing.com` 作为证书域名时，导出的 sing-box 客户端配置会写入 `tls.server_name: www.bing.com`，并设置 `tls.alpn: ["h3"]`，不会再使用默认的 `qie.local`。

复制整段输出，回到原机器执行：

```bash
sudo qie add
```

然后粘贴整段数据块即可。

导出脚本会自动尝试以下 Hysteria2 配置路径：

- `/etc/hysteria/config.yaml`
- `/etc/hysteria/config.yml`
- `/etc/hysteria/*.yaml`
- `/etc/hysteria/*.yml`

如果都不存在，会创建 `/etc/hysteria/config.yaml`。

同时也会检测以下 sing-box 配置路径中的 `hysteria2` / `hy2` inbound：

- `/etc/s-box/sb.json`
- `/etc/sing-box/config.json`
- `/usr/local/etc/sing-box/config.json`
- `/etc/s-box/*.json`
- `/etc/sing-box/*.json`

如果想手动指定落地机上的 Hysteria2 服务端配置路径：

Debian / Ubuntu：

```bash
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sudo sh -s -- -n my-node -c /path/to/config.yaml
```

Alpine：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node -c /path/to/config.yaml
```

如果自动探测的公网 IP 不正确，手动指定原机器连接落地机时使用的地址：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node --server 1.2.3.4
```

如果要指定 hy2 端口：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node -p 8443
```

## 前置条件

- 原机器和落地机都需要 `python3`
- 原机器使用 `systemd` 或 OpenRC
- 原机器已安装 `sing-box`
- 原机器 sing-box 服务名为 `sing-box`
- 落地机需要开放 Hysteria2 UDP 端口，默认 `8443/udp`
- `qie` 需要 root 权限运行

`qie` 在原机器上使用以下路径：

| 路径 | 用途 |
| --- | --- |
| `/etc/s-box/sb.json` | 当前运行配置 |
| `/etc/s-box/qie_nodes/` | 导入的落地机配置 |
| `/etc/s-box/qie_nodes.json` | 节点索引 |
| `/etc/s-box/qie_state` | 当前模式状态 |
| `/etc/s-box/sb_direct.json` | 旧版 Direct 配置，保留兼容 |

切换节点时，`qie` 不会把落地机客户端配置整体覆盖到 `/etc/s-box/sb.json`。它会读取当前运行配置，保留原有 `inbounds`、已有 `outbounds` 和 `route.rules`，只替换由 `qie` 管理的 `qie-proxy` 出口，并把 `route.final` 指向当前落地机。切回 Direct 时只把 `route.final` 改回 `direct`。

如果重启 `sing-box` 失败，`qie` 会尝试回滚到切换前的配置。

## 用法

交互菜单：

```bash
sudo qie
```

录入落地机：

```bash
sudo qie add
```

修改落地机名称：

```bash
sudo qie rename 1 新名称
sudo qie rename my-node 新名称
```

删除落地机：

```bash
sudo qie delete 1
sudo qie delete my-node
```

查看节点：

```bash
sudo qie list
```

查看版本：

```bash
qie version
```

按编号或名称切换：

```bash
sudo qie 1
sudo qie my-node
```

切换 Direct：

```bash
sudo qie direct
```

查看状态：

```bash
sudo qie status
```

测速：

```bash
sudo qie test
```

延迟显示说明：

- `sudo qie` 和 `sudo qie list` 中，HY2 / TUIC 节点显示的是 ICMP 延迟，即本机到落地机 IP 的基础网络延迟。
- `sudo qie test` 会额外启动一个临时 sing-box 客户端，通过该落地机访问 `https://www.gstatic.com/generate_204`，显示 `出站`耗时。该测试不修改 `/etc/s-box/sb.json`，结束后会关闭临时进程。
- 如果落地机禁 ping，会显示 `ICMP: 不可用`，但 `出站`测试仍可用于判断 HY2 链路是否可用。

## 更新

更新 `qie`：

```bash
tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && if [ -f /usr/local/bin/qie ] && cmp -s "$tmp" /usr/local/bin/qie; then echo "qie 已是最新"; rm -f "$tmp"; else sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp" && echo "qie 已更新"; fi
```

Alpine：

```sh
tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && if [ -f /usr/local/bin/qie ] && cmp -s "$tmp" /usr/local/bin/qie; then echo "qie 已是最新"; rm -f "$tmp"; else install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp" && echo "qie 已更新"; fi
```

## 卸载

```bash
sudo rm -f /usr/local/bin/qie
```
