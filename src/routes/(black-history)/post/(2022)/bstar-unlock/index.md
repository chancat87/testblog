---
title: bilibili 东南亚版本使用指南
---

# bilibili 东南亚版本使用指南

<vue-metadata author="swwind" time="2022-2-20" tags="bilibili" updated="2024-10-30"></vue-metadata>

---

**注意：本文内容已经严重过时，过时内容包括：谷歌已经缩紧了账号注册流程，目前基本不能跳过绑定手机号；在网页中开大会员的功能已经开放（需要使用 PayPal）等。（2024/10/30）**

**注意：目前本人不建议如此曲线救国，而且该站简体中文字幕良莠不齐，不如看看远处的巴哈姆特动画疯。**

---

本文简单讲解 bilibili 东南亚版本的情况，以及简单说明如何使用神秘手段访问其服务的方法。

## 关于 bilibili 东南亚版本

bilibili 东南亚版本是叔叔给泰国以及东南亚的用户单独分离出去的一个站点，主要用于提供泰国以及东南亚的用户使用。为了避免和国内主站叫混，也有时称其为 Bstar。

目前其网站端主域名为 `bilibili.tv`，视频流服务主要使用 `*.akamaized.net` 域名。网页端只需要有东南亚地区的 IP 地址即可访问。

安卓端的 App ID 为 `com.bstar.intl`，可以从 [Google Play](https://play.google.com/store/apps/details?id=com.bstar.intl) 上直接下载。App 中的访问限制比较多，因此解锁访问较为麻烦，下文将会详细介绍解锁过程。

## 网页端的使用

前往新加坡即可直接访问网页端的 bilibili 东南亚版本。如果 IP 地址不在东南亚地区，则会看到白屏。

注意某些软件中 `*.bilibili.tv` 和 `*.akamaized.net` 两个域名可能需要手动设置为经过代理才会生效。

网页端可以访问基本全部内容，只是不能给叔叔钱开大会员。

## 移动端的使用

如果您想要给叔叔送钱开大会员，则需要使用 bilibili 东南亚版本的 App。

首先您需要一台带有 Google Play 框架的安卓手机。

### 注册 Google 小号

**注意：如果您的 Google 帐号没有被锁区（或者没有进行过任何充值），则可以直接尝试切换到新加坡区。但是如果您已经被锁区（或者有可用余额），则不建议使用其一年一次的地区切换功能强行切换到新加坡区，个人建议可以直接注册一个新的帐号当作新加坡区小号来用。**

首先将手机切换成英文，退出原有的 Google 帐号，并且飞往美国，使用美国的 IP 地址进行下面的操作。

点开任意一个 Google 全家桶，然后按照流程注册。注意生日填写 15 岁，这时 Google 会认为你还没有手机，于是不会向您索要手机号码。

这时便可以成功注册一个没有绑定任何手机号码的 Google 小号。注意千万不要忘记关联一个自己的邮箱地址，否则以后将会无法登陆。

### 切换 Play 商店到新加坡区

保持手机语言为英文，飞往新加坡，使用新加坡的 IP 地址进行下面的操作。

设置中选择 Google Play，强制停止（Force stop），然后选择 Clear Storage 与 Clear Cache，这样就可以清除 Google Play 的缓存。

重新打开 Google Play，这时应该可以看到商店变成了新加坡区的内容。

之后直接点击安装 [bilibili](https://play.google.com/store/apps/details?id=com.bstar.intl) 即可。

### 突破访问限制

直接打开 App 也许看到的是一片空白（即使使用新加坡的 IP 地址），这是由于叔叔的技术手段限制，因此我们需要透过一些渠道去绕过它。

按照手机是否 Root 分两种情况，关于如何安装 Magisk 和启用 LSPosed 均不在本文介绍范畴。

1. 我的手机已经 Root，并且已经装好了 LSPosed：

   直接安装这个插件 [com.github.bstartweaks](https://t.me/biliroaming_chat/416377)，启用之后强制停止 bilibili 之后再打开便可以突破访问限制。

2. 我没法 Root：

   访问 Telegram 频道 [@biliroaming_lspatch](https://t.me/biliroaming_lspatch) 并查看带有 `#bstar` 标签的内容，选择适合您的方法进行突破。

突破之后登录就可以直接在手机上看 720p 画质的番剧了，不过要想看更加高清的画质就要给叔叔钱了。
