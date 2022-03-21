# Canal

## Canal基础

### 什么是Canal？

基于MySQL数据库的日志解析，获取增量变更进行同步，由此衍生出增量订阅&消费的业务

仓库地址：https://github.com/alibaba/canal

### MySQL的Binlog

#### 什么是Binlog？

MySQL的二进制日志是MySQL中最重要的日志他，它记录了所有的DDL和DML语句，以事件形式记录，还包含语句所执行的消耗时间，MySQL的二进制日志是事务安全型的。

> Binlog日志有两个重要的使用场景：
>
> 1.主从复制时，master节点将它的二进制日志传递给Slaves来达到Master-Slave数据一致
>
> 2.数据恢复，通过MySQL Binlog工具来使数据恢复

Binlog日志包含两类文件：二进制日志索引文件(后缀.index)，二进制日志文件(后缀.00000*，记录DDL和DML)

#### Binlog的分类

MySQL的Binlog 的格式有三种，分别是statement，mixed，row。三种格式的区别：

1.statement：语句级，binlog会记录每一次执行写操作的语句

​	优点：节省空间

​	缺点：可能造成数据不一致性

2.mixed：statement的升级版，解决了一些情况下statement模式不一致问题

​	优点：节省空间，同时兼顾了一定的一致性

​	缺点：还有极个别情况依旧会造成数据不一致

3.row：行级，binlog会记录每次操作后每行记录的变化

​	优点：保持数据的绝对一致性，因为不管sql语句是什么，他只记录执行后的结果

​	缺点：占用较大空间

### Canal工作原理

#### MySQL主从复制过程

1.master主库将改变记录，写到二进制日志文件binlog

2.slave从库向主库mysql master发送dump协议，将master的binlog文件拷贝到它的中继日志(relay log)

3.slave从库读取并重做中继日志中的事件，将改变的数据同步到自己的数据库

#### Canal工作原理

canal把自己伪装成slave，假装从master复制数据

### 使用场景

1.原始场景：阿里Otter中间件的一部分，Otter是阿里用于进行异地数据库之间的同步框架，Canal是其中一部分(读取解析MySQL binlog日志)

2.常见场景：更新缓存

![image-20220321122816107](https://github.com/Cola-Ice/Yarda-Canal/raw/master/doc/image/image-20220321122816107.png)

3.常见场景：抓取业务表的新增变化数据，用于制作实时数据统计

## MySQL环境准备

开启binlog，只需在etc/my.cnf文件加上以下配置：

server-id=1
log-bin=mysql-bin	
binlog_format=row	# 日志格式，行级
binlog-do-db=test	# 需要记录日志的库

查看数据库是否已开启binlog命令：

show variables like‘%log_bin%’;

创建canal用户只赋予查询权限：

#创建canal用户
create user canal identified by 'canal';
#授权
grant select,replication slave,replication client on*.* to 'canal'@'%';

## Canal下载和安装

下载地址：https://github.com/alibaba/canal/releases/

修改canal.properties配置文件(canal自身相关配置，以及需要监听的实例名配置)

修改example/instance.properties(监听实例配置，数据库连接信息等)

canal支持的服务模式：tcp、kafka、rocketmq、rabbitmq

## Canal实现的数据采集

### TCP模式

maven依赖:

```
<dependency>
    <groupId>com.alibaba.otter</groupId>
    <artifactId>canal.client</artifactId>
    <version>1.1.0</version>
</dependency>
```

官方java客户端实例：https://github.com/alibaba/canal/wiki/ClientExample

### Kafka模式

仅需要修改canal的配置即可将数据直接传输到Kafka

### RocketMQ模式



