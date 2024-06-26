---
title: MySQL索引
date: 2024-06-24 20:51:41
categories: MySQL
tags: [MySQL,数据库]
---

# 什么是索引

当我们想查找书籍中特定的内容时就会想到，去翻目录，就能快速找到目标内容。书籍的目录就是**索引**

在数据库中，索引的定义就是帮助存储引擎快速获取数据的一种数据结构，形象的说就是**索引是数据的目录**

# InnoDB索引模型

在 InnoDB 中，表都是根据主键顺序以索引的形式存放的，这种存储方式的表称为索引组织表。InnoDB 使用了 B+ 树索引模型，所以数据都是存储在 B+ 树中的。

这里顺带说一下InnoDB的表结构：

1. 在 InnoDB 中，每一张表其实就是多个 B+ 树，即一个主键索引树和多个非主键索引树。
2.  执行查询的效率，使用主键索引 > 使用非主键索引 > 不使用索引。
3. 如果不使用索引进行查询，则从主索引 B+ 树的叶子节点进行遍历。

每一个索引在 InnoDB 里面对应一棵 B+ 树。索引名称对应所有索引记录，对应一棵B+树，而不是单条记录。



假设现有一张表如下：

```sql
create table T(
	id bigint not null auto_increment primary key,
  name varchar(32) not null default '',
  age int not null default 0,
  index idx_age(age)
)engine=InnoDB;
```

表中 R1~R5 的 (id,age) 值分别为 (100,1)、(200,2)、(300,3)、(500,5) 和 (600,6)，两棵树的示例示意图如下。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/mysql_index_model.png)

根据叶子结点的内容，索引类型分为了主键索引和非主键索引。

主键索引的叶子节点存的是整行数据。在 InnoDB 里，主键索引也被称为聚簇索引（clustered index）。 key:主键的值，value:整行数据。

非主键索引的叶子节点内容是主键的值。在 InnoDB 里，非主键索引也被称为二级索引（secondary index）。 key：索引列的值， value:主键的值。

基于逐渐索引和普通索引的查询有什么区别呢？

- 如果语句是 `select * from T where id=100`，当使用主键索引进行查询的时候，只需要搜索id这颗b+树
- 如果语句是 `select * from T where age=3`，当使用普通索引查询时，先要搜索age索引树，查询到id为300，在用id索引树查到一次，这个过程称为**回表**（如果语句为 `select id from T where age=3`，这种在二级索引的B+tree就能查询到结果的过程就叫做**覆盖索引**，也就是只需要查一个B+tree就能找到数据）

所以，基于非主键索引的查询需要多扫描一棵索引树。因此，我们应用中应该多使用主键查询。

# 索引维护

B+树为了维护索引的有序性，在插入新值的时候需要做必要的维护。以上图为例，如果插入的值是700，则只要在R5的记录后边插入一个新纪录。如果新插入的值是400，就麻烦了，需要逻辑上挪动后边的数据，空出位置。

而更糟的情况是，如果 R5 所在的数据页已经满了，根据 B+ 树的算法，这时候需要申请一个新的数据页，然后挪动部分数据过去。这个过程称为页分裂。在这种情况下，性能自然会受影响。

除了性能外，页分裂操作还影响数据页的利用率。原本放在一个页的数据，现在分到两个页中，整体空间利用率降低大约 50%。

当然有分裂就有合并。当相邻两个页由于删除了数据，利用率很低之后，会将数据页做合并。合并的过程，可以认为是分裂过程的逆过程。

在很多规范里边，都要求建表语句一定要有自增主键，但是事无绝对，也有一些场景不需要。

自增主键是指自增列上定义的主键，在建表语句中一般是这么定义的： NOT NULL PRIMARY KEY AUTO_INCREMENT。

插入新记录的时候可以不指定 ID 的值，系统会获取当前 ID 最大值加 1 作为下一条记录的 ID 值。

也就是说，自增主键的插入数据模式，正符合了我们前面提到的递增插入的场景。每次插入一条新记录，都是追加操作，都不涉及到挪动其他记录，也不会触发叶子节点的分裂。

而有业务逻辑的字段做主键，则往往不容易保证有序插入，这样写数据成本相对较高。除了考虑性能外，我们还可以从存储空间的角度来看。假设你的表中确实有一个唯一字段，比如字符串类型的身份证号，那应该用身份证号做主键，还是用自增字段做主键呢？由于每个非主键索引的叶子节点上都是主键的值。如果用身份证号做主键，那么每个二级索引的叶子节点占用约 20 个字节，而如果用整型做主键，则只要 4 个字节，如果是长整型（bigint）则是 8 个字节。

**显然，主键长度越小，普通索引的叶子节点就越小，普通索引占用的空间也就越小。**

所以从性能和存储空间上分析，自增主键往往更合理。

但是，假如有的需求场景如下：

- 只有一个索引
- 索引必须是唯一索引

此时就是一个kv场景，由于没有其他索引，也不用考虑其他索引叶子结点的大小问题，这时候查询就用到`尽量使用主键查询`的原则，直接将这个索引设置为主键，避免为此搜索两棵树。

# 索引的分类

上面大概介绍了下索引，对索引有了大致了解后，从 数据结构、物理存储、字段特性、字段个数 四个方面看看都有哪些索引

- 数据结构：B+tree索引、Hash索引、Full-text索引
- 物理存储：聚簇索引（主键索引）、二级索引（辅助索引）
- 字段特性：主键索引、唯一索引、普通索引、前缀索引
- 字段个数：单列索引、联合索引

## 按数据结构分类

从数据结构分类上看，常见的有B+tree索引、Hash索引、Full-text索引灯

每一种存储引擎支持的索引类型不一定相同，具体如下表：

| 索引类型      | InnoDB引擎                                                   | MyISAM引擎 | Memory引擎 |
| ------------- | ------------------------------------------------------------ | ---------- | ---------- |
| B+tree索引    | Yes                                                          | Yes        | Yes        |
| Hash索引      | No（不支持hash索引，但是在内存结构中有一个自适应的hash索引） | No         | Yes        |
| Full-text索引 | Yes（MySQL 5.6版本后支持）                                   | Yes        | No         |

InnoDB在MySQL5.5之后成为默认的存储引擎，B+tree索引类型也是MySQL存储引擎采用最多的索引类型

在创建表时，InnoDB 存储引擎会根据不同的场景选择不同的列作为索引：

- 如果有主键，默认会使用主键作为聚簇索引的索引键（key）
- 如果没有主键，就选择第一个不包含 NULL 值的唯一列作为聚簇索引的索引键（key）
- 在上面两个都没有的情况下，InnoDB 将自动生成一个隐式自增 id 列作为聚簇索引的索引键（key）

其它索引都属于辅助索引（Secondary Index），也被称为二级索引或非聚簇索引。**创建的主键索引和二级索引默认使用的是 B+Tree 索引**。

数据库的索引和数据都是存储在硬盘的，我们可以把读取一个节点当作一次磁盘 I/O 操作。B+Tree 存储千万级的数据只需要 3-4 层高度就可以满足，这意味着从千万级的表查询目标数据最多需要 3-4 次磁盘 I/O，所以**B+Tree 相比于 B 树和二叉树来说，最大的优势在于查询效率很高，因为即使在数据量很大的情况，查询一个数据的磁盘 I/O 依然维持在 3-4次。**

为什么MySQL选择B+tree作为索引的数据结构呢？

**1、B+Tree vs B Tree**

B+Tree 只在叶子节点存储数据，而 B 树 的非叶子节点也要存储数据，所以 B+Tree 的单个节点的数据量更小，在相同的磁盘 I/O 次数下，就能查询更多的节点。

另外，B+Tree 叶子节点采用的是双链表连接，适合 MySQL 中常见的基于范围的顺序查找，而 B 树无法做到这一点。

**2、B+Tree vs 二叉树**

对于有 N 个叶子节点的 B+Tree，其搜索复杂度为`O(logdN)`，其中 d 表示节点允许的最大子节点个数为 d 个。

在实际的应用当中， d 值是大于100的，这样就保证了，即使数据达到千万级别时，B+Tree 的高度依然维持在 3\~4 层左右，也就是说一次数据查询操作只需要做 3\~4 次的磁盘 I/O 操作就能查询到目标数据。

而二叉树的每个父节点的儿子节点个数只能是 2 个，意味着其搜索复杂度为 `O(logN)`，这已经比 B+Tree 高出不少，因此二叉树检索到目标数据所经历的磁盘 I/O 次数要更多。

**3、B+Tree vs Hash**

Hash 在做等值查询的时候效率贼快，搜索复杂度为 O(1)。

但是 Hash 表不适合做范围查询，它更适合做等值的查询，这也是 B+Tree 索引要比 Hash 表索引有着更广泛的适用场景的原因。

## 按物理存储分类

在物理存储上，索引可以分为聚簇索引（主键索引）、二级索引（辅助索引）

这两个区别在前面也提到了：

- 主键索引的 B+Tree 的叶子节点存放的是实际数据，所有完整的用户记录都存放在主键索引的 B+Tree 的叶子节点里；
- 二级索引的 B+Tree 的叶子节点存放的是主键值，而不是实际数据。

所以，在查询时使用了二级索引，如果查询的数据能在二级索引里查询的到，那么就不需要回表，这个过程就是覆盖索引。如果查询的数据不在二级索引里，就会先检索二级索引，找到对应的叶子节点，获取到主键值后，然后再检索主键索引，就能查询到数据了，这个过程就是回表。

## 按字段特性分类

从字段特性上看，索引可以分为 主键索引、唯一索引、普通索引、前缀索引

### 主键索引

主键索引就是一张表的主键上的索引，一张表只能有一个主键索引，主键列的值非空且唯一。

### 唯一索引

唯一索引也就是 `UNIQUE`字段上的索引，一张表可以有多个唯一索引，但是唯一索引的每一列的值，都必须唯一

### 普通索引

普通索引是在普通字段上的索引，不要求字段为主键也不要求字段为`UNIQUE`

### 前缀索引

前缀索引是指对字符类型的前几个字符建立的索引，前缀索引可以建立在字段类型为 char、varchar、binary、varbinary、blob、text的列上

## 按字段个数分类

从字段个数上，索引可以分为单列索引、联合索引

**联合索引**：将多个字段组合成一个索引即为联合索引

现在将课程表中的课程id和课程名组成联合索引(class_id, class_name)，联合索引的b+tree示意图如下

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/index_tree.png)

如图，联合索引的非叶子结点用两个字段的值作为b+tree的key值。当联合索引查询数据时，先按`class_id`字段进行比较，在`class_id`字段相同的情况下再按照`class_name`字段进行比较。即，联合索引查询时，先按`class_id`进行排序，再按照`class_name`进行排序。

因此，在使用联合索引时，存在**最左匹配原则**，也就是按照最左优先的方式进行索引的匹配。在使用联合索引进行查询的时候，如果不遵循**最左匹配原则**，联合索引就会失效。

# 索引失效的情况

使用索引失效有6种情况

- 当使用 左 或者 左右 模糊匹配时，也就是`like %xxx`或者`linke %xxx%`这两种方式都会造成索引失效
- 在查询条件中对索引列使用函数
- 在查询条件中对表达式进行计算
- MySQL遇到字符串和数字比较时，会自动吧字符串转为数字，然后进行比较。如果字符串是索引列，那么就会发生隐式类型转换，由于隐式类型转换是通过 CAST 函数实现的，等同于对索引列使用了函数，所以就会导致索引失效。
- 联合索引中未正确遵循最左匹配原则
- WHERE字句中，如果在 OR 前的条件列是索引列，而在 OR 后的条件列不是索引列，那么索引会失效。

## 对索引使用左或者左右模糊匹配

由于索引B+树是按照索引值有序排列存储的，只能根据前缀进行比较

假设现在有二级索引的B+树结构如下图所示

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/%E4%BA%8C%E7%BA%A7%E7%B4%A2%E5%BC%95.png)

假如现在要查询name字段前缀为 宝 的数据，即`name like '%宝'`，扫描索引过程如下：

- 在根节点查询，宝这个字的拼音大小比根节点第一个索引的爱大，但是比第二个康小，所以走到二层的节点
- 在二层的节点查询，首字爱比宝小，第二个索引宝岛与之匹配，所以走到宝岛的叶子节点查询，即叶子节点的所在层的第二个
- 发现第一个索引值的前缀是宝字，与前缀匹配，于是读取该行，接着继续向右匹配，知道匹配不到宝字前缀的索引值

如果使用`name like '%宝'`方式来查询，结果可能是 宝岛、宝贝、宝宝 等，所以不知道从哪里开始比较，于是就走了全表扫描的

## 对索引使用函数

因为索引保存的是索引字段的原始值，而不是经过函数计算后的值，所以没办法走索引。

比如如下的sql

```sql
select * from product where length(name)=3;
```

这里需要全表扫描name字段，判断长度是否等于3

## 对索引进行表达式计算

如下这个sql，在执行时也是进行了全表扫描

```sql
select * from product where price +35 = 100;
```

但是如果略作修改，把条件改成  `where price = 100-35` 就可以走索引了，为什么？

因为当条件是 `price+35 = 100`时，需要全表扫描，逐个去判断，那个price+35能等于100。但是当条件改为了`price = 100-35`的时候，mysql会进行计算，然后再判断 price = 65。

## 对索引隐式类型转化

如果索引是字段是字符串类型，但是在条件查询中，输入的参数是整型，最后就会走全表扫描。

但是如果索引的字段是整型，输入的参数是字符串，还是会走索引。

因为MySQL在遇到字符串和数字进行比较的时候，会自动把字符串转为数字，然后再进行比较。



MySQL的CAST函数可以修改类型。

当索引字段是字符串类型，输入参数是整型时，会把索引字段转为整型，相当于给索引字段用了CAST函数，而如上所说，**对索引使用函数会导致索引失效**

当索引字段是整型，而输入参数是字符串时，会把字符串转为整型，相当于给入参使用了CAST函数，不影响索引，所以还是会走索引扫描。

## 联合索引非最左前缀

在联合索引的情况下，数据是按照索引第一列排序，第一列数据相同时才会按照第二列排序。

所以如果我们想使用索引中尽可能多的列，查询条件的各个列必须是联合索引中最左边开始连续的列。

联合索引有一些特殊情况，**并不是查询过程使用了联合索引查询，就代表联合索引中的所有字段都用到了联合索引进行索引查询**，也就是可能存在部分字段用到联合索引的 B+Tree，部分字段没有用到联合索引的 B+Tree 的情况。

这种特殊情况就发生在范围查询。联合索引的最左匹配原则会一直向右匹配直到遇到「范围查询」就会停止匹配。**也就是范围查询的字段可以用到联合索引，但是在范围查询字段的后面的字段无法用到联合索引**。

看几个案例：

> `select * from t_table where a > 1 and b = 2`，联合索引（a, b）哪一个字段用到了联合索引的 B+Tree？

由于联合索引是先按照a字段的值进行排序的，所以查找a>1的联合索引记录肯定是相邻的，在进行索引扫描时，可以定位到符合a>1的第一条记录，然后沿着记录的链表向后扫描，直到a>1条件不成立。

但是在符合a>1条件的联合索引记录的范围里，b字段的值是无序的。所以这条语句，只有a字段用到了联合索引，b字段没有用到。



> `select * from t_table where a >= 1 and b = 2`，联合索引（a, b）哪一个字段用到了联合索引的 B+Tree？

a>=1条件的联合索引记录范围里，b字段是无序的，但是对于符合 a = 1的联合索引范围里，b字段的值是有序的。由于联合索引，先按照a字段排序，然后再a字段相同的情况下，再按照b字段排序，所以，从符合a=1 and b=2条件的第一条记录开始扫描，而不需要从第一个a字段值为1的记录开始扫描，所以，此时a字段和b字段都用到了联合索引



> `SELECT * FROM t_table WHERE a BETWEEN 2 AND 8 AND b = 2`，联合索引（a, b）哪一个字段用到了联合索引的 B+Tree？

在MySQL中，BETWEEN包含了 value1和value2边界值，类似 >= and <=，通过上一个案例可知，在a字段等于2 和等于8的时候，b字段可以走联合索引，所以此时，a字段和b字段都可以用到联合索引



> `SELECT * FROM t_user WHERE name like 'j%' and age = 22`，联合索引（name, age）哪一个字段用到了联合索引的 B+Tree？

此时是右模糊匹配，所以左前缀不受影响，在name = j时，age字段可以走联合索引



综上所示，**联合索引的最左匹配原则，在遇到范围查询（如 >、<）的时候，就会停止匹配，也就是范围查询的字段可以用到联合索引，但是在范围查询字段的后面的字段无法用到联合索引。注意，对于 >=、<=、BETWEEN、like 前缀匹配的范围查询，并不会停止匹配**

### 索引下推

上边的案例说`select * from table where a > 1 and b = 2` 语句中的a字段可以走索引，但是b字段不能走索引，那在联合索引里边，是不是回回表呢？

在MySQL5.6之前，只能根据查到的主键值，一个一个回表，再试在MySQL5.6引入了**索引下推**，可以再联合索引遍历的过程中，对联合索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。

索引下推的大概原理是：截断的字段不会在 Server 层进行条件判断，而是会被下推到「存储引擎层」进行条件判断（因为 b 字段的值是在 `(a, b)` 联合索引里的），然后过滤出符合条件的数据后再返回给 Server 层。由于在引擎层就过滤掉大量的数据，无需再回表读取数据来进行判断，减少回表次数，从而提升了性能。

### 索引区分度

建立联合索引时的字段顺序，对索引效率也有很大影响。越靠前的字段被用于索引过滤的概率越高，实际开发工作中**建立联合索引时，要把区分度大的字段排在前面，这样区分度大的字段越有可能被更多的 SQL 使用到**。

区分度就是某个字段 column 不同值的个数「除以」表的总行数，计算公式如下：

> 区分度 = distinct(column) / count(*)

比如，性别的区分度就很小，不适合建立索引或不适合排在联合索引列的靠前的位置，而 UUID 这类字段就比较适合做索引或排在联合索引列的靠前的位置。

因为如果索引的区分度很小，假设字段的值分布均匀，那么无论搜索哪个值都可能得到一半的数据。在这些情况下，还不如不要索引，因为 MySQL 还有一个查询优化器，查询优化器发现某个值出现在表的数据行中的百分比（惯用的百分比界线是"30%"）很高的时候，它一般会忽略索引，进行全表扫描。

# 优化索引的方法

## 前缀索引优化

使用前缀索引是为了减小索引字段大小，可以增加一个索引页中存储的索引值，有效提高索引的查询速度。在一些大字符串的字段作为索引时，使用前缀索引可以帮助我们减小索引项的大小。

不过，前缀索引有一定的局限性，例如：

- order by 就无法使用前缀索引；
- 无法把前缀索引用作覆盖索引；

## 覆盖索引优化

覆盖索引是指，在索引的叶子节点上就能找到值，而不需要再回表。

比如建立一个联合索引，「name, age, address」， 当查小明的年龄和地址的时候，直接从索引中就能得到数据，不需要再回表

## 主键索引最好是自增的

**如果我们使用自增主键**，那么每次插入的新数据就会按顺序添加到当前索引节点的位置，不需要移动已有的数据，当页面写满，就会自动开辟一个新页面。因为每次**插入一条新记录，都是追加操作，不需要重新移动数据**，因此这种插入数据的方法效率非常高。

**如果我们使用非自增主键**，由于每次插入主键的索引值都是随机的，因此每次插入新的数据时，就可能会插入到现有数据页中间的某个位置，这将不得不移动其它数据来满足新数据的插入，甚至需要从一个页面复制数据到另外一个页面，我们通常将这种情况称为**页分裂**。**页分裂还有可能会造成大量的内存碎片，导致索引结构不紧凑，从而影响查询效率**。

另外，主键字段的长度不要太大，因为**主键字段长度越小，意味着二级索引的叶子节点越小（二级索引的叶子节点存放的数据是主键值），这样二级索引占用的空间也就越小**。

## 索引最好设置为NOT NULL

索引列存在 NULL 就会导致优化器在做索引选择的时候更加复杂，更加难以优化，因为可为 NULL 的列会使索引、索引统计和值比较都更复杂，比如进行索引统计时，count 会省略值为NULL 的行。

NULL 值是一个没意义的值，但是它会占用物理空间，所以会带来的存储空间的问题，因为 InnoDB 存储记录的时候，如果表中存在允许为 NULL 的字段，那么行格式 (opens new window)中**至少会用 1 字节空间存储 NULL 值列表**

## 防止索引失效

