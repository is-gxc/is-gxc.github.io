---
title: MySQL三大日志(binlog、redo log、undo log)
date: 2024-07-10 15:02:44
categories: MySQL
tags: [MySQL,数据库]
---

MySQL的日志有错误日志（Error Log）、查询日志（General Log）、慢查询日志（Slow Query Log）、二进制日志（Binary Log）、中继日志（Relay Log）、重做日志（Redo Log）、撤销日志（Undo Log）等，比较重要的是二进制日志（Binary Log）、重做日志（Redo Log）、撤销日志（Undo Log）。

需要注意的是，重做日志（Redo Log）和撤销日志（Undo Log）是InnoDB的日志，如果使用的存储引擎不是InnoDB，就不会有这两个日志。

# buffer pool

在介绍日志之前先简单介绍一下buffer pool。

MySQL的数据是存储在磁盘上的，我们在访问数据时，需要从磁盘读取数据，这个过程是比较慢的，如果在内存上有缓存，访问数据时是从内存读取而不是从磁盘读取，就会快很多。

buffer pool就是在内存上的缓存。

buffer pool具有以下特点：

- buffer pool通常以 **页(page)** 为单位缓存数据，其中的页是数据库页面的副本，页的大小通常是固定的（如 InnoDB 的默认页大小为 16KB）
- **磁盘读写，并不是按需读取，而是按页读取**，一次至少读一页数据（，如果未来要读取的数据就在页中，就能够省去后续的磁盘IO，提高效率。
- 使用特定的页面替换算法（如 LRU，最近最少使用）来决定哪些页应从内存中移出，为新的页腾出空间
- 其大小和相关参数可以通过数据库配置进行调整，以适应不同的工作负载和硬件资源。例如，InnoDB 的 buffer pool 大小可以通过参数 `innodb_buffer_pool_size` 进行配置。

buffer pool的结构如下：

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_buffer_pool.png)

1. 索引页（Index Pages）

索引页存储了InnoDB表的索引结构，包括主键索引（聚集索引）和辅助索引（非聚集索引）。这些索引页被加载到缓冲池中，以加速对表中数据的查找和访问。当执行查询操作时，InnoDB会首先检查所需的索引页是否已经在缓冲池中，如果在，则直接从缓冲池中读取，这称为缓冲池命中；如果不在，则需要从磁盘加载到缓冲池中，这称为缓冲池未命中。

2. 数据页（Data Pages）

数据页存储了InnoDB表的实际数据行。在InnoDB中，数据是按页存储的，每个数据页通常包含多行数据。当需要读取或修改表中的数据时，相关的数据页会被加载到缓冲池中。通过将数据页缓存在内存中，InnoDB可以快速地读取和修改数据，而无需每次都从磁盘加载。

3. Undo页（Undo Pages）

Undo页存储了旧版本的数据，用于支持事务的ACID属性中的隔离性（Isolation）和持久性（Durability）。当执行一个事务时，对数据的修改不会立即生效，而是先记录在Undo页中。如果其他事务需要读取被修改的数据，它可以通过Undo页来获取数据修改前的版本，从而实现多版本并发控制（MVCC）。此外，如果事务失败或回滚，Undo页中的数据可以用于恢复数据到事务开始前的状态。

4. 插入缓存（Insert Buffer）

插入缓存是InnoDB中用于优化非聚集索引插入操作的一种机制。当向一个包含非聚集索引的表中插入数据时，如果相关的索引页不在缓冲池中，InnoDB不会立即将索引键插入到索引页中，而是将其存储在插入缓存中。当相关的索引页被加载到缓冲池时，插入缓存中的索引键会被合并并插入到索引页中。这样可以减少磁盘I/O操作，并提高插入操作的性能。

需要注意的是，插入缓存只适用于非唯一索引的插入操作，并且在某些情况下，如缓冲池足够大或表很小，插入缓存可能不会被使用。

5. 自适应哈希索引（Adaptive Hash Index）

自适应哈希索引是InnoDB存储引擎的一个特性，用于自动根据访问模式创建哈希索引。当某些索引值被频繁访问时，InnoDB会将这些索引值存储在自适应哈希索引中，以加速对这些值的查找。自适应哈希索引是完全自动的，不需要用户手动创建或维护。当哈希索引不再被频繁使用时，InnoDB会自动删除它们以释放内存。

6. InnoDB的锁信息（Lock Information）

InnoDB存储引擎使用锁来确保并发访问时的数据一致性和完整性。在缓冲池中，InnoDB会维护锁信息，以跟踪哪些数据页或行被锁定，以及锁的类型（如共享锁或排他锁）。这些锁信息对于实现事务的隔离性和并发控制至关重要。当事务尝试访问被其他事务锁定的数据时，它会根据锁的类型和事务的隔离级别来决定是等待锁释放还是立即返回错误。

# redo log

## 什么是WAL技术

WAL（Write-Ahead Logging，预写日志）是一种确保数据一致性和可靠性的重要技术，广泛应用于数据库管理系统中。WAL 技术的核心思想是，在将数据更新写入数据库之前，先将这些更新记录到日志文件中。

具体步骤如下：

1. **写日志**：在执行数据修改操作之前，首先将这些修改操作记录到 WAL 日志文件中。这些日志条目通常包括事务 ID、修改的表、被修改的行、修改的具体内容等信息。
2. **写数据**：将实际的数据修改操作应用到数据库中（写入内存中的缓冲区或磁盘上的数据文件）。
3. **持久化日志**：确保日志文件中的记录被安全地写入磁盘，以防止在系统崩溃时丢失。
4. **持久化数据**：在适当的时机，将缓冲区中的数据持久化到磁盘上的数据文件中。

通过先记录日志再执行数据修改，WAL 技术确保了系统在崩溃后能够通过日志进行恢复，保证数据一致性。即使系统在数据写入过程中崩溃，也可以通过重放 WAL 日志中的记录来恢复未完成的事务。

## 什么是redo log

上述说明了什么是WAL技术，对应MySQL来说，就是写操作不是立刻写到磁盘上，而是先写日志，然后在合适的时间写到磁盘上。

redo log 用于记录对数据库的修改操作，确保在系统崩溃后可以通过这些日志进行恢复，使数据库恢复到崩溃前的一致状态。为了防止崩溃、或者断电导致的数据丢失，当有一条记录更新时，InnoDB引擎就会先更新内存（同时标记为脏页），然后将本次对这个页的修改以 redo log的形式记录下来，这时就算更新完成，后续InnoDB引擎会在适当的时候，由后台线程将缓存在buffer poll的脏页刷新到磁盘（这就是WAL技术）。

redo log是物理日志，记录的是数据页的物理变化。例如，某个事务将某一页中的某个偏移量处的数据从旧值改为新值。

在事务提交时，只要先将 redo log 持久化到磁盘即可，可以不需要等到将缓存在 Buffer Pool 里的脏页数据持久化到磁盘。

当系统崩溃时，虽然脏页数据没有持久化，但是 redo log 已经持久化，接着 MySQL 重启后，可以根据 redo log 的内容，将所有数据恢复到最新的状态。

## Redo Log 的作用

1. **数据恢复**：在系统崩溃或硬件故障后，使用 redo log 可以恢复所有已经提交的事务，将数据库恢复到最新的一致状态。
2. **崩溃恢复**：通过重放 redo log 中的记录，可以将数据库恢复到崩溃前的状态，确保数据的完整性和一致性。
3. **持久性保证**：redo log 确保事务的持久性（即事务的 ACID 特性中的 Durability），即使在系统崩溃后，已经提交的事务不会丢失。

## redo log什么时候写入磁盘

redo log也有缓存，不是直接写入磁盘，而是写入redo log的缓存——redo log buffer，那redo log什么时候写入磁盘呢？

1. MySQL正常关闭时
2. checkpoint机制，InnoDB 通过 checkpoint 机制管理 redo log 的使用和重用。当 redo log 文件接近写满时，系统会触发 checkpoint，将内存中的脏页（已修改但未写入磁盘的数据页）写入数据文件，并更新 checkpoint 位置。这使 redo log 文件能够循环重用，并确保系统在崩溃恢复时可以从 checkpoint 开始重做操作。
3. 每次事务提交时

在每次事务提交时，会将redo log写入磁盘，但是写入磁盘有三种方式，InnoDB提供了一个关键参数 `innodb_flush_log_at_trx_commit` 来控制redo log的写入和刷新策略。这个参数有三种设置，每种设置决定了redo log何时写入磁盘：

-  **`innodb_flush_log_at_trx_commit = 1`** （默认值）：

  - 每次事务提交时，InnoDB 会将 redo log buffer 中的日志条目写入到 redo log 文件，并立即刷新到磁盘。

  - 这种设置保证了每个提交的事务都持久化到磁盘，即使系统崩溃，已提交的事务也不会丢失。

-  **`innodb_flush_log_at_trx_commit = 0`** ：

  - 每次事务提交时，redo log buffer 中的日志条目不会立即写入磁盘，而是由后台线程每秒将 redo log buffer 刷新到 redo log 文件。

  - 这种设置在系统崩溃时可能会丢失最近一秒内的事务，但可以提高性能，因为减少了磁盘 I/O 操作。

-  **`innodb_flush_log_at_trx_commit = 2`** ：

  - 每次事务提交时，InnoDB 会将 redo log buffer 中的日志条目写入到 redo log 文件，但不会立即刷新到磁盘，而是依赖于操作系统的文件系统缓存刷新机制（通常每秒一次）。

  - 这种设置在系统崩溃时可能会丢失最近一秒内的事务，但性能介于 `innodb_flush_log_at_trx_commit=0` 和 `innodb_flush_log_at_trx_commit=1` 之间。

解释一下`innodb_flush_log_at_trx_commit = 2`的场景。在计算机操作系统中，用户空间(user space)下的缓冲区数据一般情况下是无法直接写入磁盘的，中间必须经过操作系统内核空间(kernel space)缓冲区(OS Buffer)。因此，redo log buffer / undo log buffer写入redo log file / undo log file实际上是先写入OS Buffer，然后再通过系统调用fsync()将其刷到redo log file / undo log file中。所以在参数等于2的时候，是将日志写入文件，但是不会立刻刷新到磁盘：

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_log_fsync.png)

## redo log组

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/redo_log_group.png)

redo log日志文件不止一个，而是以一个日志文件组出现的，每个redo log日志文件大小都是一样的，比如上图一组有4个文件，每个文件大小1GB，整个redo log日志文件组容量就有4GB。

日志的文件名字是`ib_logfile+编号`，比如上图就是`ib_logfile0、ib_logfile1……`

redo log日志是环形数组的形式，从头开始写，写到末尾就又回到开头。

日志文件组中有两个重要的属性，分别是：

- `write pos`是当前记录的位置，一边写一遍后移
- `check point`是当前要擦除的位置，也是向后移动

redo log是为了防止buffer pool中的脏页丢失而设计的，随着系统运行，buffer pool中的脏页刷新到了磁盘上，对应redo log的记录就没用了，这时就可以擦除旧记录，腾出新空间。

当write pos追上了checkpoint，说明redo log文件满了，这时，MySQL就不能再执行新的更新操作，MySQL就会阻塞（针对并发量大时，适当设置redo log文件大小），此时要停下来将buffer pool中的脏页刷新到磁盘中，然后标记redo log哪些记录可以被擦除，接着要对旧的redo log记录进行擦除，腾出新的空间，checkpoint向后移动，MySQL恢复正常，继续执行更新操作

## redo log也要写入磁盘，岂不是多余？

redo log在写的时候是追加操作，磁盘是**顺序写**，而直接写数据，要先找到数据的位置，然后再写磁盘，是**随机写**。

所以这是WAL技术的一个有点：将随机写变更顺序写提高性能

# undo log

undo log是回滚日志，用来回滚行记录到某个版本，undo log一般是逻辑日志，根据行的数据变化进行记录，可以简单的理解为：当insert一条记录时，undo log会记录一条对应的delete语句；当update一条语句时，undo log记录的是一条与之操作相反的语句。它保证了事务ACID特性中的原子性(Atomicty)

undo log跟redo log一样也是在SQL操作数据之前记录的，也就是SQL操作先记录日志，再进行操作数据

当事务需要回滚时，可以从undo log中找到相应的内容进行回滚操作，回滚后数据恢复到操作之前的状态

一条记录的每次更新操作产生的undo log都有一个roll_pointer指针和一个trx_id事务id

- 通过trx_id可以知道该记录是被那个事务修改的
- roll_pointer指针将这些undo log串成一个链表，这个链表就被成为版本链
- undo log是MVCC实现的关键，通过ReadView + undo log实现MVCC(多版本并发控制)



![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/undo_log_%E7%89%88%E6%9C%AC%E9%93%BE.png)

# binlog

binlog用于记录所有对数据库进行更改的额操作（例如，数据的插入、更新和删除，不包含查询操作），以及这些操作的顺序。主要用于数据恢复、复制和审计。在事务提交时，将该事务执行过程中产生的所有binlog统一写进binlog文件。

redo log和binlog都记录了数据库的修改操作，为什么还需要binlog？

首先看看redo log和binlog的区别

|          | redo log                                                     | binlog                                                       |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 日志种类 | redo log是物理日志，记录的是数据页的物理变化。例如，某个事务将某一页中的某个偏移量处的数据从旧值改为新值 | binlog是逻辑日志，它记录的是对数据库进行的高层次的逻辑操作，例如 SQL 语句的执行，而不是底层数据页的物理变化。 |
| 所处位置 | redo log是InnoDB引擎实现的，是在存储引擎层的，只有使用InnoDB存储引擎时，才会有这个日志 | binglog是server层实现的，所有的存储引擎都会有这个日志        |
| 作用     | redo是物理日志，具有crash-safe能力，用户掉电或者故障恢复     | binlog是逻辑日志，记录的所有逻辑操作，用于备份、主从复制     |
| 写入方式 | 循环写，日志大小固定，写满后需要刷脏页腾出空间再能再写       | 追加写，写满一个文件就会创建新的文件，保存全量日志           |

## binlog日志格式

Binlog 支持三种日志格式，用户可以根据需要进行配置：

- **STATEMENT**：记录 SQL 语句。这种方式体积小，但在某些情况下（如含有不确定因素的函数）可能无法精确重放。（STATEMENT 有动态函数的问题，比如你用了 uuid 或者 now 这些函数，你在主库上执行的结果并不是你在从库执行的结果，这种随时在变的函数会导致复制的数据不一致）
- **ROW**：记录行级别的变化。每次修改记录的具体行的变化，更加精确，但日志体积较大。比如执行批量 update 语句，更新多少行数据就会产生多少条记录，使 binlog 文件过大，而在 STATEMENT 格式下只会记录一个 update 语句而已
- **MIXED**：结合了 STATEMENT 和 ROW 的优点，根据具体情况自动选择日志格式。

## 主从复制

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_master_slave_copy.png)

MySQL主从复制依赖binlog。master服务器将数据的改变记录二进制日志，当master上的数据发生改变时，则将其改变写入二进制日志中，salve服务器会在一定时间间隔内对master二进制日志进行探测其是否发生改变，如果发生改变，则开始一个I/OThread请求master二进制事件，同时主节点为每个I/O线程启动一个dump线程，用于向其发送二进制事件，并保存至从节点本地的中继日志中，从节点将启动SQL线程从中继日志中读取二进制日志，在本地重放，使得其数据和主节点的保持一致，最后I/OThread和SQLThread将进入睡眠状态，等待下一次被唤醒。

## binlog刷盘时机

<img src="https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/binlog_cache.png" style="zoom:20%;" />

在每次事务执行过程中，会把日志写到缓存（binlog cache）中，提交时，MySQL会根据`sync_binlog`参数决定是否立刻刷盘。

| 值       | 刷盘频率                                                     |
| -------- | ------------------------------------------------------------ |
| 0        | 每次提交事务，只会调用write将日志写入文件，没有经过 fsync()，后续由操作系统决定数据什么时候写入磁盘 |
| 1        | 每次事务提交时，MySQL 都会将 Binlog 从缓存中写入到磁盘，并调用 `fsync()` 确保数据被持久化到磁盘。 |
| N(N > 1) | 每提交 `sync_binlog` 次事务才将 Binlog 刷盘一次。            |

MySQL中默认是0，这时候性能比较好，但是风险也比较大。一旦机器宕机或者服务崩溃，还没刷到磁盘的数据就没了。

如果在MySQL出现了性能瓶颈，且瓶颈在IO上，可以将 sync_binlog 设置为大于 1 的值（比较常见是 100~1000）。这样做的风险是，主机掉电时会丢 binlog 日志。

# 两阶段提交

## 什么是两阶段提交

**两阶段提交（Two-Phase Commit，2PC）** 是一种分布式事务协议，用于确保分布式系统中的多个节点能够一致地执行事务。它通过两个阶段的操作（准备阶段和提交阶段）来确保所有参与节点都能达成一致，要么全部提交事务，要么全部回滚事务。以下是对两阶段提交的详细解释：

两阶段提交的步骤：

阶段一：准备阶段（Prepare Phase）

1. **事务协调者（Transaction Coordinator）** 向所有参与者（Participants）发送准备请求（Prepare Request），询问它们是否可以执行并准备提交事务。
2. **参与者** 在接收到准备请求后，执行本地事务的预处理操作，但不提交（只是准备好提交）。然后，参与者将预处理的结果（准备好或失败）反馈给事务协调者。

阶段二：提交阶段（Commit Phase）

1. **事务协调者** 收集所有参与者的反馈：
   - 如果所有参与者都准备好了，事务协调者向所有参与者发送提交请求（Commit Request），要求它们正式提交事务。
   - 如果任何一个参与者反馈失败，事务协调者向所有参与者发送回滚请求（Rollback Request），要求它们回滚之前的预处理操作。
2. **参与者** 在接收到提交请求后，正式提交事务；在接收到回滚请求后，回滚事务。

## 两阶段提交在 MySQL 中的应用

在 MySQL 中，两阶段提交用于确保 Binlog 和 InnoDB 的事务日志（redo log）的一致性。这种机制确保了在 MySQL 主从复制中，即使在崩溃恢复的情况下，主从服务器的数据也能保持一致。

MySQL 两阶段提交的具体过程

1. **准备阶段**：

   - InnoDB 存储引擎将事务的变更记录写入 redo log，并标记为准备提交（prepare）。

2. **提交阶段**：

   - MySQL 服务器将事务的变更写入 Binlog，如果 Binlog 写入成功，InnoDB 将 redo log 标记为已提交。

   - 如果 Binlog 写入失败，InnoDB 将 redo log 回滚。

## 为什么需要两阶段提交

事务提交后，redo log和binlog都需要持久化到磁盘，但这是两个独立的逻辑，可能出现半成功的状态，造成两份日志的不一致。而redo log影响主库的数据，binlog影响从库的数据，所以redo log和binlog必须保持一致，才能保证主从数据一致。
