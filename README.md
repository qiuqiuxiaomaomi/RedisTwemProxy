# RedisTwemProxyTree
redis的主备机制，分片机制，负载均衡机制，高可用性


![](https://i.imgur.com/7uhECaJ.png)

<pre>
TwemProxy

          又称nutcracker,是一个memcache,Redis协议的轻量级代理，一个用于sharding的中间
      件。有了TwemProxy，客户端不直接访问Redis服务器，而是通过TwemProxy代理中间件直接访问。
      TwemProxy为Twitter开源产品，简单来说，TewmProxy是Twitter开发的一个Redis代理Proxy,
      类似于Nginx的反向代理或者Mysql的代理工具amoeba,TwemProxy通过引入一个代理层，可以
      将其后端的多台Redis或Memcached实例进行统一管理与分配，使应用程序只需要在TwemProxy上
      进行操作，而不用关心后边具体有多少个真实的Redis或者Memcached存储。

          一般来说，只要服务器上运行了Redis，那么就有可能造成非常可怕的局面：服务器的内存将
      被立刻占满，而且一台Redis数据库的性能终归是有限制的，那么现在如果要求保证用户的执行速度
      快，就需要使用集群的设计，而对于设计主要的问题就是解决单实例Redis的性能问题。

          TwemProxy是一个专门为了这种nosql数据库设计的一款代理工具软件，这个工具软件最大的
      特征就是可以实现数据的分片处理。所谓的分片指的是根据一定的算法将要保存的数据保存到不同的
      节点之中。有了分片之后数据的保存节点就可能有无限多个，但是理论上如果要真进行集群的搭建，
      往往3台节点起步。
</pre>

<pre>
Twemproxy代理机制的特点：

      1）支持失败节点自动删除
         可以设置重新连接该节点的时间
         可以设置连接多少次之后删除该节点
      2）支持设置hashtag
         通过hashtag可以自己设定将两个key哈希到同一个实例上。
      3）减少与Redis的直接连接数
         保持与redis的长连接
         减少了客户端直接与服务器连接的连接数量
      5）自动分片到后端多个redis实例上
         多种hash算法：md5, crc16,crc32a等
         多种分片算法random等
         支持设置后端权重
      6）避免单点故障
         可以平行部署多个代理层，通过Haproxy做负载均衡，将redis的读写分散到多个twemproxy上。
      7）支持状态监控
         可以设置状态监控ip和端口，访问ip和端口可以得到一个json格式的状态信息
         可以设置监控信息刷新间隔时间
      8）使用pipelining处理请求和响应
         连接复用，内存复用
         将多个连接请求，组成redis pipelining统一向redis请求
      9）并不是支持所有redis命令
         不支持redis事务操作
         使用SIDFF, SDIFFSTORE, SINTER, SINTERSTORE, SMOVE, SUNION and SUNIONSTORE
         命令需要保证key都在同一个分片上。

      需要注意的是：
          Twemproxy不会增加Redis的性能指标数据，据业界测算，使用twemproxy相比直接使用
          Redis会带来大约10%的性能下降。但是单个Redis进程的内存管理能力有限。据测算，单个
          Redis进程内存超过20G之后，效率会急剧下降。目前，建议单个Redis最好配置在8G以内；
          8G以上的Redis缓存需求，通过Twemproxy来提供支持。
</pre>

###集群部署架构

![](https://i.imgur.com/q1sDZDa.png)

<pre>
TwemProxy缺点：

      Twem是这些年来，应用范围最广，稳定性最高，最久经考验的分布式中间件。

      但是TwemProxy最大的痛点在于，无法平滑的扩容/缩容
          这样导致运维人员非常痛苦：业务量突增，需要增加Redis服务器，业务量萎缩，需要减少Redis
      服务器，但对于Twemproxy而言，基本很难做到。

          或者说Twemproxy更加像服务器静态sharding，有时为了避免业务量突增导致的扩容需求，甚至
      被迫新开一个基于Twemproxy的Redis集群。

          Twemproxy的另一个痛点是，运维不够友好，甚至没有控制面板
</pre>


###Redis Cluster集群模式


###Codis

![](https://i.imgur.com/wsd4hZw.png)

<pre>
Codis
   
      Codis刚好击中Twemproxy的这两大痛点，并且提供诸多其他令人激赏的特性。

      Codis引入了Group的概念，每个Group包括1个Redis Master及至少1个Redis Slave，这是
      和Twemproxy的区别之一。这样做的好处是，如果当前Master有问题，则运维人员可通过Dashboard
      “自助式”切换到Slave，而不需要小心翼翼地修改程序配置文.

      为支持数据热迁移（Auto Rebalance），出品方修改了Redis Server源码，并称之为Codis Server。
  
      Codis采用预先分片（Pre-Sharding）机制，事先规定好了，分成1024个slots（也就是说，最多能
      支持后端1024个Codis Server），这些路由信息保存在ZooKeeper中。

      ZooKeeper还维护Codis Server Group信息，并提供分布式锁等服务。

      Codis目前仍被精益求精地改进中。其性能，从最初的比Twemproxy慢20%（虽然这对于内存型应用而
      言，并不明显），到现在远远超过Twemproxy性能（一定条件下）
</pre>