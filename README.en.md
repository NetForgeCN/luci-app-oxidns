# luci-app-oxidns

Language: [中文](./README.md) | English

`luci-app-oxidns` is the LuCI management app for OxiDNS on OpenWrt. After installation, LuCI adds `Services -> OxiDNS` pages for installing the OxiDNS core binary, managing the OpenWrt service, editing configuration, and viewing logs.

This is a re-implementation on top of [svenshi/luci-app-oxidns](https://github.com/svenshi/luci-app-oxidns) that fills in three missing capabilities. Licensed under GPL-3.0-or-later.

## Differences From Upstream

| Capability | Upstream | This repository |
|---|---|---|
| Core binary location | `/usr/bin/oxidns`, lost on firmware upgrade | `/etc/oxidns/core/oxidns` (persistent), runtime path is a symlink |
| Configuration file | lost by default on sysupgrade | added to `/etc/sysupgrade.conf`, kept when "keep settings" is checked |
| Recovery after sysupgrade | re-download core, re-paste config | symlinks auto-rebuilt; reinstall the package once and it works |
| Core upgrades | no LuCI entry, must use the core's own WebUI/API | `Check Update` button on the `Core` page |
| Logs | plain listing | keyword filter plus `plugin` / `qtype` / `rcode` field filters |
| Low-flash devices | — | persistence can be disabled in `Settings` |

## Building From Source

### Option 1: OpenWrt buildroot / SDK (recommended, can be baked into firmware)

```sh
# 1. Prepare an OpenWrt build environment matching your target firmware
git clone https://github.com/openwrt/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a

# 2. Clone this repository as a package
git clone https://github.com/NetForgeCN/luci-app-oxidns.git package/luci-app-oxidns

# 3. Select it in menuconfig (LuCI -> Applications -> luci-app-oxidns)
make menuconfig

# 4. Build
make package/luci-app-oxidns/compile V=s
```

Artifacts land in `bin/packages/<arch>/`: `luci-app-oxidns_*.ipk` and `luci-i18n-oxidns-zh-cn_*.ipk`. Selecting `<*>` in menuconfig bakes the plugin directly into the firmware image, so even the plugin itself survives sysupgrade.

### Option 2: Quick packaging without the SDK

Requires `node`, `tar`, `gzip`, `sha256sum`:

```sh
git clone https://github.com/NetForgeCN/luci-app-oxidns.git
cd luci-app-oxidns
./scripts/build-luci-package.sh 0.2.0 dist
```

Artifacts land in `dist/` (ipk/apk for the main package and the Simplified Chinese translation, plus `sha256sums.txt`).

### Install On The Router

```sh
opkg install luci-app-oxidns_0.2.0-r1_all.ipk
opkg install luci-i18n-oxidns-zh-cn_0.2.0-r1_all.ipk
# or on apk-based OpenWrt:
apk add --allow-untrusted ./luci-app-oxidns_0.2.0-r1_all.apk
```

Then open LuCI → `Services -> OxiDNS`:

1. `Core` page → `Install Core` (auto-selects the release for your CPU architecture and verifies SHA256; use `Upload Core` offline);
2. `Configuration` page → paste/edit `config.yaml` → `Save & Restart`;
3. `Overview` page → `Start` + `Start on boot`.

## Behavior After Installing The Build

- **Persistent core**: the binary and webui live in `/etc/oxidns/core/` on the persistent partition; runtime paths are symlinks. They survive a sysupgrade with "keep settings".
- **Persistent configuration**: `/etc/oxidns` and `/etc/config/oxidns` are appended to `/etc/sysupgrade.conf` on install.
- **Boot self-healing**: init script, rpcd status and package postinst all recreate missing symlinks from the persistent copy. After a firmware upgrade, reinstalling the package once brings the service back — no re-download, no re-pasting.
- **LuCI core updates**: `Check Update` compares the installed version with the latest GitHub release and updates in place (with SHA256 verification).
- **Log filtering**: keyword filter plus `plugin` / `qtype` / `rcode` dropdowns with a live line counter.

## Firmware Upgrade Steps

1. LuCI → System → Backup / Flash Firmware, check **"keep settings"**;
2. Flash the new firmware;
3. Reinstall this package once (the plugin's own files are wiped by sysupgrade — that is how OpenWrt works);
4. Done. postinst rebuilds the core symlinks automatically; configuration is preserved.

Without "keep settings", everything is wiped — that is the OpenWrt mechanism itself.

## Default Paths

- Binary (runtime): `/usr/bin/oxidns` (symlink)
- Binary (persistent): `/etc/oxidns/core/oxidns`
- WebUI (runtime): `/usr/share/oxidns/webui` (symlink)
- WebUI (persistent): `/etc/oxidns/core/webui`
- Config: `/etc/oxidns/config.yaml`
- Working directory: `/var/lib/oxidns` (tmpfs, runtime only)
- Init script: `/etc/init.d/oxidns`

## Known Limitations

- Only published Linux musl release targets of OxiDNS are supported.
- Low-flash devices (less free overlay space than the core bundle) should disable `Persist core across upgrades` in `Settings`.
- The `Overview` WebUI entry is generated from the HTTP listen address; `127.0.0.1` requires local access or an SSH tunnel.
- Private core repositories or restricted networks can use a GitHub token or download proxy in `Settings` (requires `curl`).

## License

GPL-3.0-or-later. Based on [svenshi/luci-app-oxidns](https://github.com/svenshi/luci-app-oxidns) and [svenshi/oxidns](https://github.com/svenshi/oxidns). Thanks to the original author.
