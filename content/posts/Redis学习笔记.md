---
title: Redis学习笔记
copyright: true
date: 2020-07-03 21:03:43
tags: [学习]
---

Redis是一个使用ANSI C编写的开源、支持网络、基于内存、可选持久性的键值对存储数据库。从2015年6月开始，Redis的开发由Redis Labs赞助，而2013年5月至2015年6月期间，其开发由Pivotal赞助。在2013年5月之前，其开发由VMware赞助。根据月度排行网站DB-Engines.com的数据显示，Redis是最流行的键值对存储数据库。

Redis主要有这几部分知识，五大数据类型，持久化方式，发布订阅，主从复制+哨兵。

<!--more-->

## Redis的安装

### centos安装redis

```bash 
wget http://download.redis.io/releases/redis-6.0.5.tar.gz
tar -zxvf redis-6.0.5.tar.gz
cd redis-6.0.5
sudo make
sudo make install PREFIX=/opt/redis
```

### centos 设置redis开启自启动

```bash
sudo vim /usr/lib/systemd/system/redis.service

## redis.service 内容
[Unit]
Description=The redis-server Process Manager
After=syslog.target network.target

[Service]
Type=forking
PIDFile=/var/run/redis_6379.pid
ExecStart=/opt/redis/bin/redis-server /opt/redis/bin/redis.conf
ExecReload=/bin/kill -USR2 $MAINPID
ExecStop=/bin/kill -SIGINT $MAINPID

[Install]
WantedBy=multi-user.target

# 结束，保存退出
# 刷新守护进程列表
sudo systemctl daemon-reload
# 启动服务
sudo systemctl start redis.service
# 设置服务开启自启
sudo systemctl enable redis.service

```

### redis开启远程连接（redis.conf）

```bash
## redis.conf 的一些配置
# 注释 bind 127.0.0.1
bind 127.0.0.1
# 关闭保护模式
protected-mode no
# 允许后台启动
daemonize yes
```



## 五大数据类型

redis中的数据都是以key/value的形式存储的，五大数据类型主要是指value的数据类型。

### String字符串

最基础的数据类型，需要注意的是redis中的字符串大小上限是512M。

**APPEND**

使用APPEND命令时，如果key已经存在，则会直接在value后追加值，如果key不存在，则会先创建一个value为空字符串的key，然后再追加。

```shell
127.0.0.1:6379> get k1
v1
127.0.0.1:6379> append k1 v1vv
6
127.0.0.1:6379> get k1
v1v1vv
```

**GET**

GET命令用来获取对应key的value，如果key不存在则返回nil。

```bash
127.0.0.1:6379> get k1
v1v1vv
```

**GETRANGE**

GETRANGE用来返回key所对应的value的子串，子串由start和end决定，从左往右计算，如果下标是负数，则从右往左计算，其中-1表示最后一个字符，-2是倒数第二个…

```bash
127.0.0.1:6379> getrange k1 1 3
1v1
```

**GETSET**

GETSET命令可以用来获取key所对应的value，并对key进行重置。

```bash
127.0.0.1:6379> get k2
v2
127.0.0.1:6379> getset k2 v2set
v2
127.0.0.1:6379> get k2
v2set
```

**SETRANGE**

SETRANGE用来覆盖一个已经存在的key的value。

```bash
127.0.0.1:6379> setrange k2 2 range
7
127.0.0.1:6379> get k2
v2range
```

**SETEX**

SETEX用来给key设置value，同时设置过期时间，等效于先给key设置value，再给key设置过期时间。

```bash
127.0.0.1:6379> setex k4 5 v4
OK
```

**SETNX**

SETNX是 **SET** if **N**ot e**X**ists的简写，SET命令在执行时，如果key已经存在，则新值会覆盖掉旧值，而对于SETNX命令，如果key已经存在，则不做任何操作，如果key不存在，则效果等同于SET命令。

```bash
127.0.0.1:6379> setnx k4 v4v4
0
```



### List列表

LIST是一个简单的字符串列表，按照插入顺序进行排序，可以从头尾插入或弹出。

**LPUSH**

将一个或多个值value插入到列表key的表头，如果有多个value值，那么各个value值按从左到右的顺序依次插入到表头。

```bash
127.0.0.1:6379> lpush l1 v1 v2 v3 v4 v5
5
```

**LRANGE**

返回列表key中指定区间内的元素，区间以偏移量start和stop指定，下标(index)参数start和stop都以0为底，即0表示列表的第一个元素，1表示列表的第二个元素，以此类推。我们也可以使用负数下标，以-1表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

```bash
127.0.0.1:6379> lrange l1 0 -1
v5
v4
v3
v2
v1
```

**RPUSH**

RPUSH与LPUSH的功能基本一致，不同的是RPUSH的中的value值是按照从右到左的顺序依次插入。

```bash
127.0.0.1:6379> rpush l1 v6 v7
7
127.0.0.1:6379> lrange l1 0 -1
v5
v4
v3
v2
v1
v6
v7
```

**RPOP**

RPOP命令可以移除并返回列表key的尾元素。

```bash
127.0.0.1:6379> rpop l1
v7
127.0.0.1:6379> rpop l1
v6
127.0.0.1:6379> lrange l1 0 -1
v5
v4
v3
v2
v1
```

**LPOP**

LPOP和RPOP类似，不同的是LPOP移除并返回列表key的头元素。

```bash
127.0.0.1:6379> lpop l1
v5
127.0.0.1:6379> lrange l1 0 -1
v4
v3
v2
v1
```

**LINDEX**

LINDEX命令可以返回列表key中，下标为index的元素，正数下标0表示第一个元素，也可以使用负数下标，-1表示倒数第一个元素。

```bash
127.0.0.1:6379> lrange l1 0 -1
v4
v3
v2
v1
127.0.0.1:6379> lindex l1 2
v2
```



### Hash键值对

HASH类似于Java中的Map，是一个键值对集合，在redis中可以用来存储对象。

**HSET**

HSET命令可以用来设置key指定的哈希集中指定字段的值。

```bash
127.0.0.1:6379> hset hk1 k1 v1 k2 v2 k3 v3 k4 v4 k5 v5
5
```

**HGET**

HGET命令可以用来返回key指定的哈希集中该字段所关联的值。

```bash
127.0.0.1:6379> hget hk1 k1
v1
```

**HDEL**

HDEL命令可以从key指定的哈希集中移除指定的域，在哈希集中不存在的域将被忽略。

```bash
127.0.0.1:6379> hdel hk1 k1
1
```

**HSETNX**

HSETNX命令只在key指定的哈希集中不存在指定的字段时，设置字段的值，如果字段已存在，该操作无效果。

```bash
127.0.0.1:6379> hsetnx hk1 k1 v11
1
127.0.0.1:6379> hget hk1 k1
v11
127.0.0.1:6379> hsetnx hk1 k1 v12
0
127.0.0.1:6379> hget hk1 k1
v11
```

**HVALS**

HVALS命令可以返回key指定的哈希集中所有字段的值。

```bash
127.0.0.1:6379> hvals hk1
v2
v3
v4
v5
v11
```

**HKEYS**

HKEYS命令可以返回key指定的哈希集中所有字段的名字。

```bash
127.0.0.1:6379> hkeys hk1
k2
k3
k4
k5
k1
```

**HGETALL**

HEXISTS命令可以返回hash里面field是否存在。

```bash
127.0.0.1:6379> hgetall hk1
k2
v2
k3
v3
k4
v4
k5
v5
k1
v11
```

**HLEN**

HSTRLEN可以返回hash指定field的value的字符串长度，如果hash或者field不存在，返回0。

```bash
127.0.0.1:6379> hlen hk1
5
```



### Set集合

SET是String类型的无序集合，不同于LIST，SET中的元素不可以重复。

**SADD**

SADD命令可以添加一个或多个指定的member元素到集合的key中，指定的一个或者多个元素member如果已经在集合key中存在则忽略，如果集合key不存在，则新建集合key,并添加member元素到集合key中。

```bash
127.0.0.1:6379> sadd sk1 v1 v2 v3 v3 v4
4
```

**SREM**

SREM命令可以在key集合中移除指定的元素，如果指定的元素不是key集合中的元素则忽略。如果key集合不存在则被视为一个空的集合，该命令返回0。

```bash
127.0.0.1:6379> srem sk1 v5
0
127.0.0.1:6379> srem sk1 v4
1
```

**SISMEMBER**

SISMEMBER命令可以返回成员member是否是存储的集合key的成员。

```bash
127.0.0.1:6379> sismember sk1 v4
0
127.0.0.1:6379> sismember sk1 v3
1
```

**SCARD**

SCARD命令可以返回集合存储的key的基数(集合元素的数量)。

```bash
127.0.0.1:6379> scard sk1
3
```

**SMEMBERS**

SMEMBERS命令可以返回key集合所有的元素。

```bash
127.0.0.1:6379> smembers sk1
v3
v1
v2
```

**SPOP**

SPOP命令的用法和SRANDMEMBER类似，不同的是，SPOP每次选择一个随机的元素之后，该元素会出栈，而SRANDMEMBER则不会出栈，只是将该元素展示出来。

```bash
127.0.0.1:6379> spop sk1
v2
127.0.0.1:6379> smembers sk1
v3
v1
```

**SDIFF**

SDIFF可以用来返回一个集合与给定集合的差集的元素。

```bash
127.0.0.1:6379> smembers sk1
v3
v1
127.0.0.1:6379> smembers sk2
v1
v4
v2
127.0.0.1:6379> sdiff sk1 sk2
v3
```

**SDIFFSTORE**

SDIFFSTORE命令与SDIFF命令基本一致，不同的是SDIFFSTORE命令会将结果保存在一个集合中。

```bash
127.0.0.1:6379> sdiffstore sk3 sk2 sk1
2
127.0.0.1:6379> smembers sk3
v2
v4
```

**SINTER**

SINTER命令可以用来计算指定key之间元素的交集。

```bash
127.0.0.1:6379> smembers sk2
v1
v4
v2
127.0.0.1:6379> smembers sk3
v2
v4
127.0.0.1:6379> sinter sk2 sk3
v2
v4
```

**SUNION**

SUNION可以用来计算两个集合的并集。

```bash
127.0.0.1:6379> sunion sk1 sk3
v1
v3
v4
v2
```

### ZSet有序集合

ZSET和SET一样，也是STRING类型的元素的集合，不同的是ZSET中的每个元素都会关联一个double类型的分数，ZSET中的成员都是唯一的，但是所关联的分数可以重复。

**ZADD**

ZADD命令可以将所有指定成员添加到键为key的有序集合里面。添加时可以指定多个分数/成员（score/member）对。 如果指定添加的成员已经是有序集合里面的成员，则会更新该成员的分数（scrore）并更新到正确的排序位置。

```bash
127.0.0.1:6379> zadd zk1 10 v1 20 v2 30 v3 40 v4 10 v5
5
```

**ZSCORE**

ZSCORE命令可以返回有序集key中，成员member的score值。

```bash
127.0.0.1:6379> zscore zk1 v1
10
127.0.0.1:6379> zscore zk1 v5
10
```

**ZRANGE**

ZRANGE命令可以根据index返回member，该命令在执行时加上withscores参数可以连同score一起返回。

```bash
127.0.0.1:6379> zrange zk1 0 -1
v1
v5
v2
v3
v4
127.0.0.1:6379> zrange zk1 0 -1 withscores
v1
10
v5
10
v2
20
v3
30
v4
40
```

**ZCOUNT**

ZCOUNT命令可以返回有序集key中，score值在min和max之间(默认包括score值等于min或max)的成员。

```bash
127.0.0.1:6379> zcount zk1 10 30
4
```

**ZRANGEBYSCORE**

ZRANGEBYSCORE命令可以按照score范围范围元素，加上withscores可以连score一起返回。

```bash
127.0.0.1:6379> zrangebyscore zk1 10 30 
v1
v5
v2
v3
127.0.0.1:6379> zrangebyscore zk1 10 30 withscores
v1
10
v5
10
v2
20
v3
30
```

**ZRANK**

ZRANK命令可以返回有序集key中成员member的排名。其中有序集成员按score值递增(从小到大)顺序排列。排名以0为底，即score值最小的成员排名为0。

```bash
127.0.0.1:6379> zrank zk1 v2
2
127.0.0.1:6379> zrank zk1 v4
4
```

**ZREM**

ZREM命令可以从集合中弹出一个元素。

```bash
127.0.0.1:6379> zrem zk1 v1 v2
2
127.0.0.1:6379> zrange zk1 0 -1
v5
v3
v4
```



### Key相关命令

**DEL命令**

通过DEL命令我们可以删除一个已经存在的key。

```bash
127.0.0.1:6379> keys *
k2
k4
k3
sk2
zk1
sk1
hk1
l1
k1
127.0.0.1:6379> del sk2 k2 k3
3
```

**DUMP命令**

DUMP命令可以序列化给定的key，并返回序列化之后的值。

```bash
127.0.0.1:6379> dump k4
"\x00\x02v4\t\x00tx\x19\x8c\xe4\xa9\xaa\xb3"
```

**EXISTS**

EXISTS命令用来检测一个给定的key是否存在。

```bash
127.0.0.1:6379> keys *
1) "k4"
2) "zk1"
3) "sk1"
4) "hk1"
5) "l1"
6) "k1"
127.0.0.1:6379> exists k4
(integer) 1
```

**TTL命令**

TTL命令可以查看一个给定key的有效时间。

```bash
127.0.0.1:6379> ttl k4 
(integer) -1
```

**EXPIRE命令**

EXPIRE命令可以给key设置有效期，在有效期过后，key会被销毁。

```bash
127.0.0.1:6379> expire k4 120
(integer) 1
127.0.0.1:6379> ttl k3
(integer) -2  # 永久存在
127.0.0.1:6379> ttl k4
(integer) 107 # 存活107秒

```

**PERSIST命令**

PERSIST命令表示移除一个key的过期时间，这样该key就永远不会过期。

```
127.0.0.1:6379> persist k4
(integer) 1
127.0.0.1:6379> ttl k4
(integer) -1
```

**KEYS 命令**

KEYS命令可以获取满足给定模式的所有key。

```bash
KEYS *（*表示所有的key，*也可以是一个正在表达式）
```



## 发布订阅

**SUBSCRIBE **

订阅频道

```bash
127.0.0.1:6379> subscribe c1 c2 c3
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "c1"
3) (integer) 1
1) "subscribe"
2) "c2"
3) (integer) 2
1) "subscribe"
2) "c3"
3) (integer) 3
```

**PUBLISH **

发布消息

```bash
127.0.0.1:6379> PUBLISH c1 "hello redis!"
(integer) 1
```

**PSUBSCRIBE **

```bash
127.0.0.1:6379> PSUBSCRIBE c*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "c*"
3) (integer) 1
```



## 事务

不同于关系型数据库，redis中的事务出错时没有回滚，对此，官方的解释如下：

Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

**MULTI**

开启事物

**EXEC**

结束事物并执行

**WATCH**

事务中的WATCH命令可以用来监控一个key，通过这种监控，我们可以为redis事务提供(CAS)行为。 如果有至少一个被WATCH监视的键在EXEC执行之前被修改了，那么整个事务都会被取消，EXEC返回nil-reply来表示事务已经失败。

**事物中的异常行为**

redis中事务的异常情况总的来说分为两类：

1.进入队列之前就能发现的错误，比如命令输错；（拒绝执行并自动放弃这个事务）
2.执行EXEC之后才能发现的错误，比如给一个非数字字符加1；（执行产生错误，其他命令继续执行）



## 持久化

整体上来说，redis持久化有两种方式，快照持久化和AOF，在项目中我们可以根据实际情况选择合适的持久化方式，也可以不用持久化，这关键看我们的redis在项目中扮演了什么样的角色。

### 快照持久化

redis可以在某个时间点上对内存中的数据创建一个副本文件，副本文件中的数据在redis重启时会被自动加载，我们也可以将副本文件拷贝到其他地方一样可以使用。

#### 如何配置快照持久化（redis.conf）

```shell
save 900 1		# 900秒内至少一个key变更
save 300 10		# 100秒内至少10个key变更
save 60 10000   # 60秒内10000个key变更
stop-writes-on-bgsave-error yes  # 快照出错后，是否继续执行写命令
rdbcompression yes   # 快照文件是否压缩
dbfilename dump.rdb  # 快照文件名字
dir ./   # 快照文件存放目录
```

#### 手动创建快照

**SAVE**

save命令是一个阻塞命令，在这个命令执行完成之前，不在处理其他请求，其他请求将被挂起。

**BGSAVE**

bgsave命令会fork一个子进程，用这个子进程将快照写入硬盘，而父进程则继续处理客户端发来的请求。

#### 自动创建快照时机

1. 当条件满足时，如900秒内有一个key被操作，redis会触发bgsave命令进行备份；
2. 当我们执行shutdown命令时，redis会执行save命令进行备份操作，备份完成后关闭服务器；
3. 在主从模式中，当从机第一次连接上主机后会发一条sync命令来开始一次复制操作，此时主机会开始一次bgsave操作，并在bgsave操作结束后向从机发送快照数据实现数据同步；

### AOF持久化

与快照持久化不同，AOF持久化是将被执行的命令写到aof文件末尾，在恢复时只需要从头到尾执行一遍写命令即可恢复数据，AOF在redis中默认也是没有开启的，需要我们手动开启，开启方式为将redis.conf中的修改appendonly属性值为yes。

#### aof配置（redis.conf）

```shell
appendonly yes   # 开启aof持久化

appendfilename "appendonly.aof"   # aof备份文件名

# appendfsync always   # 每执行一次写操作备份一次
appendfsync everysec   # 每秒备份一次（最常用）
# appendfsync no       # 由操作系统决定备份时机

no-appendfsync-on-rewrite no   # 对aof文件进行压缩时，是否执行同步操作
auto-aof-rewrite-percentage 100   # 重写aof文件的百分比，基准为上一次重写文件大小或启动时文件大小
auto-aof-rewrite-min-size 64mb    # 最少要大于64m才对aof文件进行重写
```

### 最佳实践

1. 如果redis只做缓存服务器，那么可以不使用任何持久化方式；
2. 同时开启两种持久化方式，redis重启时优先载入aof文件来恢复原始的数据。rdb更适合用于备份数据库，快速重启，不会有aof可能潜在的bug，但是aof数据会更加完整；
3. 因为rdb文件只用作后备用途，建议只在slave上持久化rdb文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则；
4. 如果开启aof，一是会带来持续的io，二是rewrite过程中产生的新数据写到新文件造成的阻塞几乎不可避免。所以只要硬盘允许，尽量减少rewrite的频率，重写的基础可以设置5G以上。
5. 如果不开启aof，紧靠master-slave replication也可以实现高可用性。能省掉一大笔io，也减少了rewrite带来的系统波动。代价是master/slave同时倒掉，会丢失十几分钟的数据。



## 主从复制

主从复制可以在一定程度上扩展redis性能，redis的主从复制和关系型数据库的主从复制类似，从机能够精确的复制主机上的内容。实现了主从复制之后，一方面能够实现数据的读写分离，降低master的压力，另一方面也能实现数据的备份。

### 配置方式

1. 启动三个实例

   ```shell
   [root@localhost redis-4.0.8]# redis-server redis6379.conf
   [root@localhost redis-4.0.8]# redis-server redis6380.conf
   [root@localhost redis-4.0.8]# redis-server redis6381.conf
   ```

2. 连接服务

   ```shell
   redis-cli -h IP -p 端口  --raw      //登陆redis服务器时，增加参数 --raw（避免命令行中文乱码）
   ```

3. 从机上执行slaveof

   ```bash
   slaveof 127.0.0.1 6379
   ```

4. 查看每个实例当前状态

   ```bash
   INFO replication
   ```

### 主从复制注意点

1. 如果主机运行一段时间了，从机连接上来，会备份主机所有的数据；
2. 主从复制后，master可读可写，slave可读不可写；
3. 主从结构运行中，主机不行挂掉，重启之后，他依然是主机；

### 主从复制原理

```text
1.slave启动成功连接到master后会发送一个sync命令。  
2.Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令。  
3.在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步。  
4.全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。  
5.增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步。  
6.但是只要是重新连接master,一次完全同步（全量复制)将被自动执行。  
```

### 哨兵模式

**sentinel.conf**

```bash
sentinel monitor mymaster 127.0.0.1 6379 1
```

**启动哨兵**

```bash
redis-sentinel sentinel.conf
```

**注意点**

当master挂掉之后，哨兵会重新选举一个slave当master，之前的master重启之后，也只能当slave了。

由于所有的写都是在master上操作的，master和slave会有一定的延迟，当系统繁忙，slave机器较多时问题会更严重，所以我们可以通过集群来进一步提示redis性能。