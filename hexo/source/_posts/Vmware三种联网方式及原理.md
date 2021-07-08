---
title: VMware三种联网方式及原理
typora-root-url: ../../source
date: 2021-07-03 09:49:05
tags: 虚拟机
categories: 计算机网络
sticky: 
---

学习spark的standalone模式时，需要基于虚拟机进行模拟集群搭建。所以详细记录相关操作经验及原理。

## 1. VMware三种联网方式及原理

vmware提供三种虚拟机联网方法：bridge桥接模式，NAT网络地址转换模式，Host-only仅主机模式。

- 桥接模式

  > **首先理解“桥接”的概念。**
  >
  > 路由器上一般有多个LAN口，不同LAN口之间就是桥接关系。这个路由器就是“桥”。一个桥设备上有多块网卡，分别处于多个局域网中。桥设备的作用就是学习不同网口连接的mac地址，并在转发帧的时候查询mac地址和LAN口的绑定关系。

  VMware的桥也是一样的道理，采用桥接模式时，VMware会虚拟出一块网卡和真正的物理网卡进行桥接，这样发到物理网卡的所有数据包都到了VMware虚拟机，由虚拟机发出的数据包也会通过桥从物理网卡那端发出。VMware的虚拟机就像局域网中一台独立的主机，可以访问局域网内任意一台机器。

- NAT模式

  VMware的NAT是在主机和虚拟机之间用软件伪造出一块网卡，这块网卡和虚拟机的ip处于同一个地址段，同时和主机的网络接口之间进行NAT。虚拟机相当于一台内网设备，能够ping到主机IP。

- HostOnly模式

  HostOnly模式下，虚拟网络是一个全封闭的网络，唯一能访问的就是主机。

## 2. 不同方式的设置及其他操作

安装完虚拟机之后，会在控制面板-> 网络和共享中心 -> 更改适配器设置 中，新增两块虚拟网卡：

<img src="/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/image-20210703102545051.png" alt="image-20210703102545051" style="zoom:50%;" />

VMware Network Adepter VMnet1：Host用于与Host-Only虚拟网络进行通信的虚拟网卡
VMware Network Adepter VMnet8：Host用于与NAT虚拟网络进行通信的虚拟网卡

### 桥接模式设置

这种模式下，需要手工设置虚拟机的IP地址、子网掩码，必须保持和宿主机处于同一个网段，这样虚拟机才能和宿主机通信（我理解相当于通过设置为同一网段，使具备桥设备的桥接功能）

- 首先确定主机ip设置，ipconfig

  ![image-20210703110322533](/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/image-20210703110322533.png)

  将自动获取IP地址修改为指定ip

  ![image-20210703110405014](/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/image-20210703110405014.png)

- 在VMware->edit->虚拟网络编辑器中修改设置VMnet

  <img src="/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/20180415183840256" alt="img" style="zoom:50%;" />

  <img src="/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/20180415183903429" alt="img" style="zoom:50%;" />

  选择 桥接模式，然后点击 还原默认设置，然后再重新进来，不打开虚拟机，进入设置项，设置为桥接模式。

  ![img](/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/20180415183958794)

  打开虚拟机，进入settings，然后设置ipv4网络

  <img src="/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/image-20210703115033366.png" alt="image-20210703115033366" style="zoom:50%;" />

  ![image-20210703213335139](/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/image-20210703213335139.png)

  这里的192.168.124.7是和主机ip在统一网段的。重新开启网络就可以看到配置已经成功。

  ![image-20210703213435546](/images/Vmware%E4%B8%89%E7%A7%8D%E8%81%94%E7%BD%91%E6%96%B9%E5%BC%8F%E5%8F%8A%E5%8E%9F%E7%90%86/image-20210703213435546.png)

  

