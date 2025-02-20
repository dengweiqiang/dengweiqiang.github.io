---
title: MySQL学习笔记
copyright: true
date: 2020-07-13 10:26:43
tags: [学习]
---

简单介绍了在centos中mysql的安装，实现主从复制，常见的数据库拆分方案。

<!--more-->

## CentOS安装MySQL

具体的安装步骤如下：

- 检查是否安装了 mariadb，如果已经安装了则卸载：

```bash
yum list installed | grep mariadb
```

如果执行结果如下，表示已经安装了 mariadb，将之卸载：

```bash
mariadb-libs.x86_64                   1:5.5.52-1.el7                   @anaconda
```

卸载命令如下：

```bash
yum -y remove mariadb*
```

- 接下来下载官方提供的 rpm 包

如果 CentOS 上没有 wget 命令，首先通过如下命令安装 wget：

```bash
yum install wget
```

然后执行如下操作下载 rpm 包：

```bash
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

- 下载完成后，安装rpm包：

```bash
rpm -ivh mysql57-community-release-el7-11.noarch.rpm
```

- 检查 MySQL 的 yum 源是否安装成功：

```bash
yum repolist enabled | grep "mysql.*-community.*"
```

执行结果如下表示安装成功：

[![img](https://www.javaboy.org/images/mysql/2-1.png)](https://www.javaboy.org/images/mysql/2-1.png)

- 安装 MySQL

```bash
yum install mysql-server
```

- 安装完成后，启动MySQL：

```bash
systemctl start mysqld.service
```

- 停止MySQL：

```bash
systemctl stop mysqld.service
```

- 登录 MySQL：

```bash
mysql -u root -p
```

默认无密码。有的版本有默认密码，查看默认密码，首先去 /etc/my.cnf 目录下查看 MySQL 的日志位置，然后打开日志文件，可以看到日志中有一个提示，生成了一个临时的默认密码，使用这个密码登录，登录成功后修改密码即可。

- 改密码

首先修改密码策略(这一步不是必须的，如果不修改密码策略，需要取一个比较复杂的密码，松哥这里简单起见，就修改下密码策略)：

```bash
set global validate_password_policy=0; # 修改策略
set global validate_password_length=6; # 修改密码长度
```

然后重置密码：

```bash
set password=password("123456");     
flush privileges;
```

- 授权远程登录同方式一：

```bash
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
flush privileges;
```

- 授权远程登录同方式二：

修改 mysql 库中的 user 表，将 root 用户的 Host 字段的值改为 `%` ，然后重启 MySQL 即可。

- 关闭防火墙
  MySQL 要能远程访问，还需要关闭防火墙：

```bash
systemctl stop firewalld.service
```

禁止firewall开机启动:

```bash
systemctl disable firewalld.service
```



## MySQL主从配置

**准备工作**

```
主机：192.168.209.171
从机：192.168.209.172， 192.168.209.173
```

### 主机配置

1. 授权给从服务器

   ```bash
   GRANT REPLICATION SLAVE ON *.* to 'rep1'@'192.168.209.172' identified by '123456';
   GRANT REPLICATION SLAVE ON *.* to 'rep2'@'192.168.209.173' identified by '123456';
   FLUSH PRIVILEGES;
   
   # 这里表示配置从机登录用户名为 rep1，密码为 123，并且必须从 192.168.248.139这个地址登录，登录成功之后可以操作任意库中的任意表。其中，如果不需要限制登录地址，可以将 IP 地址更换为一个 %。
   ```

2. 修改主库配置文件

   ```bash
   vi /etc/my.cnf
   
   [mysqld]
   log-bin=/var/lib/mysql/binlog
   server-id=171
   binlog-do-db=db1
   
   # 开启 binlog（同步的日志路径及文件名），并设置 server-id（MySQL 在主从环境下的唯一标志符），配置binlog-do-db（要同步的数据库名）
   
   systemctl restart mysqld; # 重启服务
   ```

3. 查看主服务器当前二进制日志名和偏移量，这个操作的目的是为了在从数据库启动后，从这个点开始进行数据的恢复：

   ```
   show master status;
   ```

### 从机配置

1. 修改从机配置文件（/etc/my.cnf）

   ```bash
   vi /etc/my.cnf
   
   [mysqld]
   server-id=172
   
   # 173机器配置文件
   [mysqld]
   server-id=173
   ```

2. 使用命令来配置从机：

   ```bash
   # 172
   change master to master_host='192.168.209.171',master_port=3306,master_user='rep1',master_password='123456',master_log_file='binlog.000001',master_log_pos=154;
   # 173
   change master to master_host='192.168.209.171',master_port=3306,master_user='rep2',master_password='123456',master_log_file='binlog.000001',master_log_pos=0;
   
   # 这里配置了主机地址、端口以及从机登录主机的用户名和密码，注意最后两个参数要和 master 中的保持一致。
   ```

3. 启动slave进程

   ```bash
   start slave;
   ```

4. 查看从机状态

   ```bash
   show slave status\G;
   
   # 这两个值都要为yes才表示成功
   # Slave_IO_Running: Yes
   # Slave_SQL_Running: Yes
   ```

## 数据库拆分方案

把一个数据库切分成 N 多个数据库的优点，一是可以降低单台数据库实例的负载，二是可以方便的实现对数据库的扩容。

### 水平切分

**特点**

- 两个 DB 中表的个数都是完整的，就是原来 DB 中有几张表，现在还是几张。
- 每张表中的数据是不完整的，数据被拆分到了不同的 DB 中去了。

**分片规则**

- 按照日期划分：不容日期的数据存放到不同的数据库中。
- 对 ID 取模：对表中的 ID 字段进行取模运算，根据取模结果将数据保存到不同的实例中。
- 使用一致性哈希算法进行切分。

**优点**

1. 水平切分最大的优势在于数据库的扩展性好，提前选好切分规则，数据库后期可以非常方便的进行扩容。
2. 有效提高了数据库稳定性和系统的负载能力。拆分规则抽象好， join 操作基本可以数据库做。

**缺点**

1. 水平切分后，分片事务一致性不容易解决。
2. 拆分规则不易抽象，对架构师水平要求很高。
3. 跨库 join 性能较差。

### 垂直切分

**特点**

- 每一个数据库实例中的表的数量都是不完整的。
- 每一个数据库实例中表的数据是完整的。

垂直切分我们可以按照业务来划分，不同业务的表放到不同的数据库实例中。在实际项目中，数据库垂直切分并不是一件容易的事，因为表之间往往存在着复杂的跨库 JOIN 问题，那么这个时候如何取舍，就要考验架构师的水平了！

**优点**

1. 一般按照业务拆分，拆分后业务清晰，可以结合微服务一起食用。
2. 系统之间整合或扩展相对要容易很多。
3. 数据维护相对简单。

**缺点**

1. 最大的问题在于存在单库性能瓶颈，数据表扩展不易。
2. 跨库 join 不易。
3. 事务处理复杂。