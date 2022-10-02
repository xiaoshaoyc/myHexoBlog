---
title: 使用Headscale+Tailscale部署Mesh VPN
categories:
  - null
date: 2022-10-02 04:52:48
tags:
---


## 背景
前段时间从Oracle Cloud白嫖了一台Ampere A1的虚拟机，还把家里尘封许久的服务器搬出来组了个on-premise network。就想着能不能把这两个子网用VPN连接起来，组成一个大内网可以互相访问。但是我平时还要去学校上学，又是另一个网域，于是这个问题就变得稍显复杂——三个不同地区的内网要互相能够访问。

同学一开始给我推荐了`Tailscale`，这个软件的功能完全符合我的预期，但是美中不足在于免费账号的限制比较严格，只允许1user，20个设备，和1个子网，而我这边有两个子网。

之后又查到它的竞品`Twingate`，此产品允许两个subnet，而且默认部署为subnet的网关，操作比`Tailscale`更为简单。但是我还是有小小的一点不满意（捂脸），因为它免费版本只允许1台终端设备连接（可以简单的理解城在`Twingate`中是分server和client的，server作为网络入口和出口，client只作为网络入口）。而且说不定我以后会把腾讯云、GCP、AWS的机器并入内网，为了扩展性考虑我还是决定再做一些搜索。

[`Tinc`](https://github.com/gsliepen/tinc)是我找到的第三个软件，免费开源，磁盘占用和运存占用都很小。它有点类似于p2p下载器（如迅雷）的感觉，在设备加入一个VPN网络后可以自动发现VPN网络中其他peer。但是试用了一下比较麻烦，还要安装TUN/TAP驱动（写到这里突然想到SSTap，曾经是最好的翻墙软件，只可惜作者被警告了，长江后浪推前浪。。。）。另外`Tinc`最后的commit停留在了2021年，似乎是已经停止开发，感觉会跟不上现在技术迭代的速度。

`Wireguard`+[`wg-meshconf`](https://github.com/k4yt3x/wg-meshconf)也是一种解决方案，但是总的来说配置过于复杂而且维护起来不方便，每次添加新的子网都要手动更新所有子网的配置。`Wireguard`团队正在开发的[`wg-dynamic`](https://git.zx2c4.com/wg-dynamic/about/docs/idea.md)是以上方案的代替产品，可以关注一下进展。

此时我了解到，我要解决方案是一套类似于Mesh VPN的系统，于是做了一些更有针对性的搜索。[`netmaker`](https://github.com/gravitl/netmaker/)是**一个很有吸引力的方案，尤其是其宣传的5倍于`tailscale`的吞吐量**。但是受限于其文档较少，配置较为繁琐，再加上我已经很累了，暂时先放一边。以后有机会可以再研究一下。

最后，试着搜索能不能自建`Tailscale`或者`Twingate`的控制主机，结果还真被我找到了[`Headscale`](https://github.com/juanfont/headscale)，开源免费的`Tailscale`主控替代品。按照官方的说法
{% blockquote %}
An open source, self-hosted implementation of the Tailscale control server.
{% endblockquote %}
学习成本低，而且没有子网数量限制，完美！

## 搭建Headscale主控
主要是跟着以下两篇教程走，记得注意一下linux用户权限的[问题](https://github.com/juanfont/headscale/issues/523)。

https://icloudnative.io/posts/how-to-set-up-or-migrate-headscale/#windows

https://github.com/juanfont/headscale/blob/main/docs/running-headscale-linux.md

如果用的是Oracle Cloud的话记得要在Virtual Cloud Networks - Security List里开放端口，还要在机器的iptables把端口打开，不要问我是怎么知道的。 (-1hr)
```shell
sudo iptables -I INPUT -p tcp --dport <端口号> -j ACCEPT
```

## 修改Tailscale客户端配置
Headscale使用Tailscale的客户端进行连接，但是需要一些魔改。

Windows需要修改注册表
https://github.com/juanfont/headscale/blob/main/docs/windows-client.md

安卓系统会稍微麻烦一些，需要下载单独的软件
https://github.com/juanfont/headscale/blob/main/docs/android-client.md

Linux-cli 在安装完成后可以使用以下指令连接
```shell
tailscale up --login-server=http://your.server.address:port_num --accept-routes=true --accept-dns=false

# 如果想将tailscale作为网关，定义一个子网的话
# https://tailscale.com/kb/1019/subnets/
tailscale up --login-server=http://your.server.address:port_num --accept-routes=true --accept-dns=false --advertise-routes=XXX.XXX.XXX.XXX/X
```
也可以在这里找到所有的指令 https://tailscale.com/kb/1080/cli/

## TODO
安装、配置Headscale的[UI](https://github.com/gurucomputing/headscale-ui)


## 杂项
https://www.reddit.com/r/WireGuard/comments/inn8sl/wireguard_mesh_network_options/

https://tailscale.com/blog/how-tailscale-works/

https://www.wireguard.com/netns/#routing-network-namespace-integration

http://linux-ip.net/html/routing-tables.html

