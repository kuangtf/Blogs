# Zookeeper

## 1、Zookeeper快速入门

### 1.1、Zookeeper 概述

- Zookeeper 是一个基于观察者模式设计的分布式服务管理框架，负责存储和管理数据，接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper 就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper1.jpg" style="zoom:67%;" />

### 1.2、Zookeeper 的特点

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/Zookeeper2.jpg" style="zoom:80%;" />

1）：Zookeeper ：一个领导者 （Leader），多个跟随者 （Follower）组成的集群。

2）：集群中只要有半数以上节点存活，Zookeeper 集群就能正常服务，所以Zookeeper 适合安装奇数台服务器。

3）：全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。

4）：更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。

5）：数据更新原子性，一次数据更新要么成功，要么失败。

6）：实时性，在一定时间范围内，Client能读到最新数据。

### 1.3、数据结构

- Zookeeper 数据模型的结构与Unix 文件系统很类似，像一个分布式的小文件存储系统，整体上可以看作是一棵树，每个节点称做一个 ZNode。每一个 ZNode 默认能够存储 1MB 的数据，每个 ZNode 都可以通过其路径唯一标识。  

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/Zookeeper3.jpg" style="zoom:80%;" />

### 1.4、应用场景

- 提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等 。

- 同一服务命名：在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。例如：IP不容易记住，而域名容易记住。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper4.jpg" style="zoom:80%;" />

- 统一配置管理
    - 分布式环境下，配置文件同步非常常见：1、一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群。2、对配置文件修改之后，希望能够快速同步到各个节点上。
    - 配置管理可交由Zookeeper实现：1、可将配置信息写入Zookeeper上的一个Znode。2、各个客户端服务器监听这个Znode。3、一旦Znode中的数据被修改，Zookeeper将通知各个客户端服务器。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper5.jpg" style="zoom:80%;" />

- 同一集群管理
    - 分布式环境中，实时掌握每个节点的状态是必要的：可根据节点实时状态做出一些调整。
    - Zookeeper可以实现实时监控节点状态变化：1、可将节点信息写入Zookeeper上的一个Znode。2、监听这个Znode可获取它的实时状态变化。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper6.jpg" style="zoom:67%;" />

- 服务器动态上下线

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper7.jpg" style="zoom:67%;" />

- 负载均衡：在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户daunt请求

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper8.jpg" style="zoom:67%;" />

## 2、Zookeeper操作

### 2.1、简单操作：

1、启动Zookeeper：

```
[root@192 zookeeper-3.5.7]# bin/zkServer.sh start
```

2、查看进程是否启动

```
[root@192 zookeeper-3.5.7]# jps
102109 Jps
20286 QuorumPeerMain
```

3、查看状态

```
[root@192 zookeeper-3.5.7]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: standalone
```

4、启动客户端

```
[root@192 zookeeper-3.5.7]# bin/zkCli.sh
```

5、退出客户端

```
[zk: localhost:2181(CONNECTED) 0] quit
```

6、停止Zookeeper

```
[root@192 zookeeper-3.5.7]# bin/zkServer.sh stop
```

### 2.2、配置参数解读

- Zookeeper中的配置文件zoo.cfg中参数含义解读如下  ：
    - tickTime = 2000： 通信心跳时间， Zookeeper服务器与客户端心跳时间，单位毫秒  
    - initLimit = 10： LF初始通信时限  ：Leader和Follower初始连接时能容忍的最多心跳数（tickTime的数量）  
    - syncLimit = 5： LF同步通信时限  ：Leader和Follower之间通信时间如果超过syncLimit * tickTime， Leader认为Follwer死
        掉，从服务器列表中删除Follwer  
    - dataDir： 保存Zookeeper中的数据  。注意： 默认的tmp目录，容易被Linux系统定期删除，所以一般不用默认的tmp目录。  
    - clientPort = 2181：客户端连接端口，通常不做修改。  

## 3、Zookeeper 集群操作  

### 3.1、集群操作

#### 3.1.1、集群安装

- 集群规划：在 hadoop102、 hadoop103 和 hadoop104 三个节点上都部署 Zookeeper。  

    - 思考：如果是 10 台服务器，需要部署多少台 Zookeeper？  

- 解压安装  

    1、在 hadoop102 解压 Zookeeper 安装包到/opt/module/目录下  ：

```
[192@hadoop102 software]$ tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C /opt/module/
```

​		2、修改 apache-zookeeper-3.5.7-bin 名称为 zookeeper-3.5.7  

```
[192@hadoop102 module]$ mv apache-zookeeper-3.5.7-bin/zookeeper-3.5.7
```

- 配置服务器编号  

    1、在/opt/module/zookeeper-3.5.7/这个目录下创建 zkData  

```
[192@hadoop102 zookeeper-3.5.7]$ mkdir zkData
```

​		2、在/opt/module/zookeeper-3.5.7/zkData 目录下创建一个 myid 的文件  

```
[192@hadoop102 zkData]$ vi myid
```

​		在文件中添加与 server 对应的编号（注意：上下不要有空行，左右不要有空格）  

```
2
```

​		注意： 添加 myid 文件，一定要在 Linux 里面创建，在 notepad++里面很可能乱码  

​		3、拷贝配置好的 zookeeper 到其他机器上  

```
[192@hadoop102 module ]$ xsync zookeeper-3.5.7
```

​		并分别在 hadoop103、 hadoop104 上修改 myid 文件中内容为 3、 4  

- 配置zoo.cfg文件  

    1、重命名/opt/module/zookeeper-3.5.7/conf 这个目录下的 zoo_sample.cfg 为 zoo.cfg  

```
[192@hadoop102 conf]$ mv zoo_sample.cfg zoo.cfg
```

​		2、打开 zoo.cfg 文件  

```
[192@hadoop102 conf]$ vim zoo.cfg
```

​		\#修改数据存储路径配置  

```
dataDir=/opt/module/zookeeper-3.5.7/zkData
```

​		\#增加如下配置  

```
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
```

​		3、配置参数解读  

```
server.A=B:C:D。
```

A 是一个数字，表示这个是第几号服务器；
集群模式下配置一个文件 myid， 这个文件在 dataDir 目录下，这个文件里面有一个数据就是 A 的值， Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是哪个 server。
B 是这个服务器的地址；
C 是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口；
D 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的
Leader，而这个端口就是用来执行选举时服务器相互通信的端口。  

​		4、同步 zoo.cfg 配置文件  

```shell
[192@hadoop102 conf]$ xsync zoo.cfg
```

- 集群操作  

    1、分别启动 Zookeeper  

```
[192@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh start
[192@hadoop103 zookeeper-3.5.7]$ bin/zkServer.sh start
[192@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh start
```

​		2、查看状态

```
[192@hadoop102 zookeeper-3.5.7]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: follower
[192@hadoop103 zookeeper-3.5.7]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: leader
[192@hadoop104 zookeeper-3.4.5]# bin/zkServer.sh status
JMX enabled by default
Using config: /opt/module/zookeeper-3.5.7/bin/../conf/zoo.cfg
Mode: follower
```

#### 3.1.2 选举机制

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper9.jpg" style="zoom:67%;" />

- 第一次启动
    - 服务器1启动，发起一次选举。服务器1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器1状态保持为 LOOKING；
    - 服务器2启动，在发起一次选举。服务器1和2分别投自己一票并交换选票信息：此时服务器1发现服务器2的myid比自己目前投票推举的（服务器1）大，更改选票为推举服务器2。此时服务器1票数0票，服务器2票数2票，没有半数以上结果，选举无法完成，服务器1和2保持状态LOOKING。
    - 服务器3启动，发起一次选举。此时服务器1和2都会更改选票为服务器3.此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票，此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING。
    - 服务器4启动，发起一次选举。此时服务器1,2,3已经不是LOOKING状态，不会更改选票信息。交换选票结果：服务器3位3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOING。
    - 服务器5启动，同4一样当小弟。

> SID：服务器ID。用来唯一标识一台Zookeeper集群中的机器，每台机器不能重复，和myid一致。
>
> ZXID：事务ID。ZXID是一个事务ID，用来表示一次服务器状态的变更。在某一时刻，集群中的每台机器的ZXID值不一定完全一致，这和Zookeeper服务器对于客户端“更新请求”的处理逻辑有关。
>
> Epoch：每个Leader任期的代号。没有Leader时同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper10.jpg" style="zoom: 67%;" />

- 非第一次启动
    - 当Zookeeper集群中的一台服务器出现以下两种情况之一时，就会开始进入Leader选举：
        - 服务器初始化启动
        - 服务器运行期间无法和Leader保持连接
    - 而当一台机器进入Leader选举流程时，当前集群也可能会处于以下两种状态：
        - 集群中本来就已经存在一个Leader：机器试图去选举Leader时，会被告知当前服务器的Leader信息，对于该机器来说， 仅仅需要和Leader机器建立连接，并进行状态同步即可。
        - 集群中确实不存在Leader：假设Zookeeper由5台服务器组成，SID分别为1、2、3、4、5，ZXID分别为8、8、8、7、7，并且此时SID为3的服务器是Leader。某一时刻3和5服务器出现故障，因此开始进行Leader选举。选举规则：1、EPOCH大的直接胜出，2、EPOCH相同，事务id大的胜出，3、事务id相同，服务器id大的胜出。

#### 3.1.3 ZK 集群启动停止脚本  

- 在 hadoop102 的/home/ktf/bin 目录下创建脚本  

```
[192@hadoop102 bin]$ vim zk.sh
```

​		在脚本中编写如下内容  

```shell
#!/bin/bash
case $1 in
"start"){
	for i in hadoop102 hadoop103 hadoop104
	do
		echo ---------- zookeeper $i 启动 ------------
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh
start"
	done
};;
"stop"){
	for i in hadoop102 hadoop103 hadoop104
	do
		echo ---------- zookeeper $i 停止 ------------
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh
stop"
	done
};;
"status"){
	for i in hadoop102 hadoop103 hadoop104
	do
		echo ---------- zookeeper $i 状态 ------------
		ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh
status"
	done
};;
esac
```

- 增加脚本执行权限  

```
[192@hadoop102 bin]$ chmod u+x zk.sh
```

- Zookeeper 集群启动脚本  

```
[192@hadoop102 module]$ zk.sh start
```

- Zookeeper 集群停止脚本  

```
[192@hadoop102 module]$ zk.sh stop
```

### 3.2、客户端命令行操作  

#### 3.2.1、命令行基本语法

| 命令基本语法 | 功能描述                                       |
| ------------ | ---------------------------------------------- |
| help         | 显示所有操作命令                               |
| ls path      | 使用 ls 命令来查看当前 znode 的子节点 [可监听] |
|              | -w 监听子节点变化                              |
|              | -s 附加次级信息                                |
| create       | 普通创建                                       |
|              | -s 含有序列                                    |
|              | -e 临时（重启或者超时消失）                    |
| get path     | 获得节点的值 [可监听]                          |
|              | -w 监听节点内容变化                            |
|              | -s 附加次级信息                                |
| set          | 设置节点的具体值                               |
| stat         | 查看节点状态                                   |
| delete       | 删除节点                                       |
| deleteall    | 递归删除节点                                   |

- 启动客户端

```
[192@hadoop102 zookeeper-3.5.7]$ bin/zkCli.sh -server hadoop102:2181
```

- 显示所有操作命令

```
[zk: hadoop102:2181(CONNECTED) 1] help
```

#### 3.2.2 znode 节点数据信息  

- 查看当前znode中所包含的内容  

```
[zk: hadoop102:2181(CONNECTED) 0] ls /
[zookeeper]
```

- 查看当前节点详细数据  

```
[zk: hadoop102:2181(CONNECTED) 5] ls -s /
[zookeeper]cZxid = 0x0     // 创建节点的事务 zxid
ctime = Thu Jan 01 08:00:00 CST 1970   //znode 被创建的毫秒数（从 1970 年开始）
mZxid = 0x0   //   znode 最后更新的事务 zxid
mtime = Thu Jan 01 08:00:00 CST 1970   // znode 最后修改的毫秒数（从 1970 年开始）
pZxid = 0x0   //  znode 最后更新的子节点 zxid
cversion = -1   //   znode 子节点变化号， znode 子节点修改次数
dataVersion = 0   //  znode 数据变化号
aclVersion = 0   //  znode 访问控制列表的变化号
ephemeralOwner = 0x0   //  如果是临时节点，这个是 znode 拥有者的 session id。如果不是临时节点则是 0。
dataLength = 0   //  znode 的数据长度
numChildren = 1    //  znode 子节点数量
```

#### 3.2.3 节点类型（持久/短暂/有序号/无序号）  

- 持久（Persistent）：客户端和服务器断开连接后，创建的节点不删除
- 短暂（Ephemeral）：客户端和服务器断开连接后，创建的节点自己删除

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper11.jpg" style="zoom:80%;" />

- 持久化目录节点：客户端与Zookeeper断开连接后，该节点依旧存在
- 持久化顺序编号目录节点：客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给 该节点名称进行顺序编号
- 临时目录节点：客户端与Zookeeper断开连接后，该节点被删除
- 临时顺序编号目录节点：客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。

> 说明：创建Znode时设置顺序表示，Znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护。
>
> 注意：在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序。

1、分别创建2个普通节点（永久节点 + 不带序号）  

```
[zk: localhost:2181(CONNECTED) 3] create /sanguo "diaochan"
Created /sanguo
[zk: localhost:2181(CONNECTED) 4] create /sanguo/shuguo "liubei"
Created /sanguo/shuguo
```

> 注意：创建节点时， 要赋值  

2、获得节点的值

```
[zk: localhost:2181(CONNECTED) 5] get -s /sanguo
diaochan
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000003
mtime = Wed Aug 29 00:03:23 CST 2018
pZxid = 0x100000004
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1

[zk: localhost:2181(CONNECTED) 6] get -s /sanguo/shuguo
liubei
cZxid = 0x100000004
ctime = Wed Aug 29 00:04:35 CST 2018
mZxid = 0x100000004
mtime = Wed Aug 29 00:04:35 CST 2018
pZxid = 0x100000004
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

3、创建带序号的节点（永久节点 + 带序号）  

- 先创建一个普通的根节点/sanguo/weiguo  

```
[zk: localhost:2181(CONNECTED) 1] create /sanguo/weiguo "caocao"
Created /sanguo/weiguo
```

- 创建带序号的节点

```
[zk: localhost:2181(CONNECTED) 2] create -s /sanguo/weiguo/zhangliao "zhangliao"
Created /sanguo/weiguo/zhangliao0000000000

[zk: localhost:2181(CONNECTED) 3] create -s /sanguo/weiguo/zhangliao "zhangliao"
Created /sanguo/weiguo/zhangliao0000000001

[zk: localhost:2181(CONNECTED) 4] create -s /sanguo/weiguo/xuchu "xuchu"
Created /sanguo/weiguo/xuchu0000000002
```

> 如果原来没有序号节点，序号从 0 开始依次递增。 如果原节点下已有 2 个节点，则再排序时从 2 开始，以此类推。  

4、创建短暂节点（短暂节点 + 不带序号 or 带序号）  

- 创建短暂的不带序号的节点  

```
[zk: localhost:2181(CONNECTED) 7] create -e /sanguo/wuguo "zhouyu"
Created /sanguo/wuguo
```

- 创建短暂的带序号的节点  

```
[zk: localhost:2181(CONNECTED) 2] create -e -s /sanguo/wuguo "zhouyu"
Created /sanguo/wuguo0000000001
```

- 在当前客户端是能查看到的  

```
[zk: localhost:2181(CONNECTED) 3] ls /sanguo
[wuguo, wuguo0000000001, shuguo]
```

- 退出当前客户端然后再重启客户端  

```
[zk: localhost:2181(CONNECTED) 12] quit
[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkCli.sh
```

- 再次查看根目录下短暂节点已经删除  

```
[zk: localhost:2181(CONNECTED) 0] ls /sanguo
[shuguo]
```

- 修改节点数据值  

```
[zk: localhost:2181(CONNECTED) 6] set /sanguo/weiguo "simayi"
```

#### 3.2.4 监听器原理  

- 客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、节点删除、子目录节点增加删除）时， ZooKeeper 会通知客户端。监听机制保证 ZooKeeper 保存的任何的数据的任何改变都能快速的响应到监听了该节点的应用程序  

- 监听器详解：
    - 首先要有一个main（）线程
    - 在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信（connet），一个负责监听（listener）。
    - 通过connect线程将注册的监听事件发送给Zookeeper。
    - 在Zookeeper的注册监听列表中将注册监听事件添加到列表中。
    - Zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程。
    - listener线程内部调用了process（）方法。
- 常见的监听
    - 监听节点数据的变化：get path [watch]
    - 监听子节点增减的变化：ls path [watch]

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper12.jpg" style="zoom:80%;" />

1、节点的值变化监听

- 在hadoop104 主机上注册监听/sanguo 节点数据变化  

```
[zk: localhost:2181(CONNECTED) 26] get -w /sanguo
```

- 在 hadoop103 主机上修改/sanguo 节点的数据  

```
[zk: localhost:2181(CONNECTED) 1] set /sanguo "xisi
```

- 观察 hadoop104 主机收到数据变化的监听  

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged
path:/sanguo
```

> 注意：在hadoop103再多次修改/sanguo的值， hadoop104上不会再收到监听。因为注册一次，只能监听一次。想再次监听，需要再次注册  

2、节点的子节点变化监听（路径变化）  

- 在 hadoop104 主机上注册监听/sanguo 节点的子节点变化  

```
[zk: localhost:2181(CONNECTED) 1] ls -w /sanguo
[shuguo, weiguo]
```

- 在 hadoop103 主机/sanguo 节点上创建子节点  

```
[zk: localhost:2181(CONNECTED) 2] create /sanguo/jin "simayi"
Created /sanguo/jin
```

- 观察 hadoop104 主机收到子节点变化的监听  

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged
path:/sanguo
```

> 注意： 节点的路径变化，也是注册一次，生效一次。想多次生效，就需要多次注册。  

#### 3.2.5 节点删除与查看  

- 删除节点  

```
[zk: localhost:2181(CONNECTED) 4] delete /sanguo/jin
```

- 递归删除节点  

```
[zk: localhost:2181(CONNECTED) 15] deleteall /sanguo/shuguo
```

- 查看节点状态  

```
[zk: localhost:2181(CONNECTED) 17] stat /sanguo
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000011
mtime = Wed Aug 29 00:21:23 CST 2018
pZxid = 0x100000014
cversion = 9
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 1
```

### 3.3 客户端向服务端写数据流程

- 写流程之写入请求直接发送给Leader节点

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper13.jpg" style="zoom:80%;" />

- 写流程之写入请求发送给follower节点

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper14.jpg" style="zoom:80%;" />



## 4、ZooKeeper 分布式锁

### 1、概述

- 什么叫做分布式锁呢？
    - 比如说"进程 1"在使用该资源的时候，会先去获得锁， "进程 1"获得锁以后会对该资源 保持独占，这样其他进程就无法访问该资源， "进程 1"用完该资源以后就将锁释放掉，让其他进程来获得锁，那么通过这个锁机制，我们就能保证了分布式系统中多个进程能够有序的访问该临界资源。那么我们把这个分布式环境下的这个锁叫作分布式锁。  

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper15.jpg" style="zoom:67%;" />

### 2、原生 Zookeeper 实现分布式锁案例  

- 原生 Zookeeper 实现分布式锁案例  

```java
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import java.io.IOException;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;
public class DistributedLock {
    // zookeeper server 列表
    private String connectString ="hadoop102:2181,hadoop103:2181,hadoop104:2181";
    // 超时时间
    private int sessionTimeout = 2000;
    private ZooKeeper zk;
    private String rootNode = "locks";
    private String subNode = "seq-";
    // 当前 client 等待的子节点
    private String waitPath;
    //ZooKeeper 连接
    private CountDownLatch connectLatch = new CountDownLatch(1);
    //ZooKeeper 节点等待
    private CountDownLatch waitLatch = new CountDownLatch(1);
    // 当前 client 创建的子节点
    private String currentNode;
    // 和 zk 服务建立连接，并创建根节点
    public DistributedLock() throws IOException,
    InterruptedException, KeeperException {
        zk = new ZooKeeper(connectString, sessionTimeout, new
                           Watcher() {
                               @Override
                               public void process(WatchedEvent event) {
                                   // 连接建立时, 打开 latch, 唤醒 wait 在该 latch 上的线程
                                   if (event.getState() ==
                                       Event.KeeperState.SyncConnected) {
                                       connectLatch.countDown();
                                   }
                                   // 发生了 waitPath 的删除事件
                                   if (event.getType() ==
                                       Event.EventType.NodeDeleted && event.getPath().equals(waitPath))
                                   {
                                       waitLatch.countDown();
                                   }
                               }
                           });
        // 等待连接建立
        connectLatch.await();
        //获取根节点状态
        Stat stat = zk.exists("/" + rootNode, false);
        //如果根节点不存在，则创建根节点，根节点类型为永久节点
        if (stat == null) {
            System.out.println("根节点不存在");
            zk.create("/" + rootNode, new byte[0],
                      ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        }
    }
    // 加锁方法
    public void zkLock() {
        try {
            //在根节点下创建临时顺序节点，返回值为创建的节点路径
            currentNode = zk.create("/" + rootNode + "/" + subNode,
                                    null, ZooDefs.Ids.OPEN_ACL_UNSAFE,
                                    CreateMode.EPHEMERAL_SEQUENTIAL);
            // wait 一小会, 让结果更清晰一些
            Thread.sleep(10);
            // 注意, 没有必要监听"/locks"的子节点的变化情况
            List<String> childrenNodes = zk.getChildren("/" +
                                                        rootNode, false);
            // 列表中只有一个子节点, 那肯定就是 currentNode , 说明
            client 获得锁
                if (childrenNodes.size() == 1) {
                    return;
                } else {
                    //对根节点下的所有临时顺序节点进行从小到大排序
                    Collections.sort(childrenNodes);
                    //当前节点名称
                    String thisNode = currentNode.substring(("/" +
                                                             rootNode + "/").length());
                    //获取当前节点的位置
                    int index = childrenNodes.indexOf(thisNode);
                    if (index == -1) {
                        System.out.println("数据异常");
                    } else if (index == 0) {
                        // index == 0, 说明 thisNode 在列表中最小, 当前
                        client 获得锁
                            return;
                    } else {
                        // 获得排名比 currentNode 前 1 位的节点
                        this.waitPath = "/" + rootNode + "/" +
                            childrenNodes.get(index - 1);
                        // 在 waitPath 上注册监听器, 当 waitPath 被删除时,
                        zookeeper 会回调监听器的 process 方法
                            zk.getData(waitPath, true, new Stat());
                        //进入等待锁状态
                        waitLatch.await();
                        return;
                    }
                }
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    // 解锁方法
    public void zkUnlock() {
        try {
            zk.delete(this.currentNode, -1);
        } catch (InterruptedException | KeeperException e) {
            e.printStackTrace();
        }
    }
}
```

- 分布式锁测试  

```java
import org.apache.zookeeper.KeeperException;
import java.io.IOException;
public class DistributedLockTest {
    public static void main(String[] args) throws
        InterruptedException, IOException, KeeperException {
        // 创建分布式锁 1
        final DistributedLock lock1 = new DistributedLock();
        // 创建分布式锁 2
        final DistributedLock lock2 = new DistributedLock();
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock1.zkLock();
                    System.out.println("线程 1 获取锁");
                    Thread.sleep(5 * 1000);
                    lock1.zkUnlock();
                    System.out.println("线程 1 释放锁");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock2.zkLock();
                    System.out.println("线程 2 获取锁");
                    Thread.sleep(5 * 1000);
                    lock2.zkUnlock();
                    System.out.println("线程 2 释放锁");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

- 输出：

```
线程 1 获取锁
线程 1 释放锁
线程 2 获取锁
线程 2 释放锁
```

### 3、Curator 框架实现分布式锁

- 原生的 Java API 开发存在的问题 ：
    - 会话连接是异步的，需要自己去处理。比如使用 CountDownLatch  
    - Watch 需要重复注册，不然就不能生效  
    - 开发的复杂性还是比较高的  
    - 不支持多节点删除和创建。需要自己去递归  
- Curator 是一个专门解决分布式锁的框架，解决了原生 Java API 开发分布式遇到的问题  
- Curator 案例实操  ：

```java
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessLock;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.ExponentialBackoffRetry;

public class CuratorLockTest {
    private String rootNode = "/locks";
    // zookeeper server 列表
    private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
    // connection 超时时间
    private int connectionTimeout = 2000;
    // session 超时时间
    private int sessionTimeout = 2000;
    public static void main(String[] args) {
        new CuratorLockTest().test();
    }
    // 测试
    private void test() {
        // 创建分布式锁 1
        final InterProcessLock lock1 = new
            InterProcessMutex(getCuratorFramework(), rootNode);
        // 创建分布式锁 2
        final InterProcessLock lock2 = new
            InterProcessMutex(getCuratorFramework(), rootNode);
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock1.acquire();
                    System.out.println("线程 1 获取锁");
                    // 测试锁重入
                    lock1.acquire();
                    System.out.println("线程 1 再次获取锁");
                    Thread.sleep(5 * 1000);
                    lock1.release();
                    System.out.println("线程 1 释放锁");
                    lock1.release();
                    System.out.println("线程 1 再次释放锁");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 获取锁对象
                try {
                    lock2.acquire();
                    System.out.println("线程 2 获取锁");
                    // 测试锁重入
                    lock2.acquire();
                    System.out.println("线程 2 再次获取锁");
                    Thread.sleep(5 * 1000);
                    lock2.release();
                    System.out.println("线程 2 释放锁");
                    lock2.release();
                    System.out.println("线程 2 再次释放锁");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    // 分布式锁初始化
    public CuratorFramework getCuratorFramework (){
        //重试策略，初试时间 3 秒，重试 3 次
        RetryPolicy policy = new ExponentialBackoffRetry(3000, 3);
        //通过工厂创建 Curator
        CuratorFramework client =
            CuratorFrameworkFactory.builder()
            .connectString(connectString)
            .connectionTimeoutMs(connectionTimeout)
            .sessionTimeoutMs(sessionTimeout)
            .retryPolicy(policy).build();
        //开启连接
        client.start();
        System.out.println("zookeeper 初始化完成...");
        return client;
    }
}
```

- 观察控制台变化：  

```
线程 1 获取锁
线程 1 再次获取锁
线程 1 释放锁
线程 1 再次释放锁
线程 2 获取锁
线程 2 再次获取锁
线程 2 释放锁
线程 2 再次释放锁
```

## 5、Zookeeper源码分析

### 1、Paxos算法

- Paxos算法：一种基于消息传递且具备高度容错特性的一致性算法。
- Paxos算法解决的问题：就是如何快速正确的在一个分布式系统中对某个数据值达成一致，并且保证无论发生任何异常都不会破坏整个系统的一致性。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper16.jpg" style="zoom:67%;" />

- 在一个Paxos系统中，首先将所有节点划分为Proposer（提议者），Acceptor（接收者），和Learner（学习者）。（注意：每个节点都可以身兼数职）

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper17.jpg" style="zoom:67%;" />

- 一个完整的Paxos算法流程分为三个阶段：
    - Prepare准备阶段：Proposer向多个Acceptor发出Propose请求Promise（承诺），Acceptor针对收到的Propose请求进行Promise（承诺）
    - Accept接受阶段：Proposer收到多数Acceptor承诺的Promise后，向Acceptor发出Propose请求，Acceptor针对收到的Propose请求进行Accept处理
    - Learn学习阶段：Propose将形成的决议发送给所有Learners

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper18.jpg" style="zoom:67%;" />

- 算法流程
    - Prepare: Proposer生成全局唯一且递增的Proposal ID,向所有Acceptor发送Propose请求，这里无需携带提案内容，只携带Proposal ID即可。
     - Promise:Acceptor收到Propose请求后，做出“两个承诺，一个应答”。
          - 不再接受Proposal ID小于等于(注意：这里是<=)当前请求的Propose请求。
         - 不再接受Proposal ID小于(注意：这里是< )当前请求的Accept请求。
          - 不违背以前做出的承诺下，回复已经Accept过的提案中Proposal ID最大的那个提案的Value和Proposal ID,没有则返回空值。
     -  Propose: Proposer收到多数Acceptor的Promise应答后，从应答中选择Proposal ID最大的提案的Value,作为本次要发起的提案。如果所有应答的提案Value均为空值，则可以自己随意决定提案Value。然后携带当前Proposal ID,向所有Acceptor发送     Propose请求。
    -  Accept: Acceptor收到Propose请求后，在不违背自己之前做出的承诺下，接受并持久化当前Proposal ID和提案Value。
    - Learn: Proposer收到多数Acceptor的Accept后，决议形成，将形成的决议发送给所有Learner。
- 下面我们针对上述描述做三种情况的推演举例：为了简化流程，我们这里不设置 Learner。  

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper19.jpg" style="zoom:80%;" />

- 情况1：有A1、A2、A3、A4、A5  5位议员，就税率问题进行决议。
     - A1发起1号Proposal的Propose，等待Promise承诺；  
     - A2-A5回应Promise
     - A1在收到两份回复时就会发起税率10%的Proposal；
     - A2-A5回应Accept
     - 通过Proposal，税率10%。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper20.jpg" style="zoom:80%;" />

- 情况2：现在我们假设在A1提出提案的同时，A5决定将税率定为20%
    - A1， A5同时发起Propose（序号分别为1， 2）
    - A2承诺A1， A4承诺A5， A3行为成为关键
    - 情况1： A3先收到A1消息，承诺A1。
    - A1发起Proposal（1， 10%）， A2， A3接受。
    - 之后A3又收到A5消息， 回复A1： （1， 10%），并承诺A5。
    - A5发起Proposal（2， 20%）， A3， A4接受。之后A1， A5同时广播决议。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper21.jpg" style="zoom:80%;" />

- 现在我们假设在A1提出提案的同时, A5决定将税率定为20%
    - A1， A5同时发起Propose（序号分别为1， 2）
    - A2承诺A1， A4承诺A5， A3行为成为关键
    - 情况2： A3先收到A1消息，承诺A1。之后立刻收到A5消息，承诺A5。
    - A1发起Proposal（1， 10%），无足够响应， A1重新Propose （序号3）， A3再次承诺A1。
    -  A5发起Proposal（2， 20%），无足够相应。 A5重新Propose （序号4）， A3再次承诺A5。
    - ……
- 造成这种情况的原因是系统中有一个以上的 Proposer，多个 Proposers 相互争夺 Acceptor，造成迟迟无法达成一致的情况。 针对这种情况，一种改进的 Paxos 算法被提出：从系统中选出一个节点作为 Leader，只有 Leader 能够发起提案。 这样，一次 Paxos 流程中只有一个Proposer，不会出现活锁的情况，此时只会出现例子中第一种情况。  

- Paxos 算法缺陷：在网络复杂的情况下，一个应用 Paxos 算法的分布式系统，可能很久无法收敛，甚至陷入活锁的情况。  

### 2、ZAB 协议  

- 什么是 ZAB 算法 ：Zab 借鉴了 Paxos 算法，是特别为 Zookeeper 设计的支持崩溃恢复的原子广播协议。基于该协议， Zookeeper 设计为只有一台客户端（Leader）负责处理外部的写事务请求，然后Leader 客户端将数据同步到其他 Follower 节点。 即 Zookeeper 只有一个 Leader 可以发起提案 。

- Zab 协议包括两种基本的模式： 消息广播、 崩溃恢复 。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper22.jpg" style="zoom:67%;" />

#### 2.1、消息广播

- 客户端发起一个写操作请求。
-  Leader服务器将客户端的请求转化为事务Proposal 提案， 同时为每个Proposal 分配一个全局的ID， 即zxid。
-  Leader服务器为每个Follower服务器分配一个单独的队列， 然后将需要广播的 Proposal依次放到队列中去， 并且根据FIFO策略进行消息发送。
- Follower接收到Proposal后， 会首先将其以事务日志的方式写入本地磁盘中， 写入成功后向Leader反馈一个Ack响应消息。
-  Leader接收到超过半数以上Follower的Ack响应消息后， 即认为消息发送成功， 可以发送commit消息。
- Leader向所有Follower广播commit消息， 同时自身也会完成事务提交。 Follower 接收到commit消息后， 会将上一条事务提交。
- Zookeeper采用Zab协议的核心， 就是只要有一台服务器提交了Proposal， 就要确保所有的服务器最终都能正确提交Proposal。

- ZAB协议针对事务请求的处理过程类似于一个两阶段提交过程
    - 广播事务阶段
    - 广播提交操作
- 这两阶段提交模型如下， 有可能因为Leader宕机带来数据不一致， 比如
    - Leader 发 起 一 个 事 务Proposal1 后 就 宕 机 ， Follower 都 没 有Proposal1
    - Leader收到半数ACK宕机，没来得及向Follower发送Commit 
- 怎么解决呢？ 
    - ZAB引入了崩溃恢复模式。

#### 2.2、崩溃恢复  

##### 崩溃恢复——异常假设  

- 一旦Leader服务器出现崩溃或者由于网络原因导致Leader服务器失去了与过半 Follower的联系，那么就会进入**崩溃恢复模式**。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper23.jpg" style="zoom:80%;" />

-  假设两种服务器异常情况：
    - 假设一个事务在Leader提出之后， Leader挂了。
    -  一个事务在Leader上提交了， 并且过半的Follower都响应Ack了， 但是Leader在Commit消息发出之前挂了。
- Zab协议崩溃恢复要求满足以下两个要求：
    - 确保已经被Leader提交的提案Proposal， 必须最终被所有的Follower服务器提交。 （已经产生的提案， Follower必须执行）
    - 确保丢弃已经被Leader提出的， 但是没有被提交的Proposal。 （丢弃胎死腹中的提案）

##### 崩溃恢复——Leader选举

- 崩溃恢复主要包括两部分： **Leader选举**和**数据恢复**。

<img src="https://gitee.com/kuangtf/blogImage/raw/master/img/zookeeper24.jpg" style="zoom:67%;" />

- **Leader选举**： 根据上述要求， Zab协议需要保证选举出来的Leader需要满足以下条件：
    - 新选举出来的Leader不能包含未提交的Proposal。 即新Leader必须都是已经提交了Proposal的Follower服务器节点。
    - 新选举的Leader节点中含有最大的zxid。 这样做的好处是可以避免Leader服务器检查Proposal的提交和丢弃工作。  

##### 崩溃恢复——数据恢复

- 崩溃恢复主要包括两部分： Leader选举和数据恢复。
- Zab如何数据同步：
    - 完成Leader选举后， 在正式开始工作之前（接收事务请求， 然后提出新的Proposal） ， Leader服务器会首先确认事务日
        志中的所有的Proposal 是否已经被集群中过半的服务器Commit。
    - Leader服务器需要确保所有的Follower服务器能够接收到每一条事务的Proposal， 并且能将所有已经提交的事务Proposal应用到内存数据中。 等到Follower将所有尚未同步的事务Proposal都从Leader服务器上同步过， 并且应用到内存数据中以后，Leader才会把该Follower加入到真正可用的Follower列表中。  

### 3、CAP

- CAP理论告诉我们， 一个分布式系统不可能同时满足以下三种
    - 一致性（C:Consistency）
    - 可用性（A:Available）
    - 分区容错性（ P:Partition Tolerance）
- 这三个基本需求， 最多只能同时满足其中的两项， 因为P是必须的， 因此往往选择就在CP或者AP中。
- 一致性（ C:Consistency）
    - 在分布式环境中， 一致性是指数据在多个副本之间是否能够保持数据一致的特性。 在一致性的需求下， 当一个系统在数据一致的状态下执行更新操作后， 应该保证系统的数据仍然处于一致的状态。
- 可用性（A:Available）
    - 可用性是指系统提供的服务必须一直处于可用的状态， 对于用户的每一个操作请求总是能够在有限的时间内返回结果。
- 分区容错性（ P:Partition Tolerance）
    - 分布式系统在遇到任何网络分区故障的时候， 仍然需要能够保证对外提供满足一致性和可用性的服务， 除非是整个网络
        环境都发生了故障。
- ZooKeeper保证的是CP
    - ZooKeeper不能保证每次服务请求的可用性。 （注：在极端环境下， ZooKeeper可能会丢弃一些请求， 消费者程序需要
        重新请求才能获得结果） 。 所以说， ZooKeeper不能保证服务可用性。
    - 进行Leader选举时集群都是不可用  。



> 来自尚硅谷的学习笔记！

