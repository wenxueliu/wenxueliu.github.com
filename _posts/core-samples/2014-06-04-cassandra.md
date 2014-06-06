---
layout: post
category : cassandra
tagline: "调研"
tags : [cassandra]
---
{% include JB/setup %}

**作者**

    刘文学

**版本**

    0.1 : 初始化

**注**
    本文档仅从宏观了解 cassandra 的各方面指标，作为存储系统选择的参考
    性能见附件

Cassandra 简介
--------------------
Cassandra 是结合了 Google Bigtable 数据模型和 Amazon Dynamo 分布式系统的一个高可用、最终一致的的 key-value 数据存储库。Cassandra 设计之初目的是解决 Facebook 收件箱搜索的存储需要。在 Facebook,这意味着这个系统需要能够处理非常大的写吞吐量,每天几十亿的写请求,随着用户数的规模而增长.通过在地理上分布的数据中心对用户进行服务的,因此支持跨越多个数据中心的数据复制。由 Facebook 在 2008 年开源，目前 是Apache 的顶级项目。

目前商业支持公司 [DataStax](http://www.datastax.com)


安装
--------------------
非常简单，支持 deb , rpm 和源码安装。前提是安装 java 7。

[特点][1]
--------------------
* 高可用
* 线性增长
* 最终一致性(低延迟)
* 一致性和延迟可互换
* 最小化运维
* 没有单点故障

架构
--------------------
无中心节点

存储机制
--------------------

采用 [CommitLog][commiglog] --> [Memtable][Memtable] --> [SSTable][SSTable] --> Compaction的方式

* CommitLog：主要记录下客户端提交过来的数据以及操作。这个数据将被持久化到磁盘中，以便数据没有被持久化到磁盘时可以用来恢复。当一 个Commitlog 文件写满以后，会新建一个的文件。当旧的 Commitlog 文件不再需要时，会自动清除。 。Commitlog 是 server 级别的，不是 Column Family级别的。
* Memtable：Memory table 的简写，常驻内存的 key/value 键值字符对。
* SSTable： Sorted Strings Table的简写，是一个 key/value 的字符对，这又分为 Data、Index 和 Filter 三种数据格式。

[Commiglog]: http://wiki.apache.org/cassandra/ArchitectureCommitLog
[Memtable]: http://wiki.apache.org/cassandra/ArchitectureSSTable
[SSTable]: http://wiki.apache.org/cassandra/MemtableSSTable

写
------------------------

**特点**

* No reads
* No seeks
* Fast
* Atomic within ColumnFamily
* Always writable 


**方式**

* Quorum写: 阻塞直到 quorum 到达
* 异步写: 发送请求到任意节点，这个节点立即返回，之后将数据推送到合适的节点。

**过程**

Cassandra 在写数据之前，也需要先记录日志，称之为 Commitlog，然后数据才会写入到 Column Family 对应的 Memtable 中，并且 Memtable 中的内容是按照 key 排序好的。Memtable 是一种内存结构，满足一定条件后批量刷新到磁盘上，存储为SSTable。这种机制，相当于缓存写回机制(Write-back Cache)，优势在于将随机 IO 写变成顺序 IO 写，降低大量的写操作对于存储系统的压力。SSTable 一旦完成写入，就不可变更，只能读取。下一次 Memtable 需要刷新到一个新的SSTable文件中。所以对于 Cassandra 来说，可以认为只有顺序写，没有随机写操作。

CommitLog 数据持久化

* Periodic : 异步
* Batch : 同步

Memtable 写入磁盘的时机

* 超过规定的内存空间
* 超过 128 个 key
* 超时（不是cluster clock，而是由客户端提供）

SSTable 组成

* DataFile : 存储实际数据的 key/value 键值对
* Index File : key/offset 对，offset 采用 [Bloom filter](en.wikipedia.org/wiki/Bloom_filter) 算法



读
---------------------

**特点**

* Read multiple SSTables 
* Slower than writes (but still fast) 
* Seeks can be mitigated with more RAM 
* Scales to billions of rows 


**过程**

Cassandra 系统通过主键来来索引所有数据，磁盘上的数据文件被分解成一系列的块，每个块内最多包含 128 个键，并通过一个块索引来区分，块索引抓取块内的键的相对偏移量以及其数据大小，当内存数据结构被转储到磁盘时，系统会为其生成一个索引，它的偏移量会被当作索引写到磁盘上。内存中也会维护一份这个索引以提供快速访问。一个典型的读操作总是会先检索内存数据结构，如果找到就将数据返回给应用程序，因为内存数据结构中包含任何键的最新数据。如果没有找到，那么我们就需要对所有磁盘数据文件按照时间逆序来执行磁盘IO。由于总是寻求最新的数据，我们就先查阅最新的文件,一旦找到数据就返回。


因为SSTable数据不可更新，可能导致同一个 Column Family 的数据存储在多个 SSTable 中，这时查询数据时，需要去合并读取 Column Family 所有的 SSTable 和Memtable，这样到一个Column Family的数量很大的时候，可能导致查询效率严重下降。因此需要有一种机制能快速定位查询的 Key 落在哪些SSTable中，而不需要去读取合并所有的SSTable。Cassandra 采用的是 Bloom Filter 算法，通过多个 hash 函数将 key 映射到一个位图中，来快速判断这个 key 属于哪个 SSTable。关于Bloom Filter 参考文章[这里][4],[这里][5]和[还有这里][6]。 

为了避免大量SSTable带来的性能影响，Cassandra 也提供一种定期将多个 SSTable 合并成一个新的 SSTable 的机制，因为每个 SSTable 中的 key 都是已经排序好的，因此只需要做一次合并排序就可以完成该任务，代价还是可以接受的。所以在 Cassandra 的数据存储目录中，可以看到三种类型的文件，格式类似于： 

* Column Family Name-序号-Data.db ： SSTable数据文件，按照 key 排序后存储 key/value 键值字符对
* Column Family Name-序号-Filter.db ： 索引文件，保存的是每个key在数据文件中的偏移位置
* Column Family Name-序号-index.db ： Bloom Filter算法生产的映射文件

__*注*__
> Cassandra 读写操作实际上都是无锁操作

压缩
-------------------------------
每隔一段时间,就会运行一个主压缩程序来将所有相关的数据文件压缩成一个大文件。这个压缩进程是一个磁盘 IO 密集型的操作，需要对此做大量的优化以做到不影响后续的读请求。

通过只解压指定部分的数据块，Cassandra对从磁盘上读取数据的性能也有提升。

插件式压缩策略 

* Tiered Compaction : 将大小相似的 SStable 压缩，适合写
* Leveled Compaction : 纵向排序数据，适合读

删除
-------------------------------
将待删除文档作标记，等待定期的垃圾回收进程删除文档。

[数据模型][module1]
-------------------------
表是一个按照主键索引的分布式多维图。它的值是一个高度结构化的对象。表中的记录键是一个没有大小限制的字符串，虽然它通常都只有16-36个字节的长度。无论需要读写多少列，单一记录键的每个副本的每次操作都是一个原子操作。多个列可以组合在一起形成一个称为 column family 的列的集合,

keyspace : 类似关系型数据库的 database
column family： 类似关系型数据库的 table。简单的 column family、超级的 column family（想象成column family里面嵌入column family）
key : 代表一行的关键词
column : 列名、值、时间戳
	

列的排序顺序：按时间对列进行排序、按名称对列进行排序。

具体参考[这里][data_model1]，[这里][data_model2]，还有[这里][data_model3]

**注**
> 可以理解为Lua的table

[module1]: http://wiki.apache.org/cassandra/DataModel
[1]: http://wiki.apache.org/cassandra/ArchitectureOverview
[data_model1]: http://www.slideshare.net/planetcassandra/c-summit-eu-2013-apache-cassandra-20-data-model-on-fire
[data_model2]: http://www.infoq.com/cn/articles/best-practice-of-cassandra-data-model-design
[data_model3]: http://www.infoq.com/cn/articles/best-practices-cassandra-data-model-design-part2
[data_model4]: http://www.datastax.com/dev/blog/leveled-compaction-in-apache-cassandra
[data_module]: http://www.datastax.com/dev/blog/when-to-use-leveled-compaction


原理
--------------------------

一个键的请求可能被路由到 Cassandra 集群的任何一个节点去处理。这个节点会确定这个特定的键的副本，对于写操作来讲，系统会将请求路由到副本上，并且等待仲裁数量的副本以确认写操作完成。对于读操作来讲，基于客户端要求的一致性保证,系统要么将请求路由到最近的副本，要么将请求路由到所有的副本并等待达到仲裁数量的响应。

**复制**

Cassandra 使用复制来实现高可用性与持久性

复制策略：机架不可知”(rack unaware)、”机架可知”(rack aware)(同一个数据中心内)与”数据中心可知”(data-center aware)

复制模式：同步写(synchronous write)、异步写(asynchronous write).


**持久化**

Cassandra 系统要依赖于本地文件系统做数据的持久化。这些数据是以一种易于高效检索的格式存储在磁盘上。如果 MemTable 数据丢失通过 Commitlog 来恢复

**监控**

Cassandra 系统与 Ganglia 做了很好的集成，参考附录中的工具集。

**扩展性**

高可用，在线可扩展：单点故障不影响集群服务，可线性扩展。

**运维**

Cassandra的日常维护工作很简单。任一时刻都可以在某台节点上进行升级工作，而当某个节点停机时，其它节点会保留
本应应用在该节点上的升级内容，并在该节点恢复后将升级内容重新发送给它。此外，添加新节点的操作可以在整个集群
中并行进行，在操作完成后也无需重新进行平衡。 

**备份**

同步或异步，自动备份支持多节点和多数据中心


[CQL Driver](http://wiki.apache.org/cassandra/ClientOptions)
-------------------------

DataStax 官方支持(Apache License, Version 2.0)

* [Java](https://github.com/datastax/java-driver) 
* [C#](https://github.com/datastax/csharp-driver) 
* [Python](https://github.com/datastax/python-driver) 
* [Cpp](https://github.com/datastax/cpp-driver)

社区

* [Clojure](https://github.com/mpenet/hayt) 
* [Elang](https://github.com/iamaleksey/seestar)
* [Go](https://github.com/gocql/gocql) 
* [Haskell](http://hackage.haskell.org/package/cassandra-cql)
* [Node.js](https://github.com/jorgebay/node-cassandra-cql) 
* [Perl](https://metacpan.org/pod/Net::Async::CassandraCQL) 
* [PHP](code.google.com/a/apache-extras.org/p/cassandra-pdo/) 
* [Ruby](https://github.com/iconara/cql-rb) 
* [Scala](https://github.com/eklavya/Scqla)


使用场景
-----------------------------
Cassandra 确实非常适合时间序列数据，因为可以周期性的将时间序列数据写入1列，
然后通过部分字符串对比来查询一定时间范围内的数据。这样 的话使用列就比使用行更加的高效，
而加载单行可以让你获得巨大的I/O性能。

使用MongoDB时，写操作的数量会严重影响到读的性能，而使用Cassandra就不会有这种情况发生。

案例
-----------------------------




附录
-------------------------

涉及算法

* [ Gossip ][gossip]：复制，同步
* 向量时钟(vector lock)：版本冲突
* 一致性散列(consistent hash)：带有虚拟节点的一致性Hash
* [布隆过滤器][bloom filter1](bloom filter)：索引
* 最终一致性：CAP 选择 AP，C保证最终一致性

[bloom filter1]: http://blog.csdn.net/jiaomeng/archive/2007/01/27/1495500.aspx
[bloom filter2]: http://www.hellodba.net/2009/04/bloom_filter.html
[bloom filter3]: http://www.googlechinablog.com/2007/07/bloom-filter.html
[gossip]: http://blog.csdn.net/wanghai__/article/details/5783561

在ACID(Atomic Consistent Isolated Durable)方面 cassandra 支持：

* Atomicity
* Tunable consistency
* Isolation
* Durability

工具集

* cqlsh : 类似关系数据库的 SQL 语句
* nodetool : 命令行管理工具
* [OpsCenter](www.datastax.com/documentation/opscenter/4.1/opsc/about_c.html) ： DataStax 的图形管理管理和监控 cassandra 集群工具
* JConsole : GUI 管理工具
* 支持 hadoop的工具集



