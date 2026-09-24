---
title: "VRP操作系统"
date: 2026-04-23 11:50:12
permalink: "net/vrp操作系统/"
updated: 2026-04-23 11:50:12
categories:
  - "计算机网络"
tags:
  - "VRP"
  - "数通"
  - "网络设备"
description: "VRP系统概念 路由、交换机、无线、安全设备等都可以通过命令行配置。一个常见的安装在网络设备上的操作系统是隶属于华为操作系统平台的vrp操作系统，接下来所有示范均为vrp操作系统。目前盒式设备使用vr"
cover: /img/covers/cover-019.jpg
top_img: false
---

## VRP系统概念

路由、交换机、无线、安全设备等都可以通过命令行配置。一个常见的安装在网络设备上的操作系统是隶属于华为操作系统平台的vrp操作系统，接下来所有示范均为vrp操作系统。目前盒式设备使用vrp操作系统的v5版本，框式设备则使用v8版本

### 文件系统

vrp系统内的主要文件类型有以下几种

1. **系统软件**

   以.cc结尾，启动系统必须的文件
2. **配置文件**

   通常以.zip .cfg等结尾，保存配置的文件
3. **补丁文件**

   .pat结尾，补丁文件，作用跟名字一致
4. **PAF文件**

   .bin结尾，控制系统功能

### 存储结构

网络设备也有类似个人pc的cpu、内存等硬件，一些重要硬件如下

- **SD-RAM**

  设备的运行内存
- **Flash**

  非易失存储器，是设备的默认存储空间，也是系统文件配置文件等文件存储的地方
- **NVRAM**

  用于日志缓存文件
- **SD Card**

  外置存储卡
- **usb接口**

  外接大容量存储设备，主要用于设备升级，数据传输等

### 启动过程

设备上电之后，首先运行 BootROM 初始化硬件，接着加载系统软件 .cc 文件，从默认路径加载配置文件，完成设备初始化。**所有网络设备初始登录都需要密码验证**

## VRP系统管理方式

1. 通过命令的方式对设备进行维护和管理，这种方式能够精细化控制 ，但需要熟练掌握命令行，对操作人员要求较高。通常通过console口（本地）或者telnet和ssh（远程）来连接系统并管理设备
2. 通过web网管方式管理，这种方式的优点是有图形化操作界面，能更加直观的管理设备

## 命令行管理

### 级别

登录系统的用户根据级别拥有不同的权限

- **0 参观级**

  简单命令排查，基本无法配置
- **1 监控级**

  系统命令查看
- **2 配置级**

  对路由、层次进行配置等
- **3-15 管理级**

  配置，配置的删除下载之类

### VRP视图

- **用户视图**

  刚进入系统时的视图，提示符为尖括号，如`<huawei>`

  用户视图只能进行一些最基本的操作，如查看运行状态和统计信息，无法对设备进行配置

  通过`system-view`命令可以进入系统视图
- **系统视图**

  系统视图的提示符为方括号，如`[huawei]`

  在系统视图里可以进行修改设备名以及一些系统参数的配置，通过`interface`进入接口视图，或其他命令进入协议视图、配置AAA、路由协议等
- **接口视图**

  接口视图的提示符如`[huawei-GE0/0/0]`

  > GE / GigabitEthernet表示千兆以太网接口，速率1 Gbit/s
  >
  > 10GE / Ten-GigabitEthernet表示万兆以太网接口，速率 10 Gbit/s
  >
  > XGigabitEthernet / XGE也表示万兆口，速率 10 Gbit/s

  可以专门配置某个接口的ip地址、开关等
- **协议视图**

## 部分命令介绍

用户视图下一些命令

| 命令 | 作用 |
| --- | --- |
| pwd | 显示路径 |
| dir | 查看目录下文件信息 |
| cd | 切换目录 |
| mkdir | 创建目录 |
| rmdir | 删除目录 |
| copy | 复制 |
| rename | 重命名 |
| delete | 删除 |
| undelete | 撤销删除 |
| move | 移动 |

其他命令

```text
display

display version 查看当前设备信息
display current-configuration 当前配置
display interface 接口
display startup 已保存信息
```
```text
sysname ar0 修改设备名为ar0
interface GigabitEthernet 0/0/0 进入接口 
ip address 192.168.10.3 24 配置当前接口ip地址和掩码
display this 查看当前视图配置

        save 保存配置
        undo ip address 撤销配置 在哪里配置在哪里撤销
        shutdown 关闭接口
        reset saved-configuration 
        startup save-configuration 指定下次启动的配置文件

        ? 可以提示命令，tab自动补全
```

## telnet远程登录

telnet是应用层协议，使用tcp传输，端口号23。telnet采用明文传输，安全性差，一般在安全的情况下使用

### 被登录方配置

登录设备需要验证身份，有密码验证和AAA验证两种方式。密码验证：密码存储在设备上，无需用户名只需密码即可登录。aaa验证：需要用户名和密码，密码可以存储在服务器上

- **AAA认证**

```text
    aaa进入aaa视图
    local-user huawei password cipher 1235 创建用户密码
                      privilege level 等级
                      sveice-type telnet 
    authentication-mode aaa 设为aaa认证模式(在vty配置中）
```

- **密码认证**

```text
    user-interface vty 0 4 进入vty管理后台
    authentication-mode password 认证模式为密码
    set authentication password cipher xxxxx 设置登录密码
    user privilege level 3 给予权限3
    idle-timeout 5 超时5分钟未操作需重新登录
    protocol inbound telnet 绑定telnet协议
    telnet server port 1234 修改默认端口，增加安全性
```

### 登陆方

```text
telnet ip port
interface loopback 0 登录后设置环回地址
```
