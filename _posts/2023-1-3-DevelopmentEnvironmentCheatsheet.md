---
layout: article
title: Development Environment Setup Cheatsheet
excerpt: "A bunch of tools, scripts, walkthroughs to setup environments I use often. E.g. Golang"
categories: [article]
tags: [workflow, development, tutorial]
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

# Generate Pubkey Pair via OpenSSL

1. Generate private key
```bash
openssl genrsa -out private_key.pem 2048
```

2. Generate public key
```bash
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

# Convert Python2 to Python3

1. Address tabs vs space issues
  - In vim,
```
:set tabstop=8    # or 4 depending on file
:set expandtab
:retab
```

2. Go to [https://python2to3.com/](https://python2to3.com/)
3. Repeat step 1 until works

# Configure Copy/Paste for ESXi VMs

1. Shutdown VM
2. Edit VM settings > VM Options > Advanced > Configuration Parameters > Edit Configuration
3. Add the following configuration parameters:

| Name | Value |
| ---- | ----- |
| isolation.tools.copy.disable | FALSE |
| isolation.tools.paste.disable | FALSE |
| isolation.tools.setGUIOptions.enable | TRUE |

4. Click OK
5. Power on VM

# Set Static IP

After configuring static IP, make sure to restart networking service or interface.

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

Edit `/etc/sysconfig/network-scripts/ifcfg-ens192` and add:

```
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.200
GATEWAY=192.168.1.1
PREFIX=24
DNS=8.8.8.8
```
