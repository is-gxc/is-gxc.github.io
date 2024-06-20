---
title: MongoDB集群
date: 2024-06-19 14:20:27
categories: MongoDB
tags: [MongoDB,NoSQL,数据库]
---

# 副本集

## 简介

MongoDB副本集（Replica Set）是一组维护相同数据集的mongod服务，旨在提供冗余和高可用性。它由多个mongod进程组成，这些进程在不同的机器上运行，共同维护相同的数据集。

副本集的工作原理类似于具有自动故障恢复功能的主从集群，通过多台机器进行同一数据的异步同步，确保多台机器拥有同一数据的多个副本。当主库发生故障时，副本集能够自动切换到其他备份服务器作为新的主库，无需用户干预，从而保证了数据的可用性和系统的持续性运行。此外，副本集还可以利用副本服务器作为只读服务器，实现读写分离，提高系统的负载能力。

（1）冗余和数据可用性

复制提供冗余并提高数据可用性。 通过在不同数据库服务器上提供多个数据副本，复制可提供一定级别的容错功能，以防止丢失单个数据库服务器。

在某些情况下，复制可以提供增加的读取性能，因为客户端可以将读取操作发送到不同的服务上， 在不同数据中心维护数据副本可以增加分布式应用程序的数据位置和可用性。 您还可以为专用目的维护其他副本，例如灾难恢复，报告或备份。

（2）MongoDB中的复制

副本集是一组维护相同数据集的mongod实例。 副本集包含多个数据承载节点和可选的一个仲裁节点。在承载数据的节点中，一个且仅一个成员被视为主节点，而其他节点被视为次要（从）节点。

主节点接收所有写操作。 副本集只能有一个主要能够确认具有{w：“most”}写入关注的写入; 虽然在某些情况下，另一个mongod实例可能暂时认为自己也是主要的。主要记录其操作日志中的数据集的所有更改，即oplog。

辅助(副本)节点复制主节点的oplog并将操作应用于其数据集，以使辅助节点的数据集反映主节点数据集。 如果主要人员不在，则符合条件的中将举行选举以选出新的主要人员。

（3）主从复制和副本集区别

主从集群和副本集最大的区别就是副本集没有固定的“主节点”；整个集群会选出一个“主节点”，当其挂掉后，又在剩下的从节点中选中其他节点为“主节点”，副本集总有一个活跃点(主、primary)和一个或多个备份节点(从、secondary)。

## 副本集的三个角色

副本集有两种类型三种角色

两种类型：

- 主节点（Primary）类型：数据操作的主要连接点，可读写。

- 次要（辅助、从）节点（Secondaries）类型：数据冗余备份节点，可以读或选举。

三种角色：

主要成员（Primary）：主要接收所有写操作。就是主节点。

副本成员（Replicate）：从主节点通过复制操作以维护相同的数据集，即备份数据，不可写操作，但可以读操作（但需要配置）。是默认的一种从节点类型。

仲裁者（Arbiter）：不保留任何数据的副本，只具有投票选举作用。当然也可以将仲裁服务器维护为副本集的一部分，即副本成员同时也可以是仲裁者。也是一种从节点类型。

```shell
+-----------------+
|    Client       |
+-----------------+
       |
       v
+-----------------+
|   Primary Node  |
+-----------------+
       |
       v
+------------------+        +------------------+
| Secondary Node 1 | <----> | Secondary Node 2 |
+------------------+        +------------------+
       ^
       |
+------------------+
|   Arbiter Node   |
+------------------+

```

关于仲裁者的额外说明：

您可以将额外的mongod实例添加到副本集作为仲裁者。 仲裁者不维护数据集。 仲裁者的目的是通过响应其他副本集成员的心跳和选举请求来维护副本集中的仲裁。 因为它们不存储数据集，所以仲裁器可以是提供副本集仲裁功能的好方法，其资源成本比具有数据集的全功能副本集成员更便宜。

如果您的副本集具有偶数个成员，请添加仲裁者以获得主要选举中的“大多数”投票。 仲裁者不需要专用硬件。

仲裁者将永远是仲裁者，而主要人员可能会退出并成为次要人员，而次要人员可能成为选举期间的主要人员。

如果你的副本+主节点的个数是偶数，建议加一个仲裁者，形成奇数，容易满足大多数的投票。

如果你的副本+主节点的个数是奇数，可以不加仲裁者。

## 副本集架构目标

一主一副本一仲裁

```shell
+-----------------+
|    Client       |
+-----------------+
       |
       v
+-----------------+
|   Primary Node  |
+-----------------+
       |
       v
+------------------+        +------------------+
| Secondary Node 1 | <----> | Secondary Node 2 |
+------------------+        +------------------+
       ^
       |
+------------------+
|   Arbiter Node   |
+------------------+


```

## 副本集的创建

### 创建主节点

简历存放数据和日志的目录

```shell
#-----------myrs
#主节点
mkdir -p /mongodb/replica_sets/myrs_27017/log \ &
mkdir -p /mongodb/replica_sets/myrs_27017/data/db
```

新建或修改配置文件：

```shell
vim /mongodb/replica_sets/myrs_27017/mongod.conf
```

myrs_27017：

```yaml
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/replica_sets/myrs_27017/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
  dbPath: "/mongodb/replica_sets/myrs_27017/data/db"
  journal:
  #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
  enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式。
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/replica_sets/myrs_27017/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.0.2
  #bindIp
  #绑定的端口
  port: 27017
replication:
  #副本集的名称
  replSetName: myrs
```

启动节点服务：

```shell
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/myrs_27017/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 54257
child process started successfully, parent exiting
```

### 创建副本节点

建立存放数据和日志的目录

```shell
#副本节点
mkdir -p /mongodb/replica_sets/myrs_27018/log \ &
mkdir -p /mongodb/replica_sets/myrs_27018/data/db
```

新建或修改配置文件：

```shell
vim /mongodb/replica_sets/myrs_27018/mongod.conf
```

myrs_27018：

```shell
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/replica_sets/myrs_27018/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
  dbPath: "/mongodb/replica_sets/myrs_27018/data/db"
  journal:
  #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
  enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式。
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/replica_sets/myrs_27018/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.0.2
  #bindIp
  #绑定的端口
  port: 27018
replication:
  #副本集的名称
  replSetName: myrs
```

启动节点服务：

```shell
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/myrs_27018/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 54361
child process started successfully, parent exiting
```

### 创建仲裁节点

建立存放数据和日志的目录

```shell
#-----------myrs
#仲裁节点
mkdir -p /mongodb/replica_sets/myrs_27019/log \ &
mkdir -p /mongodb/replica_sets/myrs_27019/data/db
```

仲裁节点：

新建或修改配置文件：

```shell
vim /mongodb/replica_sets/myrs_27019/mongod.conf
```

myrs_27019：

```shell
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/mongodb/replica_sets/myrs_27019/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
  dbPath: "/mongodb/replica_sets/myrs_27019/data/db"
  journal:
  #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
  enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式。
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/mongodb/replica_sets/myrs_27019/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost,192.168.0.2
  #bindIp
  #绑定的端口
  port: 27019
replication:
  #副本集的名称
  replSetName: myrs
```

启动节点服务：

```shell
 /usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/myrs_27019/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 54410
child process started successfully, parent exiting
```

### 初始化配置副本集和主节点

使用客户端命令连接任意一个节点，但这里尽量要连接主节点(27017节点)：

```shell
/usr/local/mongodb/bin/mongo --host=180.76.159.126 --port=27017
```

结果，连接上之后，很多命令无法使用，，比如 show dbs 等，必须初始化副本集才行

准备初始化新的副本集：

语法：

```shell
rs.initiate(configuration)
```

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| configuration | document | Optional. A document that specifies [configuration](https://www.mongodb.com/docs/manual/reference/replica-configuration/#replica-set-configuration-document) for the new replica set. If a configuration is not specified, MongoDB uses a default replica set configuration. |

【示例】

使用默认的配置来初始化副本集：

```shell
rs.initiate()
```

执行结果：

```shell
rs.initiate()
{
	"info2" : "no configuration specified. Using a default configuration for
the set",
  "me" : "180.76.159.126:27017",
  "ok" : 1,
  "operationTime" : Timestamp(1565760476, 1),
  "$clusterTime" : {
    "clusterTime" : Timestamp(1565760476, 1),
    "signature" : {
      "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
      "keyId" : NumberLong(0)
		}
	}
}
myrs:SECONDARY>
myrs:PRIMARY>
```

提示：

1）“ok”的值为1，说明创建成功。

2）命令行提示符发生变化，变成了一个从节点角色，此时默认不能读写。稍等片刻，回车，变成主节点。

### 查看副本集的配置内容

说明：

返回包含当前副本集配置的文档。

语法：

```shell
rs.conf(configuration)
```

提示：

rs.config() 是该方法的别名。

configuration：可选，如果没有配置，则使用默认主节点配置。

【示例】

在27017上执行副本集中当前节点的默认节点配置

```shell
myrs:PRIMARY> rs.conf()
{
  "_id" : "myrs",
  "version" : 1,
  "protocolVersion" : NumberLong(1),
  "writeConcernMajorityJournalDefault" : true,
  "members" : [
    {
    "_id" : 0,
    "host" : "180.76.159.126:27017",
    "arbiterOnly" : false,
    "buildIndexes" : true,
    "hidden" : false,
    "priority" : 1,
    "tags" : {
    },
    "slaveDelay" : NumberLong(0),
    "votes" : 1
 	 }
  ],
  "settings" : {
    "chainingAllowed" : true,
    "heartbeatIntervalMillis" : 2000,
    "heartbeatTimeoutSecs" : 10,
    "electionTimeoutMillis" : 10000,
    "catchUpTimeoutMillis" : -1,
    "catchUpTakeoverDelayMillis" : 30000,
    "getLastErrorModes" : {
	  },
  "getLastErrorDefaults" : {
    "w" : 1,
    "wtimeout" : 0
  },
  "replicaSetId" : ObjectId("5d539bdcd6a308e600d126bb")
  }
}
```

说明：

1） "_id" : "myrs" ：副本集的配置数据存储的主键值，默认就是副本集的名字

2） "members" ：副本集成员数组，此时只有一个： "host" : "180.76.159.126:27017" ，该成员不是仲裁节点： "arbiterOnly" : false ，优先级（权重值）： "priority" : 1,

3） "settings" ：副本集的参数配置。



提示：副本集配置的查看命令，本质是查询的是 system.replset 的表中的数据：

```shell
myrs:PRIMARY> use local
switched to db local
myrs:PRIMARY> show collections
oplog.rs
replset.election
replset.minvalid
replset.oplogTruncateAfterPoint
startup_log
system.replset
system.rollback.id
myrs:PRIMARY> db.system.replset.find()
{ "_id" : "myrs", "version" : 1, "protocolVersion" : NumberLong(1),
"writeConcernMajorityJournalDefault" : true, "members" : [ { "_id" : 0, "host" :
"180.76.159.126:27017", "arbiterOnly" : false, "buildIndexes" : true, "hidden" :
false, "priority" : 1, "tags" : { }, "slaveDelay" : NumberLong(0), "votes" : 1
} ], "settings" : { "chainingAllowed" : true, "heartbeatIntervalMillis" : 2000,
"heartbeatTimeoutSecs" : 10, "electionTimeoutMillis" : 10000,
"catchUpTimeoutMillis" : -1, "catchUpTakeoverDelayMillis" : 30000,
"getLastErrorModes" : { }, "getLastErrorDefaults" : { "w" : 1, "wtimeout" : 0
}, "replicaSetId" : ObjectId("5d539bdcd6a308e600d126bb") } }
myrs:PRIMARY>
```

### 查看副本集状态

检查副本集状态。

说明：

返回包含状态信息的文档。此输出使用从副本集的其他成员发送的心跳包中获得的数据反映副本集的当前状态。

语法：

```shell
rs.status()
```

【示例】

在27017上查看副本集状态：

```shell
myrs:PRIMARY> rs.status()
{  
  "set": "myrs",  
  "date": "ISODate(\"2019-08-14T05:29:45.161Z\")",  
  "myState": 1,  
  "term": "NumberLong(1)",  
  "syncingTo": "",  
  "syncSourceHost": "",  
  "syncSourceId": -1,  
  "heartbeatIntervalMillis": "NumberLong(2000)",  
  "optimes": {  
    "lastCommittedOpTime": {  
      "ts": "Timestamp(1565760578, 1)",  
      "t": "NumberLong(1)"  
    },  
    "readConcernMajorityOpTime": {  
      "ts": "Timestamp(1565760578, 1)",  
      "t": "NumberLong(1)"  
    },  
    "appliedOpTime": {  
      "ts": "Timestamp(1565760578, 1)",  
      "t": "NumberLong(1)"  
    },  
    "durableOpTime": {  
      "ts": "Timestamp(1565760578, 1)",  
      "t": "NumberLong(1)"  
    }  
  },  
  "lastStableCheckpointTimestamp": "Timestamp(1565760528, 1)",  
  "members": [  
    {  
      "_id": 0,  
      "name": "180.76.159.126:27017",  
      "health": 1,  
      "state": 1,  
      "stateStr": "PRIMARY",  
      "uptime": 419,  
      "optime": {  
        "ts": "Timestamp(1565760578, 1)",  
        "t": "NumberLong(1)"  
      },  
      "optimeDate": "ISODate(\"2019-08-14T05:29:38Z\")",  
      "syncingTo": "",  
      "syncSourceHost": "",  
      "syncSourceId": -1,  
      "infoMessage": "could not find member to sync from",  
      "electionTime": "Timestamp(1565760476, 2)",  
      "electionDate": "ISODate(\"2019-08-14T05:27:56Z\")",  
      "configVersion": 1,  
      "self": true,  
      "lastHeartbeatMessage": ""  
    }  
  ],  
  "ok": 1,  
  "operationTime": "Timestamp(1565760578, 1)",  
  "$clusterTime": {  
    "clusterTime": "Timestamp(1565760578, 1)",  
    "signature": {  
      "hash": "BinData(0,\"AAAAAAAAAAAAAAAAAAAAAAAAAAA=\")",  
      "keyId": "NumberLong(0)"  
    }  
  }  
}
```

说明：

1） "set" : "myrs" ：副本集的名字

2） "myState" : 1：说明状态正常

3） "members" ：副本集成员数组，此时只有一个： "name" : "180.76.159.126:27017" ，该成员的角色是 "stateStr" : "PRIMARY", 该节点是健康的： "health" : 1 

### 添加副本从节点

在主节点添加从节点，将其他成员加入到副本集

语法：

```shell
rs.add(host, arbiterOnly)
```

选项：

| **Parameter** | **Type**           | **Description**                                              |
| ------------- | ------------------ | ------------------------------------------------------------ |
| host          | string or document | 要添加到副本集的新成员。 指定为字符串或配置文档：1）如果是一个字符串，则需要指定新成员的主机名和可选的端口号；2）如果是一个文档，请指定在members数组中找到的副本集成员配置文档。 您必须在成员配置文档中指定主机字段。有关文档配置字段的说明，详见下方文档：“主机成员的配置文档” |
| arbiteronly   | boolean            | 可选的。 仅在 <host> 值为字符串时适用。 如果为true，则添加的主机是仲裁者。 |

主机成员的配置文档：

```shell
{
  _id: <int>,
  host: <string>, // required
  arbiterOnly: <boolean>,
  buildIndexes: <boolean>,
  hidden: <boolean>,
  priority: <number>,
  tags: <document>,
  slaveDelay: <int>,
  votes: <number>
}
```

【示例】

将27018的副本节点添加到副本集中：

```shell
myrs:PRIMARY> rs.add("180.76.159.126:27018")
{  
  "ok": 1,  
  "operationTime": "Timestamp(1565761757, 1)",  
  "$clusterTime": {  
    "clusterTime": "Timestamp(1565761757, 1)",  
    "signature": {  
      "hash": "BinData(0,\"AAAAAAAAAAAAAAAAAAAAAAAAAAA=\")",  
      "keyId": "NumberLong(0)"  
    }  
  }  
}
```

说明：

1） "ok" : 1 ：说明添加成功。

查看副本集状态：

```shell
myrs:PRIMARY> rs.status()
{
    "set" : "myrs",
    "date" : ISODate("2019-08-14T05:50:05.738Z"),
    "myState" : 1,
    "term" : NumberLong(1),
    "syncingTo" : "",
    "syncSourceHost" : "",
    "syncSourceId" : -1,
    "heartbeatIntervalMillis" : NumberLong(2000),
    "optimes" : {
        "lastCommittedOpTime" : {
            "ts" : Timestamp(1565761798, 1),
            "t" : NumberLong(1)
        },
        "readConcernMajorityOpTime" : {
            "ts" : Timestamp(1565761798, 1),
            "t" : NumberLong(1)
        },
        "appliedOpTime" : {
            "ts" : Timestamp(1565761798, 1),
            "t" : NumberLong(1)
        },
        "durableOpTime" : {
            "ts" : Timestamp(1565761798, 1),
            "t" : NumberLong(1)
        }
    },
    "lastStableCheckpointTimestamp" : Timestamp(1565761798, 1),
    "members" : [
        {
            "_id" : 0,
            "name" : "180.76.159.126:27017",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
            "uptime" : 1639,
            "optime" : {
                "ts" : Timestamp(1565761798, 1),
                "t" : NumberLong(1)
            },
            "optimeDate" : ISODate("2019-08-14T05:49:58Z"),
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "electionTime" : Timestamp(1565760476, 2),
            "electionDate" : ISODate("2019-08-14T05:27:56Z"),
            "configVersion" : 2,
            "self" : true,
            "lastHeartbeatMessage" : ""
        },
        {
            "_id" : 1,
            "name" : "180.76.159.126:27018",
            "health" : 1,
            "state" : 2,
            "stateStr" : "SECONDARY",
            "uptime" : 48,
            "optime" : {
                "ts" : Timestamp(1565761798, 1),
                "t" : NumberLong(1)
            },
            "optimeDurable" : {
                "ts" : Timestamp(1565761798, 1),
                "t" : NumberLong(1)
            },
            "optimeDate" : ISODate("2019-08-14T05:49:58Z"),
            "optimeDurableDate" : ISODate("2019-08-14T05:49:58Z"),
            "lastHeartbeat" : ISODate("2019-08-14T05:50:05.294Z"),
            "lastHeartbeatRecv" : ISODate("2019-08-14T05:50:05.476Z"),
            "pingMs" : NumberLong(0),
            "lastHeartbeatMessage" : "",
            "syncingTo" : "180.76.159.126:27017",
            "syncSourceHost" : "180.76.159.126:27017",
            "syncSourceId" : 0,
            "infoMessage" : "",
            "configVersion" : 2
        }
    ],
    "ok" : 1,
    "operationTime" : Timestamp(1565761798, 1),
    "$clusterTime" : {
        "clusterTime" : Timestamp(1565761798, 1),
        "signature" : {
            "hash" : BinData(0, "AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    }
}

```

说明：

1） "name" : "180.76.159.126:27018" 是第二个节点的名字，其角色是 "stateStr" : "SECONDARY"

### 添加仲裁节点

添加一个仲裁节点到副本集

语法：

```shell
rs.addArb(host)
```

将27019的仲裁节点添加到副本集中：

```shell
myrs:PRIMARY> rs.addArb("180.76.159.126:27019")
{
    "ok" : 1,
    "operationTime" : Timestamp(1565761959, 1),
    "$clusterTime" : {
        "clusterTime" : Timestamp(1565761959, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    }
}

```

说明：

1） "ok" : 1 ：说明添加成功。

查看副本集状态：

```shell
myrs:PRIMARY> rs.status()
{
    "set" : "myrs",
    "date" : ISODate("2019-08-14T05:53:27.198Z"),
    "myState" : 1,
    "term" : NumberLong(1),
    "syncingTo" : "",
    "syncSourceHost" : "",
    "syncSourceId" : -1,
    "heartbeatIntervalMillis" : NumberLong(2000),
    "optimes" : {
        "lastCommittedOpTime" : {
            "ts" : Timestamp(1565761998, 1),
            "t" : NumberLong(1)
        },
        "readConcernMajorityOpTime" : {
            "ts" : Timestamp(1565761998, 1),
            "t" : NumberLong(1)
        },
        "appliedOpTime" : {
            "ts" : Timestamp(1565761998, 1),
            "t" : NumberLong(1)
        },
        "durableOpTime" : {
            "ts" : Timestamp(1565761998, 1),
            "t" : NumberLong(1)
        }
    },
    "lastStableCheckpointTimestamp" : Timestamp(1565761978, 1),
    "members" : [
        {
            "_id" : 0,
            "name" : "180.76.159.126:27017",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
            "uptime" : 1841,
            "optime" : {
                "ts" : Timestamp(1565761998, 1),
                "t" : NumberLong(1)
            },
            "optimeDate" : ISODate("2019-08-14T05:53:18Z"),
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "electionTime" : Timestamp(1565760476, 2),
            "electionDate" : ISODate("2019-08-14T05:27:56Z"),
            "configVersion" : 3,
            "self" : true,
            "lastHeartbeatMessage" : ""
        },
        {
            "_id" : 1,
            "name" : "180.76.159.126:27018",
            "health" : 1,
            "state" : 2,
            "stateStr" : "SECONDARY",
            "uptime" : 249,
            "optime" : {
                "ts" : Timestamp(1565761998, 1),
                "t" : NumberLong(1)
            },
            "optimeDurable" : {
                "ts" : Timestamp(1565761998, 1),
                "t" : NumberLong(1)
            },
            "optimeDate" : ISODate("2019-08-14T05:53:18Z"),
            "optimeDurableDate" : ISODate("2019-08-14T05:53:18Z"),
            "lastHeartbeat" : ISODate("2019-08-14T05:53:25.668Z"),
            "lastHeartbeatRecv" : ISODate("2019-08-14T05:53:26.702Z"),
            "pingMs" : NumberLong(0),
            "lastHeartbeatMessage" : "",
            "syncingTo" : "180.76.159.126:27017",
            "syncSourceHost" : "180.76.159.126:27017",
            "syncSourceId" : 0,
            "infoMessage" : "",
            "configVersion" : 3
        },
        {
            "_id" : 2,
            "name" : "180.76.159.126:27019",
            "health" : 1,
            "state" : 7,
            "stateStr" : "ARBITER",
            "uptime" : 47,
            "lastHeartbeat" : ISODate("2019-08-14T05:53:25.668Z"),
            "lastHeartbeatRecv" : ISODate("2019-08-14T05:53:25.685Z"),
            "pingMs" : NumberLong(0),
            "lastHeartbeatMessage" : "",
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "infoMessage" : "",
            "configVersion" : 3
        }
    ],
    "ok" : 1,
    "operationTime" : Timestamp(1565761998, 1),
    "$clusterTime" : {
        "clusterTime" : Timestamp(1565761998, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    }
}

```

说明：

1） "name" : "180.76.159.126:27019" 是第二个节点的名字，其角色是 "stateStr" : "ARBITER"

## 副本集的读写操作

目标：测试三个不同角色的节点的数据读写情况。

登录主节点27017，写入和读取数据：

```shell
[root@bobohost ~]# /usr/local/mongodb/bin/mongo --host 180.76.159.126 --port
27017
myrs:PRIMARY> use articledb
switched to db articledb
myrs:PRIMARY> db
articledb
myrs:PRIMARY> db.comment.insert({"articleid":"100000","content":"今天天气真好，阳光
明媚","userid":"1001","nickname":"Rose","createdatetime":new Date()})
WriteResult({ "nInserted" : 1 })
myrs:PRIMARY> db.comment.find()
{ "_id" : ObjectId("5d4d2ae3068138b4570f53bf"), "articleid" : "100000",
"content" : "今天天气真好，阳光明媚", "userid" : "1001", "nickname" : "Rose",
"createdatetime" : ISODate("2019-08-09T08:12:19.427Z") }
```

登录从节点27018

```shell
[root@bobohost ~]# /usr/local/mongodb/bin/mongo --host 180.76.159.126 --port
27018
myrs:SECONDARY> show dbs;
2019-09-10T10:56:51.953+0800 E QUERY [js] Error: listDatabases failed:{
  "operationTime" : Timestamp(1568084204, 1),
  "ok" : 0,
  "errmsg" : "not master and slaveOk=false",
  "code" : 13435,
  "codeName" : "NotMasterNoSlaveOk",
  "$clusterTime" : {
    "clusterTime" : Timestamp(1568084204, 1),
    "signature" : {
      "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
      "keyId" : NumberLong(0)
    }
  }
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:139:1
shellHelper.show@src/mongo/shell/utils.js:882:13
shellHelper@src/mongo/shell/utils.js:766:15
@(shellhelp2):1:1
```

发现，不能读取集合的数据。当前从节点只是一个备份，不是奴隶节点，无法读取数据，写当然更不行。

因为默认情况下，从节点是没有读写权限的，可以增加读的权限，但需要进行设置。

设置读操作权限：

说明：

设置为奴隶节点，允许在从成员上运行读的操作

语法：

```shell
rs.slaveOk()
#或
rs.slaveOk(true)
```

提示：

该命令是 db.getMongo().setSlaveOk() 的简化命令。

【示例】

在27018上设置作为奴隶节点权限，具备读权限：

```shell
rs:SECONDARY> rs.slaveOk()
```

此时，在执行查询命令，运行成功！

但仍然不允许插入。

```shell
myrs:SECONDARY> rs.slaveOk()
myrs:SECONDARY> show dbs;
admin 0.000GB
articledb 0.000GB
config 0.000GB
local 0.000GB
myrs:SECONDARY> use articledb
switched to db articledb
myrs:SECONDARY> show collections
comment
myrs:SECONDARY> db.comment.find()
{ "_id" : ObjectId("5d7710c04cfd7eee2e3cdabe"), "articleid" : "100000",
"content" : "今天天气真好，阳光明媚", "userid" : "1001", "nickname" : "Rose",
"createdatetime" : ISODate("2019-09-10T02:56:00.467Z") }
myrs:SECONDARY> db.comment.insert({"_id":"1","articleid":"100001","content":"我们
不应该把清晨浪费在手机上，健康很重要，k一杯温水幸福你我
他。","userid":"1002","nickname":"相忘于江湖","createdatetime":new Date("2019-08-
05T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"})
WriteCommandError({
  "operationTime" : Timestamp(1568084434, 1),
  "ok" : 0,
  "errmsg" : "not master",
  "code" : 10107,
  "codeName" : "NotMaster",
  "$clusterTime" : {
  "clusterTime" : Timestamp(1568084434, 1),
    "signature" : {
      "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
      "keyId" : NumberLong(0)
    }
  }
})
```

现在可实现了读写分离，让主插入数据，让从来读取数据。

如果要取消作为奴隶节点的读权限：

```shell
myrs:SECONDARY> rs.slaveOk(false)
myrs:SECONDARY> db.comment.find()
Error: error: {
  "operationTime" : Timestamp(1568084459, 1),
  "ok" : 0,
  "errmsg" : "not master and slaveOk=false",
  "code" : 13435,
  "codeName" : "NotMasterNoSlaveOk",
  "$clusterTime" : {
    "clusterTime" : Timestamp(1568084459, 1),
    "signature" : {
      "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
      "keyId" : NumberLong(0)
    }
  }
}
```

仲裁者节点，不存放任何业务数据的，可以登录查看

```shell
[root@bobohost ~]# /usr/local/mongodb/bin/mongo --host 180.76.159.126 --port
27019
myrs:ARBITER> rs.slaveOk()
myrs:ARBITER> show dbs
local 0.000GB
myrs:ARBITER> use local
switched to db local
myrs:ARBITER> show collections
replset.minvalid
replset.oplogTruncateAfterPoint
startup_log
system.replset
system.rollback.id
myrs:ARBITER>
```

发现，只存放副本集配置等数据。

## 主节点的选举原则

MongoDB在副本集中，会自动进行主节点的选举，主节点选举的触发条件：

1） 主节点故障

2） 主节点网络不可达（默认心跳信息为10秒）

3） 人工干预（rs.stepDown(600)）

一旦触发选举，就要根据一定规则来选主节点。

选举规则是根据票数来决定谁获胜：

- 票数最高，且获得了“大多数”成员的投票支持的节点获胜。“大多数”的定义为：假设复制集内投票成员数量为N，则大多数为 N/2 + 1。例如：3个投票成员，则大多数的值是2。当复制集内存活成员数量不足大多数时，整个复制集将无法选举出Primary，复制集将无法提供写服务，处于只读状态。
- 若票数相同，且都获得了“大多数”成员的投票支持的，数据新的节点获胜。数据的新旧是通过操作日志oplog来对比的。

在获得票数的时候，优先级（priority）参数影响重大。

可以通过设置优先级（priority）来设置额外票数。优先级即权重，取值为0-1000，相当于可额外增加0-1000的票数，优先级的值越大，就越可能获得多数成员的投票（votes）数。指定较高的值可使成员更有资格成为主要成员，更低的值可使成员更不符合条件。

默认情况下，优先级的值是1

```shell
myrs:PRIMARY> rs.conf()
{
    "_id" : "myrs",
    "version" : 3,
    "protocolVersion" : NumberLong(1),
    "writeConcernMajorityJournalDefault" : true,
    "members" : [
        {
            "_id" : 0,
            "host" : "180.76.159.126:27017",
            "arbiterOnly" : false,
            "buildIndexes" : true,
            "hidden" : false,
            "priority" : 1,
            "tags" : {},
            "slaveDelay" : NumberLong(0),
            "votes" : 1
        },
        {
            "_id" : 1,
            "host" : "180.76.159.126:27018",
            "arbiterOnly" : false,
            "buildIndexes" : true,
            "hidden" : false,
            "priority" : 1,
            "tags" : {},
            "slaveDelay" : NumberLong(0),
            "votes" : 1
        },
        {
            "_id" : 2,
            "host" : "180.76.159.126:27019",
            "arbiterOnly" : true,
            "buildIndexes" : true,
            "hidden" : false,
            "priority" : 0,
            "tags" : {},
            "slaveDelay" : NumberLong(0),
            "votes" : 1
        }
    ],
    "settings" : {
        "chainingAllowed" : true,
        "heartbeatIntervalMillis" : 2000,
        "heartbeatTimeoutSecs" : 10,
        "electionTimeoutMillis" : 10000,
        "catchUpTimeoutMillis" : -1,
        "catchUpTakeoverDelayMillis" : 30000,
        "getLastErrorModes" : {},
        "getLastErrorDefaults" : {
            "w" : 1,
            "wtimeout" : 0
        },
        "replicaSetId" : ObjectId("5d539bdcd6a308e600d126bb")
    }
}

```

可以看出，主节点和副本节点的优先级各为1，即，默认可以认为都已经有了一票。但选举节点，优先级是0，（要注意是，官方说了，选举节点的优先级必须是0，不能是别的值。即不具备选举权，但具有投票权）

【了解】修改优先级

比如，下面提升从节点的优先级：

1）先将配置导入cfg变量

```shell
myrs:SECONDARY> cfg=rs.conf()
```

2）然后修改值（ID号默认从0开始）：

```shell
myrs:SECONDARY> cfg.members[1].priority=2
2
```

3）重新加载配置

```shell
myrs:SECONDARY> rs.reconfig(cfg)
{ "ok" : 1 }
```

稍等片刻会重新开始选举。

## 故障测试

### 副本节点故障测试

关闭27018副本节点：

发现，主节点和仲裁节点对27018的心跳失败。因为主节点还在，因此，没有触发投票选举。

如果此时，在主节点写入数据。

```shell
db.comment.insert({"_id":"1","articleid":"100001","content":"我们不应该把清晨浪费在
手机上，健康很重要，一杯温水幸福你我他。","userid":"1002","nickname":"相忘于江
湖","createdatetime":new Date("2019-08-
05T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"})
```

再启动从节点，会发现，主节点写入的数据，会自动同步给从节点。

### 主节点故障测试

关闭27017节点

发现，从节点和仲裁节点对27017的心跳失败，当失败超过10秒，此时因为没有主节点了，会自动发起投票。

而副本节点只有27018，因此，候选人只有一个就是27018，开始投票。

27019向27018投了一票，27018本身自带一票，因此共两票，超过了“大多数”

27019是仲裁节点，没有选举权，27018不向其投票，其票数是0.

最终结果，27018成为主节点。具备读写功能。

在27018写入数据查看。

```shell
db.comment.insert({"_id":"2","articleid":"100001","content":"我夏天空腹喝凉开水，冬
天喝温开水","userid":"1005","nickname":"伊人憔悴","createdatetime":new Date("2019-
08-05T23:58:51.485Z"),"likenum":NumberInt(888),"state":"1"})
```

再启动27017节点，发现27017变成了从节点，27018仍保持主节点。

登录27017节点，发现是从节点了，数据自动从27018同步。

从而实现了高可用。

### 仲裁节点和主节点故障

先关掉仲裁节点27019，

关掉现在的主节点27018

登录27017后，发现，27017仍然是从节点，副本集中没有主节点了，导致此时，副本集是只读状态，无法写入。

为啥不选举了？因为27017的票数，没有获得大多数，即没有大于等于2，它只有默认的一票（优先级是1）

如果要触发选举，随便加入一个成员即可。

- 如果只加入27019仲裁节点成员，则主节点一定是27017，因为没得选了，仲裁节点不参与选举，但参与投票。（不演示）
- 如果只加入27018节点，会发起选举。因为27017和27018都是两票，则按照谁数据新，谁当主节点。

### 仲裁节点和从节点故障

先关掉仲裁节点27019，

关掉现在的副本节点27018

10秒后，27017主节点自动降级为副本节点。（服务降级）

副本集不可写数据了，已经故障了。

## 连接副本集

MongoDB客户端连接语法：

```shell
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]]
[/[database][?options]]
```

- **mongodb://** 这是固定的格式，必须要指定。

- **username:password@** 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库

- **host1** 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。

- **portX** 可选的指定端口，如果不填，默认为27017

- **/database** 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开test 数据库。

- **?options** 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开

标准的连接格式包含了多个选项(options)，如下所示：

| 选项                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| replicaSet=name     | 验证replica set的名称。 Impliesconnect=replicaSet.           |
| slaveOk=true\|false | true:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。false: 在connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。 |
| safe=true\|false    | true: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS).false: 在每次更新之后，驱动不会发送getLastError来确保更新成功 |
| w=n                 | 驱动添加 { w : n } 到getLastError命令. 应用于safe=true。     |
| wtimeoutMS=ms       | 驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true. |
| fsync=true\|false   | true: 驱动添加 { fsync : true } 到 getlasterror 命令.应用于safe=true.false: 驱动不会添加到getLastError命令中。 |
| journal=true\|false | 如果设置为 true, 同步到 journal (在提交到数据库前写入到实体中).应用于 safe=true |
| connectTimeoutMS=ms | 可以打开连接的时间。                                         |
| socketTimeoutMS=ms  | 发送和接受sockets的时间。                                    |



# 分片

## 概念

分片（sharding）是一种跨多台机器分布数据的方法， MongoDB使用分片来支持具有非常大的数据集

和高吞吐量操作的部署。

换句话说：分片(sharding)是指将数据拆分，将其分散存在不同的机器上的过程。有时也用分区(partitioning)来表示这个概念。将数据分散到不同的机器上，不需要功能强大的大型计算机就可以储存更多的数据，处理更多的负载。

具有大型数据集或高吞吐量应用程序的数据库系统可以会挑战单个服务器的容量。例如，高查询率会耗尽服务器的CPU容量。工作集大小大于系统的RAM会强调磁盘驱动器的I / O容量。

有两种解决系统增长的方法：垂直扩展和水平扩展。

垂直扩展意味着增加单个服务器的容量，例如使用更强大的CPU，添加更多RAM或增加存储空间量。可用技术的局限性可能会限制单个机器对于给定工作负载而言足够强大。此外，基于云的提供商基于可用的硬件配置具有硬性上限。结果，垂直缩放有实际的最大值。水平扩展意味着划分系统数据集并加载多个服务器，添加其他服务器以根据需要增加容量。虽然单个机器的总体速度或容量可能不高，但每台机器处理整个工作负载的子集，可能提供比单个高速大容量服务器更高的效率。扩展部署容量只需要根据需要添加额外的服务器，这可能比单个机器的高端硬件的总体成本更低。权衡是基础架构和部署维护的复杂性增加。

MongoDB支持通过分片进行水平扩展。

## 分片集群包含的组件

MongoDB分片群集包含以下组件：

- 分片（存储）：每个分片包含分片数据的子集。 每个分片都可以部署为副本集。

- mongos（路由）：mongos充当查询路由器，在客户端应用程序和分片集群之间提供接口。

- config servers（“调度”的配置）：配置服务器存储群集的元数据和配置设置。 从MongoDB 3.4开始，必须将配置服务器部署为副本集（CSRS）。

下图描述了分片集群中组件的交互：

![分片](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/%E5%88%86%E7%89%87.png)

MongoDB在集合级别对数据进行分片，将集合数据分布在集群中的分片上。

## 分片集群架构目标

两个分片节点副本集（3+3）+一个配置节点副本集（3）+两个路由节点（2），共11个服务节点。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/%E5%88%86%E7%89%87%E9%9B%86%E7%BE%A4%E6%9E%B6%E6%9E%84%E7%9B%AE%E6%A0%87.png)

