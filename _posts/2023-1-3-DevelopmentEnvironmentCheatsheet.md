---
layout: article
title: Development Environment Setup Cheatsheet
excerpt: "A bunch of tools, scripts, walkthroughs to setup environments I use often. E.g. Golang"
categories: []
categories: [tutorial]
tags: [workflow, development]
aside:
  toc: true
---
# Quick Intro
Hello and welcome. Hopefully this helps you if you need to quickly setup an environment using some of this programming languages. This list will continue to grow as I encounter it. This is mostly just for myself. 

# Dotfiles
<a class="button button--primary button--pill" href="https://github.com/dbaseqp/dotfiles">Dotfiles</a>

# Golang Setup
Install `curl` and run the following oneliner.

```bash=
curl -L https://git.io/vQhTU | bash
```

# Set Static IP

## Debian 10+
Edit `/etc/networks/interfaces`:
```
auto ens192
iface ens192 inet static
 address 192.168.1.200
 netmask 255.255.255.0
 gateway 192.168.1.1
 dns-nameservers 8.8.8.8
```

## Ubuntu
Edit `/etc/netplan/<config file>.yaml`:
```
network:
  ethernets:
    ens160:
      dhcp4: no
      addresses:
        - 192.168.1.200/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
  version: 2
```

## RHEL-based
Edit `/etc/sysconfig/network-scripts/ifcfg-ens192`:
```
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.200
GATEWAY=192.168.1.1
PREFIX=24
DNS=8.8.8.8
```
