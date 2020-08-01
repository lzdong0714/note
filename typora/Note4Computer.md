



#### 网络传输协议

| **层级** | **名称**   | **包含的协议**                                               |
| -------- | ---------- | ------------------------------------------------------------ |
| 7        | 应用层     | 例如HTTP、SMTP、SNMP、FTP、Telnet、SIP、SSH、NFS、RTSP、XMPP、Whois、ENRP |
| 6        | 表示层     | 例如XDR、ASN.1、SMB、AFP、NCP                                |
| 5        | 会话层     | 例如ASAP、TLS、SSH、ISO 8327 / CCITT X.225、RPC、NetBIOS、ASP、Winsock、BSD sockets |
| 4        | 传输层     | 例如TCP、UDP、RTP、SCTP、SPX、ATP、IL                        |
| 3        | 网络层     | 例如IP、ICMP、IGMP、IPX、BGP、OSPF、RIP、IGRP、EIGRP、ARP、RARP、 X.25 |
| 2        | 数据链路层 | 例如以太网、令牌环、HDLC、帧中继、ISDN、ATM、IEEE 802.11、FDDI、PPP |
| 1        | 物理层     | 例如线路、无线电、光纤、信鸽                                 |



| **层级** | **名称**   | 功能                                   |
| -------- | ---------- | -------------------------------------- |
| 7        | 应用层     | 文件传输，电子邮件，文件服务，虚拟终端 |
| 6        | 表示层     | 数据格式化，代码转换，数据加密         |
| 5        | 会话层     | 解除或建立与别的结点的联系             |
| 4        | 传输层     | 提供端对端的接口                       |
| 3        | 网络层     | 为数据包选择路由                       |
| 2        | 数据链路层 | 传输有地址的帧以及错误检测功能         |
| 1        | 物理层     | 以二进制数据形式在物理媒体上传输数据   |

`TCP/IP`是传输层协议，主要解决数据如何在网络中传输；而`HTTP`是应用层协议，主要解决如何包装数据。

把`IP`想像成一种`高速公路`，它允许其它协议在上面行驶并找到到其它电脑的出口。`TCP`和`UDP`是高速公路上的`“卡车”`，它们携带的货物就是像`HTTP`，文件传输协议`FTP`这样的协议等。(可以这样理解:`TCP`和`UDP`都是用来传输其他协议的)

而`Socket`是对`TCP/IP`协议的`封装`，`Socket`本身并不是协议，而是一个调用接口（`API`），通过`Socket`，我们才能使用`TCP/IP`协议。

###### **ip地址**

每个`IP地址`包括两个`标识码`（ID），即`网络ID`和`主机ID`。同一个物理网络上的所有主机都使用同一个`网络ID`，网络上的一个主机（包括网络上工作站，服务器和路由器等）有一个`主机ID`与其对应。

`Internet`委员会定义了5种`IP地址`类型以适合不同容量的网络，即A类~E类。
 其中A、B、C3类（如下表格）由`InternetNIC`在全球范围内统一分配，D、E类为特殊地址。



## LINUX

### 系统知识

Filesystem Hierarcy Standard 标准，令常用Linux目录结构一样。

  /usr   /var  /home三个目录区别与使用

|        | 可分享的（可以网络挂载）                 | 不可分享的           |
| ------ | ---------------------------------------- | -------------------- |
| 不变的 | /usr （软件放置处，类是C盘programFiles） | /etc  配置文件       |
|        | /opt                                     | /boot 开机与核心文档 |
| 可变的 | /var/mail                                | /var/run             |
|        |                                          |                      |

以上应该都是归管理员管理，管理员创建的用户应当登录后的当前路径为：/home/user

日常开发测试，应当在/home 下进行，正式部署在另外的层级目录下，避免开发测试的干扰。

### 常用命令

``` shell
ll /proc/PID

cwd符号链接的是进程运行目录；

exe符号连接就是执行程序的绝对路径；

cmdline就是程序运行时输入的命令行命令；

environ记录了进程运行时的环境变量；

fd目录下是进程打开或使用的文件的符号连接。
```

建立新用户，设定用户密码

https://www.runoob.com/linux/linux-user-manage.html

设定后，记得修改用户使用的shell 命令方式：usermod -s /bin/bash username username是建立用户的用户名

### shell

基本帮助指令

```shell
command -options parameter1 parameter2 #基本格式
command --help  #用来查看帮助

#输入指令，部分，连按两次tab，会补全查询所有的指令
ca[tab][tab]

Ctrl + c 中断
Ctrl + d 退出

man [command] #查看指令的操作说明
info [command] #查看在线操作说明
#EP:
date --help
man date
info date

#关机
who #查看链接用户状态
netstat -a #查看网络联机状态
ps -aux #查看后台执行程序状态
sync #数据同步写入硬盘中
shutdown #惯用关机
reboot,halt,poweroff # 重新启动
```

#### 文件

[权限] [连结] [拥有者] [群组] [文件容量] [修改日期]  [文件名]

``` shell
#root 权限下的指令：
	chgrp #change group
    chown #change owner
#user 权限
chmod # change 权限 r：读（1） w:写（2） x：执行（4）
# 三组权限分别事 onwer/group/other
chmod 770 fileName 
chmod u=rwx,g=rwx,o=--- #等价写法1
chmod ug=rwx,o=---
```

文件类型

|      |                          |                            | r            | w            | x                |
| ---- | ------------------------ | -------------------------- | ------------ | ------------ | ---------------- |
| d    | 目录                     |                            | 读到目录名   | 修改目录名   | 进入该目录的权限 |
| -    | 文件                     |                            | 读到文件内容 | 修改文件内容 | 执行文件         |
| l    | 链接文档                 | 类是快捷方式               |              |              |                  |
| b    | 区块设备档（集中在/dev） | 提供系统随机存取的设备     |              |              |                  |
| c    | 字符设备档（集中在/dev） | 串行设备接口               |              |              |                  |
| s    | 资料接口文件socket       |                            |              |              |                  |
| p    | 数据传输文件             | FIFO解决多程序存取文件问题 |              |              |                  |

在权限的目录下，无权访问的文件，可以执行"目录操作"，即删除，移动文件。就是不能打开访问文件。

### Linux查看进程运行的完整路径方法



通过[ps](http://lovesoo.org/tag/ps)及[top](http://lovesoo.org/tag/top)命令查看进程信息时，只能查到[相对路径](http://lovesoo.org/tag/相对路径)，查不到的进程的详细信息，如[绝对路径](http://lovesoo.org/tag/绝对路径)等。这时，我们需要通过以下的方法来查看进程的详细信息：

[Linux](http://lovesoo.org/tag/linux)在启动一个进程时，系统会在/[proc](http://lovesoo.org/tag/proc)下创建一个以PID命名的文件夹，在该文件夹下会有我们的进程的信息，其中包括一个名为exe的文件即记录了[绝对路径](http://lovesoo.org/tag/绝对路径)，通过[ll](http://lovesoo.org/tag/ll)或[ls](http://lovesoo.org/tag/ls) –l命令即可查看。

[ll](http://lovesoo.org/tag/ll) /[proc](http://lovesoo.org/tag/proc)/PID

cwd符号链接的是进程运行目录；

exe符号连接就是执行程序的绝对路径；

cmdline就是程序运行时输入的命令行命令；

environ记录了进程运行时的环境变量；

fd目录下是进程打开或使用的文件的符号连接。