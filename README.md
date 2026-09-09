# iptv-spider-pro

上海电信 IPTV Spider 的**单仓库自包含**版本：PVE/CT 编排脚本与应用本体打包在
同一个仓库、同一个 Release 里，全新安装不再依赖其它 GitHub 仓库（旧架构的
`sh-iptv-manager` 已不再需要）。

在一个全新 Debian 12 CT 上把 PVE 侧一键装成 "DHCP-direct" 的 IPTV Spider 节点：
`eth1` 在 VLAN85 专网自行获取 DHCP 租约，专网路由与源 IP 随租约自动维护，无需
RouterOS SNAT/静态路由（RouterOS 只做二层桥接 + IGMP 组播代理）。

## 结构

- `install.sh`：PVE 一键引导。无参数运行=交互向导（扫描空闲 CT 号/管理 IP、冲突
  重输、机顶盒抓包/手工）；也支持参数化透传。首次从本仓库 tag 下载同版本包。
- `pve-iptv-dhcp-create.sh`：PVE 编排。**默认自包含**：自动把本仓库 `app/` 打包
  推入新 CT 作为应用发行包，全程不访问其它仓库；也可 `--pkg-dir <dir>` 指定本地
  发行目录。其它参数（--vmid/--mgmt-ip/--eth1-mac/--ssh-pubkey/--routeros-key/
  --template/--destroy-existing/...）见脚本内 `--help`。
- `install-dhcp.sh`：CT 内引导。eth1 DHCP + dhclient hooks（专网路由/源 IP 随租约
  维护、config.yaml `stb.ip` 自动同步）；机顶盒参数支持 `STB_MODE=manual` 与
  `STB_MODE=capture`（RouterOS 抓包）；本地 MariaDB + config.yaml + systemd。
  GitHub 兜底下载只指向本仓库 Release（`iptv-spider-pro-app-<ver>-linux-amd64.tar.gz`）。
- `app/`：应用本体（iptv-spider 程序源码 + `bin/` 预编译 + assets/logos +
  `status.sh`/`manage.sh`/`uninstall.sh`/覆盖升级 `install.sh` 等）。
- 版本：仓库 tag（如 `v1.0.0`）= 应用版本；`app/VERSION` 与 Release asset 同步。

## 快速开始（全新安装）

前置：Proxmox VE root Shell；Debian 12 CT 模板
（`local:vztmpl/debian-12-standard_*.tar.zst`）；RouterOS 侧已有 VLAN85 二层路径；
`STB_MODE=capture` 需实体机顶盒可断电上电。

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/driftbottle61/iptv-spider-pro/v1.0.0/install.sh)
```

- 无参数=交互向导；按提示选择机顶盒参数获取方式（RouterOS 抓包 / 手工）。
- 抓包模式下：向导只登记 RouterOS 连接参数，真正抓包在 CT 建好、专网 DHCP 就绪后
  自动开始，届时按提示断电重启机顶盒。
- 安装完成会显示：TiviMate `http://<管理IP>:8888/tv.m3u`、IPTV#
  `http://<管理IP>:8888/iptvsharp.m3u`、EPG `http://<管理IP>:8888/api/epg?daysAgo=7`。
- CT 内 `iptv-spider-status` 会显示同样三个链接；管理菜单 `iptv-spider`。

## 自包含说明

- 全新安装：`install.sh` → 本仓库 tag 包 → `pve-iptv-dhcp-create.sh` 默认使用
  `app/` 本地发行包 → CT 安装全程无外部仓库依赖。
- 已装节点覆盖升级：CT 内 `/opt/sh-iptv-spider/install.sh`（app 自带），或重跑
  一键安装选择全新 CT。
- 卸载：CT 内 `iptv-spider-uninstall`。

应用运行细节、RouterOS 抓包说明、配置项见 `app/README_CN.md`。

## 参考文档与致谢

- `docs/routeros-iptv-guide-jeffz.md`：RouterOS + 猫棒打通上海电信 IPTV 二层
  链路指引。整理自 jeffz 的博客文章《上海电信 IPTV 折腾记录 ROS+猫棒》
  （<https://jeffz.page/posts/4277824729/>）。本项目的 RouterOS 猫棒 IPTV
  链路即参考 jeffz 的文章打通，在此致谢。
