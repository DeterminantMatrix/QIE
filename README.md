# QIE

`qie` 是一个用于切换 sing-box 节点配置的命令行脚本，支持 SG、TW、JP、Direct、Auto、状态查看和 TCP 测速。

## 一键安装

Debian / Ubuntu：

```bash
sudo apt update && sudo apt install -y curl python3 && tmp=$(mktemp) && curl -fsSL https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie -o "$tmp" && sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

安装后运行：

```bash
sudo qie
```

## 前置条件

- 系统使用 `systemd`
- 已安装 `sing-box`
- sing-box 服务名为 `sing-box`
- 需要 root 权限运行
- 配置目录为 `/etc/s-box`

脚本默认使用以下配置文件：

| 模式 | 配置文件 |
| --- | --- |
| SG | `/etc/s-box/sb_sg.json` |
| TW | `/etc/s-box/sb_taiwan.json` |
| JP | `/etc/s-box/sb_jp.json` |
| Direct | `/etc/s-box/sb_direct.json` |
| 当前运行配置 | `/etc/s-box/sb.json` |

切换时脚本会把目标配置复制到 `/etc/s-box/sb.json`，然后重启 `sing-box`。如果重启失败，会尝试回滚到原配置。

## 用法

交互菜单：

```bash
sudo qie
```

直接切换：

```bash
sudo qie sg
sudo qie tw
sudo qie jp
sudo qie direct
```

自动选择 TCP 延迟最低的可用节点：

```bash
sudo qie auto
```

查看状态：

```bash
sudo qie status
```

测速：

```bash
sudo qie test
```

## 更新

```bash
tmp=$(mktemp) && curl -fsSL https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie -o "$tmp" && sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

## 卸载

```bash
sudo rm -f /usr/local/bin/qie
```
