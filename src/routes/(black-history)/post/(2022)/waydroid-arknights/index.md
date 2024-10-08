---
title: 在 Arch Linux 上用 Waydroid 玩舟游
---

# 在 Arch Linux 上用 Waydroid 玩舟游

<vue-metadata author="swwind" time="2022-6-7" tags="waydroid,arknights,archlinux"></vue-metadata>

隔壁的 Anbox 基本已经不维护了，咱还是换到 Waydroid 玩 Arknights 吧。

## 系统说明

博主的目前桌面系统是 Plasma KDE 5.24.5，当前内核最新版本是 5.18.1，如果时间过去许久，本文也许很难再作为参考。

## 内核

要使用 Waydroid 咱们需要使用 `binderfs` 和 `memfd` 这两个内核模块，这两个都在 `linux-zen` (>= 5.18) 包中有提供。

```bash
# 安装 linux-zen 内核
sudo pacman -S linux-zen
# 更新你的 grub（非常重要）
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## 显卡驱动

首先您必须要有一个非 NVIDIA 的显卡，集显也可以，因为 Wayland 在 NVIDIA 驱动下[完全不工作](https://www.youtube.com/watch?v=_36yNWw_07g)。

可以使用 `lspci | grep -E "(VGA|3D)"` 查看当前系统有哪些显卡可以用，本机的输出如下。

```plain
01:00.0 VGA compatible controller: NVIDIA Corporation TU106M [GeForce RTX 2060 Mobile] (rev a1)
06:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Renoir (rev c6)
```

可以看到有一个 NVIDIA 的显卡，而且还有一个 AMD 的核显。

笔者是 AMD 核显 + NVIDIA 独显的配置，因此这里要装下面一些驱动。

```bash
# 这一堆是 AMD 的显卡驱动全家桶
sudo pacman -S mesa xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau
# 这一堆是 NVIDIA 的显卡驱动
sudo pacman -S linux-headers linux-zen-headers
sudo pacman -S nvidia-dkms
```

顺便，如果是使用 AMD 的显卡驱动，可能还需要编辑 `/etc/mkinitcpio.conf` 文件，把 `MODULES` 开头的那一行加一个 `amdgpu`。

```conf
MODULES=(amdgpu)
```

然后对于笔记本的双显卡的情况，咱们使用 `optimus-manager` 来管理双显卡下的驱动。

注意先检查是否有 `/etc/X11/xorg.conf` 这个文件，如果有就删掉。

然后检查 `/etc/X11/xorg.conf.d/` 目录下面是否有其他配置文件，通通删掉。

```bash
sudo pacman -S optimus-manager
sudo systemctl enable --now optimus-manager.service

# 切换到集成显卡模式（禁用独立显卡）
optimus-manager --switch integrated
# 如果要切换到混合模式，可以用 hybrid
# optimus-manager --switch hybrid
# 不过建议先禁用独立显卡，之后重启没问题之后再尝试启用独立显卡
```

之后重启，应该可以正常进入 X11 的桌面系统。

## 安装 Wayland

只需要装一个 `plasma-wayland-session` 就可以了。

```bash
sudo pacman -S plasma-wayland-session
```

安装完成之后登出，再从 DM 中应该可以找到 Plasma KDE (Wayland) 选项，选择这个登录进去。

如果您出现了闪屏、抽风等问题，多半是因为显卡驱动的原因（Wayland 用 NVIDIA 驱动就抽风），尝试用 optimus-manager 禁用独立显卡之后应该可以变正常。

## 安装 Waydroid

分三步，第一步装 `waydroid` (AUR)。

```bash
# 或者别的 AUR 安装工具
pikaur -S waydroid
```

第二步装镜像，可以通过 `waydroid-image` (AUR) 安装，不过可能需要手动修改一下镜像的版本，一般使用最新的构建会比较好，因为咱用老版本他不工作，然后咱手动把版本更新到最新的镜像版本之后就可以了。

或者可以不通过包管理器装镜像，直接跳转到下一步也是可以的。

第三步初始化，直接通过 `waydroid init` 就可以了。

```bash
sudo waydroid init
```

注意如果第二步没有通过包管理器安装镜像的话初始化脚本会自动联网下载最新的构建版本。

如果切换了镜像，或者做了一些别的什么修改，可以加一个 `-f` 来强制重新初始化一遍。

最后启动服务

```bash
sudo systemctl enable --now waydroid-container.service
```

最后直接从桌面启动就可以了。

## 安装 Arknights

通过 Waydroid 给的工具安装 apk 文件。

```bash
waydroid app install Downloads/arknights-hg-1801.apk
```

然后直接在里面打开应该就可以用了。

## Troubleshooting

### Waydroid 内联网失败

编辑 `/etc/dnsmasq.conf`，找到 `#bind-interfaces` 这一行，把他去掉注释。

然后重启 dnsmasq 服务。

```bash
sudo systemctl restart dnsmasq.service
sudo systemctl restart waydroid-container.service
```

### 我明明有双显卡为什么只能检测到一个 NVIDIA 显卡？

笔者电脑是联想拯救者 R7000P，BIOS 中默认开启了独显直连，会导致只能看到 NVIDIA 的显卡信息。

从 BIOS 中把这个选项关掉就可以了。

### 我基建里面怎么没法拖动了啊

不知道，这个貌似是个特性。

### 我肉鸽里面也没法拖动了啊，后面的路线都不到了

不知道，这个貌似也是个特性。

### Anbox 就没有上面的莫名其妙的问题，是不是 Waydroid 不行啊

确实，我也是这么认为的。

~~但是 Anbox 要用的 ashmem 模块在 linux-zen (>= 5.18) 之后就不提供支持了，除了自己编译内核之外可能没有别的选项了~~

<figure>
  <img width="200" src="/assets/sticker.webp" />
</figure>
