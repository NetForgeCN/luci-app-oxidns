# luci-app-oxidns

语言：中文 | [English](./README.en.md)

`luci-app-oxidns` 是 OxiDNS 的 OpenWrt / LuCI 管理插件。安装后，LuCI 会出现 `Services -> OxiDNS` 页面，用来安装 OxiDNS 内核二进制、管理 OpenWrt 服务、编辑配置和查看日志。

在原版功能之上，本版本额外提供：

- **内核持久化存储与开机自愈**：内核二进制和 WebUI 默认保存在 `/etc/oxidns/core`（持久分区），`/usr/bin/oxidns` 与 `/usr/share/oxidns/webui` 以符号链接指向它。安装插件时会把 `/etc/oxidns` 写入 `/etc/sysupgrade.conf`，因此**升级固件后内核和配置不会丢失**；重新安装本插件或启动服务时会自动重建符号链接，无需重新下载内核、无需重新粘贴配置。
- **LuCI 内核更新**：`Core` 页面提供 `Check Update` 按钮，对比当前版本与 GitHub 最新 release，确认后可一键下载（含 SHA256 校验）、替换并重启服务。
- **日志字段过滤**：`Logs` 页面支持关键词过滤，以及按 `plugin`（如 log_cache / log_blocked / log_abroad）、`qtype`、`rcode` 字段筛选，并显示筛选后行数。

## 你需要安装什么

- `luci-app-oxidns`：LuCI 管理页面、rpcd 后端和 OpenWrt init 服务脚本。
- `luci-i18n-oxidns-zh-cn`：可选简体中文语言包。

## 安装 LuCI 插件

推荐在 OpenWrt 上以 root 用户执行官方安装脚本：

```sh
curl -fsSL https://oxidns.org/install.sh | sh
```

如果系统没有 `curl`，也可以使用 `wget`：

```sh
wget -O- https://oxidns.org/install.sh | sh
```

脚本会检测 OpenWrt 包管理器，从 `luci-app-oxidns` Releases 读取最新包，按系统选择 `.ipk` 或 `.apk`，安装 `luci-app-oxidns` 和可选简体中文语言包，并自动重启 `rpcd`。更多脚本选项见 <https://oxidns.org/openwrt>。

也可以从本仓库 Release 下载对应的 LuCI 包后手动安装：

```sh
opkg install ./luci-app-oxidns_0.1.0-r1_all.ipk
opkg install ./luci-i18n-oxidns-zh-cn_0.1.0-r1_all.ipk
```

在使用 `apk` 的 OpenWrt 系统上：

```sh
apk add --allow-untrusted --no-network ./luci-app-oxidns_0.1.0-r1_all.apk
apk add --allow-untrusted --no-network ./luci-i18n-oxidns-zh-cn_0.1.0-r1_all.apk
```

如果安装后菜单没有出现，重启 `rpcd`：

```sh
/etc/init.d/rpcd restart
```

然后打开 LuCI：`Services -> OxiDNS`。

## 安装 OxiDNS 内核

如果你是从旧的 OpenWrt `oxidns` 包模式迁移，先停止服务并移除旧内核包，避免它继续拥有 `/usr/bin/oxidns` 或 `/etc/init.d/oxidns`：

```sh
/etc/init.d/oxidns stop
opkg remove oxidns
```

如果系统使用 `apk`，对应执行：

```sh
/etc/init.d/oxidns stop
apk del oxidns
```

1. 打开 `Services -> OxiDNS -> Settings`，确认 `Core repository` 为 `svenshi/oxidns`，`Core bundle` 为 `full`。
2. 打开 `Services -> OxiDNS -> Core`，点击 `Install Core`；离线环境也可以点击 `Upload Core` 上传官方 `.tar.gz` archive 或单个 `oxidns` 二进制。
3. 安装成功后，到 `Overview` 启动并启用服务。

LuCI 会按当前设备 CPU 架构选择 OxiDNS Linux musl release archive，例如 `oxidns-x86_64-unknown-linux-musl.tar.gz`。安装时会校验 GitHub release asset 的 SHA256 digest。

如果内核已经安装，`Core` 页面提供 `Repair Reinstall` 和 `Upload Core`，用于重新下载当前已安装版本或用上传文件修复二进制 / WebUI 文件；它不会安装 latest，也不会作为升级入口。

## 常用页面

- `Overview`：查看内核状态、服务状态、WebUI 入口、配置路径和日志状态。
- `Core`：首次安装、上传安装、检查更新、一键升级、修复重装或删除 OxiDNS 内核二进制。
- `Configuration`：查看、保存和校验配置文件。
- `Logs`：查看运行日志，支持刷新、暂停，以及关键词和 plugin / qtype / rcode 字段过滤。
- `Settings`：设置 core repository、bundle、代理、配置路径、工作目录和内核持久化开关。

## 默认路径

- 二进制（运行时）：`/usr/bin/oxidns`（指向持久化副本的符号链接）
- 二进制（持久化）：`/etc/oxidns/core/oxidns`
- WebUI（运行时）：`/usr/share/oxidns/webui`（符号链接）
- WebUI（持久化）：`/etc/oxidns/core/webui`
- 配置：`/etc/oxidns/config.yaml`
- 工作目录：`/var/lib/oxidns`
- 服务脚本：`/etc/init.d/oxidns`

## 升级与删除

升级 OxiDNS 内核有两种方式：

- 在 LuCI 的 `Core` 页面点击 `Check Update`，如有新版本点击 `Update to vX.Y.Z`，LuCI 会下载最新 release、校验 SHA256、替换内核并重启服务。
- 继续使用 OxiDNS 自带的升级能力（内核 WebUI / API / CLI）。

升级 LuCI 插件时，下载新版本 `luci-app-oxidns` 包并重新安装即可。LuCI 页面内不提供插件自升级。

删除 OxiDNS 内核可以在 LuCI 的 `Core` 页面点击 `Remove Core`。该操作会停止并禁用服务，删除 `/usr/bin/oxidns`、`/usr/share/oxidns/webui` 和持久化目录 `/etc/oxidns/core`，但保留 `/etc/oxidns/config.yaml` 和 `/var/lib/oxidns`。

## 固件升级（sysupgrade）行为

安装本插件后，`/etc/oxidns` 与 `/etc/config/oxidns` 会被追加到 `/etc/sysupgrade.conf`。在 LuCI 升级页面勾选“保留配置”执行固件升级时：

- 配置文件 `/etc/oxidns/config.yaml`、UCI 设置和持久化的内核二进制 / WebUI 都会保留。
- 固件自身会清掉插件文件（LuCI 页面、init 脚本等），升级完成后**重新安装一次本插件的 ipk/apk**，postinst 会自动重建 `/usr/bin/oxidns` 符号链接，服务恢复运行，无需重新下载内核或重新粘贴配置。
- 如果不勾选“保留配置”，所有数据都会被清除，这是 OpenWrt 机制本身的行为。

低闪存设备（overlay 剩余空间小于内核包体积）可以在 `Settings` 中关闭“Persist core across upgrades”，此时安装行为与原版一致（直接写入 `/usr/bin`，升级后丢失）。

## 私有仓库与下载

路由器需要能直接访问 GitHub Releases 和 release archive。私有仓库或受限网络环境可以在 `Settings` 中配置 GitHub token 或下载代理。Token 和代理值保存后不会在 LuCI 页面中回显，需要清除时请勾选对应清除选项。配置下载代理时需要安装 `curl`，否则 LuCI 会返回明确错误；也可以在 `Core` 页面手动上传 release archive 或二进制进行离线安装。

## 已知限制

- 目前只支持 OxiDNS 已发布的 Linux musl release target。
- 固件升级后插件自身文件会被清除（OpenWrt sysupgrade 机制），需要重新安装一次本插件；持久化的内核与配置会自动恢复，无需重新下载。
- `Overview` 的 WebUI 入口根据配置文件中的 HTTP listen 地址生成；如果监听 `127.0.0.1`，LuCI 会保留链接并提示需要本机访问或 SSH 隧道。
- 日志页读取 OpenWrt `logread` 中的 OxiDNS 服务 stdout/stderr 输出。
- 可在 `Settings` 中关闭“写入系统日志”（默认开启），保存后需重启 OxiDNS 才会生效。LuCI 会停止服务，等待旧 procd 实例被移除（最多 10 秒），再启动新实例，以应用 stdout/stderr 设置。状态查询失败或停止超时时，重启会中止。替换正在运行的内核前也会执行相同的停止等待。
- `Overview` 上的服务操作（Start / Stop / Restart / Enable / Disable）始终会向系统日志写入一条以 `luci-app-oxidns` 为 tag 的审计记录，包含动作和成功或失败结果。内核未安装、init 脚本缺失或操作失败时同样会记录。这些记录可在 `logread` 原始输出和 LuCI 系统日志页看到；由于日志页只显示 `oxidns[PID]:` 格式的内核输出，审计记录不会出现在 OxiDNS 日志页。
