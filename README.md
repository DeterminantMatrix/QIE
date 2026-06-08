# QIE

`qie` 是一个用于管理和切换 sing-box 落地机配置的命令行脚本。

它不主动 SSH 到落地机。正确流程是：

1. 在落地机 SSH 中执行导出命令，生成 `QIE_NODE_BEGIN` / `QIE_NODE_END` 数据块。
2. 回到原机器运行 `sudo qie` 或 `sudo qie add`。
3. 粘贴落地机生成的数据块。
4. `qie` 自动保存该落地机配置，之后可以在菜单中切换。

## 安装 qie

Debian / Ubuntu：

```bash
sudo apt update && sudo apt install -y curl python3 && tmp=$(mktemp) && curl -fsSL https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie -o "$tmp" && sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

安装后首次运行：

```bash
sudo qie
```

第一次使用时，`qie` 默认没有任何落地机配置，会提示粘贴落地机导出的数据块。

## 在落地机生成导入数据

在落地机 SSH 中执行：

```bash
sudo apt update && sudo apt install -y curl python3 && curl -fsSL https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie-export-node | sudo bash -s -- -n my-node
```

把 `my-node` 换成你想显示在 `qie` 菜单里的名称。

命令会输出类似下面的数据块：

```text
QIE_NODE_BEGIN
...
QIE_NODE_END
```

复制整段输出，回到原机器执行：

```bash
sudo qie add
```

然后粘贴整段数据块即可。

如果落地机的 sing-box 配置不是 `/etc/s-box/sb.json`，可以指定路径：

```bash
curl -fsSL https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie-export-node | sudo bash -s -- -n my-node -c /path/to/sb.json
```

## 前置条件

- 原机器和落地机都需要 `python3`
- 原机器使用 `systemd`
- 原机器已安装 `sing-box`
- 原机器 sing-box 服务名为 `sing-box`
- `qie` 需要 root 权限运行

`qie` 在原机器上使用以下路径：

| 路径 | 用途 |
| --- | --- |
| `/etc/s-box/sb.json` | 当前运行配置 |
| `/etc/s-box/qie_nodes/` | 导入的落地机配置 |
| `/etc/s-box/qie_nodes.json` | 节点索引 |
| `/etc/s-box/qie_state` | 当前模式状态 |
| `/etc/s-box/sb_direct.json` | Direct 配置 |

切换节点时，`qie` 会把目标配置复制到 `/etc/s-box/sb.json`，然后重启 `sing-box`。如果重启失败，会尝试回滚到原配置。

## 用法

交互菜单：

```bash
sudo qie
```

录入落地机：

```bash
sudo qie add
```

查看节点：

```bash
sudo qie list
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

## 更新

更新 `qie`：

```bash
tmp=$(mktemp) && curl -fsSL https://raw.githubusercontent.com/DeterminantMatrix/QIE/main/qie -o "$tmp" && sudo install -m 755 "$tmp" /usr/local/bin/qie && rm -f "$tmp"
```

## 卸载

```bash
sudo rm -f /usr/local/bin/qie
```
