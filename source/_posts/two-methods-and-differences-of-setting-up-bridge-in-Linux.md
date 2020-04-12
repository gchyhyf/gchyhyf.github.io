---
title: LINUX下两种设置网桥的方法与区别
description: ovs-vsctl是openvswitch中提供的命令，它创建的网桥通常是自己的私有网桥；在配置文件中当做一个网卡进行设置;而brctl是LINUX本身的网桥管理命令.
date: 2020-03-15 23:28:00
categories:
- [运维,网络]
tags:
- 网桥
- 虚拟网桥
- 虚拟网卡
---

# 1.ovs-vsctl
ovs-vsctl是openvswitch中提供的命令，它创建的网桥通常是自己的私有网桥；在配置文件中当做一个网卡进行设置，例如如下配置中，br-ex就像一个独立网卡一样，但实际上它是由ovs-vsctl创建的：
```shell
test@cloundvm:~$ cat /etc/netplan/50-cloud-init.yaml
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eno2:
            dhcp4: false
        eno3:
            dhcp4: false
        eno4:
            dhcp4: false       
        eno1:
            dhcp4: false
        br-ex:
            dhcp4: false        
            addresses:
            - 192.168.1.154/24
            gateway4: 192.168.1.1
            nameservers: 
                addresses:
                - 114.114.114.114
                - 223.5.5.5
                - 223.6.6.6
    version: 2
```
>注：openstack正是使用了此种方式
# 2.brctl
brctl是LINUX本身的网桥管理命令，它的设置示例如下,明确指定了br0是网桥：
```shell
network:
    ethernets:
        enp0s3:
            dhcp4: false
            dhcp6: false
            #addresses:
    bridges:
        br0:
            interfaces: [enp0s3]
            dhcp4: false
            dhcp6: false
            addresses:
            - 192.168.1.126/24
            #netmask: 255.255.255.0
            gateway4: 192.168.1.1
            nameservers:
                addresses:
                - 114.114.114.114
                - 223.5.5.5
                - 223.6.6.6
    version: 2
```
# 3.进一步测试理解两者之间的关系
## 3.1首先创建ovs-bridge
```shell
ovs-vsctl add-br br-int
ovs-vsctl add-br br-tun
ovs-vsctl add-port br-int patch-int -- set Interface patch-int type=patch -- set Interface patch-int options:peer=patch-tun
ovs-vsctl add-port br-tun patch-tun -- set Interface patch-tun type=patch -- set Interface patch-tun options:peer=patch-int
```
## 3.2创建linux bridge

创建网桥brtest：

```shell
brctl addbr brtest
ip link set brtest up
```
创建虚拟网卡vbtest
```shell
ip link add brtest type veth peer name vbtest
```
linux虚拟网卡vbtest附加到网桥brtest
```shell
brctl addif brtest vbtest
```
## 3.3 将linux虚拟网卡vbtest附加(连接)到openswitch网桥br-int
```shell
ovs-vsctl add-port br-int vbtest
ovs-vsctl set port vbtest tag=100
ip link set vbtest up
ip link set brtest up
ovs-vsctl show
brctl show
```
>这说明：ovs-vsctl创建的网桥应该与外部的物理网卡（上面一开始是物理网卡）或者虚拟网卡连接

本文运行测试环境：unbuntu 18.04