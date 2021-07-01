https://www.jellythink.com/archives/379

### 单节点

我们先从单机版安装开始，因为我们在搭建开发环境时，都是安装的单机版，所以单机版的安装是运维人员、开发人员必备的技能。

安装单机版主要分为以下几步： 1.下载源码 打开https://redis.io/download网站，下载Redis最新稳定版源码。 2.上传至服务器，解压缩、编译

```
#解压
tar -zxf redis-5.0.3.tar.gz

#进入redis目录
cd redis-5.0.3

#编译
make

#测试编译结果
make test
-------------------
报错:需要安装tcl
yum install tcl -y
然后重新执行make test
-------------------
```

3.启动redis

```
#编译生成的可执行文件都在src目录下
cd src

#指定配置文件运行redis
./redis-server ../redis.conf
```

4.使用自带的redis-cli客户端连接redis-server测试

```
./redis-cli -h 127.0.0.1 -p 6379
```

### 主从模式

一主多从。主机和从机的数据完全一致，主机支持数据的写入和读取等各项操作，而从机则只支持与主机数据的同步和读取，也就是说，客户端可以将数据写入到主机，由主机自动将数据的写入操作同步到从机。主从模式很好的解决了数据备份问题，并且由于主从服务数据几乎是一致的，因而可以将写入数据的命令发送给主机执行，而读取数据的命令发送给不同的从机执行，从而达到读写分离的目的。
我们都知道每个Redis实例都是一个单独的进程，都会有一个对应的进程。为了搭建一主三从这个例子，我准备以下三个配置文件：

| 角色 | 配置文件        | 监听接口 |
| ---- | --------------- | -------- |
| 主   | redis-6379.conf | 6379     |
| 从   | redis-6380.conf | 6380     |
| 从   | redis-6381.conf | 6381     |
| 从   | redis-6382.conf | 6382     |

redis-6379.conf

```
bind 127.0.0.1
port 6379
daemonize yes
logfile "6379.log"
dbfilename "dump-6379.rdb"
```

redis-6380.conf

```
bind 127.0.0.1
port 6380
daemonize yes
logfile "6380.log"
dbfilename "dump-6380.rdb"
slaveof 127.0.0.1 6379
```

redis-6381.conf

```
bind 127.0.0.1
port 6381
daemonize yes
logfile "6381.log"
dbfilename "dump-6381.rdb"
slaveof 127.0.0.1 6379
```

redis-6382.conf

```
bind 127.0.0.1
port 6382
daemonize yes
logfile "6382.log"
dbfilename "dump-6382.rdb"
slaveof 127.0.0.1 6379
```

启动redis

```
#将配置文件放到redis5.0.3/conf目录中
mkdir conf
mv redis-6379.conf conf/
mv redis-6380.conf conf/
mv redis-6381.conf conf/
mv redis-6382.conf conf/

cd src
./redis-server ../conf/redis-6379.conf
./redis-server ../conf/redis-6380.conf
./redis-server ../conf/redis-6381.conf
./redis-server ../conf/redis-6382.conf
```

然后使用redis-cli连接redis实例

```
./redis-cli -h 127.0.0.1 -p 6379
./redis-cli -h 127.0.0.1 -p 6380
./redis-cli -h 127.0.0.1 -p 6381
./redis-cli -h 127.0.0.1 -p 6382
```

分别再4个命令行工具中执行一个get命令,获取键名为website的数据

```
127.0.0.1:6379>get website
(nil)
127.0.0.1:6380>get website
(nil)
127.0.0.1:6381>get website
(nil)
127.0.0.1:6382>get website
(nil)
```

4个redis实例中都不存在键为website的数据,在6379上设置一个键为website的数据

```
127.0.0.1:6379>set website https://www.jellythink.com
127.0.0.1:6379>get website
"https://www.jellythink.com"
```

设置成功,此时在6380、6381、6382实例上执行`get website`命令

```
127.0.0.1:6380>get website
"https://www.jellythink.com"
```

说明redis的主从模式配置成功

### redis sentinel哨兵配置

通过上面配置的最简单的主从模式，我们可以实现读写分离，解决了数据备份和单例可能存在的性能问题，但是也引入了新的问题。由于主从模式配置了四个redis实例，并且每个实例都使用不同的ip（如果在不同的机器上）和端口号，根据前面所述，主从模式下可以将读写操作分配给不同的实例进行从而达到提高系统吞吐量的目的，但也正是因为这种方式造成了使用上的不便，因为每个客户端连接redis实例的时候都是指定了ip和端口号的，如果所连接的redis实例因为故障下线了，而主从模式也没有提供一定的手段通知客户端另外可连接的客户端地址，因而需要手动更改客户端配置重新连接。另外，主从模式下，如果主节点由于故障下线了，那么从节点因为没有主节点而同步中断，因而需要人工进行故障转移工作。

为了解决上面的这个不能自动进行故障转移的问题，在2.8版本之后redis正式提供了sentinel（哨兵）架构。

Sentinel（哨兵）是Redis的高可用性解决方案：**由一个或多个Sentinel实例组成的Sentinel系统可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器**。

Redis的Sentinel系统用于管理多个Redis服务器(instance) 该系统执行以下三个任务：

- 监控(Monitoring): Sentinel会不断地定期检查主服务器和从服务器是否运作正常
- 提醒(Notification): 当被监控的某个Redis服务器出现问题时，Sentinel可以通过API向管理员或者其他应用程序发送通知
- 自动故障迁移(Automaticfailover): 当一个主服务器不能正常工作时，Sentinel会开始一次自动故障迁移操作，它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器; 当客户端试图连接失效的主服务器时，集群也会向客户端返回新主服务器的地址，使得集群可以使用新主服务器代替失效服务器

根据上一节的内容做好redis主从模式配置,再看sentienl

| 角色     | 配置文件            | 监听端口 |
| -------- | ------------------- | -------- |
| sentienl | sentienl-23679.conf | 26379    |
| sentienl | sentienl-23680.conf | 26380    |
| sentienl | sentienl-23681.conf | 26381    |

建议哨兵至少部署3个,并且使用奇数个哨兵。Redis Sentinel的节点数量要满足2n+1(n>=1)的奇数个

sentienl-23679.conf

```
port 26379
bind 127.0.0.1
daemonize yes
dir /usr/local/src/redis-5.0.3/data/sentinel-26379
logfile "/usr/local/src/redis-5.0.3/data/sentinel-26379/sentinel.log"
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

sentienl-23680.conf

```
port 26380
bind 127.0.0.1
daemonize yes
dir /usr/local/src/redis-5.0.3/data/sentinel-26380
logfile "/usr/local/src/redis-5.0.3/data/sentinel-26380/sentinel.log"
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

sentienl-23681.conf

```
port 26381
bind 127.0.0.1
daemonize yes
dir /usr/local/src/redis-5.0.3/data/sentinel-26381
logfile "/usr/local/src/redis-5.0.3/data/sentinel-26381/sentinel.log"
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

配置说明

```
1.sentinel monitor mymaster 127.0.0.1 6379 2
sentinel监控的master的名字叫mymaster,地址是127.0.0.1,端口号6379,2表示当集群中有两个sentinel认为master死了时,才能真正认为该master已经不可用了

2.sentinel down-after-milliseconds mymaster 30000
表示master在多少毫秒内无反应，哨兵会开始进行master-slave间的切换，使用“选举”机制

3.sentinel parallel-syncs mymaster 1
在发生failover主备切换时，这个选项指定了最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为1来保证每次只有一个slave处于不能处理命令请求的状态

4.sentinel failover-timeout mymaster 180000
 - 同一个sentinel对同一个master两次failover之间的间隔时间
 - 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时所对应的最大时间
 - 当想要取消一个正在进行的failover所需要的时间
 - 当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
```

启动redis和redis sentinel,启动顺序如下

```
Redis Master -> Redis Slave -> Redis Sentinel
```

启动sentinel

```
./redis-sentinel ../conf/sentienl-23679.conf --sentinel
./redis-sentinel ../conf/sentienl-23680.conf --sentinel
./redis-sentinel ../conf/sentienl-23681.conf --sentinel
```

启动成功后,可以通过redis-cli来连接sentinel,从而完成管理

```
./redis-cli -h 127.0.0.1 -p 26379
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=3,sentinels=3
```

现在我们通过关闭master来模拟master故障,从而检测redis sentinel哨兵

```
./redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379>shutdown
```

接下来再连接sentinel,检查sentinel信息

```
./redis-cli -h 127.0.0.1 -p 26379
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6380,slaves=3,sentinels=3
```

现在master变成了127.0.0.1:6380,此时master-slave信息如下

```
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=858977,lag=1
slave1:ip=127.0.0.1,port=6382,state=online,offset=858977,lag=1
master_replid:94cd21d5f654f52f56e4de4d729e215a3dab16d0
master_replid2:8c589ce46bc707d0d062ba4e035ce55c84faf67d
master_repl_offset:859110
second_repl_offset:841553
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:859110
```

一主两备,再将127.0.0.1:6379启动

```
./redis-server ../redis-6379.conf

# 此时，准备信息又发生了变化
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:3
slave0:ip=127.0.0.1,port=6381,state=online,offset=889947,lag=0
slave1:ip=127.0.0.1,port=6382,state=online,offset=889548,lag=1
slave2:ip=127.0.0.1,port=6379,state=online,offset=889681,lag=1
master_replid:94cd21d5f654f52f56e4de4d729e215a3dab16d0
master_replid2:8c589ce46bc707d0d062ba4e035ce55c84faf67d
master_repl_offset:889947
second_repl_offset:841553
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:889947
```

变成了一主三备

### Redis cluster集群配置

Redis Cluster是一种服务器Sharding技术，3.0版本开始正式提供。Redis Cluster并没有使用一致性hash，而是采用slot(槽)的概念，一共分成16384个槽。将请求发送到任意节点，接收到请求的节点会将查询请求发送到正确的节点上执行。当客户端操作的key分配到该node上时，就像操作单一Redis实例一样，当客户端操作的key没有分配到该node上时，Redis会返回转向指令，指向正确的node，这有点儿像浏览器页面的302 redirect跳转。

Redis集群，要保证16384个槽对应的node都正常工作，如果某个node发生故障，那它负责的slots也就失效，整个集群将不能工作。为了增加集群的可访问性，官方推荐的方案是将node配置成主从结构，即一个master主节点，挂n个slave从节点。这时，如果主节点失效，Redis Cluster会根据选举算法从slave节点中选择一个上升为主节点，整个集群继续对外提供服务，具有以下特点：

- 无中心架构，支持动态扩容，对业务透明
- 具备Sentinel的监控和自动Failover能力
- 客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可
- 高性能，客户端直连redis服务，免去了proxy代理的损耗

缺点是运维也很复杂，数据迁移需要人工干预，只能使用0号数据库，不支持批量操作，分布式逻辑和存储模块耦合等。

官方推荐集群至少需要六个节点，即三主三从。六个节点的配置文件基本相同，只需要修改端口号。Redis cluster规划如下：

| 角色 | 配置文件          | 监听接口 |
| ---- | ----------------- | -------- |
| 主   | cluster-6383.conf | 6383     |
| 主   | cluster-6384.conf | 6384     |
| 主   | cluster-6385.conf | 6385     |
| 备   | cluster-6386.conf | 6386     |
| 备   | cluster-6387.conf | 6387     |
| 备   | cluster-6388.conf | 6388     |

配置文件如下(端口不同)

```
port 6383
bind 127.0.0.1
daemonize yes
cluster-enabled yes
cluster-config-file nodes_6383.conf
cluster-node-timeout 15000
```

使用`reids-server`将6个节点启动,然后使用redis官方提供的`redis-trib.rb`工具创建集群

```
./redis-trib.rb create --replicas 1 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 127.0.0.1:6387 127.0.0.1:6388
```

==注:在任意一台Redis节点上运行，不要在每台机器上都运行，一台就够了 ==

但是要运行`redis-trib.rb命令，我们的主机还是要安装一些软件的。

ruby:http://www.ruby-lang.org/en/downloads/

```
#解压
tar -zxf ruby-2.6.1.tar.gz
cd ruby-2.6.1

#生成编译配置
./configure --prefix=/home/ruby

#编译并安装
make && make install

#配置环境变量
PATH=$PATH:/home/ruby/bin

#测试
ruby -v
```

zlib:http://www.zlib.net/

```
## 解压缩源码
tar -xzvf zlib-1.2.11.tar.gz
cd zlib-1.2.11

# 生成编译配置
./configure --prefix=/home/zlib

# 编译并安装
make && make install
```

rubygems:https://rubygems.org/pages/download

```
#编译ruby中的zlib
cd /usr/local/ruby-2.6.1/ext/zlib
ruby extconf.rb --with-zlib-include=/home/zlib/include/ --with-zlib-lib=/home/zlib/lib/

#解压
tar -zxf rubygems-3.0.3.tar.gz
cd rubygems-3.0.3

# 执行安装脚本，在这个过程中可能需要安装zlib
ruby setup.rb

# 验证gem是否可以使用
gem -v
```

rubygems的redis api:https://rubygems.org/gems/redis/versions/4.0.1

```
gem install -l redis-4.0.1.gem
gem list redis
```

然后再执行一下命令即可

```
# 创建集群
cd src
./redis-trib.rb create --replicas 1 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 127.0.0.1:6387 127.0.0.1:6388

---------------------------------
报错:[ERR] Node 127.0.0.1:6383 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.
此时需要将redis启动目录中的appendonly.aof,nodes.conf,dump.rdb文件删除后执行即可
---------------------------------

# 查看集群信息
./redis-cli -h 127.0.0.1 -p 6383
127.0.0.1:6383> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:717
cluster_stats_messages_pong_sent:250
cluster_stats_messages_sent:967
cluster_stats_messages_ping_received:250
cluster_stats_messages_pong_received:221
cluster_stats_messages_received:471

# 查看集群当前节点信息
127.0.0.1:6383> cluster nodes
97ec5f3606ef995f952ffa17f05a4b1e70dbc392 127.0.0.1:6383@16383 myself,master - 0 1531145554000 1 connected 0-5460
1bddffff033ccf2182b3937e2d5f8af257d97416 127.0.0.1:6388@16388 slave 85162c300cf1701ff7ab5ef2bb14b9e5d7a9fc9c 0 1531145554000 6 connected
85162c300cf1701ff7ab5ef2bb14b9e5d7a9fc9c 127.0.0.1:6384@16384 master - 0 1531145555880 2 connected 5461-10922
c7b6fef3312033ede2993fcb0b0acabe3f6b9d78 127.0.0.1:6386@16386 slave 6edac8e90957060ef176cf5ac627f9f68bac054e 0 1531145554874 4 connected
3865043ff47b60ddd099c93741d76abda3be9ecc 127.0.0.1:6387@16387 slave 97ec5f3606ef995f952ffa17f05a4b1e70dbc392 0 1531145555000 5 connected
6edac8e90957060ef176cf5ac627f9f68bac054e 127.0.0.1:6385@16385 master - 0 1531145553000 3 connected 10923-16383

# 查看集群状态信息
./redis-trib.rb check 127.0.0.1:6383
>>> Performing Cluster Check (using node 127.0.0.1:6383)
M: 97ec5f3606ef995f952ffa17f05a4b1e70dbc392 127.0.0.1:6383
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 1bddffff033ccf2182b3937e2d5f8af257d97416 127.0.0.1:6388
   slots: (0 slots) slave
   replicates 85162c300cf1701ff7ab5ef2bb14b9e5d7a9fc9c
M: 85162c300cf1701ff7ab5ef2bb14b9e5d7a9fc9c 127.0.0.1:6384
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: c7b6fef3312033ede2993fcb0b0acabe3f6b9d78 127.0.0.1:6386
   slots: (0 slots) slave
   replicates 6edac8e90957060ef176cf5ac627f9f68bac054e
S: 3865043ff47b60ddd099c93741d76abda3be9ecc 127.0.0.1:6387
   slots: (0 slots) slave
   replicates 97ec5f3606ef995f952ffa17f05a4b1e70dbc392
M: 6edac8e90957060ef176cf5ac627f9f68bac054e 127.0.0.1:6385
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```