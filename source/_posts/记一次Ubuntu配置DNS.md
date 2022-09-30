---
title: 记一次Ubuntu配置DNS
categories:
  - null
date: 2022-09-30 04:24:17
tags:
---

## 起
最近玩[NextDNS](https://nextdns.io/zh)玩的很嗨，想让所有的设备都走`DNS over TLS`和`DNS over HTTPS`。在学校的时候发现不能用，后来想起来是学校只允许用内部的DNS服务器，把公网上的全部屏蔽了，有点不爽，以后看看有没有什么办法绕过这个限制，但这个是以后的事情了，今天先讲Ubuntu上碰到的问题。

首先查到`systemd-resolve`这个服务支持`DOT`，它在`127.0.0.53`的53端口上运行。于是修改`/etc/systemd/resolved.conf`配置文件，主要修改了以下三行
```conf
[Resolve]
DNS=45.90.28.0#custom--name-xxxxxx.dns1.nextdns.io
DNS=45.90.30.0#custom--name-xxxxxx.dns2.nextdns.io
DNSOverTLS=yes
```
在重启服务后（`sudo systemctl restart systemd-resolved`），发现DNS无法查询。具体出错信息为 `resolve call failed: All attempts to contact name servers or networks failed`。

## 承
通过`resolvectl`了解到出口网卡(ens160)的DNS指向`192.168.100.1`，即网关。最初判断是此网卡因为某些原因DNS指向没有被修改，后来了解到这个是通过`DHCP`动态获取到的DNS服务器。尝试把DNS改为`127.0.0.53`（通过`/etc/netplan/xxxx.yaml`），无果。大量查阅了资料，尝试了各种可能（我今天就是要用上systemd-resolve里的DOT（恼）），后来能解析出结果，我还以为是终于成功了，但是看了一下NextDNS的日志是空的。emmm，something went wrong，继续和这个问题死磕。

## 转
尝试看一下`systemd-resolved`的日志，根据[这篇教程](https://unix.stackexchange.com/a/432077)把log改成debug级别，发现内部报错为`Failed to invoke gnutls_handshake: Error in the certificate verification.`

Google搜索报错信息后线索来到了这个[systemd仓库中的issue](https://github.com/systemd/systemd/issues/16531)，说是有bug，在`systemd version 246`中得到修复。我查了一下我的systemd版本，嗯。。。那么整件事情基本上清楚了。
```shell
$ systemctl --version
systemd 245 (245.4-4ubuntu3.15)
+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-hierarchy=hybrid
```

## 合
手动升级systemd太麻烦（其实是我不会，于是我打算直接升级`Ubuntu 22.04`版本，运行指令`sudo do-release-upgrade`。升级完成后恢复正常，在NextDNS中可以看到查询日志。（记得要把`systemd-resolved`的debug模式改回来


## 伏笔
运行`resolvectl`查看DNS信息，可以看到Current DNS Server显示为`192.168.100.1`(DHCP中动态获取的)，但是NextDNS也确实收到了查询的请求，难道是进入了Fallback？
```TEXT
$ resolvectl 
Global
         Protocols: -LLMNR -mDNS +DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub
Current DNS Server: 45.90.28.0#homelab-5c1745.dns1.nextdns.io
       DNS Servers: 45.90.28.0#homelab-5c1745.dns1.nextdns.io 45.90.30.0#homelab-5c1745.dns2.nextdns.io

Link 2 (ens160)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS +DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.100.1
       DNS Servers: 127.0.0.53 192.168.100.1

Link 3 (docker0)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS +DNSOverTLS DNSSEC=no/unsupported
```