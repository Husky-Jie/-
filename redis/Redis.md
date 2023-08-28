# 1.NoSql数据库



## 1.1NoSql概述

NoSQL(NoSQL = **Not Only SQL** )，意即“不仅仅是SQL”，泛指**非关系型的数据库**。 

NoSQL 不依赖业务逻辑方式存储，而以简单的key-value模式存储。因此大大的增加了数据库的扩展能力。

1. 不遵循SQL标准。
2. 不支持ACID。
3. 远超于SQL的性能。



## 1.2NoSql的适用场景

1. 对数据高并发的读写
2. 海量数据的读写
3. 对数据高可扩展性的

不适用场景

1. 需要事务支持
2. 基于sql的结构化查询存储，处理复杂的关系,需要即席查询。



## 1.3Redis

数据都在内存中，支持持久化，主要用作备份恢复。除了支持简单的key-value模式，还支持多种数据结构的存储，比如 list、set、hash、zset等。一般是作为缓存数据库辅助持久化的数据库。

1. Redis是一个开源的key-value存储系统。
2. 和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。
3. 这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。
4. 在此基础上，Redis支持各种不同方式的排序。
5. 与memcached一样，为了保证效率，数据都是缓存在内存中。
6. 区别的是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件。
7. 并且在此基础上实现了master-slave(主从)同步。

Redis是单线程+多路IO复用技术

多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）



### 1.3.1应用场景

配合关系型数据库做高速缓存

1. 高频次，热门访问的数据，降低数据库IO
2. 分布式架构，做session共享

多样的数据结构存储持久化数据



### 1.3.2原子操作

所谓原子操作是指不会被线程调度机制打断的操作；
这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。
（1）在单线程中， 能够在单条指令中完成的操作都可以认为是"原子操作"，因为中断只能发生于指令之间。
（2）在多线程中，不能被其它进程（线程）打断的操作就叫原子操作。
Redis单命令的原子性主要得益于Redis的单线程。



# 2.Redis的安装

在官网https://redis.io下载，并安装在lunix中（自己上网查）

默认安装目录：

redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何

redis-check-aof：修复有问题的AOF文件，rdb和aof后面讲

redis-check-dump：修复有问题的dump.rdb文件

redis-sentinel：Redis集群使用

redis-server：Redis服务器启动命令

redis-cli：客户端，操作入口

## 2.1Redis启动



### 2.1.1前台启动（不推荐）

命令：redis-server

前台启动，命令行窗口不能关闭，否则服务器停止



### 2.1.2后台启动（推荐）

1. 拷贝一份redis.conf到其他目录，命令：cp  /opt/redis-6.2.1/redis.conf  /myredis（拷贝到的目录不唯一）

2. 后台启动设置daemonize no改成yes，（改变拷贝后的redis.conf），将redis.conf中的daemonize no改成yes，修改redis.conf(128行)文件将里面的daemonize no 改成 yes，让服务在后台启动

3. Redis启动，命令 redis-server /myredis/redis.conf

4. 用客户端访问：redis-cli

   如有密码：输入密码命令：auth 密码

5. 测试验证： ping

6. Redis关闭

   单实例关闭：命令redis-cli shutdown

   也可以进入终端后再关闭 命令shutdown

### 2.1.3 liunx防火墙操作

查看防火墙状态
systemctl status firewalld、 firewall-cmd --state
暂时关闭防火墙
systemctl stop firewalld
永久关闭防火墙（慎用）
systemctl disable firewalld
开启防火墙
systemctl start firewalld
开放指定端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent
关闭指定端口
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
立即生效
firewall-cmd --reload
查看开放的端口
firewall-cmd --zone=public --list-ports

# 3.Redis的常用五大数据类型



## 3.1Redis常用命令

keys *查看当前库所有key  (匹配：keys *1)

exists key判断某个key是否存在

type key 查看你的key是什么类型

del key    删除指定的key数据

unlink key  根据value选择非阻塞删除

仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。

expire key 10  10秒钟：为给定的key设置过期时间

ttl key 查看还有多少秒过期，-1表示永不过期，-2表示已过期

select命令切换数据库

dbsize查看当前数据库的key的数量

flushdb清空当前库

flushall通杀全部库

![image-20220826112435190](assets\image-20220826112435190.png)

NX：当数据库中key不存在时，可以将key-value添加数据库

XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥

EX：key的超时秒数

PX：key的超时毫秒数，与EX互斥



## 3.2字符串（String）

String是Redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

String类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M



### 3.2.1字符串常用命令

1. set   <key><value>添加键值对
2. get   <key>查询对应键值
3. append  <key><value>将给定的<value> 追加到原值的末尾
4. strlen  <key>获得值的长度
5. setnx  <key><value>只有在 key 不存在时    设置 key 的值
6. incr  <key>将 key 中储存的数字值增1，只能对数字值操作，如果为空，新增值为1
7. decr  <key>将 key 中储存的数字值减1，只能对数字值操作，如果为空，新增值为-1
8. incrby / decrby  <key><步长>将 key 中储存的数字值增减。自定义步长。
9. mset  <key1><value1><key2><value2>  ..... 同时设置一个或多个 key-value对  
10. mget  <key1><key2><key3> .....同时获取一个或多个 value  
11. msetnx <key1><value1><key2><value2>  ..... 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。原子性，有一个失败则都失败
12. getrange  <key><起始位置><结束位置>获得值的范围，类似java中的substring，前包，后包
13. setrange  <key><起始位置><value>用 <value>  覆写<key>所储存的字符串值，从<起始位置>开始(索引从0开始)。
14. setex  <key><过期时间><value>设置键值的同时，设置过期时间，单位秒。
15. getset <key><value>以新换旧，设置了新值同时获得旧值。



## 3.3列表（List）

1）**单键多值**

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。



2）**数据结构**

List的数据结构为快速链表quickList。

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。

它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

当数据量比较多的时候才会改成quicklist。

因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。   

Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。



### 3.3.1常用命令

1. lpush/rpush <key><value1><value2><value3> .... 从左边/右边插入一个或多个值。
2. lpop/rpop <key>从左边/右边吐出一个值。值在键在，值光键亡。
3. rpoplpush <key1><key2>从<key1>列表右边吐出一个值，插到<key2>列表左边。
4. lrange <key><start><stop>按照索引下标获得元素(从左到右)
5. lrange mylist 0 -1  0左边第一个，-1右边第一个，（0-1表示获取所有）
6. lindex <key><index>按照索引下标获得元素(从左到右)
7. llen <key>获得列表长度 
8. linsert <key> before <value><newvalue>在<value>的后面插入<newvalue>插入值
9. lrem <key><n><value>从左边删除n个value(从左到右)
10. lset<key><index><value>将列表key下标为index的值替换成value



## 3.4集合（set）

1）Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以**自动排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的**复杂度都是O(1)**。

一个算法，随着数据的增加，执行时间的长短，如果是O(1)，数据增加，查找数据的时间不变



2）**数据结构**

Set数据结构是dict字典，字典是用哈希表实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

### 3.4.1常用命令

1. sadd <key><value1><value2> ..... 
2. 将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
3. smembers <key>取出该集合的所有值。
4. sismember <key><value>判断集合<key>是否为含有该<value>值，有1，没有0
5. scard<key>返回该集合的元素个数。
6. srem <key><value1><value2> .... 删除集合中的某个元素。
7. spop <key>**随机从该集合中吐出一个值。**
8. srandmember <key><n>随机从该集合中取出n个值。不会从集合中删除 。
9. smove <source><destination>value把集合中一个值从一个集合移动到另一个集合
10. sinter <key1><key2>返回两个集合的交集元素。
11. sunion <key1><key2>返回两个集合的并集元素。
12. sdiff <key1><key2>返回两个集合的**差集**元素(key1中的，不包含key2中的)



## 3.5哈希（hash）

1）Redis hash 是一个键值对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

类似Java里面的Map<String,Object>

用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储。

2）**数据结构**

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

### 3.5.1常用命令

1. hset <key><field><value>给<key>集合中的 <field>键赋值<value>
2. hget <key1><field>从<key1>集合<field>取出 value 
3. hmset <key1><field1><value1><field2><value2>... 批量设置hash的值
4. hexists<key1><field>查看哈希表 key 中，给定域 field 是否存在。 
5. hkeys <key>列出该hash集合的所有field
6. hvals <key>列出该hash集合的所有value
7. hincrby <key><field><increment>为哈希表 key 中的域 field 的值加上增量 1  -1
8. hsetnx <key><field><value>将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在



## 3.6有序集合（sorted set）

1）Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分（score）**,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

2）**数据结构**

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。



### 3.6.1常用命令

1. zadd <key><score1><value1><score2><value2>…将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
2. **zrange <key><start><stop> [WITHSCORES]**  返回有序集 key 中，下标在<start><stop>之间的元素
3. 带WITHSCORES，可以让分数一起和值返回到结果集。
4. zrangebyscore key minmax [withscores] [limit offset count] 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 
5. zrevrangebyscore key maxmin [withscores] [limit offset count] 同上，改为从大到小排列。 
6. zincrby <key><increment><value>   为元素的score加上增量
7. zrem <key><value>删除该集合下，指定值的元素
8. zcount <key><min><max>统计该集合，分数区间内的元素个数 
9. zrank <key><value>返回该值在集合中的排名，从0开始



# 4.事务和锁机制

## 4.1事务的定义

Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

Redis事务的主要作用就是串联多个命令防止别的命令插队。

## 4.2mult、exec、discard

1）从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。组队的过程中可以通过discard来放弃组队。

2）事务的错误处理

1. 组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。
2. 如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。

## 4.3悲观锁

![image-20220829110906187](assets\image-20220829110906187.png)

**悲观锁(Pessimistic Lock)**, 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。**传统的关系型数据库里边就用到了很多这种锁机制**，比如**行锁**，**表锁**等，**读锁**，**写锁**等，都是在做操作之前先上锁。



## 4.4乐观锁

![image-20220829110934032](assets\image-20220829110934032.png)

**乐观锁(Optimistic Lock),** 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis就是利用这种check-and-set机制实现事务的。



## 4.5watch和unwatch

**1）watch：**在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务**执行之前这个(****或这些) key** **被其他命令所改动，那么事务将被打断。**

**2）unwatch：**取消 WATCH 命令对所有 key 的监视。如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。



## 4.6Redis事务的三大特性

1. **单独的隔离操作** 

   事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

2. **没有隔离级别的概念** 

   队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

3. **不保证原子性** 

   事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 



# 5.Redis持久化

Redis 提供了2个不同形式的持久化方式。

RDB（Redis DataBase）

AOF（Append Of File）

## 5.1RDB

1）在指定的时间间隔内将内存中的数据集快照写入磁盘， 也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

2）Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**RDB****的缺点是最后一次持久化后的数据可能丢失**。

3）Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程

 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了“**写时复制技术**”

**一般情况父进程和子进程会共用同一段物理内存**，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。



### 5.1.1如何触发RDB快照

1、配置文件中默认的快照配置

![image-20220901112038712](assets\image-20220901112038712.png)

save ：save时只管保存，其它不管，全部阻塞。手动保存。不建议。

bgsave：Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。

可以通过lastsave 命令获取最后一次成功执行快照的时间



### 5.1.2优势与劣势及停止

**优势**

1. 适合大规模的数据恢复
2. 对数据完整性和一致性要求不高更适合使用
3. 节省磁盘空间
4. 恢复速度快

​                               

 **劣势**

1. Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
2. 虽然Redis在fork时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能。
3. 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

**动态停止RDB：redis-cli config set save ""#save后给空值，表示禁用保存策略**

## 5.2AOF

1）以**日志**的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(**读操作不记录**)， **只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

2） AOF默认不开启

可以在redis.conf中配置文件名称，默认为 appendonly.aof；AOF文件的保存路径，同RDB的路径一致。

3）AOF和RDB同时开启，系统默认取AOF的数据（数据不会存在丢失）

### 5.2.1AOF持久化流程

（1）客户端的请求写命令会被append追加到AOF缓冲区内；

（2）AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；

（3）AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；

（4）Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的；

### 5.2.2AOF启动/修复/恢复

AOF的备份机制和性能虽然和RDB不同, 但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载。

**正常恢复**

修改默认的appendonly no，改为yes

将有数据的aof文件复制一份保存到对应目录(查看目录：config get dir)

恢复：重启redis然后重新加载

 

**异常恢复**

修改默认的appendonly no，改为yes

如遇到**AOF**文件损坏**，通过/usr/local/bin/**redis-check-aof--fix appendonly.aof**进行恢复

备份被写坏的AOF文件

恢复：重启redis，然后重新加载



### 5.2.3AOF同步频率设置

**appendfsync always**

始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好

**appendfsync everysec**

每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。

**appendfsync no**

redis不主动进行同步，把同步时机交给操作系统。



### 5.2.4AOF优势和劣势

 **优势**

1. 备份机制更稳健，丢失数据概率更低。
2. 可读的日志文本，通过操作AOF稳健，可以处理误操作。

**劣势**

1. 比起RDB占用更多的磁盘空间。
2. 恢复备份速度要慢。
3. 每次读写都同步的话，有一定的性能压力。
4. 存在个别Bug，造成恢复不能。

## 5.3总结

1）官方推荐两个都启用。如果对数据不敏感，可以选单独用RDB。不建议单独用 AOF，因为可能会出现Bug。如果只是做纯内存缓存，可以都不用。

2）因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。     如果使用AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。  代价,一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。  只要硬盘许可，应该尽量减少AOF  rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。  默认超过原大小100%大小时重写可以改到适当的数值。  

# 6.Redis集群

Redis 集群实现了对Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。

Redis 集群通过分区（partition）来提供一定程度的可用性（availability）： 即使集群中有一部分节点失效或者无法进行通讯， 集群也可以继续处理命令请求。

## 6.1redis cluster配置修改

cluster-enabled yes  打开集群模式

cluster-config-file nodes-6379.conf 设定节点配置文件名

cluster-node-timeout 15000  设定节点失联时间，超过该时间（毫秒），集群自动进行主从切换。

## 6.2slots

一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个， 

集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。

集群中的每个节点负责处理一部分插槽。 举个例子， 如果一个集群可以有主节点， 其中：

节点 A 负责处理 0 号至 5460 号插槽。

节点 B 负责处理 5461 号至 10922 号插槽。

节点 C 负责处理 10923 号至 16383 号插槽。

## 6.3集群的优缺点

**Redis** **集群提供了以下好处**

实现扩容

分摊压力

无中心配置相对简单

**Redis** **集群的不足**

多键操作是不被支持的 

多键的Redis事务是不被支持的。lua脚本不被支持

由于集群方案出现较晚，很多公司已经采用了其他的集群方案，而代理或者客户端分片的方案想要迁移至redis cluster，需要整体迁移而不是逐步过渡，复杂度较大。

# 7.Redis应用问题解决

## 7.1缓存穿透

key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

![image-20220901115151573](assets\image-20220901115151573.png)

**解决方案**

一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。

解决方案：

**（1）**  **对空值缓存：**如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟

**（2）**  **设置可访问的名单（白名单）：**

使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量，每次访问和bitmap里面的id进行比较，如果访问id不在bitmaps里面，进行拦截，不允许访问。

**（3）**  **采用布隆过滤器**：(布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。

布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。)

将所有可能存在的数据哈希到一个足够大的bitmaps中，一个一定不存在的数据会被 这个bitmaps拦截掉，从而避免了对底层存储系统的查询压力。

**（4）**  **进行实时监控：**当发现Redis的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务

## 7.2缓存击穿

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

![image-20220901115317796](assets\image-20220901115317796.png)

**解决方案**

key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。

解决问题：

**（1）预先设置热门数据：**在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长

**（2）实时调整：**现场监控哪些数据热门，实时调整key的过期时长

**（3）使用锁：**

1. 就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db。
2. 先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key
3. 当操作返回成功时，再进行load db的操作，并回设缓存,最后删除mutex key；
4. 当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法。
5. ![image-20220901115442053](assets\image-20220901115442053.png)



## 7.3缓存雪崩

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

缓存雪崩与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key正常访问

![image-20220901115550421](assets\image-20220901115550421.png)

**缓存失效瞬间**

![image-20220901115618987](assets\image-20220901115618987.png)



**解决方案**

缓存失效时的雪崩效应对底层系统的冲击非常可怕！

解决方案：

**（1）**  **构建多级缓存架构：**nginx缓存 + redis缓存 +其他缓存（ehcache等）

**（2）**  **使用锁或队列：**

用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况

**（3）**  **设置过期标志更新缓存：**

记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。

**（4）**  **将缓存失效时间分散开：**

比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。



# 8.分布式锁

随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要一种跨JVM的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题！

分布式锁主流的实现方案：

1.  基于数据库实现分布式锁
2.  基于缓存（Redis等）
3.  基于Zookeeper

每一种分布式锁解决方案都有各自的优缺点：

1.  性能：redis最高
2. 可靠性：zookeeper最高

这里，我们就基于redis实现分布式锁。

## 8.1解决方案：使用redis实现分布式锁

redis:命令

\# set sku:1:info “OK” NX PX 10000

EX second ：设置键的过期时间为 second 秒。 SET key value EX second 效果等同于 SETEX key second value 。

PX millisecond ：设置键的过期时间为 millisecond 毫秒。 SET key value PX millisecond 效果等同于 PSETEX key millisecond value 。

NX ：只在键不存在时，才对键进行设置操作。 SET key value NX 效果等同于 SETNX key value 。

XX ：只在键已经存在时，才对键进行设置操作。

![image-20220901115926319](assets\image-20220901115926319.png)

1. 多个客户端同时获取锁（setnx）

2. 获取成功，执行业务逻辑{从db获取数据，放入缓存}，执行完成释放锁（del）

3. 其他客户端等待重试

