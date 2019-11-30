---
layout: post
title: "Build your own router"
---

## What is a router

应对特定场景 / 领域的低成本计算机

### Routing

将数据包传递到目标地址

### Network Address Translation

修改数据包地址

### Dynamic Host Configuration Protocol

动态配置网络参数

### Domain Name Service

域名解析

## Why do it the hard way

1. 需要对网络数据包做动作，而硬件路由无法满足
2. OpenWRT(LEDE)、RouterOS、pfSense、iKuai 各有利弊，很难兼得

## Router hardware

低功耗工控机， Intel 3855u / 4G RAM / 60G SSD

## Router operating system

debian x64

## Configuring network interfaces


### Prepare System

```bash
~$ apt-get update
~$ apt-install openssh-server vim git curl
```
clone confbook to /root

```
~$ git clone https://github.com/song940/confbook.git
```

link apt source

```bash
~$ apt-get install dirmngr
~$ rm -rf /etc/apt
~$ ln -s /root/confbook/apt /etc/apt
~$ rm -rf /var/lib/apt # clean old list
```

link nginx

```bash
~$ apt-get install nginx
~$ useradd nginx
~$ rm -rf /etc/nginx
~$ ln -s /root/confbook/nginx /etc/nginx
```

link network

```bash
~$ rm -rf /etc/network
~$ ln -s /root/confbook/network /etc/network
```

### Static Network

### PPPoE

```bash
~$ apt-get install pppoeconf
```

### Bridge Network

https://wiki.debian.org/BridgeNetworkConnections

```
~$ apt-get install bridge-utils
```

## Enabling NAT

iptables NAT

## Setting up DHCP and DNS

```bash
~$ apt-get install dnsmasq -y
```

## Wireless AP

Unifi AP

### AP Controller

Unifi AP Controller