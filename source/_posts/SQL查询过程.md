---
title: SQL查询过程
date: 2024-06-23 21:58:08
categories: MySQL
tags: [MySQL, 数据库]
---

# MySQL基本架构

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/MySQL%E5%9F%BA%E6%9C%AC%E6%9E%B6%E6%9E%84.png)

从上图看，MySQL分为两部分：Server层 和 存储引擎 

**Server层**：负责建立连接，分析和执行SQL。涵盖MySQL的大多数核心服务功能，主要包括连接器，查询缓存、解析器、预处理器、优化器、执行器等，以及所有的内置函数（如日期、时间、数学和加密函数等）

**存储引擎层**：负责数据的存储和提取。支持 InnoDB、MyISAM、Memory 等多个存储引擎。现在最常用的存储引擎是 InnoDB，它从 MySQL 5.5.5 版本开始成为了默认存储引擎。

# 连接器

执行SQL语句首先得连接到数据库上，这时候就会遇到连机器。连接器负责跟客户端建立连接、获取权限、维持和管理链接。链接命令如下：

```shell
mysql -h$ip -P$port -u$user -p
```

为了密码的安全，不在如上命令后写密码，而是直接执行上述命令在后续的对话框中输入密码。

连接的过程需要先经过 TCP 三次握手，因为 MySQL 是基于 TCP 协议进行传输的，如果 MySQL 服务并没有启动，则会报一个 "Can't not connect to MySQL through socket ...."

如果MySQL服务正常，完成TCP连接之后，连接器就要开始验证用户名和密码

- 如果用户名或者密码不对，则会报 "Access denied for user"
- 如果用户名密码认证通过，连接器会到权限表里面查出你拥有的权限。之后，这个连接里面的权限判断逻辑，都将依赖于此时读到的权限

因为在成功建立连接后，连接器已经校验了权限，所以后续管理员账号哪怕对这个权限做了修改，也不会影响已经存在的连接的权限，只有新建的连接才会使用新的权限。



如何查看MySQL的连接呢？

连接完成后，如果你没有后续的动作，这个连接就处于空闲状态，你可以在 `show processlist` 命令中看到它。其中的 Command 列显示为“Sleep”的这一行，就表示现在系统里面有一个空闲连接。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5.png)

共有两个用户名为 root 的用户连接了 MySQL 服务，其中 id 为 6 的用户的 Command 列的状态为 `Sleep` ，这意味着该用户连接完 MySQL 服务就没有再执行过任何命令，也就是说这是一个空闲的连接，并且空闲的时长是 736 秒（ Time 列）。

客户端如果太长时间没动静，连接器就会自动将它断开。这个时间是由参数 wait_timeout 控制的，默认值是 8 小时。(`mysql> show variables like 'wait_timeout';` 可以查看此配置)

也可以手动断开空闲的连接，使用 `kill connection + id` 

如果在连接被断开之后，客户端不会马上感知，当再次发送请求的时，就会收到一个错误提醒： Lost connection to MySQL server during query。这时候如果你要继续，就需要重连，然后再执行请求了。

建立连接的过程通常是比较复杂的，所以在使用中要尽量减少建立连接的动作，也就是尽量使用长连接。（使用长连接，通过连接池维护一些长连接）

但是MySQL的连接数有限制，支持的最大连接数由 max_connections 参数控制，如果超过这个限制就会拒绝接下来的连接请求，并报错提示："Too many connections"，而且使用长连接后可能会占用内存增多，因为 MySQL 在执行查询过程中临时使用内存管理连接对象，这些连接对象资源只有在连接断开时才会释放。如果长连接累计很多，将导致 MySQL 服务占用内存太大，有可能会被系统强制杀掉，这样会发生 MySQL 服务异常重启的现象。



那如何解决长连接占用内存问题呢？

1. 定期断开长连接。使用一段时间，或者程序里面判断执行过一个占用内存的大查询后，断开连接，之后要查询再重连。
2. 如果用的是 MySQL 5.7 或更新版本，可以在每次执行一个比较大的操作后，通过执行 mysql_reset_connection 来重新初始化连接资源。这个过程不需要重连和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。注意这是接口函数不是命令，那么当客户端执行了一个很大的操作后，在代码里调用 mysql_reset_connection 函数来重置连接，达到释放内存的效果。



小结：

- 与客户端进行 TCP 三次握手建立连接；
- 校验客户端的用户名和密码，如果用户名或密码不对，则会报错；
- 如果用户名和密码都对了，会读取该用户的权限，然后后面的权限逻辑判断都基于此时读取到的权限；



# 查询缓存

连接建立完成后，执行逻辑就会来到第二步：查询缓存。

MySQL 拿到一个查询请求后，会先到查询缓存看看，之前是不是执行过这条语句。之前执行过的语句及其结果可能会以 key-value 对的形式，被直接缓存在内存中。key 是查询的语句，value 是查询的结果。如果你的查询能够直接在这个缓存中找到 key，那么这个 value 就会被直接返回给客户端。如果查询的语句没有命中查询缓存中，那么就要往下继续执行，等执行完后，查询的结果就会被存入查询缓存中。

**但是查询缓存往往弊大于利**

**查询缓存的失效非常频繁，只要有对一个表的更新，这个表上所有的查询缓存都会被清空。**对于更新比较频繁的表，查询缓存的命中率很低的，如果刚缓存了一个查询结果很大的数据，还没被使用的时候，刚好这个表有更新操作，查询缓冲就被清空了。

查询缓存受 `query_cache_type`参数控制，将其设置成 "DEMAND"，SQL就默认不使用查询缓存，对于要使用查询缓存的，可以用 SQL_CACHE 显示指定，例如：

```SQL
select SQL_CACHE * from T where ID=1；
```

在 MySQL 8.0版本之后，查询缓存的功能被移除了。注意：此处移除的是server层的查询缓存，不是InnoDB存储引擎中的 buffer pool。

# 分析器

没有命中查询缓存的查询SQL，就要开始真正执行了。所以接下来要对SQL进行分析

分析器会做两种分析：**词法分析**、**语法分析**

词法分析：MySQL根据输入的字符串识别出关键字来，比如根据输入的 "select" 识别出这是一个查询语句。

语法分析：语法分析会根据语法规则，判断输入的这个SQL是否符合MySQL的语法，如果语句有错误，就会收到"You have an error in your SQL syntax"的错误提醒，

# 执行SQL

经过解析器后，就要进入执行SQL查询语句的流程了，查询分三个阶段：预处理阶段、优化阶段、执行阶段

## 预处理器

- 检查SQL查询语句中的表或者字段是否存在
- 将 `select *` 中的`*`符号，扩展为表中所有的列

## 优化器

在预处理过后，还需要优化器进行处理

**优化器主要负责将 SQL 查询语句的执行方案确定下来**，比如在表里面有多个索引的时候，决定使用哪个索引，或者在一个语句有多表关联（join）的时候，决定各个表的连接顺序。

如何知道优化器选择了哪个索引？

可以在查询语句最前面加个 `explain` 命令，这样就会输出这条 SQL 语句的执行计划，然后执行计划中的 key 就表示执行过程中使用了哪个索引，当 key 为 `PRIMARY` 就是使用了主键索引。

> type显示的是访问类型，访问类型表示我是以何种方式去访问我们的数据，最容易想的是全表扫描，直接暴力的遍历一张表去寻找需要的数据，效率非常低下，访问的类型有很多，效率从最好到最坏依次是：
>
> 所有的连接类型中，上面的最好，越往下越差。在常用的链接类型中：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
>
> 以上访问类型除了 all，都能用到索引,一般情况下，得保证查询至少达到range级别，最好能达到ref
>
> --all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。
> explain select * from emp;
>
> --index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序
> explain  select empno from emp;
>
> --range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN()
> explain select * from emp where empno between 7000 and 7500;
>
> --index_subquery：利用索引来关联子查询，不再扫描全表
> explain select * from emp where emp.job in (select job from t_job);
>
> --unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引
>  explain select * from emp e where e.deptno in (select distinct deptno from dept);
>
> --index_merge：在查询过程中需要多个索引组合使用
>
> --ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式
> explain select * from emp e where  e.mgr is null or e.mgr=7369;
>
> --ref：使用了非唯一性索引进行数据的查找，查询用到了非唯一性索引，或者关联操作只使用了索引的最左前缀。
>  create index idx_3 on emp(deptno);
>  explain select * from emp e,dept d where e.deptno =d.deptno;
>
> --eq_ref ：通常出现在多表的 join 查询，被驱动表通过唯一性索引（UNIQUE 或 PRIMARY KEY）进行访问，此时被驱动表的访问方式就是 eq_ref。eq_ref 是除 const 之外最好的访问类型。使用唯一性索引进行数据查找
> explain select * from emp,emp2 where emp.empno = emp2.empno;
>
> --const：主键索引或者唯一索引,这个表至多有一个匹配行；
> explain select * from emp where empno = 7369;
>
> --system：system 是 const 的一种特例，只有一行满足条件，对于 MyISAM、Memory 的表，只查询到一条记录，也是 system。比如系统库的这张表（8.0 的版本中系统表全部变成 InnoDB 存储引擎了）：
> 表只有一行记录（等于系统表），这是const类型的特例，平时不会出现；
> EXPLAIN SELECT * FROM mysql.proxies_priv;

## 执行器

MySQL 通过分析器知道了你要做什么，通过优化器知道了该怎么做，于是就进入了执行器阶段，开始执行语句。

开始执行的时候，要先判断一下你对这个表 T 有没有执行查询的权限，如果没有，就会返回没有权限的错误(在工程实现上，如果命中查询缓存，会在查询缓存返回结果的时候，做权限验证。查询也会在优化器之前调用 precheck 验证权限)。
