---
title: "DHCP协议"
date: 2026-04-29 14:36:17
permalink: "net/dhcp协议/"
updated: 2026-04-29 14:36:17
categories:
  - "计算机网络"
tags:
  - "dhcp"
description: "手动配置网络中的各个设备的ip地址会出现参数多、工作量大、理解难等问题。对于上百台主机分别配置ip地址属于重复性劳动且灵活性极差，更换主机办公区域就需要重新配置ip地址，手工配置ip地址对ip地址的利"
cover: /img/covers/cover-016.jpg
top_img: false
---

手动配置网络中的各个设备的ip地址会出现参数多、工作量大、理解难等问题。对于上百台主机分别配置ip地址属于重复性劳动且灵活性极差，更换主机办公区域就需要重新配置ip地址，手工配置ip地址对ip地址的利用率也低下，无法高效利用。利用DHCP协议就可以解决这些问题。

**DHCP：**Dynamic Host Configuration Protocol，动态主机配置协议，是一个应用在局域网中的网络协议，采用C/S架构，使用传输层的UDP协议进行工作，client使用端口68、server使用端口67。DHCP的作用是通过网络动态分配ip地址给主机进行使用。

DHCP优点：

- **统一管理：**  
    
  DHCP服务端记录地址的使用情况，对可用ip地址进行统一管理和分配
- **ip地址利用率高：**  
    
  分配的ip地址具有租期时间的特性，超过租期时间回收

## DHCP工作原理

1. client发送discover广播报文，发现当前网络中的server
2. server发送offer单播报文，携带分配给当前client的ip地址
3. client发送request广播报文，请求使用该ip地址
4. server发送ACK单播报文，告知client可以使用该ip地址

![](/img/wp/1777442487-Snipaste_2026-04-29_14-01-16.png)

通过wireshark抓包可以发现，client明明在接收dhcp-offer后就得到了dhcp-server的mac地址和自己被分配的ip地址，为什么dhcp-request还是使用广播呢？

这是因为在dhcp过程中，client可能收到来自多个dhcp-server的dhcp-offer，并且client无法确认所有的网络设备。为了确保dhcp-server能收到client的确认，client发送的dhcp-request仍然是广播的

获得分配的ip地址后，client会发送arp请求广播，询问网络上是否有其他设备使用相同的IP地址，这是为了避免IP地址冲突，通过这种方式，client也通知其他设备它正在使用该IP地址

以上是一般情况下的客户端服务端交流过程，dhcp协议中还有其他报文

- **dhcp-decline**  
    
  client收到server回应的ACK报文后，通过地址冲突检测发现server分配的地址冲突或由于其它原因导致不能使用，则发送dhcp-declineE报文，通知server所分配的IP地址不可用
- **dhcp-nak**  
    
  server对client的request报文的拒绝响应报文，如果server没有相应的租约记录，就会发送dhcp-nak报文给client
- **dhcp-release**  
    
  client端主动释放server端分配给它的IP是，就会发送dhcp-release报文给server，server收到这个报文后，就会回收这个IP地址
- **dhcp-inform**  
    
  在client已经获得了IP地址，需要从server端获得更详细的配置信息时，就会发送dhcp-inform报文向server请求，server在收到这个报文后，会根据租约查找，找到相应的配置信息后，就会回应dhcp-ack报文给client

### DHCP租期

地址租期过50%时，client将会发送request单播报文给server进行续租，一般地址续租的时间为1天，如果未成功续租，则当地址租期超过87.5%时client将会发送request广播报文寻找新的dhcp-server，进行重绑定。

## DHCP配置命令

路由设备可以将自身当成dhcp服务器，配置命令如下

```text
接口作为地址池下发（dhcp-server与client在同一网段）

dhcp enable 在系统视图下开启dhcp服务

interface GigabitEthernet 0/0/0 进入连接终端设备的接口
ip address   配置连接client的接口的ip地址

dhcp select interface 选择使用该接口
dhcp server dns-list  设置下发的dns服务器地址
dhcp server lease day 0 hour 2 设置租期时间为0天2小时
dhcp server static-bind ip-address  mac-address  对应mac指定分配ip
dhcp server exclued-ip-address  不参与自动分配的地址，可一次性输入多个

#################################################################################
创建地址池下发（dhcp-servr和client不在同一网段）

dhcp enable 在系统视图下开启dhcp服务

ip pool 10 创建名为10的地址池并进入地址池视图
network 192.168.10.0 mask 24 设置地址范围
gatway-list  设置下发的网关地址
server dns-list  设置下发的dns服务器地址
lease day 0 hour 2 设置租期时间为0天2小时
exclued-ip-address  不参与自动分配的地址，可一次性输入多个
static-bind ip-address  mac-address  对应mac指定分配ip

interface GigabitEthernet 0/0/0 进入连接终端设备的接口
ip address   配置连接client的接口的ip地址
dhcp select global 选择使用全局地址池
```

## DHCP中继

在大型网络组网中，dhcp-server与client往往不在同一个网段，要使用一台dhcp-server满足ip地址的下发，需要跨网段发送dhcp报文。dhcp中继就可以满足不同网段间传送dhcp报文的需求，让dhcp报文透明的在dhcp-server和client之间传输。

dhcp中继会对dhcp报文部分字段进行修改，只有最靠近终端设备的一个路由设备算dhcp中继设备，需要配置。中继设备发送给dhcp-server的报文都是单播报文，特殊情况下中继设备回应给client的报文可以是广播和单播任意一种。

### DHCP中继配置命令

```text
dhcp enable

进入接口后
dhcp select relay 使用中继功能
dhcp relay server-ip  选择dhcp服务器

存在多个dhcp服务器可以创建服务器组
dhcp server group 1 名为1的服务器组
dhcp-server  
dhcp-server  在组里添加服务器ip
dhcp relay server-select 1 使用组1

路由器接口使用dhcp只需进入对应接口使用下列命令即可
ip address dhcp-alloc
```

## DHCP Snooping

由于dhcp-server和dhcp-client之间没有认证机制，所以如果在网络上随意添加一台dhcp-server，它就可以为client分配ip地址以及其他网络参数。如果该dhcp-server为用户分配错误的ip地址和其他网络参数，将会对网络造成非常大的危害

DHCP Snooping是dhcp的一种安全特性，用于保证dhcp-client从合法的dhcp-server获取ip地址。DHCP Snooping通过信任功能和绑定表可以抵御网络中针对dhcp的各种攻击

dhcp snooping一般配置在交换机上

```text
dhcp snooping enable ipv4 系统视图下开启snooping功能

进入接口 
dhcp snooping enable 
dhcp snooping trusted  设置信任接口

只有信任接口的dhcp报文会转发，非信任接口的dhcp报文将会丢弃
```
