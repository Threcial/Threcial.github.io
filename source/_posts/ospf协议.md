---
title: "OSPF协议"
date: 2026-05-02 17:42:32
permalink: "net/ospf协议/"
updated: 2026-05-02 17:42:32
categories:
  - "计算机网络"
tags:
  - "ospf"
  - "动态路由协议"
description: "OSPF 全称为 Open Shortest Path First，开放式最短路径优先协议。OSPF 协议属于内部网关协议和链路状态路由协议，使用组播地址224.0.0.5，采用多区域设计，支持大规模"
cover: /img/covers/cover-014.png
top_img: false
---

OSPF 全称为 Open Shortest Path First，开放式最短路径优先协议。OSPF 协议属于内部网关协议和链路状态路由协议，使用组播地址`224.0.0.5`，采用多区域设计，支持大规模网络 ，具备灵活性好、易于扩展、收敛速度快等优点，被广泛应用于企业内网、园区网、运营商内部网络等场景。

## OSPF协议工作原理

在链路状态路由协议中，通告的不是路由表而是链路状态，OSPF 也是如此，它的工作原理为：

1. 运行运行 OSPF 的路由器会通过发送 **hello报文** 发现邻居并建立邻居关系
2. 然后彼此之间进一步建立邻接关系并开始交互 **LSA（链路状态通告）** ，LSA 信息中包含：这台路由器是谁、它有哪些接口、它连接了哪些网段、它连接了哪些邻居、每条链路的 cost 等信息
3. 将收集到的 LSA 信息放入 **LSDB** 表中，同一区域的路由器中 LSDB 是一致的
4. 路由器根据 LSDB 中的拓扑信息通过 **SPF** 最短路径优先算法计算最优路径，将计算结果放入 OSPF 路由表，视情况加入全局路由表中

OSPF 目前有两个版本，v2 主要用于 IPv4，v3 主要用于 IPv6

## OSPF 区域 area

OSPF 通过区域 area 来划分网络。area 可以理解为 OSPF 的逻辑分区，一个 OSPF 网络被拆分为多个区域，每个区域内部独立维护 LSDB。多区域有以下优点：可以组建更大规模的网络，在区域边界设备上可以进行路由汇总减少路由表规模，限制 LSA泛洪 的范围来进行网络优化

area 分为骨干区域 `area0` 和非骨干区域 `非area0`

area0 是 OSPF 的核心区域，OSPF 规定非骨干区域之间的路由信息必须通过骨干区域 area0 传递，这样可以形成清晰的层次化结构，减少环路风险并提高扩展性

OSPF 的区域是配置在接口上的，一台路由器上可以有多个区域的配置

### Router-ID

Router-ID 是一种 OSPF 协议标识符，格式类似 IPv4 地址，用于在该 OSPF 路由域中唯一标识一台路由器的身份，注意不是一个区域而是整个路由域

一般情况下，Router-ID 的选择规则是：

1. 如果手动配置了 Router ID，优先使用手动配置
2. 如果没有手动配置，选择环回接口中最大的 IPv4 地址
3. 如果没有环回接口，选择物理接口中最大的 IPv4 地址

一般情况下都是手动配置 Router-ID ，这样便于管理

配置命令

```text
ospf 进入ospf视图
ospf router-id 2.2.2.2 手动配置Router-ID
reset ospf process 配置完需要重启ospf进程
```

## OSPF 度量值

在 OSPF 中每个接口都维护着一个度量值，计算公式为 `100M/接口带宽`，最小为1

配置命令

```text
进入接口
ospf cost 100 修改cost为100

display ospf interface 可以查看相关信息
```

## OSPF 报文类型

OSPF 有五种报文类型

|  |  |
| --- | --- |
| Hello | 发现和维护邻居关系 |
| DD | 描述本地 LSDB 摘要 |
| LSR | 请求所需的 LSA |
| LSU | 携带具体的 LSA |
| LSAck | 确认收到的 LSA |

## OSPF 状态

OSPF 有七种状态

|  |  |
| --- | --- |
| Down | 双方设备初始情况下没有发送 hello 信息时，处于 down |
| Init | 收到对方 Hello，但对方 Hello 中没有自己的 Router ID |
| 2-Way | Hello 中携带对方的 Router-ID，处于 2-Way 中表示已经建立邻居关系 |
| Exstart | 发送空 DD 报文，双方开始协商主从关系，并确定 DD 报文序列号 |
| Exchange | 双方正式交换 DD 报文，描述自己的 LSDB 摘要，用于同步 |
| Loading | 对缺少的 LSA 发送 LSR 请求，对方使用 LSU 返回具体 LSA，收到后使用 LSAck 确认 |
| Full | 双方 LSDB 同步完成，邻接关系正式建立 |

## OSPF 三大表项

邻居表、LSDB 表、OSPF 路由表

```text
display ospf peer brief 显示邻居表
display ospf lsdb 显示LSDB表
display ospf routing 显示OSPF路由表
```

## OSPF 支持的网络类型

|  |
| --- |
| 广播型多路访问 BMA |
| 非广播型多路访问 NBMA |
| 点到点 P2P |
| 点到多点 P2MP |

```text
在接口中
ospf network-type p2p 修改网络类型为p2p
```

## OSPF 路由器类型

OSPF 路由器类型主要分为以下几种

|  |  |
| --- | --- |
| 区域内路由器 IR | 所有接口都属于同一个非骨干区域 |
| 骨干路由器 BR | 至少有一个接口属于 area0 |
| 边界路由器 ABR | 同时连接多个 area |
| 自治系统边界路由器 ASBR | 将外部路由引入 OSPF 的路由器 |

一台路由器可以同时属于多个类型

```text
ospf视图中
import-route rip 引入rip协议，引入的路由优先级为150

rip协议视图中也能引入ospf
```

## DR BDR 和 DRother

在 ma （多路访问）网络中如果每台设备都建立邻接关系，会导致过多关系存在，造成设备资源损耗、负担增大等问题，所以 OSPF 在 BMA 和 NBMA 网络中引入（P2P和P2MP网络中不存在）：

- **DR，Designated Router，指定路由器**  
    
  代表该网段与其他路由器建立邻接， 负责在该多路访问网段中集中同步 LSA， 减少邻接数量
- **BDR，Backup Designated Router，备份指定路由器**  
    
  作为 DR 的备份，当 DR 故障时，BDR 接替成为新的 DR
- **DRother，非 DR/BDR 路由器**  
    
  与 DR 建立 Full 邻接，与 BDR 建立 Full 邻接，与其他 DRother 只保持 2-Way 邻居关系

每个网段都会产生 DR 和 BDR，DR 和 BDR 通过接口 OSPF优先级 选出（越大越优先），默认为1，0代表不参与选举，优先级相同则选择 Router-ID 较大的。DR 拥有不可抢占原则，选举成功后不可通过优先级修改

```text
在接口中 
ospf dr-priority 10 修改优先级为10
```

## OSPF 虚连接

OSPF 要求所有非骨干区域必须连接到骨干区域，但有时因为网络设计不规范或其他原因，有些非骨干区域无法直接连接骨干区域，这时就可以通过虚连接让两个区域相连

虚连接通常建立在两个 ABR 之间，并且这两个 ABR 必须共同连接到一个普通区域，这个区域叫做中转区域

配置命令

```text
ospf 1
area 1
vlink-peer 2.2.2.2

ospf 1
area 1
vlink-peer 1.1.1.1
```

在相同的中转区域内，两台 ABR 互相配置 `vlink-peer 对端 Router ID`，即可建立 OSPF 虚连接

## OSPF 配置命令

```text
ospf 1 router-id 1.1.1.1 启动ospf并配置Router-ID
area 0 进入区域0
network 192.168.1.0 0.0.0.255 通告ip，注意要用反掩码
display ospf peer brief 
display ospf lsdb
display ospf routing
display ospf interface 查看ospf接口
ospf cost 100 接口内，修改cost
ospf network-type p2p 接口内 修改网络类型
ospf dr-priority 10 接口内，修改DR优先级
reset ospf process 重启ospf进程
```
