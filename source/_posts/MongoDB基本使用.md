---

title: MongoDB基本使用
date: 2024-06-17 14:29:12
categories: MongoDB
tags: [MongoDB,NoSQL,数据库]
---

# 1. MongoDB介绍

## 1.1 MongoDB简介

MongoDB是一个开源、高性能、无模式的文档型数据库，当初的设计就是用于简化开发和方便扩展，是NoSQL数据库产品中的一种。是最像关系型数据库（MySQL）的非关系型数据库。

它支持的数据结构非常松散，是一种类似于 JSON 的 格式叫BSON，所以它既可以存储比较复杂的数据类型，又相当的灵活。

MongoDB中的记录是一个文档，它是一个由字段和值对（field:value）组成的数据结构。MongoDB文档类似于JSON对象，即一个文档认为就是一个对象。字段的数据类型是字符型，它的值除了使用基本的一些类型外，还可以包括其他文档、普通数组和文档数组。

## 1.2 业务应用场景

传统的关系型数据库（如MySQL），在数据操作的“三高”需求以及应对Web2.0的网站需求面前，显得力不从心。**而MongoDB可以应对。**

解释：“三高”需求：

• High performance - 对数据库高并发读写的需求。

• Huge Storage - 对海量数据的高效率存储和访问的需求。

• High Scalability && High Availability- 对数据库的高可扩展性和高可用性的需求。



MongoDB与MySQL对比

<table>
    <tr>
      <td> </td>
      <td> </td>
      <td>MongoDB</td>
      <td>MySQL</td>
    </tr>
    <tr>
      <td rowspan="2">高并发性</td>
      <td>架构设计</td>
      <td>采用无模式（schema-less）设计，文档存储（BSON 格式），可以灵活处理变更数据结构。写入操作不需要锁定整个表或重新定义表结构，写性能较高。</td>
      <td>使用固定模式（schema）设计，表结构变更（例如增加列）可能需要锁定整个表，影响写入性能。虽然支持事务和行级锁定，但在高并发写入时，锁争用仍可能成为瓶颈。</td>
    </tr>
    <tr>
      <td>并发控制</td>
      <td>使用优化的锁机制（如 WiredTiger 存储引擎中的多文档级锁），并且能够通过分片在多个节点上分散写入压力。</td>
      <td>尽管支持行级锁（如 InnoDB 引擎），但在高并发场景下，锁冲突和死锁的概率增大，可能导致性能下降。</td>
    </tr>
    <tr>
      <td rowspan="2">高存储能力</td>
      <td>扩展性</td>
      <td>内置分片（sharding）机制，可以将数据水平拆分到多个节点，几乎无限扩展存储容量，且分片管理自动化。</td>
      <td>缺乏原生的分片机制，水平扩展需要手动配置分区、分库分表，管理复杂且成本高。MySQL Cluster 也能提供一定的扩展性，但配置和维护复杂度较高。</td>
    </tr>
    <tr>
      <td>数据模型</td>
      <td>文档模型（BSON）允许嵌套和数组，适用于复杂和非结构化数据，减少了表关联（JOIN）和多表查询的需求。</td>
      <td>使用关系模型，数据规范化存储，复杂查询常需要 JOIN 操作。随着数据量增加，JOIN 操作的性能瓶颈更明显。</td>
    </tr>
    <tr>
      <td rowspan="2">高可用性</td>
      <td>复制与故障转移</td>
      <td>内置复制集（Replica Sets）机制，支持自动故障转移。当主节点故障时，副本节点可以自动提升为主节点，确保高可用性。</td>
      <td>支持主从复制和半同步复制，但自动故障转移需要额外配置（如使用 MHA 或 Percona XtraDB Cluster），增加了系统复杂度和维护难度。</td>
    </tr>
    <tr>
      <td>分布式架构</td>
      <td>从设计上就是分布式系统，支持地理分布的分片和复制，提高了系统的灾难恢复能力和全球可用性。</td>
      <td>典型部署是集中式架构，虽然可以通过复制和分区实现一定程度的分布式，但不是原生支持，配置和维护更复杂。</td>
    </tr>
  </table>


## 1.3 概念解析

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| :----------- | :--------------- | :---------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |



## 1.4 数据模型

MongoDB的最小存储单位就是文档(document)对象。文档(document)对象对应于关系型数据库的行。数据在MongoDB中以BSON（Binary-JSON）文档的格式存储在磁盘上。

BSON（Binary Serialized Document Format）是一种类json的一种二进制形式的存储格式，简称Binary JSON。BSON和JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型。

BSON采用了类似于 C 语言结构体的名称、对表示方法，支持内嵌的文档对象和数组对象，具有轻量性、可遍历性、高效性的三个特点，可以有效描述非结构化数据和结构化数据。这种格式的优点是灵活性高，但它的缺点是空间利用率不是很理想。

Bson中，除了基本的JSON类型：string,integer,boolean,double,null,array和object，mongo还使用了特殊的数据类型。这些类型包括date,object id,binary data,regular expression 和code。每一个驱动都以特定语言的方式实现了这些类型，查看你的驱动的文档来获取详细信息。

## 1.5 优缺点

MongoDB 是一种 NoSQL 数据库，以其灵活的文档模型和高性能而闻名。以下是 MongoDB 的主要优缺点：

### 优点

1. **灵活的文档模型**
   - MongoDB 使用 BSON（类似 JSON 的二进制格式）存储数据，这允许存储复杂的嵌套数据结构。
   - 模型灵活，模式自由，可以轻松处理数据模型的变化。
2. **高性能**
   - MongoDB 对于读取和写入操作都有很好的性能表现，尤其适用于高吞吐量和低延迟的应用场景。
3. **水平扩展**
   - MongoDB 支持水平扩展（sharding），能够通过将数据分布在多个服务器上来扩展存储容量和计算能力，适应大规模数据存储和访问需求。
4. **高可用性**
   - 通过复制集（replica sets）实现数据的高可用性和自动故障转移，确保数据的可靠性和服务的连续性。
5. **丰富的查询语言**
   - MongoDB 提供了强大的查询语言，可以进行复杂的查询、排序、投影和聚合操作，满足各种数据查询需求。
6. **强大的社区和生态系统**
   - MongoDB 有广泛的社区支持和丰富的第三方工具和库，提供了良好的文档、教程和支持。
7. **地理空间查询**
   - 内置支持地理空间查询，适用于位置数据的存储和查询。

### 缺点

1. **内存消耗高**
   - 由于 MongoDB 的文档存储格式和索引机制，其内存消耗相对较高，对于内存有限的环境可能会有压力。
2. **事务支持有限**
   - 虽然 MongoDB 从 4.0 版本开始支持多文档 ACID 事务，但在某些复杂的事务场景下，仍然不如传统的关系型数据库。
3. **不支持传统的 SQL**
   - MongoDB 的查询语言不同于传统的 SQL，对于习惯于 SQL 的开发者需要学习新的查询语法和操作方法。
4. **数据一致性**
   - 在默认配置下，MongoDB 的写操作是异步的，这意味着在某些情况下可能会导致数据一致性问题。虽然可以通过配置来保证更高的一致性，但可能会影响性能。
5. **索引大小**
   - MongoDB 的索引在内存中占用的空间较大，需要合理设计索引以避免性能问题。
6. **备份和恢复复杂**
   - 尽管 MongoDB 提供了备份和恢复的工具，但在大规模数据场景下，备份和恢复操作可能会比较复杂和耗时。


# 2. MongoDB操作

## 2.1 数据库操作

### 选择和创建数据库

选择和创建数据库的语法格式：

```shell
use 数据库名称
```

如果数据库不存在则自动创建，例如，以下语句创建testdb数据库：

```shell
use testdb
```

查看有权限查看的所有数据库命令

```
show dbs
或
show databases
```

> 注意：在MongoDB中，集合只有在内容插入后才会创建，就是说，创建集合（数据表）后要在插入一个文档（记录），集合才会真正创建

### 查看当前正在使用的数据库命令

```shell
db
```

MongoDB中默认的数据库为test，如果你没有选择数据库，集合将存放在test数据库中。

另外：

- 数据库名可以是满足以下条件的任意UTF-8字符串。

- 不能是空字符串（"")。

- 不得含有' '（空格)、.、$、/、\和\0 (空字符)。

- 应全部小写。

- 最多64字节。

有一些数据库名是保留的，可以直接访问这些有特殊作用的数据库。

- **admin**： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。

- **local:** 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合

- **config**: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

### 数据库的删除

MongoDB删除数据的语法格式如下：

```shell
db.dropDatabase()
```

提示：主要用来删除已经持久化的数据库

## 2.2 集合操作

### 集合的显示创建

基本语法格式：

```shell
db.createCollection(name)
```

参数说明：

- name：要创建的集合名称

例如：创建一个名为mycollection的普通集合

```shell
db.createCollection("mycollection")
```

查看当前库中的表：show tables命令

```shell
show collections
或
show tables
```

集合的命名规范：

- 集合名不能是空字符串""。

- 集合名不能含有\0字符（空字符)，这个字符表示集合名的结尾。

- 集合名不能以"system."开头，这是为系统集合保留的前缀。

- 用户创建的集合名字不能含有保留字符。有些驱动程序的确支持在集合名里面包含，这是因为某些系统生成的集合中包含该字符。除非你要访问这种系统创建的集合，否则千万不要在名字里出现$。

### 集合的隐式创建

当向一个集合中插入一个文档的时候，如果集合不存在，则会自动创建集合。

> 提示：通常我们使用隐式创建集合即可

## 2.3 集合的删除

集合删除语法格式如下：

```shell
db.collection.drop()
或
db.集合.drop()
```

**返回值**

如果成功删除选定集合，则drop()方法返回true，否则返回flase。

例如：要删除mycollection集合

```shell
db.mycollection.drop()
```

## 2.4 文档基本CRUD

文档（document）的数据结构和 JSON 基本一样。

所有存储在集合中的数据都是 BSON 格式。

### 文档的插入

（1）单个文档插入

使用insert()或者save()方法向集合中插入文档，语法如下：

```
db.collection.insert(
	<document or array of document>,
	{
		writeConcern: <document>,
		ordered: <boolean>
	}
)
```

参数：

| Parameter    | Type              | Description                                                  |
| ------------ | ----------------- | ------------------------------------------------------------ |
| document     | document or array | 要插入到集合中的文档或者文档数组。（json格式）               |
| writeConcern | document          | Optional. A document expressing the [write concern](https://docs.mongodb.com/manual/reference/write-concern/). Omit to use the default [write concern](https://docs.mongodb.com/manual/reference/write-concern/).See Write Concern.Do not explicitly set the write concern for the operation if run in a transaction. To use write concern with transactions, see [Transactions and Write Concern](https://docs.mongodb.com/manual/core/transactions/#transactions-write-concern). |
| ordered      | boolean           | 可选。如果为真，则按顺序插入数组中的文档，如果其中一个文档出现错误，MongoDB将返回而不处理数组中的其余文档。如果为假，则执行无序插入，如果其中一个文档出现错误，则继续处理数组中的主文档。在版本2.6+中默认为true |

【示例】

要向comment的集合（表）中插入一条数据：

```shell
db.comment.insert(
{
	"articleid":"100000",
	"content":"今天天气真好，阳光明媚",
	"userid":"1001",
	"nickname":"Rose",
	"createdatetime":new Date(),
	"likenum":NumberInt(10),
	"state":null
	}
)
```

提示：

1）comment集合如果不存在，则会隐式创建

2）mongo中的数字，默认情况下是double类型，如果要存整型，必须使用函数NumberInt(整型数字)，否则取出来就有问题了。

3）插入当前日期使用 new Date()

4）插入的数据没有指定 _id ，会自动生成主键值

5）如果某字段没值，可以赋值为null，或不写该字段。



执行后，如下，说明插入一个数据成功了

```
writeResult({"nInserted": 1})
```

注意：

1. 文档中的键/值对是有序的。
2. 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)。
3. MongoDB区分类型和大小写。
4. MongoDB的文档不能有重复的键。
5. 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。

文档键命名规范：

- 键不能含有\0 (空字符)。这个字符用来表示键的结尾。

- .和$有特别的意义，只有在特定环境下才能使用。

- 以下划线"_"开头的键是保留的(不是严格要求的)。



（2）批量插入

语法：

```
db.collection.insertMany(
	[ <document 1> , <document 2>, ... ],
	{
		writeConcern: <document>,
		ordered: <boolean>
	}
)
```

【示例】

批量插入多条文章评论：

```shell
db.comment.insertMany([
{"_id":"1","articleid":"100001","content":"我们不应该把清晨浪费在手机上，健康很重要，一杯温水幸福你我他。","userid":"1002","nickname":"相忘于江湖","createdatetime":new Date("2019-08-05T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"},
{"_id":"2","articleid":"100001","content":"我夏天空腹喝凉开水，冬天喝温开水","userid":"1005","nickname":"伊人憔悴","createdatetime":new Date("2019-08-05T23:58:51.485Z"),"likenum":NumberInt(888),"state":"1"},
{"_id":"3","articleid":"100001","content":"我一直喝凉开水，冬天夏天都喝。","userid":"1004","nickname":"杰克船长","createdatetime":new Date("2019-08-06T01:05:06.321Z"),"likenum":NumberInt(666),"state":"1"},
{"_id":"4","articleid":"100001","content":"专家说不能空腹吃饭，影响健康。","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-08-06T08:18:35.288Z"),"likenum":NumberInt(2000),"state":"1"},
{"_id":"5","articleid":"100001","content":"研究表明，刚烧开的水千万不能喝，因为烫
嘴。","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-08-
06T11:01:02.521Z"),"likenum":NumberInt(3000),"state":"1"}
]);
```

提示：

插入时指定了 _id ，则主键就是该值。

如果某条数据插入失败，将会终止插入，但已经插入成功的数据不会回滚掉。

因为批量插入由于数据较多容易出现失败，因此，可以使用try catch进行异常捕捉处理，测试的时候可以不处理。如：

```
try {
db.comment.insertMany([
{"_id":"1","articleid":"100001","content":"我们不应该把清晨浪费在手机上，健康很重要，一杯温水幸福你我他。","userid":"1002","nickname":"相忘于江湖","createdatetime":new Date("2019-08-05T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"},
{"_id":"2","articleid":"100001","content":"我夏天空腹喝凉开水，冬天喝温开水","userid":"1005","nickname":"伊人憔悴","createdatetime":new Date("2019-08-05T23:58:51.485Z"),"likenum":NumberInt(888),"state":"1"},
{"_id":"3","articleid":"100001","content":"我一直喝凉开水，冬天夏天都喝。","userid":"1004","nickname":"杰克船长","createdatetime":new Date("2019-08-06T01:05:06.321Z"),"likenum":NumberInt(666),"state":"1"},
{"_id":"4","articleid":"100001","content":"专家说不能空腹吃饭，影响健康。","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-08-06T08:18:35.288Z"),"likenum":NumberInt(2000),"state":"1"},
{"_id":"5","articleid":"100001","content":"研究表明，刚烧开的水千万不能喝，因为烫
嘴。","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-08-
06T11:01:02.521Z"),"likenum":NumberInt(3000),"state":"1"}
]);
} catch (e) {
print (e);
}
```

### 文档的基本查询

查询数据的语法格式如下：

```shell
db.collection.find(<query>, [projection])
```

参数：

| Parameter  | Type     | Description                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| query      | document | 可选。使用查询运算符指定选择筛选器。若要返回集合中的所有文档，请省略此参数或传递空文档({}) |
| projection | document | 可选。指定要在与查询筛选器匹配的文档中返回的字段（投影）。若要返回匹配文档中的所有字段，请忽略此参数。 |

【示例】

（1）查询所有

如果我们要查询comment集合的所有文档，我们输入以下命令

```shell
db.comment.find()
或
db.comment.find({})
```

这里你会发现每条文档会有一个叫_id的字段，这个相当于我们原来关系数据库中表的主键，当你在插入文档记录时没有指定该字段，MongoDB会自动创建，其类型是ObjectID类型。

如果我们在插入文档记录时指定该字段也可以，其类型可以是ObjectID类型，也可以是MongoDB支持的任意类型。

如果我想按一定条件来查询，比如我想查询userid为1003的记录，怎么办？很简单！只 要在find()中添加参数即可，参数也是json格式，如下：

```shell
db.comment.fin({userid:'1003'})
```

如果你只需要返回符合条件的第一条数据，我们可以使用findOne命令来实现，语法和find一样。

如：查询用户编号是1003的记录，但只最多返回符合条件的第一条记录：

```shell
db.comment.findOne({userid:'1003'})
```

（2）投影查询（Projection Query）：

如果要查询结果返回部分字段，则需要使用投影查询（不显示所有字段，只显示指定的字段）。

如：查询结果只显示 _id、userid、nickname :

```shell
db.comment.find({userid:"1003"},{userid:1,nickname:1})
{ "_id" : "4", "userid" : "1003", "nickname" : "凯撒" }
{ "_id" : "5", "userid" : "1003", "nickname" : "凯撒" }
```

默认 _id 会显示。

如：查询结果只显示 、userid、nickname ，不显示 _id ：

```shell
db.comment.find({userid:"1003"},{userid:1,nickname:1,_id:0})
{ "userid" : "1003", "nickname" : "凯撒" }
{ "userid" : "1003", "nickname" : "凯撒" }
```

再例如：查询所有数据，但只显示 _id、userid、nickname :

```shell
db.comment.find({},{userid:1,nickname:1})
```

### 文档的更新

更新文档的语法：

```shell
db.collection.update(query, update, options)
//或
db.collection.update(
	<query>,
	<update>,
	{
		upsert: <boolean>,
		multi: <boolean>,
		writeConcern: <document>
	}
)
```

参数：

| Parameter    | Type                 | Description                                                  |
| ------------ | -------------------- | ------------------------------------------------------------ |
| query        | document             | update的查询条件，类似sql update查询的where子句。            |
| update       | document or pipeline | update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为 sql update查询的set子句 |
| upsert       | boolean              | 可选。如果设置为true，则在没有与查询条件匹配的文档时创建新文档。默认值为false，如果找不到匹配项，则不会插入新文档。 |
| multi        | boolean              | 可选。如果设置为true，则更新符合查询条件的多个文档。如果设置为false，则更新一个文档。默认值为false。 |
| writeConcern | document             | 可选。表示写问题的文档。抛出异常的级别。                     |

提示：

主要关注前四个参数即可。

【示例】

（1）覆盖的修改

如果我们想修改_id为1的记录，点赞量为1001，输入以下语句：

```shell
db.comment.update({_id:"1"},{likenum:NumberInt(1001)})
```

执行后，我们会发现，这条文档除了likenum字段其它字段都不见了，

（2）局部修改

为了解决这个问题，我们需要使用修改器$set来实现，命令如下：

我们想修改_id为2的记录，浏览量为889，输入以下语句：

```shell
db.comment.update({_id:"2"},{$set:{likenum:NumberInt(889)}})
```

这样就OK啦。

（3）批量的修改

更新所有用户为 1003 的用户的昵称为 凯撒大帝 。

```shell
//默认只修改第一条数据
db.comment.update({userid:"1003"},{$set:{nickname:"凯撒2"}})
//修改所有符合条件的数据
db.comment.update({userid:"1003"},{$set:{nickname:"凯撒大帝"}},{multi:true})
```

提示：如果不加后面的参数，则只更新符合条件的第一条记录

（3）列值增长的修改

如果我们想实现对某列值在原有值的基础上进行增加或减少，可以使用 $inc 运算符来实现。

需求：对3号数据的点赞数，每次递增

```
db.comment.update({_id:"3"},{$inc:{likenum:NumberInt(1)}})
```

### 删除文档

删除文档的语法结构：

```shell
db.集合名称.remove(条件)
```

以下语句可以将数据全部删除，请慎用

```shell
db.comment.remove({})
```

如果删除_id=1的记录，输入以下语句

```shell
db.comment.remove({_id:"1"})
```

## 2.5 文档的分页查询

### 统计查询

统计查询使用count()方法，语法如下：

```shell
db.collection.count(query, options)
```

参数：

| Parameter | Type     | Description                  |
| --------- | -------- | ---------------------------- |
| query     | document | 查询选择条件                 |
| options   | document | 可选。用于修改计数的额外选项 |

提示：

可选项暂时不使用。

【示例】

（1）统计所有记录数：

统计comment集合的所有记录数：

```shell
db.comment.count()
```

（2）按条件统计记录数

例如：统计userid为1003的记录条数

```shell
db.comment.count({userid:"1003"})
```

提示：默认情况下 count()方法返回符合条件的全部记录条数。

### 分页列表查询

可以使用limit()方法来读取指定数量的数据，使用skip()方法来跳过指定数量的数据。

基本语法如下所示：

```shell
db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```

如果想返回指定条数的记录，可以在find方法后调用limit来返回结果（TopN），默认值20，例如：

```shell
db.comment.find().limit(3)
```

skip方法同样接收一个数字参数作为跳过的记录条数。（前N个不要）， 默认值是0

```shell
db.comment.fin().skip(3)
```



分页查询：需求： 每页2个，从第二页开始：跳过前两条数据，接着值显示3和4条数据

```shell
//第一页
db.comment.find().skip(0).limit(2)
//第二页
db.comment.find().skip(2).limit(2)
//第三页
db.comment.find().skip(4).limit(2)
```

### 排序查询

sort() 方法对数据进行排序，sort() 方法可以通过参数指定排序的字段，并使用1和-1来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。

语法如下所示：

```shell
db.COLLECTION_NAME.find().sort({KEY:1})
或
db.集合名称.find().sort(排序方式)
```

例如：

对userid降序排列，并对访问量进行升序排列

```shell
db.comment.find().sort({userid:-1,likenum:1})
```

提示：

skip(), limilt(), sort()三个放在一起执行的时候，执行的顺序是先 sort(), 然后是 skip()，最后是显示的 limit()，和命令编写顺序无关。

## 2.6 文档的更多查询

### 正则的复杂条件查询

MongoDB的模糊查询是通过**正则表达式**的方式实现的。格式为：

```shell
db.collection.find({field:/正则表达式/})
或
db.集合.find({字段:/正则表达式/})
```

提示：正则表达式是js的语法，直接量的写法。

例如，我要查询评论内容包含“开水”的所有文档，代码如下：

```shell
db.comment.find({content:/开水/})
```

如果要查询评论的内容中以“专家”开头的，代码如下：

```
db.comment.find({content:/^专家/})
```

### 比较查询

<, <=, >, >= 这个操作符也是很常用的，格式如下:

```shell
db.集合名称.find({ "field" : { $gt: value }}) // 大于: field > value
db.集合名称.find({ "field" : { $lt: value }}) // 小于: field < value
db.集合名称.find({ "field" : { $gte: value }}) // 大于等于: field >= value
db.集合名称.find({ "field" : { $lte: value }}) // 小于等于: field <= value
db.集合名称.find({ "field" : { $ne: value }}) // 不等于: field != value
```

示例：查询评论点赞数量大于700的记录

```shell
db.comment.find({likenum:{$gt:NumberInt(700)}})
```

### 包含查询

包含使用$in操作符。 示例：查询评论的集合中userid字段包含1003或1004的文档

```shell
db.comment.find({userid:{$in:["1003","1004"]}})
```

不包含使用$nin操作符。 示例：查询评论集合中userid字段不包含1003和1004的文档

```shell
db.comment.find({userid:{$nin:["1003","1004"]}})
```

### 条件连接查询

我们如果需要查询同时满足两个以上条件，需要使用$and操作符将条件进行关联。（相 当于SQL的and） 格式为：

```shell
$and:[ { },{ },{ } ]
```

示例：查询评论集合中likenum大于等于700 并且小于2000的文档：

```shell
db.comment.find({$and:[{likenum:{$gte:NumberInt(700)}},{likenum:{$lt:NumberInt(2000)}}]})
```

如果两个以上条件之间是或者的关系，我们使用 操作符进行关联，与前面 and的使用方式相同 格式为：

```shell
$or:[ { },{ },{ } ]
```

示例：查询评论集合中userid为1003，或者点赞数小于1000的文档记录

```shell
db.comment.find({$or:[ {userid:"1003"} ,{likenum:{$lt:1000} }]})
```

## 2.7 常用命令小结

```shell
选择切换数据库：use articledb
插入数据：db.comment.insert({bson数据})
查询所有数据：db.comment.find();
条件查询数据：db.comment.find({条件})
查询符合条件的第一条记录：db.comment.findOne({条件})
查询符合条件的前几条记录：db.comment.find({条件}).limit(条数)
查询符合条件的跳过的记录：db.comment.find({条件}).skip(条数)
修改数据：db.comment.update({条件},{修改后的数据}) 或db.comment.update({条件},{$set:{要修改部分的字段:数据})
修改数据并自增某字段值：db.comment.update({条件},{$inc:{自增的字段:步进值}})
删除数据：db.comment.remove({条件})
统计查询：db.comment.count({条件})
模糊查询：db.comment.find({字段名:/正则表达式/})
条件比较运算：db.comment.find({字段名:{$gt:值}})
包含查询：db.comment.find({字段名:{$in:[值1，值2]}})或db.comment.find({字段名:{$nin:[值1，值2]}})
条件连接查询：db.comment.find({$and:[{条件1},{条件2}]})或db.comment.find({$or:[{条件1},{条件2}]})
```

## 2.8 聚合

MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果。有点类似sql语句中的 count(*)。

### aggregate() 方法

MongoDB中聚合的方法使用 aggregate()。

语法如下：

```shell
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

【实例】

集合中的数据如下：

```shell
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by_user: 'w3cschool.cn',
   url: 'http://www.w3cschool.cn',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   _id: ObjectId(7df78ad8902d)
   title: 'NoSQL Overview', 
   description: 'No sql database is very fast',
   by_user: 'w3cschool.cn',
   url: 'http://www.w3cschool.cn',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 10
},
{
   _id: ObjectId(7df78ad8902e)
   title: 'Neo4j Overview', 
   description: 'Neo4j is no sql database',
   by_user: 'Neo4j',
   url: 'http://www.neo4j.com',
   tags: ['neo4j', 'database', 'NoSQL'],
   likes: 750
},
```

现在我们通过以上集合计算每个作者所写的文章数，使用`aggregate()`计算结果如下：

```shell
db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "w3cschool.cn",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
```

以上实例类似sql语句：

```shell
select by_user, count(*) from mycol group by by_user
```

在上面的例子中，我们通过字段by_user字段对数据进行分组，并计算by_user字段相同值的总和。

### 管道

MongoDB 的管道（Pipeline）是一个用于数据处理和分析的概念，它允许你通过一系列阶段（stages）来处理文档集合中的数据。每个阶段对输入文档执行操作，然后将结果传递到下一个阶段。最终，经过一系列操作后，管道输出处理后的结果。

管道在 MongoDB 中主要用于聚合框架，通过这种方式可以实现数据的过滤、转换、分组、排序、连接等复杂操作。整个管道的结构类似于 Unix 的管道命令，通过一系列简单的步骤，组合起来完成复杂的数据处理任务。

#### 管道的主要特点

1. **逐步处理**：数据经过一系列操作步骤，每个阶段只处理当前的数据并传递给下一个阶段。
2. **组合灵活**：不同的阶段可以自由组合，以满足各种数据处理需求。
3. **高效执行**：MongoDB 会对管道进行优化，尽量减少内存和计算资源的使用。

#### 常用的管道阶段

**$match**：过滤文档。用于筛选文档，仅通过满足指定条件的文档。类似于 SQL 中的 `WHERE` 子句。

```shell
{
  $match: { status: "A" }
}
```

这将筛选出所有 `status` 字段为 "A" 的文档。



**$project**：用于包括或排除文档中的特定字段，还可以创建新的字段。类似于 SQL 中的 `SELECT` 子句。

```shell
{
  $project: { item: 1, status: 1, total: { $multiply: ["$price", "$quantity"] } }
}
```

这会选择 `item` 和 `status` 字段，并创建一个 `total` 字段，其值是 `price` 和 `quantity` 的乘积。



**$group**：用于将文档分组，并对每个组应用聚合操作。类似于 SQL 中的 `GROUP BY` 子句。

```shell
{
  $group: { _id: "$status", totalQuantity: { $sum: "$quantity" } }
}
```

这会根据 `status` 字段对文档进行分组，并计算每个组的 `quantity` 字段总和。



**$sort**：用于对文档进行排序。类似于 SQL 中的 `ORDER BY` 子句。

```shell
{
  $sort: { totalQuantity: -1 }
}
```

这会按 `totalQuantity` 字段降序排列文档。



**$limit**：用于限制返回的文档数量。类似于 SQL 中的 `LIMIT` 子句。

```shell
{
  $limit: 5
}
```

这会限制返回的文档数量为 5。



**$skip**：用于跳过指定数量的文档。通常与 `$limit` 一起使用，实现分页功能。

```shell
{
  $skip: 10
}
```

这会跳过前 10 个文档。



**$unwind**：用于将数组字段拆分为多个文档，每个数组元素对应一个文档。

```shell
{
  $unwind: "$items"
}
```

这会将 `items` 数组字段中的每个元素拆分为单独的文档。



**$lookup**：联表查询。用于在集合之间进行左外连接，类似于 SQL 中的 `JOIN` 操作。

```shell
{
  $lookup:
    {
      from: "otherCollection",
      localField: "itemId",
      foreignField: "id",
      as: "itemDetails"
    }
}
```

这会将当前集合中的 `itemId` 字段与 `otherCollection` 中的 `id` 字段进行匹配，并将匹配结果存储在 `itemDetails` 字段中。



**$addFields**：添加新字段。用于向文档添加新的字段。

```shell
{
  $addFields: { totalPrice: { $multiply: ["$price", "$quantity"] } }
}
```

这会添加一个 `totalPrice` 字段，其值为 `price` 和 `quantity` 的乘积。



**$replaceRoot**：替换根文档。用于用指定的文档替换输入文档。通常用于嵌套文档。

```shell
{
  $replaceRoot: { newRoot: "$itemDetails" }
}
```

这会用 `itemDetails` 字段的内容替换当前文档。



**$count**：计算文档数量。用于计算通过管道的文档数量，并返回结果。

```shell
{
  $count: "total"
}

```

这会返回一个包含文档总数的字段 `total`。



**$facet**：允许在单个聚合管道中同时运行多个子管道。

```shell
{
  $facet: {
    "categorizedByStatus": [
      { $match: { status: { $exists: true } } },
      { $group: { _id: "$status", count: { $sum: 1 } } }
    ],
    "categorizedByPrice": [
      { $match: { price: { $exists: true } } },
      { $group: { _id: { $cond: { if: { $gte: ["$price", 100] }, then: "expensive", else: "cheap" } }, count: { $sum: 1 } } }
    ]
  }
}

```

这会分别计算按 `status` 和 `price` 分类的文档数量。



**$out**：将聚合结果输出到一个指定的集合。

```shell
{
  $out: "outputCollection"
}
```

这会将聚合结果保存到 `outputCollection` 集合中。



**$merge**：合并结果到指定集合。

```shell
{
  $merge: { into: "outputCollection", whenMatched: "merge", whenNotMatched: "insert" }
}
```



【示例】

假设有一个包含销售数据的集合 `sales`，我们想要查询销售状态为 "A" 的文档，并计算每个商品的总销售量，最后按总销售量降序排列，并只取前 5 个结果。

```shell
db.sales.aggregate([
  { $match: { status: "A" } },
  { $group: { _id: "$item", totalQuantity: { $sum: "$quantity" } } },
  { $sort: { totalQuantity: -1 } },
  { $limit: 5 }
]);
```

#### 管道的作用

管道使得 MongoDB 的聚合框架非常强大和灵活。通过管道，你可以：

- **清洗数据**：过滤和转换原始数据，得到更干净的数据集。
- **聚合分析**：进行复杂的统计分析，比如求和、平均值、最大值、最小值等。
- **数据转换**：对数据进行格式化、计算新字段、展开数组等操作。
- **数据集成**：通过联表查询将多个集合的数据合并起来。

# 3. MongoDB索引

## 3.1 概述

索引支持在MongoDB中高效地执行查询。如果没有索引，MongoDB必须执行全集合扫描，即扫描集合中的每个文档，以选择与查询语句匹配的文档。这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。

如果查询存在适当的索引，MongoDB可以使用该索引限制必须检查的文档数。

索引是特殊的数据结构，它以易于遍历的形式存储集合数据集的一小部分。索引存储特定字段或一组字段的值，按字段值排序。索引项的排序支持有效的相等匹配和基于范围的查询操作。此外，MongoDB还可以使用索引中的排序返回排序结果。

官网文档：https://docs.mongodb.com/manual/indexes/

MongoDB索引使用B树数据结构（确切的说是B-Tree，MySQL是B+Tree）

## 3.2 索引的类型

### 单字段索引

MongoDB支持在文档的单个字段上创建用户定义的升序/降序索引，称为单字段索引（Single Field Index）。

对于单个字段索引和排序操作，索引键的排序顺序（即升序或降序）并不重要，因为MongoDB可以在任何方向上遍历索引。

### 复合索引

MongoDB还支持多个字段的用户定义索引，即复合索引（Compound Index）。

复合索引中列出的字段顺序具有重要意义。例如，如果复合索引由 { userid: 1, score: -1 } 组成，则索引首先按userid正序排序，然后在每个userid的值内，再在按score倒序排序。

### 其他索引

地理空间索引（Geospatial Index）、文本索引（Text Indexes）、哈希索引（Hashed Indexes）。

地理空间索引（Geospatial Index）

为了支持对地理空间坐标数据的有效查询，MongoDB提供了两种特殊的索引：返回结果时使用平面几何的二维索引和返回结果时使用球面几何的二维球面索引。

文本索引（Text Indexes）

MongoDB提供了一种文本索引类型，支持在集合中搜索字符串内容。这些文本索引不存储特定于语言的停止词（例如“the”、“a”、“or”），而将集合中的词作为词干，只存储根词。

哈希索引（Hashed Indexes）

为了支持基于散列的分片，MongoDB提供了散列索引类型，它对字段值的散列进行索引。这些索引在其范围内的值分布更加随机，但只支持相等匹配，不支持基于范围的查询。

## 3.3 索引的管理操作

### 索引的查看

说明：

返回一个集合中的所有索引的数组。

语法：

```shell
db.collection.getIndexes()
```

【示例】

查看comment集合中所有的索引情况

```shell
db.comment.getIndexes()
[
	{
		"v" : 2,
		"key" : {
		"_id" : 1
	},
		"name" : "_id_",
		"ns" : "articledb.comment"
	}
]
```

结果中显示的是默认 _id 索引。

默认_id索引：

MongoDB在创建集合的过程中，在 _id 字段上创建一个唯一的索引，默认名字为 _id_ ，该索引可防止客户端插入两个具有相同值的文档，您不能在_id字段上删除此索引。

注意：该索引是唯一索引，因此值不能重复，即 _id 值不能重复的。在分片集群中，通常使用 _id 作为片键。

### 索引的创建

说明：

在集合上创建索引。

语法：

```shell
db.collection.createIndex(keys, options)
```

参数：

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| keys          | document | 包含字段和值对的文档，其中字段是索引键，值描述该字段的索引类型。对于字段上的升序索引，请指定值1；对于降序索引，请指定值-1。比如： {字段:1或-1} ，其中1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。另外，MongoDB支持几种不同的索引类型，包括文本、地理空间和哈希索引。 |
| options       | document | 可选。包含一组控制索引创建的选项的文档。有关详细信息，请参见选项详情列表。 |

options（更多选项）列表：

| Parameter          | Type          | Description                                                  |
| :----------------- | :------------ | :----------------------------------------------------------- |
| background         | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| unique             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| name               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups           | Boolean       | 在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse             | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 |
| v                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights            | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language   | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override  | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |

提示：

注意在 3.0.0 版本前创建索引方法为 db.collection.ensureIndex() ，之后的版本使用了 db.collection.createIndex() 方法，ensureIndex() 还能用，但只是 createIndex() 的别名。

【示例】

（1）单字段索引示例：对 userid 字段建立索引：

```shell
db.comment.createIndex({userid:1})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

参数1：按升序创建索引

可以查看一下：

```shell
db.comment.getIndexes()
[
	{
		"v" : 2,
		"key" : {
		"_id" : 1
	},
    "name" : "_id_",
    "ns" : "articledb.comment"
	},
	{
    "v" : 2,
    "key" : {
    "userid" : 1
  },
    "name" : "userid_1",
    "ns" : "articledb.comment"
  }
]
```

索引名字为 userid_1

（2）复合索引：对 userid 和 nickname 同时建立复合（Compound）索引：

```shell
db.comment.createIndex({userid:1,nickname:-1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 2,
  "numIndexesAfter" : 3,
  "ok" : 1
}
```

查看一下索引：

```shell
db.comment.getIndexes()
[
  {
    "v" : 2,
    "key" : {
    	"_id" : 1
  	},
    "name" : "_id_",
    "ns" : "articledb.comment"
	},
  {
    "v" : 2,
    "key" : {
    	"userid" : 1
  },
    "name" : "userid_1",
    "ns" : "articledb.comment"
  },
  {
    "v" : 2,
    "key" : {
    "userid" : 1,
    "nickname" : -1
  },
    "name" : "userid_1_nickname_-1",
    "ns" : "articledb.comment"
	}
]
```

### 索引的移除

说明：可以移除指定的索引，或移除所有索引

一、指定索引的移除

语法：

```shell
db.collection.dropIndex(index)
```

参数：

| Parameter | Type               | Description                                                  |
| --------- | ------------------ | ------------------------------------------------------------ |
| index     | string or document | 指定要删除的索引。可以通过索引名称或索引规范文档指定索引。若要删除文本索引，请指定索引名称。 |

【示例】

删除 comment 集合中 userid 字段上的升序索引：

```shell
db.comment.dropIndex({userid:1})
{ "nIndexesWas" : 3, "ok" : 1 }
```

查看已经删除了。

二、所有索引的移除

语法：

```shell
db.collection.dropIndexes()
```

【示例】

删除 spit 集合中所有索引。

```shell
db.comment.dropIndexes()
{
  "nIndexesWas" : 2,
  "msg" : "non-_id indexes dropped for collection",
  "ok" : 1
}
```

提示： _id 的字段的索引是无法删除的，只能删除非 _id 字段的索引。

## 3.4 索引的使用

### 执行计划

分析查询性能（Analyze Query Performance）通常使用执行计划（解释计划、Explain Plan）来查看查询的情况，如查询耗费的时间、是否基于索引查询等。

那么，通常，我们想知道，建立的索引是否有效，效果如何，都需要通过执行计划查看。

语法

```shell
db.collection.find(query,options).explain(options)
```

【示例】

查看根据userid查询数据的情况：

```shell
db.comment.find({userid:"1003"}).explain()
{
    "queryPlanner" : {
    "plannerVersion" : 1,
    "namespace" : "articledb.comment",
    "indexFilterSet" : false,
    "parsedQuery" : {
    "userid" : {
    "$eq" : "1003"
 	 }
 	 },
    "winningPlan" : {
    "stage" : "COLLSCAN",
    "filter" : {
    "userid" : {
    "$eq" : "1003"
	  }
 	 },
    "direction" : "forward"
 	 },
    "rejectedPlans" : [ ]
 	 },
    "serverInfo" : {
    "host" : "9ef3740277ad",
    "port" : 27017,
    "version" : "4.0.10",
    "gitVersion" : "c389e7f69f637f7a1ac3cc9fae843b635f20b766"
 	 },
    "ok" : 1
}
```

关键点看： "stage" : "COLLSCAN", 表示全集合扫描

下面对userid建立索引

```shell
db.comment.createIndex({userid:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

再次查看执行计划：

```shell
db.comment.find({userid:"1013"}).explain()
{
  "queryPlanner" : {
  "plannerVersion" : 1,
  "namespace" : "articledb.comment",
  "indexFilterSet" : false,
  "parsedQuery" : {
    "userid" : {
   	 "$eq" : "1013"
    }
  },
  "winningPlan" : {
    "stage" : "FETCH",
    "inputStage" : {
      "stage" : "IXSCAN",
      "keyPattern" : {
      	"userid" : 1
	  	},
      "indexName" : "userid_1",
      "isMultiKey" : false,
      "multiKeyPaths" : {
      	"userid" : [ ]
      },
      "isUnique" : false,
      "isSparse" : false,
      "isPartial" : false,
      "indexVersion" : 2,
      "direction" : "forward",
      "indexBounds" : {
        "userid" : [
        	"[\"1013\", \"1013\"]"
 				 ]
  			}
  		}
  },
  "rejectedPlans" : [ ]
  },
  "serverInfo" : {
    "host" : "9ef3740277ad",
    "port" : 27017,
    "version" : "4.0.10",
    "gitVersion" : "c389e7f69f637f7a1ac3cc9fae843b635f20b766"
  },
  "ok" : 1
}
```

关键点看： "stage" : "IXSCAN" ,基于索引的扫描

### 涵盖的查询

Covered Queries

当查询条件和查询的投影仅包含索引字段时，MongoDB直接从索引返回结果，而不扫描任何文档或将文档带入内存。 这些覆盖的查询可以非常有效。

https://www.mongodb.com/docs/manual/core/query-optimization/#read-operations-covered-query

