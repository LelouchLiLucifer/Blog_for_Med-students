# Openwrt 25.x发行版配置指南

后续配置基于使用 `ssh root@IP` 连接软路由。

与先前不同的是，25.x 发行版的 OpenWrt 使用 `.apk` 格式的安装包，并且不再使用 `opkg` 管理软件。

## 安装中文语言包

在 OpenWrt 25.x 的 SSH 终端执行：

```sh
apk update
apk add luci-i18n-base-zh-cn
```

然后刷新 LuCI，进入“系统 → 系统 → 语言与风格”，选择“简体中文”并保存应用。

如果想直接用命令设为中文：

```sh
uci set luci.main.lang='zh_cn'
uci commit luci
/etc/init.d/uhttpd restart
```

若“软件包管理”和“防火墙”等页面仍有英文，可补装对应翻译：

```sh
apk add luci-i18n-package-manager-zh-cn luci-i18n-firewall-zh-cn
```

`luci-i18n-base-zh-cn` 是 OpenWrt 25.x 官方软件源提供的 LuCI 基础简体中文包。

## 磁盘空间扩容

{% hint style="warning" %}
要扩的是 OpenWrt 的可写空间 `/overlay`，不是 `/tmp`。`/tmp` 是内存盘，重启会清空。
{% endhint %}

启动 OpenWrt 后，通过 SSH 执行：

```sh
apk update
apk add parted losetup resize2fs blkid

wget -U "" -O /tmp/expand-root.sh \
"https://openwrt.org/_export/code/docs/guide-user/advanced/expand_root?codeblock=1"

chmod +x /tmp/expand-root.sh
. /tmp/expand-root.sh
sh /etc/uci-defaults/70-rootpt-resize
```

脚本会自动重启两次；重新登录后确认：

```sh
df -h /overlay
```

看到 `/overlay` 容量接近实际分配的磁盘容量即可。OpenWrt 官方 x86 镜像默认根分区只有约 104 MB，磁盘其余空间通常尚未分区使用；官方提供的这个脚本会自动识别根盘并扩展分区与文件系统。详见[官方扩容指南](https://openwrt.org/docs/guide-user/advanced/expand_root)。

{% hint style="warning" %}
若 `lsblk` 显示根盘是 `/dev/nvme0n1`，不要执行上述自动脚本，官方提示其目前存在已知风险；PVE 常见的 `sda` 或 `vda` 虚拟盘则适用。
{% endhint %}

## 给 OpenWrt 添加中国大陆可用的软件源

可以把官方软件仓库切换到中科大镜像；它同步的是官方 OpenWrt 软件包，不会改变系统版本、架构或包签名校验。

OpenWrt 25.x 执行：

```sh
cp /etc/apk/repositories.d/distfeeds.list \
   /etc/apk/repositories.d/distfeeds.list.bak

sed -i 's|https://downloads.openwrt.org|https://mirrors.ustc.edu.cn/openwrt|g' \
  /etc/apk/repositories.d/distfeeds.list

apk update
```

确认源地址已替换：

```sh
cat /etc/apk/repositories.d/distfeeds.list
```

正常情况下，每一行都应以：

```
https://mirrors.ustc.edu.cn/openwrt/
```

开头，且仍保留原本的 `25.12.x` 版本号、`x86_64` 架构和内核版本；注意不要改这些路径。

若需恢复官方源：

```sh
cp /etc/apk/repositories.d/distfeeds.list.bak \
   /etc/apk/repositories.d/distfeeds.list

apk update
```

这只是“官方源的大陆镜像”，不等于加入各种第三方插件源；第三方源必须严格匹配你的 OpenWrt 版本和 `x86_64` 架构，不能随意添加。中科大镜像对 OpenWrt 25.12+ 的 `apk` 配置路径和替换方式有明确说明：[USTC OpenWrt 镜像帮助](https://mirrors.ustc.edu.cn/help/openwrt.html)。

## 安装 Argon 主题

```sh
apk update
cd /tmp
wget -O argon.apk https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.4.7/luci-theme-argon-2.4.7-r1.apk
apk add --allow-untrusted ./argon.apk
```

可选：安装 Argon 的背景、配色等设置界面：

```sh
wget -O argon-config.apk https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.4.7/luci-app-argon-config-2.4.7-r1.apk
apk add --allow-untrusted ./argon-config.apk

wget -O argon-config-zh-cn.apk https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.4.7/luci-i18n-argon-config-zh-cn-26.103.13761.3e099a3.apk
apk add --allow-untrusted ./argon-config-zh-cn.apk
```

完成后，LuCI 里进入“系统 → 系统 → 语言与风格”，主题选择 **Argon**，保存并应用即可。

实际安装时，请到 [Argon Releases](https://github.com/jerrykuku/luci-theme-argon/releases) 选最新版本，并把命令中的版本号和文件名一并替换。

## 安装 [OpenClash](https://github.com/vernesong/openclash)

OpenWrt 25.x 默认应使用 **nftables / firewall4**，按 OpenClash 上游的 APK 安装流程即可。不要安装 `iptables` 那套依赖。

{% stepper %}
{% step %}
### 确认 firewall4 并检查空间

在 OpenWrt SSH 执行：

```sh
command -v fw4
df -h /overlay
```

若第一条显示 `/usr/sbin/fw4`，继续执行。
{% endstep %}

{% step %}
### 安装依赖及 OpenClash APK

```sh
apk update
apk add bash dnsmasq-full curl ca-bundle ip-full ruby ruby-yaml \
  kmod-tun kmod-inet-diag unzip kmod-nft-tproxy \
  luci-compat luci luci-base

curl -fL --retry 2 \
  https://api.github.com/repos/vernesong/OpenClash/releases/latest \
  -o /tmp/openclash_version

download_url="$(jsonfilter -i /tmp/openclash_version \
  -e '@.assets[*].browser_download_url' | grep '\.apk$' | head -n 1)"

curl -fL --retry 2 "$download_url" -o /tmp/openclash.apk

apk add -q --force-overwrite --clean-protected --allow-untrusted \
  /tmp/openclash.apk
```

`dnsmasq-full` 会替换系统自带的精简 `dnsmasq`，这是 OpenClash 的正常依赖变化；安装期间可能短暂中断 DNS。`--allow-untrusted` 是因为 OpenClash 的 APK 来自其上游 Release，而非 OpenWrt 官方签名源。上面的依赖和 APK 安装参数与 [OpenClash 官方 Release 安装说明](https://github.com/vernesong/OpenClash/releases)一致。
{% endstep %}

{% step %}
### 确认架构

```sh
uname -m
```

若结果是 `x86_64`，继续执行。
{% endstep %}

{% step %}
### 下载 Meta 内核

```sh
uci set openclash.config.core_type='Meta'
uci set openclash.config.core_version='linux-amd64-compatible'
uci commit openclash

/usr/share/openclash/openclash_core.sh Meta
```
{% endstep %}

{% step %}
### 验证内核并启动服务

下载完成后验证内核：

```sh
/etc/openclash/core/clash_meta -v
```

能显示 Mihomo 版本号就成功了。随后导入配置后再启动服务：

```sh
/etc/init.d/openclash restart
```
{% endstep %}
{% endstepper %}

若自动下载失败，可让 OpenClash 脚本使用指定内核地址：

```sh
/usr/share/openclash/openclash_core.sh Meta \
"https://raw.githubusercontent.com/vernesong/OpenClash/core/master/meta/clash-linux-amd64-compatible.tar.gz"
```

OpenClash 会将内核放到 `/etc/openclash/core/clash_meta` 并自动验证。详见[上游内核安装脚本](https://github.com/vernesong/OpenClash/blob/master/luci-app-openclash/root/usr/share/openclash/openclash_core.sh)

## 安装 TTYD 终端

OpenWrt 已自带 SSH 命令行；若想在 LuCI 网页中直接使用终端，安装官方 `ttyd`：

```sh
apk update
apk add ttyd luci-app-ttyd

/etc/init.d/ttyd enable
/etc/init.d/ttyd restart
```

刷新 LuCI 后，进入“服务 → 终端”即可打开网页版 Shell。

{% hint style="warning" %}
请保持 ttyd 仅绑定 LAN，不要给它配置 WAN 访问或端口转发；它提供的是 root 权限终端。
{% endhint %}

日常更推荐在 Windows Terminal 中使用
