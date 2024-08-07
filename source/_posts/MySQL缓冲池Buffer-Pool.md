---
title: MySQL缓冲池Buffer Pool
date: 2024-08-07 20:07:53
categories: MySQL
tags: [MySQL,数据库]
---

# 什么是Buffer Pool

MySQL的数据是存储在磁盘上的，我们在访问数据时，需要从磁盘读取数据，这个过程是比较慢的，如果在内存上有缓存，访问数据时是从内存读取而不是从磁盘读取，就会快很多。

Buffer Pool就是MySQL在内存上的缓存。

Buffer Pool是InnoDB存储引擎的。处于引擎层。

有了Buffer Pool之后，在读取数据时，如果数据存在于Buffer Pool中，就会直接读取Buffer Pool中的数据。在修改数据时，如果数据在Buffer Pool中，则直接Buffer Pool中数据所在的页，然后将其设置为脏页，最后由后台线程将脏页写入磁盘。

buffer pool具有以下特点：

- buffer pool通常以 **页(page)** 为单位缓存数据，其中的页是数据库页面的副本，页的大小通常是固定的（如 InnoDB 的默认页大小为 16KB）
- **磁盘读写，并不是按需读取，而是按页读取**，一次至少读一页数据（，如果未来要读取的数据就在页中，就能够省去后续的磁盘IO，提高效率。
- 使用特定的页面替换算法（如 LRU，最近最少使用）来决定哪些页应从内存中移出，为新的页腾出空间
- 其大小和相关参数可以通过数据库配置进行调整，以适应不同的工作负载和硬件资源。例如，InnoDB 的 buffer pool 大小可以通过参数 `innodb_buffer_pool_size` 进行配置。默认是128M，一般建议设置为物理内存的60%~80%

# Buffer Pool的结构

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

# Buffer Pool的管理

InnoDB把存储的数据划分成了若干个 `页` 。以页作为和磁盘交互的基本单位，一个页的默认大小是16KB。因此，Buffer Pool同样需要按照 `页`来划分。

为了更好的管理这些在Buffer Pool中的缓存页，InnoDB为每一个缓存页都创建了一个「控制块」，控制块信息包括「缓存页的表空间、页号、缓存页地址、链表节点」等。

<img src="https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_buffer-pool_control_block.png" style="zoom:150%;" />

每个控制块大约占缓存页的5%，而系统设置的innodb_buffer_pool_size的大小不包括控制块，所以Innodb存储引擎向操作系统申请的内存空间，会比设置的innodb_buffer_pool_size大5%左右。控制块排在前面，缓存页排在后面，他们之间会存在没有分配完的内存碎片。

## Free链表

Free链表的作用是管理空闲页。

当MySQL运行一段时间后，Buffer Pool中既有已使用的，也有空闲的。

从磁盘读取一页数据，需要把他加载到Buffer Pool中时，需要知道那些缓存页是空闲的， 此时如果从头遍历，查看那些缓存页是空闲的，就显得太低效。

所以，为了高效的找到空闲的缓存页，InnoDB使用链表存放所有的空闲缓存页的控制块，当需要把一页数据加载到Buffer Pool时，从Free链表中获取一个控制块，根据控制块记录的缓存页的地址，就可以找到一个空闲缓存页，然后把查询到的页数据加入到缓存页中，把此页所属的表空间编号、页号信息写入此缓存页对应的控制块，最后从free链表中移出此控制块。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_buffer-pool_free_list.png)

## Flush链表

Flush链表的作用是管理脏页。

当客户端更新数据时，如果数据在Buffer Pool中，就会去更新Buffer Pool中对应的缓存页的数据，这个缓存页也就会被标记为「脏页」，然后再由后台线程将脏页写入磁盘。

同理，为了能够高效的找到脏页，于是就设计出了Flush链表，作用和Free链表类似，链表的节点也是控制块，区别在于Flush链表的元素都是脏页。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_buffer-pool_free_list.png)

### 脏页什么时候刷盘

当数据在Buffer Pool中时，修改数据，就是修改Buffer Pool中数据所在的页，然后将其设置为脏页，但是磁盘中的数据还没有改变。

如果此时宕机，或者MySQL进程挂了，数据怎么办？

InnoDB的更新采用了WAL（write ahead log）的方式，先写了日志（redo log），再写磁盘，通过redo log让MySQL具有crash safe能力。

刷盘时机：

- 当redo log日志满了，就会主动触发脏页刷新到磁盘
- Buffer Pool空间不足，需要将一部分数据淘汰掉，如果被淘汰的数据是脏页，则要先将脏页刷盘
- MySQL空闲时，后台线程定期将适量的脏页刷入磁盘
- MySQL正常关闭前，会把所有的脏页刷入磁盘

## LRU链表

Buffer Pool的大小是有限的。每次添加新数据时，都会消耗一个空闲缓存页，当Free链表中的空闲页为空，再添加新数据时，就需要淘汰掉一些数据来腾出缓存页。

这时肯定是淘汰掉使用频率最低的缓存页，所以InnoDB就使用了LRU（least recently used 最近最少使用算法）来淘汰缓存页，LRU链表记录访问过的缓存页对应的控制块。

简单的LRU思路：

- 当访问的页在Buffer Pool中，就直接把该页放到LRU链表的头部
- 当访问的页不在Buffer Pool中，先淘汰掉LRU链表末尾的节点，再把访问页放到LRU链表头部

但是InnoDB有预读机制，如果如同预读机制加载进来的页，试用频率不高，但是因为加载这个页，可能把使用频率高的页淘汰掉。全表扫描没有加where条件加载的页，也可能会把使用频率高的页淘汰掉。于是需要对LRU进行优化。

### 预加载优化

InnoDB把LRU链表分成了两部分：young区域、old区域

young区域存储使用频率高的缓存页，也叫热数据。

old区域存储使用频率低的缓存页，也叫冷数据。

old区域占据整个LRU链表长度的比例可以通过`innodb_old_blocks_pct`参数来设置，默认是37，表示LRU链表中，young区域与old区域的比例是 63:37。

当某个页初次加载到Buffer Pool中时，缓存页对应的控制块加到LRU链表的old区域的头部，这样通过预读加载进来的页，放到old区域，不影响young区域的数据，后续会从old区域逐渐被淘汰。

当再次访问old区域控制块对应的缓存页时，此控制块会从old区域移动到young区域的头部，此缓存页作为热数据处理。

当young区域放满后，又有新的控制块需要从old区域移动到young，则young区域的链尾移动到old区域的链头；当再次访问young区域控制块对应的缓存页时，此控制块移动到young区域的头部。
![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_buffer-pool_lru_list.png)

### 全表扫描优化

以上的优化，虽然在预加载的时候，不会淘汰掉热点数据，但是在全表扫描还是有问题。

全表扫描，一个页有多条记录需要访问，每访问一条记录就算访问一次缓存页，所以这样也算是多次访问，从而清洗young区域。

针对此问题，InnoDB对每次访问old区域控制块时，都会记录一下访问时间，当再次访问，会算出访问的时间间隔与阈值进行对比，若是小于阈值，则不会把控制块移动到young区域，此间隔阈值由`innodb_old_blocks_time`控制，默认是1000，单位毫秒，这样进行全表扫描时，多次访问一个缓存页，就不会移动到young区域。

### young区域优化

为了防止young区域控制块频繁移动到头部，访问young区域前 1/4 区域的控制块，不会进行移动，只有访问后面的 3/4 才会移动。
