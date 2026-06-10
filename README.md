# QIE

`qie` 用来在原机器上管理和切换落地机出口。落地机不需要被原机器 SSH 登录；你只需要在落地机 SSH 里运行 `luodi`，复制生成的 `QIE_NODE` 数据块，再回到原机器用 `qie add` 导入。

## 安装 qie

Debian / Ubuntu：

```bash
sudo apt update && sudo apt install -y curl python3 && tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

Alpine：

```sh
apk add --no-cache curl python3 && tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

首次运行：

```bash
sudo qie
```

## 更新 qie

Debian / Ubuntu：

```bash
tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && if [ -f /usr/local/bin/qie ] && cmp -s "$tmp" /usr/local/bin/qie; then echo "qie 已是最新"; rm -f "$tmp"; else sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp" && echo "qie 已更新"; fi
```

Alpine：

```sh
tmp=$(mktemp) && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie?$(date +%s)" -o "$tmp" && if [ -f /usr/local/bin/qie ] && cmp -s "$tmp" /usr/local/bin/qie; then echo "qie 已是最新"; rm -f "$tmp"; else install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp" && echo "qie 已更新"; fi
```

## 生成落地机导入数据

在落地机 SSH 中运行：

Debian / Ubuntu：

```bash
sudo apt update && sudo apt install -y curl python3 && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sudo sh
```

Alpine / LXC：

```sh
apk add --no-cache curl python3 && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh
```

Alpine / LXC 公网 IP 经常无法自动识别，建议直接指定：

```sh
apk add --no-cache curl python3 && curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node --server 1.2.3.4
```

如果落地机没有可用协议服务，`luodi` 会让你选择：

- `Hysteria2 / UDP`
- `Shadowsocks TCP`

直接安装 Shadowsocks TCP：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node --protocol ss --ss-port 8388
```

直接安装 Hysteria2，并指定 UDP 端口：

```sh
curl -fsSL "https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/luodi?$(date +%s)" | sh -s -- -n my-node --protocol hy2 -p 8443
```

运行完成后复制整段输出：

```text
QIE_NODE_BEGIN
...
QIE_NODE_END
```

回到原机器导入：

```bash
sudo qie add
```

## 常用命令

```bash
sudo qie              # 打开交互菜单
sudo qie add          # 导入落地机
sudo qie list         # 查看节点和延迟
sudo qie 1            # 切换到第 1 个节点
sudo qie direct       # 切回直连
sudo qie test         # 测试节点出站
sudo qie rename 1 新名称
sudo qie delete 1
qie version
```

## 说明

- `qie` 运行在原机器上，需要 root 权限。
- 原机器需要已安装并运行 `sing-box`。
- `qie` 切换节点时不会覆盖原有入站，只会替换自己管理的出站。
- 切换节点后会检测是否能出站；失败会自动回退到 Direct。
- 落地机端口需要在面板或防火墙中放行：Hy2 默认 `8443/udp`，Shadowsocks 默认 `8388/tcp`。

## 卸载

```bash
sudo rm -f /usr/local/bin/qie
```
