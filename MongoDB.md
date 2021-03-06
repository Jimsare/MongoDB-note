## mongodb连接标准格式
> mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

### 参数说明  
|         |                                          |
| :------ | :--------------------------------------- |
| Mongodb | 必填的前缀，标识当前字符串为便准链接格式 |
|username:password@|可选项，给出用户名和密码后，在连接数据库服务器后，驱动都会尝试登陆这个数据库|
|host|uri里唯一的必填项，数据库的连接地址,人如果需要连接副本集，需要制定多个主机地址|
|:port|可选项，如果不填则默认为27017端口|
|database|希望连接到的数据库名称，只有在设置username:password@后才会有效，如果不指定，则默认为admin数据库|
|?options|可选项，如果不适用/database,则需要在前面加上/。所有连接选项都是键值对name=value格式，键值对之间使用&或;（分号）分割|

#### opthons 参数说明  
|||
|:----|:----|
|connect=direct&#124;replicaset|**direct**: 直接建立一个到服务器的连接。如果指定了多个host，将按先后顺序挨个尝试建立连接，直到连接建立成功为止。如果只指定了一个host，则 direct 为默认值。<br>***replicaset***: 使用creplica set semantics建立连接（即使只提供了一个host）。指定的host作为种子列表来查找完整的replica set。当指定多个host时 replicaset 为默认值。|
|replicaset=name| 驱动验证建立连接的replica set的名字。应用于 connect=replicaset。|
|slaveok=true&#124;false | **true**: 对于 connect=direct 模式，驱动对列表中的第一个服务器建立连接，即使它不是主服务器。对 connect=replicaset 模式，驱动将所有写操作发送到主节点，将所有读操作按round robin顺序分发到从节点。<br>**false**: 对 connect=direct 模式，驱动按顺序尝试所有host直到找到主节点。对 connect=replicaset 模式，驱动将只连接到主节点，并将所有读操作和写操作都发送到主节点。|
|safe=true&#124;false|**true**: 驱动在每次更新操作后都发送 getlasterror 命令以确保更新成功（参考 w 和 wtimeout）。<br>**false**: 驱动每次更新操作后不发送 getlasterror 命令。
|w=n|w：代表server的数量<br>w=-1 不等待，不做异常检查<br>w=0 不等待，只返回网络错误<br>w=1 检查本机，并检查网络错误<br>w>1 检查w个server，并返回网络错,应用于safe=true|
|wtimeoutMS=ms|写操作超时的时间，应用于 safe=true.|
|fsync=true&#124;false|是不是等待刷新数据到磁盘，应用于safe=true|
|journal=true&#124;false|是不是等待提交的数据已经写入到日志，并刷新到磁盘，应用于safe=true|
|maxPoolSize=n<br>minPoolSize=n|一些驱动会把没用的连接关闭。 然而,如果连接数低于minPoolSize值之下， 它们不会关闭空闲的连接。注意：连接会按照需要进行创建，因此当连接池被许多连接预填充的时候，minPoolSize不会生效。|
|waitQueueTimeoutMS=ms|在超时之前，线程等待连接生效的总时间。如果连接池到达最大并且所有的连接都在使用，这个参数就生效了。|
|waitQueueMultiple=n|驱动强行限制线程同时等待连接的个数。 这个限制了连接池的倍数。|
|connectTimeoutMS=ms|可以打开连接的时间。|
|socketTimeoutMS=ms|发送和接受sockets的时间|
|ReadPreference|**primary**:主节点，默认模式，读操作只在主节点，如果主节点不可用，报错或者抛出异常。<br>**primaryPreferred**:首选主节点，大多情况下读操作在主节点，如果主节点不可用，如故障转移，读操作在从节点。<br>**secondary**:从节点，读操作只在从节点， 如果从节点不可用，报错或者抛出异常。<br>***secondaryPreferred**首选从节点，大多情况下读操作在从节点，特殊情况（如单主节点架构）读操作在主节点。<br>**nearest**最邻近节点，读操作在最邻近的成员，可能是主节点或者从节点

##### 连接示例
连接本地数据库服务器，端口默认
```monodb://localhost```

使用用户名abs 密码firelock登录localhost的***admin***数据库
```mogodb://abs:firelock@localhost```

使用用户名abs 密码firelock登录localhost的***lol***数据库
```mogodb://abs:firelock@localhost/lol```

连接 replica pair, 服务器1为example1.com服务器2为example2。
```mongodb://example1.com:27017,example2.com:27017```

连接 replica set 三台服务器 (端口 27017, 27018, 和27019):
```mongodb://localhost,localhost:27018,localhost:27019```

连接 replica set 三台服务器, 写入操作应用在主服务器 并且分布查询到从服务器。
```mongodb://host1,host2,host3/?slaveOk=true```

直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器
```mongodb://host1,host2,host3/?connect=direct;slaveOk=true```

#### 当你的连接服务器有优先级，还需要列出所有服务器，你可以使用上述连接方式。

安全模式连接到localhost:
```mongodb://localhost/?safe=true```

以安全模式连接到replica set，并且等待至少两个复制服务器成功写入，超时时间设置为2秒。
```mongodb://host1,host2,host3/?safe=true;w=2;wtimeoutMS=2000```

## MongoDB .config配置参数详解
>  mongodb 3.0之后配置文件采用YAML格式，这种格式非常简单，使用```<key>:<value>```表示，开头使用"空格"作为缩进。需要注意的是，":"之后有value的话，需要紧跟一个空格，如果key只是表示层级，则无需在":"后增加空格（比如：systemLog:后面既不需要空格）。按照层级，每行4个空格缩进，第二级则8个空格，依次轮推，顶层则不需要空格缩进
1. systemLog
```
systemLog.verbosity

integer

日志文件输出的级别，越大级别越低。

systemLog.quite

boolean

在quite模式下会限制输出信息：

数据库命令输出，副本集活动，连接接受事件，连接关闭事件。

systemLog.traceAllExceptions

string

打印verbose信息来调试，用来记录证额外的异常日志。

systemLog.syslogFacility

string，默认为user

指定syslog日志信息的设备级别。需要指定--syslog来使用这个选项。

systemLog.path string

发送所有的诊断信息日志，默认重启后会覆盖。

systemLog.logAppend

boolean

是否启用追加日志。

systemLog.logRotate

string

V3.0.0版本中新特性，默认值为rename

使用rename，mongod或mongos通过在文件名称末尾添加UTC(GMT)时间戳的方式重命名当前的日志文件，然后打开新的日志文件，关闭之前旧的日志文件，并发送所有新的日志信息到新的日志文件中。

reopen 关闭并重新打开日志文件遵循典型的Linux/Unix日志切换行为。当使用Linux/Unix logrotate工具时，使用reopen避免日志丢失。

如果指定reopen时，也必须同时使用—logappend

 

systemLog.destination

string

指定一个文件或syslog。如果指定为文件，必须同时指定systemLog.path

systemLog.timeStampFormat

string，默认为iso8601-local

日志信息中的时间戳格式：

ctime,iso8601-utc,iso8601-local
```

2. processManagement
```
processManagement.pidFilePath

string

指定进程的ID，与--fork配合使用，不指定则不会创建。

processManagement.fork

boolean，默认为false

是守护进程在后台运行
```
3. net
```
net.port

interger，默认为27017

mongodb实例监听的端口号。

net.bindIp

string,2.6版本默认为127.0.0.1

指定mongodb实例绑定的ip，为了绑定多个ip，可以使用逗号分隔。

net.maxIncomingConnections

integer 默认为1000000

mongodb实例接受的最多连接数，如果高于操作系统接受的最大线程数，设置无效。

net.wireObjectCheck

boolean，默认为true

检查文档的有效性。会稍微影响性能。

net.http.enabled

boolean,默认为false

打开http端口，会导致更多的不安全因素。

net.unixDomainSocket.enabled

boolean,默认为false

停止UNIX domain socket监听。

mongodb实例会一直监听UNIX

socket,除非net.unixDomainSocket.enabled设置为true,bindIp没有设置，bindIp没有默认指定为127.0.0.1。

net.unixDomainSocket.pathPrefix

string，默认为/tmp

unix Socket所在的路径。

net.ipv6

boolean，默认为false

打开IPV6功能，默认为关闭的。

net.http.JSONPEnabled

boolean，默认为false

运行json访问http端口，打开会导致更多的不安全因素。

net.http.RESTInterfaceEnabled

boolean，默认为false

即使http接口选项关闭，打开也会暴露http接口，会导致更多的不安全因素。
```
4. security
```
security.keyFile

string

指定分片集或副本集成员之间身份验证的key文件存储位置。

security.clusterAuthMode

string

集群认证中利用到这个模式，如果使用x.509安全机制，可以在这里指定。

keyFile,sendKeyFile,sendX509,x509

默认的mongodb发行版是不支持ssl的，可以使用专业版的或重新自行编译mongodb。

security.authorization

string,默认为disabled

打开访问数据库和进行操作的用户角色认证。

enabled,disabled
```
5. operationProfiling
```operationProfiling.slowOpThresholdMs

integer,默认100

指定慢查询时间，单位毫秒，如果打开功能，则向system.profile集合写入数据。

operationProfiling.mode

integer,默认0

改变分析日志输出级别。

0，1，2,分别对应关闭，仅打开慢查询，记录所有操作。
```
6. storage
```
storage.dbPath

string

指定数据文件的路径。

storage.directoryPerDB

boolean,默认关闭

指定存储每个数据库文件到单独的数据目录。如果在一个已存在的系统使用该选项，需要事先把存在的数据文件移动到目录。

storage.indexBuildRetry

boolean,默认为true

指定数据库在索引建立过程中停止，重启后是否重新建立索引。

storage.preallocDataFiles

boolean,默认true

是否预先分片好数据文件。

storage.nsSize

integer,默认16

指定命名空间的大小，即.ns后缀的文件。最大为2047MB,16M文件可以提供大约24000个命名空间。

storage.quota.enforced

boolean,默认false

限制每个数据库的数据文件数目。可以通过maxFilesPerDB调整数目。

storage.quota.maxFilesPerDB

integer,默认为8

限制每个数据库的数据文件数目。

storage.smallFiles

boolean,默认为false

限制mongodb数据文件大小为512MB，减小journal文件从1G到128M,适用于有很多数量小的数据文件。

storage.syncPeriodSecs

number,默认60

mongodb文件刷新频率，尽量不要在生产环境下修改。

storage.repairPath

string，默认为指定dbpath下的_tmp目录。

指定包含数据文件的根目录，进行--repair操作。

storage.journal.enabled

boolean,默认64bit为true，32bit为false

记录操作日志，防止数据丢失。

storage.journal.debugFlags

integer

提供数据库在非正常关闭下的功能测试。

storage.journal.commitIntervalMs

number，默认为100或30

journal操作的最大间隔时间。可以是2-300ms之间的值，低的值有助于持久化，但是会增加磁盘的额外负担。

如果journal和数据文件在同一磁盘上，默认为100ms。如果在不同的磁盘上为30ms。

如果强制mongod提交日志文件，可以指定j:true，指定后，时间变为原来的三分之一。
```
7. replication
```
replication.oplogSizeMB

integer,默认为磁盘的5%

指定oplog的最大尺寸。对于已经建立过oplog.rs的数据库，指定无效。

replication.replSetName

string

指定副本集的名称。

replication.secondaryIndexPrefetch

string，默认为all

指定副本集成员在接受oplog之前是否加载索引到内存。默认会加载所有的索引到内存。

none，不加载;all，加载所有;_id_only，仅加载_id。

```
8. shading
```
sharding.clusterRole

string

指定分片集的mongodb角色。

configsvr,配置服务器，端口27019;shardsvr,分片实例，端口27018。

sharding.archiveMovedChunks

integer

在块移动过程中，该选项强制mongodb实例保存所有移动的文档到moveChunk目录。
```
9.auditLog
```
auditLog.destination

string

syslog,以json格式保存身份验证到syslog，windows下不可用，serverity级别为info，facility级别为user。

console,以json格式输出信息到标准输出。

file,以json格式输出信息到文件。

auditLog.format

string

指定输出文件的格式

JSON,输出json格式文件;BSON,输出bson二进制格式文件。

auditLog.path

string

如果--auditDestination的值为file，则该选项指定文件路径。

auditLog.filter

document

指定过滤系统身份验证的格式为:

{ atype : <expression> }

{ atype: <expression>, "param.db": <database> }
```
10. snmp
```snmp.subagent

boolean

运行SNMP为一个子代理。

snmp.master

boolean

运行SNMP为一个主进程。

 

仅mongos选项

replication.localPingThresholdMs

integer，默认15

当客户端选定副本集进行读操作时受影响。

sharding.autoSplit

boolean

防止mongos自动在一个分片集合中插入元数据。

因为任何的mongos都可以创建一个分离，如果打开该选项，将会导致分片不平衡，需要谨慎使用。

sharding.configDB

string

指定配置数据库。可以使用逗号分隔一到三个服务器。

如果处于不同的位置，需要指定最近的一个。

不能移除配置服务器，即使不可用或者离线了。

sharding.chunkSize

integer,默认为64

每个块的大小。64MB是理想大小，小的会导致不能在不同节点间高效移动。

仅仅在初始化时有效。
```
> 参考示例
```
wireObjectCheck: true

ipv6: false

http:

enabled: false

JSONPEnabled: false

RESTInterfaceEnabled: false

#ssl:

#mode: <string>

#PEMKeyFile: <string>

#PEMKeyPassword: <string>

#security:

#keyFile: D:\mongodb\keyfile

#clusterAuthMode: keyFile

#authorization: disabled

operationProfiling:

slowOpThresholdMs: 100

mode: slowOp

replication:

oplogSizeMB: 50

replSetName: reptestname

secondaryIndexPrefetch: all

#enableMajorityReadConcern: <boolean>

#sharding:

#clusterRole: <string>

#archiveMovedChunks: <boolean>
```

## 数据库的相关操作
1. 切换/创建数据库
   > use dbname
2. 查看当前所处的数据库
   > db
3. 在数据库中插入数据
   > db.dbname.insert({……})
4. 删除当前所处数据库
   > db.dropDatabase()
5. 查看当前db版本
   > db.version()
6. 查看当前db的连接机器地址
   >db.getMongo()
7. 从指定主机上克隆数据库
   >db.cloneDatabase("127.0.0.1")
8. 从指定的机器上复制指定的数据库到某个数据库
   >db.copyDatabase("db1","db2","127.0.0.1")
9. 数据库修复
    >db.reqairDatabase()
	>在MongoDB中频繁的进行数据增删改时，如果记录变了，例如数据大小发生了变化，这时候容易产生一些数据碎片，出现碎片引发的结果，
	>一个是索引会出现性能问题，另外一个就是在一定的时间后，所占空间会莫明其妙地增大，所以要定期把数据库做修复，定期重新做索引，这样会提升MongoDB的稳定性和效率

## 为数据库添加用户及设置密码
  >db.createUser({user:'balabala',pwd:'******',rules:['options']})
  rules参数详情  

  |||
  |:---|:---|
  |**数据库用户角色(Database  User Roles)**|
  |read|只读权限|
  |readWrite |读写权限|
  |**数据库管理角色(Database Administration Roles)**|
  |dbAdmin|在当前数据库中执行管理操作|
  |dbOwnder|在当前db中执行任意操作|
  |userAdmin|在当前db中管理user|
  |**备份和还原角色**|
  |buckup|备份|
  |restore|还原|
  |**跨库角色(All-Database Roles)**|
  |readAnyDatabase|在所有db上读取数据|
  |readWriteAnyDatabase|在所有db上读写数据|
  |userAdminAnyDatabase|在所有数据库上管理user|
  |dbAdminAnyDatabase|管理所有数据库|
  |**集群管理角色(Cluster Administration Roles)**|
  |clusterAdmin|管理集群的最高权限|
  |clusterManager|管理和监控集群的权限A user with this role can access the config and local databases, which are used in sharding and replication, respectively.|
  |clusterMonitor|集群监控权限，对监控工具有readonly的权限|
  |hostManager|管理Server|


### 数据库登录
> mongo -umiaobt -p --authenticationDatabase
> mongo -umiaobt -p331sharp --authenticationDatabase

## 数据库中collection相关操作
1. 创建集合
   > db.createCollection(name,option)  
   >option为可选项，可以为对象  
	在使用db.collectionname.insert()插入数据时，可以自动创建集合
	option参数

	|字段名|类型|描述|
	|:----|:----:|:----|
	| capped|布尔|（可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数。|
	| autoIndexld|布尔|（即将弃用）如为 true，自动在 _id 字段创建索引。默认为 false。|
	| size|number|（可选）为固定集合指定一个最大值（以字节计）。如果 capped 为 true，也需要指定该字段。|
	| max|number|（可选）指定固定集合中包含文档的最大数量。|

	在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。
2. 删除集合
   >db.collectionname.drop()
3. 更新集合
   >db.cname.update()
   >>添加字段：db.cname.update(
	   {},
	   {$set:{……}}，
	   {},
   )
   >>删除字段db.cname.update(
	   {},
	   {$unset:{……}}，
	   {},
4. 固定集合
   > 固定集合是固定大小的集合支持基于文档插入顺序的高吞吐率的插入、检索、删除操作。限制收集工作在某种程度上类似于循环缓冲区：一旦一个文档填满分配给它的空间，他将通过在限制集中重写老文档来给新文档让出空间。 固定集合能够保留插入顺序。因此，查询并不需要索引来保证以插入顺序来返回文档。减少了索引的消耗， 固定集合可以支持更高的插入吞吐。 为了为新文档腾出空间，在不需要脚本或显式删除操作的前提下，固定集合自动删除集合中最旧的文档。 固定集合有一个 _id 字段并且默认在 _id 字段上创建索引。 注意事项： 如果一个更新或替换操作改变了文档大小，操作将会失败 不能从固定集合中进行删除文档，可以使用drop() 方法来删除集合然后重新创建限制集。 固定集合不支持分片
5. 检查一个集合是否为固定集合
   > db.collectionname.isCapped()
6. 使用命令convertToCapped转换一个非限制集合为一个限制集合
   > db.runCommand({"convertToCapped":"collectionname",size:10000})
## 集合中文档的增删改
1. 插入数据
   >db.cname.insert({……})或者db.cname.save()
   >> save()方法接收一个指定文档的_id，如果没有那么久和insert()使用功能一样
2. 更新数据
   >db.cname.update({query},{upate{upsert:boolean,multi:boolean,writeContent:document}})
   >>说明  

   |||
   |:----|:----|
   | query | update的查询条件，类似sql update查询内where后面的|
   | update | update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的|
   | upsert | 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。|
   | multi |可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。|
   | wirteContent |可选，抛出异常的级别。|
   > save()语法  
   db.cname.save()方法利用传入该方法的文档来替换已有文档（通俗解释：如果save方法传入的文档结构A与原文档结构B不一致时，则会使用新的文档结构A替换原来的文档结构B）。如果没有指定文档的 _id，那么 save() 就和 insert() 完全一样了。如果指定了文档的 _id，那么它会覆盖掉含有 save() 方法中指定的 _id的文档的全部数据  
   db.cname.save(document,{wirteContent:doucment})  
   db.users.save({"_id" : ObjectId("582194dd7f953cec73197bb1"),'status':"OK"})

3. 删除数据
   >db.cname.remove({query},{justOne:boolean,writeContent:document})
