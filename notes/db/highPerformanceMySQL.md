---
layout: post
notes: true
subtitle: "高性能MySQL（第3版）"
comments: false
author: "【美】Baron Schwartz, Peter Zaitsev, Vadim Tkachenko 著（宁海元 周振兴 彭立勋 翟卫祥 等译）"
date: 2016-11-25 00:00:00

---

![](/img/notes/db/highPerformanceMySQL/high_performance_mysql.jpg)

*   目录
{:toc }

# 第1章 MySQL架构与历史

MySQL最重要、最与众不同的特性是它的存储引擎架构，这种架构的设计将查询处理（Query Processing）及其他系统任务（Server Task）和数据的存储/提取相分离。

## 1.1 MySQL逻辑架构

![](/img/notes/db/highPerformanceMySQL/logic_architect.png)

### 1.1.1 连接管理与安全性

每个客户端连接都会在服务器进程中拥有一个线程，服务器会负责缓存线程，因此不需要为每一个新建的连接创建或者销毁线程。

### 1.1.2 优化与执行

优化器并不关心表使用的是什么存储引擎，但存储引擎对于优化查询是有影响的。优化器会请求存储引擎提供容量或某个具体操作的开销信息，以及表数据的统计信息等。

## 1.2 并发控制

MySQL在两个层面的并发控制：服务器层与存储引擎层。

### 1.2.1 读写锁

*   共享锁(shared lock)：读锁(read lock)是共享的，多个客户在同一时刻可以同时读取同一个资源，而互不干扰。
*   排他锁(exclusive lock)：写锁(write lock)则是排他的，一个写锁会阻塞其他的写锁和读锁。

确保在给定的时间里，只有一个用户能执行写入，并防止其他用户读取正在写入的同一资源。

### 1.2.2 锁粒度

*   表锁(table lock)：MySQL中最基本的锁策略，并且是开销最小的策略。
*   行级锁(row lock)：行级锁可以最大程度地支持并发处理（同时也带来最大的锁开销）。行级锁只在存储引擎层实现（InnoDB、XtraDB），而MySQL服务器层没有实现。

## 1.3 事务

*   原子性(atomicity)：一个事务必须被视为一个不可分割的最小工作单元，整个事务中所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行其中的一部分操作。
*   一致性(consistency)：数据库总是从一个一致性的状态转换到另外一个一致性的状态。
*   隔离性(isolation)：通常来说，一个事务所做的修改在最终提交以前，对其他事务是不可见的。
*   持久性(durability)：一旦事务提交，则其所做的修改就会永久保存到数据库中。

### 1.3.1 隔离级别

*   READ UNCOMMITTED（未提交读）：事务中的修改，即使没有提交，对其他的事务也都是可见的。事务可以读取未提交的数据，这也称为脏读（Dirty Read）。
*   READ COMMITED（提交读）：大多数数据库系统的默认隔离级别都是READ COMMITTED（但MySQL不是）。一个事务开始时，只能"看见"已经提交的事务所做的修改。话句话说，一个事务从开始直到提交之前，所做的任务修改对其他事务都是不可见的。这个级别有时候也叫做不可重复读（nonrepeatable read），因为两次执行同样的查询，可能会得到不一样的结果。
*   REPEATABLE READ（可重复读）：解决了脏读问题。该级别保证了在同一个事务中多次读取同样记录的结果是一致的。但无法解决幻读（Phantom Read）问题。所谓幻读，指的是当某个事务在读取某个范围内的记录时，另外一个事务又在该范围内插入了新的记录，当之前的事务再次读取该范围的记录时，会产生幻行（Phantom Row）。可重复读是MySQL的默认事务隔离级别。
*   SERIALIZABLE（可串行化）：最高的隔离级别。它通过强制事务串行执行，避免了前面说的幻读的问题。

### 1.3.2 死锁

死锁是指两个或者多个事务在同一个资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环的现象。当多个事务试图以不同的顺序锁定资源时，就可能会产生死锁。多个事务同时锁定同一资源时，也会产生死锁。

为了解决死锁问题，数据库系统实现了各种死锁检测和死锁超时机制。

死锁发生以后，只有部分或者完全回滚其中一个事务，才能打破死锁。对于事务型的系统，这是无法避免的，所以应用程序在设计时必须考虑如何处理死锁。大多数情况下只需重新执行因死锁回滚的事务即可。

### 1.3.3 事务日志

事务日志可以帮助提高事务的效率。使用事务日志，存储引擎在修改表的数据时只需要修改其内存拷贝，再把修改行为记录持久在硬盘上的事务日志中，而不用每次都将修改的数据本身持久化到磁盘。

事务日志采用的是追加的方式，因此写日志的操作是磁盘上一小块区域内的顺序I/O。

事务日志持久以后，内存中被修改的数据在后台可以慢慢地刷回到磁盘。

如果数据的修改已经记录到事务日志并持久化，但数据本身还没有写回磁盘，此时系统崩溃，存储引擎在重启时能够自动恢复这部分修改的数据。

### 1.3.4 MySQL中的事务

MySQL提供了两种事务型的存储引擎：InnoDB和NDB Cluster。

#### 自动提交（AUTOCOMMIT）

MySQL默认采用自动提交（AUTOCOMMIT）模式。如果不是显示地开始一个事务，则每个查询都被当作一个事务执行提交操作。

    mysql> SHOW VARIABLES LIKE 'AUTOCOMMIT';
    mysql> SET AUTOCOMMIT = 1;

当AUTOCOMMIT=0时，所有的查询都是在一个事务中，直到显示地执行COMMIT提交或者ROLLBACK回滚，该事务结束，同时又开始了另一个新事物。修改AUTOCOMMIT对非事务型的表，比如MyISAM或者内存表，不会有任何影响。

MySQL可以通过执行SET TRANSACTION ISOLATION LEVEL命令来设置隔离级别。新的隔离级别会在下一个事务开始的时候生效。可以在配置文件中设置整个数据库的隔离级别，也可以只改变当前会话的隔离级别：

    mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

#### 在事务中混合使用存储引擎

在同一个事务中，使用多种存储引擎是不可靠的。

为每张表选择合适的存储引擎非常重要。

#### 隐式和显式锁定

InnoDB采用的是两阶段锁定协议（two-phase locking protocal）。在事务执行过程中，随时都可以执行锁定，锁只有在执行COMMIT或者ROLLBACK的时候才会释放，并且所有的锁是在同一时刻被释放。前面描述的锁定都是隐式锁定，InnoDB会根据隔离级别在需要的时候自动加锁。

另外，InnoDB也支持通过特定的语句进行显式锁定，这些语句不属于SQL规范：

    SELECT ... LOCK IN SHARE MODE
    SELECT ... FOR UPDATE

MySQL也支持LOCK TABLES和UNLOCK TABLES语句，这是在服务器层实现的，和存储引擎无关。

建议：除了事务中禁用了AUTOCOMMIT，可以使用LOCK TABLE之外，其他任何时候都不要显示地执行LOCK TABLES，不管使用的是什么存储引擎。

## 1.4 多版本并发控制

多版本并发控制（MVCC）是行级锁的一个变种，但是它在很多情况下避免了加锁操作，因此开销更低。

MVCC的实现，是通过保存数据在某个时间点的快照来实现的。

不同存储引擎的MVCC实现不同，典型的有乐观（optimistic）并发控制和悲观（pessimistic）并发控制。

InnoDB的MVCC，是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了行的创建时间，一个保存行的过期时间（或删除时间）。当然存储的并不是实际的时间值，而是系统版本号（system version number）。每开始一个新的事务，系统版本号都会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询到的每行记录的版本号进行比较。

保存这两个额外系统版本号，使大多数读操作都可以不用加锁。这样设计使得读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行。不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作。

MVCC只在REPEATABLE READ和READ COMMITTED两个隔离级别下工作。

# 第10章 复制

复制功能不仅有利于构建高性能的应用，同时也是高可用性、可扩展性、灾难恢复、备份以及数据仓库等工作的基础。

## 10.1 复制概述

复制解决的基本问题是让一台服务器的数据与其他服务器保持同步。一台主库的数据可以同步到多台备库上，备库本身也可以被配置成另外一台服务器的主库。主库和备库之间可以有多重不同的组合方式。

MySQL支持两种复制方式：基于行的复制和基于语句的复制。这两种方式都是通过在主库上记录二进制日志、在备库重放日志的方式来实现异步的数据复制。这意味着，在同一时间点备库上的数据可能与主库存在不一致，并且无法保证主备之间的延迟。一些大的语句可能导致备库产生几秒、几分钟甚至几个小时的延迟。

MySQL复制大部分是向后兼容的，新版本的服务器可以作为老版本服务器的备库，反之则不行。

通过复制可以将读操作指向备库来获得更好的读扩展，但对于写操作，除非设计得当，否则并不适合通过复制来扩展写操作。

### 10.1.1 复制解决的问题

*	数据分布
*	负载均衡
*	备份
*	高可用性和故障切换
*	MySQL升级测试

### 10.1.2 复制如何工作

1.	在主库上把数据更改记录二进制日志（Binary Log）中（这些记录被称为二进制日志事件）。
2.	备库将主库上的日志复制到自己的中继日志（Relay Log）中。
3.	备库读取中继日志中的事件，将其重放到备库数据库之上。

![](/img/notes/db/highPerformanceMySQL/replication_work.png)

在主库上并发运行的查询在备库只能串行化执行，因为只有一个SQL线程来重放中继日志中的事件。

## 10.2 配置复制

大致步骤：

1.	在每台服务器上创建复制账号。
2.	配置主库和备库
3.	通知备库连接到主库并从主库复制数据。

### 10.2.1 创建复制账号

	mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.*
	    -> TO repl@'192.168.0.%' IDENTIFIED BY 'p4ssword';
		
### 10.2.2 配置主库和备库

假设主库是服务器server1，需要打开二进制日志并指定一个独一无二的服务器ID（server ID），在主库的my.cnf文件中增加或修改如下内容：

	log_bin = mysql-bin
	server_id = 10

备库上也需要在my.cnf中增加类似的配置，并且同样需要重启服务器。

	log_bin = mysql-bin
	server_id = 2
	relay_log = /var/lib/mysql/mysql-relay-bin
	log_slave_updates = 1
	read_only = 1
	
### 10.2.3 启动复制

下一步是告诉备库如何连接到主库并重放其二进制日志。这一步不要通过修改my.cnf来配置，而是使用CHANGE MASTER TO语句，该语句完全替代了my.cnf中相应的设置，并且允许以后指向别的主库时无须重启备库。

	mysql> CHANGE MASTER TO MASTER_HOST='server1',
	    -> MASTER_USER='repl',
		-> MASTER_PASSWORD='p4ssword',
		-> MASTER_LOG_FILE='mysql-bin.000001',
		-> MASTER_LOG_POS=0;
		
MASTER_LOG_POS参数被设置为0，因为要从日志的开头读起。当执行完这条语句后，可以通过SHOW SLAVE STATUS语句来检查复制是否正确执行。

	mysql> SHOW SLAVE STATUS\G
	
运行下面的命令开始复制：

	mysql> START SLAVE;
	
可以用SHOW SLAVE STATUS命令检查：

	mysql> SHOW SLAVE STATUS\G
	
我们还可以从线程列表中看到复制线程。在主库上可以看到由备库I/O线程向主库发起的连接。

	mysql> SHOW PROCESSLIST\G

在备库也可以看到两个线程，一个是I/O线程，一个是SQL线程。

### 10.2.4 从另一个服务器开始复制

大多数情况下有一个已经运行了一段时间的主库，然后用一台新安装的备库与之同步，此时这台备库还没有数据。

初始化备库或者从其他服务器克隆数据到备库：

*	从主库复制数据
*	从另外一台备库克隆数据
*	使用最近的一次备份来启动备库

三个条件来让主库和备库保持同步：

*	在某个时间点的主库的数据快照
*	主库当前的二进制日志文件，和获得数据快照时在该二进制日志文件中的偏移量
*	从快照时间到现在的二进制日志。

一些从别的服务器克隆备库的方法：

*	使用冷备份
*	使用热备份
*	使用mysqldump
*	使用快照或备份
*	使用Percona Xtrabackup
*	使用另外的备库

### 10.2.5 推荐的复制配置

在主库上二进制日志最重要的选项是sync_binlog：

	sync_binlog=1
	
如果开启该选项，MySQL每次在提交事务前会将二进制日志同步到磁盘上，保证在服务器崩溃时不会丢失事件。

如果无法容忍服务器崩溃导致表损坏，推荐使用InnoDB。

如果使用InnoDB，我们强烈推荐设置如下选项：

	innodb_flush_logs_at_trx_commit		# Flush every log write
	innodb_support_xa=1					# MySQL 5.0 and newer only
	innodb_safe_binlog					# MySQL 4.1 only, roughly equivalent to
	
推荐明确指定二进制日志的名字，以保证二进制日志在所有服务器上是一致的：

	log_bin=/var/lib/mysql/mysql-bin
	
在备库上，我们同样推荐如下配置选项，为中继日志指定绝对路径：

	relay_log=/path/to/logs/relay-bin
	skip_slave_start
	read_only
	
*	skip_slave_start选项能够阻止备库在崩溃后自动启动复制（有机会来修复可能发生的问题）。
*	read_only选项可以阻止大部分用户更改非临时表。

即使开启了所有我们建议的选项，备库仍然可能在崩溃后被中断，因为master.info和中继日志文件都不是崩溃安全的。如果正在使用MySQL 5.5并且不介意额外的fsync()导致的性能开销，最好设置以下选项：

	sync_master_info = 1
	sync_relay_log = 1
	sync_relay_log_info = 1