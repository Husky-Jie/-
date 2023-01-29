# 1.zookeeper

- Zookeeper是ApacheHadoop项目下的一个子项目，是一个树形目录服务。
- Zookeeper翻译过来就是动物园管理员，他是用来管Hadoop(大象)、Hive(蜜蜂)、Pig(小猪)的管理员。简称zk 
- Zookeeper是一个分布式的、开源的分布式应用程序的协调服务。

# 2.zookeeper数据模型

ZooKeeper是一个树形目录服务，其数据模型和Unix的文件系统目录树很类似，拥有一个层次化结构。

这里面的每一个节点都被称为:ZNode，每个节点上都会保存自己的数据和节点信息。

节点可以拥有子节点，同时也允许少量(1MB)数据存储在该节点之下。节点可以分为四大类:

 PERSISTENT持久化节点

 EPHEMERAL临时节点:-e			一旦客户端断开后，则节点就会删除

 PERSISTENT SEQUENTIAL持久化顺序节点:-s

EPHEMERALSEQUENTIAL临时顺序节点:-es

# 3.zookeeper客户端命令

- **./zkCli.sh -server ip:port**       启动客户端

- **quit**       退出客户端

- **ls /目录**    查看节点

- **ls -s /目录**		查询节点详细信息

- 详细信息描述：

  ![image-20221112192030455](D:\Java学习\java笔记\image-20221112192030455.png)

- **create  (不写/-e/-s/-es)   /节点  数据**		创建（持久化/临时/持久化顺序/临时顺序）节点

- **get /节点**				获取节点数据

- **set /节点  数据**				设置节点数据

- **delete /节点**					删除单个节点（节点有子节点则不能删除）

- **deleteall /节点**				删除节点（节点有子节点则会全部删除）



# 4.zookeeper Java Api操作

Curator Api 使用起来比原生的zookeeper Api操作更简便

## 4.1Curator

zookeeper 为3.5.x版本以上  要使用Curator4.0以上的版本但不能使用

zookeeper 为3.4.x版本以上  也可使用Curator4.0版本（向下兼容）

## 4.2Curator Api操作

```xml
<!--Curator Api操作环境-->
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>4.3.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
        <version>4.3.0</version>
    </dependency>
    <!--日志-->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.32</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.36</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.10.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```



### 4.2.1建立连接

```java
// 建立连接
    @Test
    public void testConnect() {

        // 第一种方式
        // 重试策略，重试设置次数，重试之间的睡眠时间增加
        /*
        * new ExponentialBackoffRetry();
        * Params:
            baseSleepTimeMs – 重试之间等待的初始时间 单位ms
            maxRetries – 重试的最大次数*/
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(3000,3);
        /*
        * CuratorFrameworkFactory.newClient();
        * params:
        *   connectString – 连接字符串 zk -server ip:port "192.168.108.128:2181(可接多个，逗号分隔)"
            sessionTimeoutMs – 会话超时时间 单位ms
            connectionTimeoutMs – 连接超时时间 单位ms
            retryPolicy – 重试策略
            * */
        CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.108.128:2181", 60 * 1000,
                60 * 1000, retryPolicy);
        // 开启连接
        client.start();

        // 第二种方式
        /*
        * namespace("husky"):空间命名，每一次操作系统认为所命名的名称为根目录*/
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString("192.168.108.128:2181")
                .connectionTimeoutMs(60 * 1000)
                .sessionTimeoutMs(60 * 1000)
                .retryPolicy(retryPolicy).namespace("husky").build();

        client.close();
    }
```

### 4.2.2创建节点

**1.创建节点**

```java
@Test
public void testCreate() {
    try {
        // 如果创建的节点没有指定数据，则默认存储当前客户端的ip地址
        client.create().forPath("/app1");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**2.创建节点并添加数据**

```java
@Test
public void testCreateData() {
    try {
        // 如果创建的节点没有指定数据，则默认存储当前客户端的ip地址
        client.create().forPath("/app2","test".getBytes(StandardCharsets.UTF_8));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**3.创建节点并设置节点类型**

```java
 @Test
public void testCreateType() {
    try {
        // 如果创建的节点没有指定数据，则默认存储当前客户端的ip地址
    client.create().withMode(CreateMode.EPHEMERAL).forPath("/app3","test".getBytes(StandardCharsets.UTF_8));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**4.创建多级节点**

```java
@Test
public void testCreateMore() {
    try {
        // 如果创建的节点没有指定数据，则默认存储当前客户端的ip地址
        // creatingParentsIfNeeded() 如果父节点不存在，则创建父节点
      client.create().creatingParentsIfNeeded().forPath("/app3/p1","test".getBytes(StandardCharsets.UTF_8));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



### 4.2.3查询节点

**1.获取数据**

```java
// 获取数据：get /路径
@Test
public void testGet () {
    try {
        byte[] data = client.getData().forPath("/app3/p1");
        System.out.println(new String(data));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**2.查询子节点： ls /路径**

```java
// 查询子节点： ls /路径
@Test
public void testGetSon () {
    try {
        List<String> data = client.getChildren().forPath("/app3");
        System.out.println(data);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**3.查询节点状态信息：ls -s**

```java
// 查询节点状态信息：ls -s
@Test
public void testQueryStatus () {
    try {
        Stat status = new Stat();
        System.out.println(status);
        // 在获取节点数据时，storingStatIn(status)封装节点状态信息
        client.getData().storingStatIn(status).forPath("/app3/p1");
        System.out.println(status);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



### 4.2.4修改节点

**1.修改节点数据**

```java
// 修改节点数据
@Test
public void textSet() {
    try {
        client.setData().forPath("/app3/p1","hello".getBytes(StandardCharsets.UTF_8));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**2.根据版本修改节点数据**

```java
// 根据版本修改节点数据
@Test
public void textSetForVersion() {
    try {
        Stat status = new Stat();
        client.getData().storingStatIn(status).forPath("/app3/p1");
        // version是通过查询出来的，目的不让其他线程干扰
        System.out.println(status.getVersion());     client.setData().withVersion(status.getVersion()).forPath("/app3/p1","helloWorld".getBytes(StandardCharsets.UTF_8));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



### 4.2.5删除节点

**1.删除单个子节点**

```java
// 删除单个子节点
@Test
public void testOneDelete() {
    try {
        client.delete().forPath("/app1");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**2.删除带有子节点的节点**

```java
// 删除带有子节点的节点
@Test
public void testDeleteAll() {
    try {
        client.delete().deletingChildrenIfNeeded().forPath("/app3");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**3.必须删除成功**

```java
// 必须删除成功，防止网络抖动
@Test
public void testMustDelete() {
    try {
        client.delete().guaranteed().forPath("/app2");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



**4.回调**

```java
// 回调
@Test
public void testCallbackDelete() {
    try {
        client.delete().guaranteed().inBackground(new BackgroundCallback() {
            @Override
            public void processResult(CuratorFramework client, CuratorEvent event) throws Exception {
                System.out.println("删除成功");
                System.out.println(event);
            }
        }).forPath("/app2");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



## 4.3Curator Api-Watch监听

**1.**ZooKeeper 允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，Zookeeper 服务端会将事件通知到感兴趣的客户端上去，该机制是 ZooKeeper实现分布式协调服务的重要特性。ZooKeeper 中引入了Watcher机制来实现了发布/订阅功能能，能够让多个订阅者同时监听某一个对象，当一个对象自身状态变化时，会通知所有订阅者。

**2.**ZooKeeper 原生支持通过注册Watcher来进行事件监听，但是其使用并不是特别方便需要开发人员自己反复注册Watcher，比较繁琐

**3.Curator引入了Cache来实现对 ZooKeeper 服务端事件的监听ZooKeeper提供了三种Watcher:**

- NodeCache: 只是监听某一个特定的节点.
- PathChildrenCache:监控一个ZNode的子节点
- TreeCache:可以监控整个树上的所有节点，类似于PathChildrenCache和NodeCache的组合



### 4.3.1NodeCache

```java
// 指定一个节点注册监听器
@Test
public void testNodeCache() {
    // 创建NodeCache对象
    NodeCache nodeCache = new NodeCache(client,"/app1");
    // 注册监听
    nodeCache.getListenable().addListener(new NodeCacheListener() {
        @Override
        public void nodeChanged() throws Exception {
            System.out.println("节点改变");
            // 获取修改后的数据
            byte[] data = nodeCache.getCurrentData().getData();
            System.out.println(new String(data));
        }
    });
    // 开启监听，如果设置为true，则开启监听
    try {
        nodeCache.start(true);
        Thread.sleep(60*1000);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



### 4.3.2PathChildrenCache

```java
// 监听某个节点下的子节点，不包括自身
@Test
public void testPathChildrenCache() {
    // 创建对象
    PathChildrenCache pathChildrenCache = new PathChildrenCache(client,"/app1",true);
    System.out.println("连接成功");
    // 注册监听
    pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
        @Override
        public void childEvent(CuratorFramework client, PathChildrenCacheEvent event) throws Exception {
            System.out.println(event);
            // 监听子节点数据变更
            PathChildrenCacheEvent.Type type = event.getType();
            byte[] data = event.getData().getData();
            // 判断是否为update
            if (type.equals(PathChildrenCacheEvent.Type.CHILD_UPDATED)) {
                System.out.println(LocalDateTime.now()+"数据变更了");
                System.out.println(new String(data));
            }
        }
    });
    // 开启监听，如果设置为true，则开启监听
    try {
        pathChildrenCache.start();
        Thread.sleep(5*60*1000);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



### 4.3.3TreeCache

```java
// 监听某个节点下的子节点，包括自身
@Test
public void testTreeCache() {
    // 创建对象
    TreeCache treeCache = new TreeCache(client,"/app1");
    // 注册监听
    treeCache.getListenable().addListener(new TreeCacheListener() {
        @Override
        public void childEvent(CuratorFramework client, TreeCacheEvent event) throws Exception {
            System.out.println(event);
            byte[] data = event.getData().getData();
            System.out.println("数据变更了");
            System.out.println(new String(data));
        }
    });
    // 开启监听，如果设置为true，则开启监听
    try {
        treeCache.start();
        Thread.sleep(5*60*1000);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



## 4.4zookeeper分布式锁

在我们进行单机应用开发，涉及并发同步的时候，我们往往采用synchronized或者Lock的方式来解决多线程间的代码同步问题这时多线程的运行都是在同一个JVM之下，没有任何问题。但当我们的应用是分布式集群工作的情况下，属于多JVM下的工作环境，跨JVM之间已经无法通过多线程的锁解决同步问题，那么就需要一种更加高级的锁机制，来**处理种跨机器的进程之间的数据同步问题一一这就是分布式锁**



核心思想:当客户端要获取锁，则创建节点，使用完锁，则删除该节点

客户端获取锁时，在lock节点下创建**临时顺序**节点。
然后获取lock下面的所有子节点，客户端获取到所有的子节点之后，如果发现自己创建的子节点序号最小，那么就认为该客户端获取到了锁。使用完锁后，将该节点删除。
如果发现自己创建的节点并非lock所有子节点中最小的，说明自己还没有获取到锁，此时客户端需要找到比自己小的那个节点，同时对其注册事件监听器，监听删除事件。
如果发现比自己小的那个节点被删除，则客户端的Watcher会收到相应通知，此时再次判断自己创建的节点是否是lock子节点中序号最小的，如果是则获取到了锁如果不是则重复以上步骤继续获取到比自己小的一个节点并注册监听。

![image-20221113194305920](D:\Java学习\java笔记\zookeeper\image-20221113194305920.png)

### 4.4.1curator实现分布式锁api

**资源在谁那里，分布锁就锁哪里**

在Curator中有五种锁方案

- InterProcessSemaphoreMutex: 分布式排它锁(非可重入锁)
- InterProcessMutex: 分布式可重入排它锁
- InterProcessReadWriteLock: 分布式读写锁
- InterProcessMultiLock: 将多个锁作为单个实体管理的容器
- InterProcessSemaphoreV2: 共享信号量

模拟12306购票：

12306线程：

```java
public class Ticket12306 implements Runnable{
    private int tickets = 10;

    private InterProcessMutex lock;

    // 初始化对象
    public Ticket12306() {
        // 重试策略
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(3 * 1000,3);
        // 创建连接对象
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString("192.168.108.128:2181")
                .sessionTimeoutMs(60 * 1000)
                .connectionTimeoutMs(60 * 1000)
                .retryPolicy(retryPolicy).build();
        client.start();
        // 创建InterProcessMutex对象（分布式可重入排它锁）
        lock = new InterProcessMutex(client,"/lock");
    }

    @Override
    public void run() {
        while (true) {
            try {
                // 获取锁
                lock.acquire();
                if (tickets > 0) {
                    System.out.println(Thread.currentThread()+"票数还剩"+tickets);
                    Thread.sleep(300);
                    tickets--;
                }
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                // 释放锁
                try {
                    lock.release();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

第三方代理买票：

```java
public class LockTest {
    public static void main(String[] args) {
        Ticket12306 ticket12306 = new Ticket12306();

        Thread t1 = new Thread(ticket12306,"携程");
        Thread t2 = new Thread(ticket12306,"飞猪");

        t1.start();
        t2.start();
    }
}
```

# 5.zookeeper集群

(1)Zookeeper:一个领导者(Leader)，多个跟随者(Follower）组成的集群。
(2)Zookeepe集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。所以zookeeper适合安装奇数台服务器。
(3)全局数据一致:每个server保存一份相同的数据副本，client无论连接到哪个Server，数据都是一致的。
(4)更新请求顺序执行，来自同一个client的更新请求按其发送顺序依次执行，即先进先出。
(5)数据更新原子性，一次数据更新要么成功，要么失败。
(6)实时性，在一定时间范围内，client能读到最新数据。



# 6.zookeeper实际应用场景

1. ​		场景一：统一命名服务

   　　有一组服务器向客户端提供某种服务，我们希望客户端每次请求服务端都可以找到服务端集群中某一台服务器，这样服务端就可以向客户端提供客户端所需的服务。对于这种场景，我们的程序中一定有一份这组服务器的列表，每次客户端请求时候，都是从这份列表里读取这份服务器列表。那么这份列表显然不能存储在一台单节点的服务器上，否则这个节点挂掉了，整个集群都会发生故障，我们希望这份列表时高可用的。

      　　高可用的解决方案是：这份列表是分布式存储的，它是由存储这份列表的服务器共同管理的，如果存储列表里的某台服务器坏掉了，其他服务器马上可以替代坏掉的服务器，并且可以把坏掉的服务器从列表里删除掉，让故障服务器退出整个集群的运行，而这一切的操作又不会由故障的服务器来操作，而是集群里正常的服务器来完成。这是一种主动的分布式数据结构，能够在外部情况发生变化时候主动修改数据项状态的数据机构。Zookeeper框架提供了这种服务。这种服务名字就是：统一命名服务，它和JavaEE里的JNDI服务很像。 

2. 　　场景二：分布式锁服务

      　　当分布式系统操作数据，例如：读取数据、分析数据、最后修改数据。在分布式系统里这些操作可能会分散到集群里不同的节点上，那么这时候就存在数据操作过程中一致性的问题，如果不一致，我们将会得到一个错误的运算结果，在单一进程的程序里，一致性的问题很好解决，但是到了分布式系统就比较困难，因为分布式系统里不同服务器的运算都是在独立的进程里，运算的中间结果和过程还要通过网络进行传递，那么想做到数据操作一致性要困难的多。Zookeeper提供了一个锁服务解决了这样的问题，能让我们在做分布式数据运算时候，保证数据操作的一致性。 

3. 　　场景三：配置管理

      　　在分布式系统里，我们会把一个服务应用分别部署到n台服务器上，这些服务器的配置文件是相同的(例如：我设计的分布式网站框架里，服务端就有4台服务器，4台服务器上的程序都是一样，配置文件都是一样)，如果配置文件的配置选项发生变化，那么我们就得一个个去改这些配置文件，如果我们需要改的服务器比较少，这些操作还不是太麻烦，如果我们分布式的服务器特别多，比如某些大型互联网公司的hadoop集群有数千台服务器，那么更改配置选项就是一件麻烦而且危险的事情。

      　　这时候zookeeper就可以派上用场了，**我们可以把zookeeper当成一个高可用的配置存储器，把这样的事情交给zookeeper进行管理，我们将集群的配置文件拷贝到zookeeper的文件系统的某个节点上，然后用zookeeper监控所有分布式系统里配置文件的状态，一旦发现有配置文件发生了变化，每台服务器都会收到zookeeper的通知，让每台服务器同步zookeeper里的配置文件，zookeeper服务也会保证同步操作原子性，确保每个服务器的配置文件都能被正确的更新。** 

4. 　　场景四：为分布式系统提供故障修复的功能

      　　集群管理是很困难的，在分布式系统里加入了zookeeper服务，能让我们很容易的对集群进行管理。集群管理最麻烦的事情就是节点故障管理，zookeeper可以让集群选出一个健康的节点作为master，master节点会知道当前集群的每台服务器的运行状况，一旦某个节点发生故障，master会把这个情况通知给集群其他服务器，从而重新分配不同节点的计算任务。Zookeeper不仅可以发现故障，也会对有故障的服务器进行甄别，看故障服务器是什么样的故障，如果该故障可以修复，zookeeper可以自动修复或者告诉系统管理员错误的原因让管理员迅速定位问题，修复节点的故障。大家也许还会有个疑问，master故障了，那怎么办了？zookeeper也考虑到了这点，zookeeper内部有一个“选举领导者的算法”，master可以动态选择，当master故障时候，zookeeper能马上选出新的master对集群进行管理。

