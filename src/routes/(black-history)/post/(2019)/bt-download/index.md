---
title: BT 下载指南
---

# BT 下载指南

<vue-metadata author="swwind" time="2019-10-12" tags="bt"></vue-metadata>

记录一下本人在使用 BT 下载时候遇到过的坑。

## 迅雷

我最先了解到的还是迅雷，如果不管它被人骂死的吸血行为，其拥有的离线下载功能还是很不错的。

这个离线加速下载其实就是他们会把一个种子的文件预先下载到自己的服务器上，然后通过直接从迅雷的服务器处下载从而加速传统的 P2P 下载。换句话说，就是有一个专门为迅雷用户上传的巨大的 leech，这对于非迅雷用户来讲是很不受欢迎的，但是正因为有这个 leech 在，一些没有速度的老种子才有可能可以在迅雷上下载成功。

但是你不氪金就不要想用这个了（~~以及开车~~）。

不支持 Linux 平台，如果你仍然想用，可以安装 AUR 里的 `deepin-wine-thunderspeed`。

## BitComet

比特彗星，我第二个长期使用的 BT 下载工具。他在一定程度上和迅雷一样封闭和商业化。

比特彗星有一个和迅雷差不多的优点，他的种子库是使用自己的服务的，所以索引特别快。

只支持 Windows 系统。

## qBittorrent

这是我要重点推荐的一款 BT 下载软件。

- [x] 跨平台
- [x] 开源
- [x] 纯正简洁

### 安装

```bash
sudo pacman -S qbittorrent
```

qBittorrent 并不能开箱就用，在下载之前需要稍微配置一下。

首先我们需要一张能用的 Tracker 列表，可以从 [ngosang/trackerslist][trackerslist] 中复制。

接着打开 qBittorrent 的 Tools > Preferences > BitTorrent

这个选项面板的最下面有一个 Automatically add these trackers to new downloads，把它选上，然后在下面的框框里面粘贴复制来的 Trackers，最后点 OK。

### 下载

直接打开 magnet 链接或者种子文件就行了。

如果不出所料，qBittorrent 会陷入无止境的 Retrieving metadata 的状态，不用担心，选择好保存地址之后点击 OK 开始。

接着选中这个 Retrieving metadata 的下载任务，点击下面的 Trackers，你会发现 qBittorrent 在一个一个尝试能用的 Tracker，或者发现了一个能用但是没有人的 Tracker 然后吊死在了那里。

这时候就需要手动救援了，选中一个 Not contacted yet 或者 Not working 的 Tracker，然后点击一下 Trackers 右边的一个向上的按钮（虽然我并不知道那个按钮到底是啥），qBittorrent 就会同时向所有 Tracker 发起请求。经过这样一轮的扫描，能够通讯的 peer 应该会多不少，Retrieving metadata 也应该马上会变成 Downloading。

接着就是享受满带宽下载的快感了。

（~~然后发现是满带宽上传~~）

## 其他下载工具

- `aria2`：命令行的 BT 下载工具，可惜没速度。
- μTorrent：？
- transmission：？
- ktorrent：？（KDE 原生的？？？）
- fragments：？（给 GNOME 的？？？）

[trackerslist]: https://github.com/ngosang/trackerslist
