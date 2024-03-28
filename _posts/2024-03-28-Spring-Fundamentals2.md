---
title: Redis-Start
date: 2024-03-28 18:46:49
tags:
- redis
categories:
- database
---

---
typora-root-url: ./
---

> > 在此特别感谢黑马程序员提供的Redis课程

https://www.bilibili.com/video/BV1cr4y1671t/?vd_source=93b849a2e05b8dcf37cd770c34797c3c

## 初识Redis

Redis是一种键值型的NoSQL数据库，这里有两个关键字

+   键值型
+   NoSQL

其中`键值型`是指Redis中存储的数据都是以Key-Value键值对的形式存储，而Value的形式多种多样，可以是字符串、数值甚至Json

而NoSQL则是相对于传统关系型数据库而言，有很大差异的一种数据库

### 认识NoSQL

`NoSql`可以翻译做Not Only Sql（不仅仅是SQL），或者是No Sql（非Sql的）数据库。是相对于传统关系型数据库而言，有很大差异的一种特殊的数据库，因此也称之为`非关系型数据库`。

#### 结构化与非结构化

传统关系型数据库是结构化数据，每张表在创建的时候都有严格的约束信息，如字段名、字段数据类型、字段约束等，插入的数据必须遵循这些约束

而NoSQL则对数据库格式没有约束，可以是键值型，也可以是文档型，甚至是图格式

#### 关联与非关联

传统数据库的表与表之间往往存在关联，例如外键约束  
而非关系型数据库不存在关联关系，要维护关系要么靠代码中的业务逻辑，要么靠数据之间的耦合

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br></pre></td><td class="code"><pre><span class="line"><span class="punctuation">{</span></span><br><span class="line">  id<span class="punctuation">:</span> <span class="number">1</span><span class="punctuation">,</span></span><br><span class="line">  name<span class="punctuation">:</span> <span class="string">"张三"</span><span class="punctuation">,</span></span><br><span class="line">  orders<span class="punctuation">:</span> <span class="punctuation">[</span></span><br><span class="line">    <span class="punctuation">{</span></span><br><span class="line">       id<span class="punctuation">:</span> <span class="number">1</span><span class="punctuation">,</span></span><br><span class="line">       item<span class="punctuation">:</span> <span class="punctuation">{</span></span><br><span class="line">	 id<span class="punctuation">:</span> <span class="number">10</span><span class="punctuation">,</span> title<span class="punctuation">:</span> <span class="string">"荣耀6"</span><span class="punctuation">,</span> price<span class="punctuation">:</span> <span class="number">4999</span></span><br><span class="line">       <span class="punctuation">}</span></span><br><span class="line">    <span class="punctuation">}</span><span class="punctuation">,</span></span><br><span class="line">    <span class="punctuation">{</span></span><br><span class="line">       id<span class="punctuation">:</span> <span class="number">2</span><span class="punctuation">,</span></span><br><span class="line">       item<span class="punctuation">:</span> <span class="punctuation">{</span></span><br><span class="line">	 id<span class="punctuation">:</span> <span class="number">20</span><span class="punctuation">,</span> title<span class="punctuation">:</span> <span class="string">"小米11"</span><span class="punctuation">,</span> price<span class="punctuation">:</span> <span class="number">3999</span></span><br><span class="line">       <span class="punctuation">}</span></span><br><span class="line">    <span class="punctuation">}</span></span><br><span class="line">  <span class="punctuation">]</span></span><br><span class="line"><span class="punctuation">}</span></span><br></pre></td></tr></tbody></table>

例如此处要维护张三与两个手机订单的关系，不得不冗余的将这两个商品保存在张三的订单文档中，不够优雅，所以建议使用业务逻辑来维护关联关系

#### 查询方式

传统关系型数据库会基于Sql语句做查询，语法有统一的标准

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line"><span class="keyword">SELECT</span> id, age <span class="keyword">FROM</span> tb_user <span class="keyword">WHERE</span> id <span class="operator">=</span> <span class="number">1</span></span><br></pre></td></tr></tbody></table>

而不同的非关系型数据库查询语法差异极大

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">Redis:  get user:1</span><br><span class="line">MongoDB: db.user.find({_id: 1})</span><br><span class="line">elasticsearch:  GET http://localhost:9200/users/1</span><br></pre></td></tr></tbody></table>

#### 事务

传统关系型数据库能满足事务的ACID原则(原子性、一致性、独立性及持久性)  
而非关系型数据库汪汪不支持事务，或者不能要个保证ACID的特性，只能实现计本的一致性

#### 总结

|          | SQL                | NoSQL    |
| -------- | ------------------ | -------- |
| 数据结构 | 结构化(Structured) | 非结构化 |
| 数据关联 | 关联的(Relational) | 无关联的 |
| 查询方式 | SQL查询            | 非SQL    |
| 事务特性 | ACID               | BASE     |
| 存储方式 | 磁盘               | 内存     |
| 扩展性   | 垂直               | 水平     |
| 使用场景 | 1）数据结构固定<br />2)对一致性、安全性要求不高| 1)数据结构不固定 <br />2)相关业务对数据安全性、一致性要求较高<br />3）对性能要求|

+   存储方式
    +   关系型数据库基于磁盘进行存储，会有大量的磁盘IO，对性能有一定影响
    +   非关系型数据库，他们的操作更多的是依赖于内存来操作，内存的读写速度会非常快，性能自然会好一些

+   扩展性
    +   关系型数据库集群模式一般是主从，主从数据一致，起到数据备份的作用，称为垂直扩展。
    +   非关系型数据库可以将数据拆分，存储在不同机器上，可以保存海量数据，解决内存大小有限的问题。称为水平扩展。
    +   关系型数据库因为表之间存在关联关系，如果做水平扩展会给数据查询带来很多麻烦

### 认识Redis

Redis诞生于2009年，全称是Remote Dictionary Server远程词典服务器，是一个基于内存的键值型NoSQL数据库。

特征：

+   键值(Key-Value)型，Value支持多种不同的数据结构，功能丰富
+   单线程，每个命令具有原子性
+   低延迟，速度快(基于内存、IO多路复用、良好的编码)
+   支持数据持久化
+   支持主从集群、分片集群
+   支持多语言客户端

作者：Antirez

Redis官网：[https://redis.io/](https://redis.io/)

### 安装Redis

关于Redis的安装，我在之前的这篇文章做过详细的说明，这里就不赘述了

### Redis桌面客户端

安装完成Redis，我们就可以操作Redis，实现数据的CRUD了。这需要用到Redis客户端，包括：

+   命令行客户端
+   图形化桌面客户端
+   编程客户端

#### Redis命令行客户端

Redis安装完成后就自带了命令行客户端：redis-cli，使用方式如下：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">redis-cli [options] [commonds]</span><br></pre></td></tr></tbody></table>

其中常见的options有：

+   `-h 127.0.0.1`：指定要连接的redis节点的IP地址，默认是127.0.0.1
+   `-p 6379`：指定要连接的redis节点的端口，默认是6379
+   `-a 123321`：指定redis的访问密码

其中的commonds就是Redis的操作命令，例如：

+   `ping`：与redis服务端做心跳测试，服务端正常会返回\`pong

#### 图形化桌面客户端

安装包：[https://github.com/lework/RedisDesktopManager-Windows/releases](https://github.com/lework/RedisDesktopManager-Windows/releases)

Redis默认有16个仓库，编号从0至15. 通过配置文件可以设置仓库数量，但是不超过16，并且不能自定义仓库名称。

如果是基于redis-cli连接Redis服务，可以通过select命令来选择数据库：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line"><span class="comment">## 选择0号数据库</span></span><br><span class="line"><span class="keyword">select</span> 0</span><br></pre></td></tr></tbody></table>

## Redis常用命令

Redis是典型的key-value数据库，key一般是字符串，而value包含很多不同的数据类型  
![](https://pic1.imgdb.cn/item/63527eb616f2c2beb12beb85.jpg)

### Redis通用命令

常用的通用命令有以下几个

| 命令         | 描述                                                     |
| ------------ | -------------------------------------------------------- |
| KEYs pattern | 查找所有符合给定模式(pattern)的key                       |
| EXISTs key   | 检查给定key是否存在                                      |
| TYPE key     | 返回key所储存的值的类型                                  |
| TTL key      | 返回给定key的剩余生存时间(TTL, time to live)，以秒为单位 |
| DEL key      | 该命令用于在key存在是删除key                             |

+   `KEYS`：查看符合模板的所有key
    +   不建议在生产环境设备上使用，因为Redis是单线程的，执行查询的时候会阻塞其他命令，当数据量很大的时候，使用KEYS进行模糊查询，效率很差
+   `DEL`：删除一个指定的key
    +   也可以删除多个key，`DEL name age`，会将name和age都删掉
+   `EXISTS`：判断key是否存在
    +   `EXISTS name`，如果存在返回1，不存在返回0
+   `EXPIRE`：给一个key设置有效期，有效期到期时该key会被自动删除
    +   `EXPIRE name 20`，给name设置20秒有效期，到期自动删除
+   `TTL`：查看一个key的剩余有效期(Time-To-Live)
    +   `TTL name`，查看name的剩余有效期，如果未设置有效期，则返回-1

### String类型

String类型，也就是字符串类型，是Redis中最简单的存储类型  
其value是字符串，不过根据字符串的格式不同，又可以分为3类

+   `string`：普通字符串
+   `int`：整数类型，可以做自增、自减操作
+   `float`：浮点类型，可以做自增、自减操作  
    不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同，字符串类型的最大空间不能超过512M

#### String的常用命令

String的常用命令有

| 命令        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| SET         | 添加或者修改一个已经存在的String类型的键值对                 |
| GET         | 根据key获取String类型的value                                 |
| MEST        | 批量添加多个String类型的键值对                               |
| MGET        | 根据多个key获取多个String类型的value                         |
| INCR        | 让一个整形的key自增1                                         |
| INCRBY      | 让一个整形的key自增并指定步长值，例如：incrby num 2，让num值自增2 |
| INCRBYFLOAT | 让一个浮点类型的数字自增并指定步长值                         |
| SETNX       | 添加一个String类型的键值对，前提是这个key不存在，否则不执行，可以理解为真正的`新`增 |
| SETEX       | 添加一个String类型的键值对，并指定有效期                     |

#### Key结构

+   Redis没有类似MySQL中Table的概念，那么我们该如何区分不同类型的Key呢？
+   例如：需要存储用户、商品信息到Redis，有一个用户的id是1，有一个商品的id恰好也是1，如果此时使用id作为key，那么就回冲突，该怎么办？
+   我们可以通过给key添加前缀加以区分，不过这个前缀不是随便加的，有一定的规范
    
    +   Redis的key允许有多个单词形成层级结构，多个单词之间用`:`隔开，格式如下
    
    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">项目名:业务名:类型:id</span><br></pre></td></tr></tbody></table>
    
    +   这个格式也并非是固定的，可以根据自己的需求来删除/添加词条，这样我们就可以把不同数据类型的数据区分开了，从而避免了key的冲突问题
    +   例如我们的项目名叫reggie，有user和dish两种不同类型的数据，我们可以这样定义key
        +   user相关的key：`reggie:user:1`
        +   dish相关的key：`reggie:dish:1`
+   如果value是一个Java对象，例如一个User对象，则可以将对象序列化为JSON字符串后存储

| KEY           | VALUE                                       |
| ------------- | ------------------------------------------- |
| reggie:user:1 | {“id”:1, “name”: “Jack”, “age”: 21}         |
| reggie:dish:1 | {“id”:1, “name”: “鲟鱼火锅”, “price”: 4999} |

+   并且在Redis的桌面客户端中，也会以相同前缀作为层次结构，让数据看起来层次分明，关系清晰

### Hash类型

+   Hash类型，也叫散列，其中value是一个无序字典，类似于Java中的HashMap结构
+   String结构是将对象序列化为JSON字符串后存储，当我们要修改对象的某个属性值的时候很不方便

| KEY           | VALUE                                       |
| ------------- | ------------------------------------------- |
| reggie:user:1 | {“id”:1, “name”: “Jack”, “age”: 21}         |
| reggie:dish:1 | {“id”:1, “name”: “鲟鱼火锅”, “price”: 4999} |

+   Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD

<table><tbody><tr><td rowspan="2">KEY</td><td colspan="2">VALUE</td></tr><tr><td>field</td><td>value</td></tr><tr><td rowspan="2">reggie:user:1</td><td>name</td><td>Jack</td></tr><tr><td>age&nbsp;</td><td>21</td></tr><tr><td rowspan="2"><br>reggie:user:2</td><td>name&nbsp;</td><td>Rose</td></tr><tr><td>age&nbsp;</td><td>18</td></tr></tbody><colgroup><col> <col> <col></colgroup></table>

+   Hash的常用命令有

| 命令                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| HSET key field value | 添加或者修改hash类型key的field的值                           |
| HGET key field       | 获取一个hash类型key的field的值                               |
| HMSET                | 批量添加多个hash类型key的field的值                           |
| HMGET                | 批量获取多个hash类型key的field的值                           |
| HGETALL              | 获取一个hash类型的key中的所有的field和value                  |
| HKEYS                | 获取一个hash类型的key中的所有的field                         |
| HINCRBY              | 让一个hash类型key的字段值自增并指定步长                      |
| HSETNX               | 添加一个hash类型的key的field值，前提是这个field不存在，否则不执行 |

### List类型

+   Redis中的List类型与Java中的LinkedList类似，可以看做是一个双向链表结构。既可以支持正向检索和也可以支持反向检索。
    
+   特征也与LinkedList类似：
    
    +   有序
    +   元素可以重复
    +   插入和删除快
    +   查询速度一般
+   常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。
    
+   List的常见命令有：
    

| 命令                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| LPUSH key element … | 向列表左侧插入一个或多个元素                                 |
| LPOP key            | 移除并返回列表左侧的第一个元素，没有则返回nil                |
| RPUSH key element … | 向列表右侧插入一个或多个元素                                 |
| RPOP key            | 移除并返回列表右侧的第一个元素                               |
| LRANGE key star end | 返回一段角标范围内的所有元素                                 |
| BLPOP和BRPOP        | 与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil |

### Set类型

+   Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征：
    +   无序
    +   元素不可重复
    +   查找快
    +   支持交集、并集、差集等功能
+   Set的常见命令有：

| 命令                 | 描述                        |
| -------------------- | --------------------------- |
| SADD key member …    | 向set中添加一个或多个元素   |
| SREM key member …    | 移除set中的指定元素         |
| SCARD key            | 返回set中元素的个数         |
| SISMEMBER key member | 判断一个元素是否存在于set中 |
| SMEMBERS             | 获取set中的所有元素         |
| SINTER key1 key2 …   | 求key1与key2的交集          |
| SUNION key1 key2 …   | 求key1与key2的并集          |
| SDIFF key1 key2 …    | 求key1与key2的差集          |

练习题：

1.  将下列数据用Redis的Set集合来存储：

+   张三的好友有：李四、王五、赵六

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; sadd zhangsan lisi wangwu zhaoliu</span><br><span class="line">(<span class="built_in">integer</span>) 3</span><br></pre></td></tr></tbody></table>

+   李四的好友有：王五、麻子、二狗

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; sadd lisi wangwu mazi ergou</span><br><span class="line">(<span class="built_in">integer</span>) 3</span><br></pre></td></tr></tbody></table>

2.  利用Set的命令实现下列功能：

+   计算张三的好友有几人

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; scard zhangsan</span><br><span class="line">(<span class="built_in">integer</span>) 3</span><br></pre></td></tr></tbody></table>

+   计算张三和李四有哪些共同好友

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; sinter zhangsan lisi</span><br><span class="line">1) <span class="string">"wangwu"</span></span><br></pre></td></tr></tbody></table>

+   查询哪些人是张三的好友却不是李四的好友

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; sdiff zhangsan lisi</span><br><span class="line">1) <span class="string">"zhaoliu"</span></span><br><span class="line">2) <span class="string">"lisi"</span></span><br></pre></td></tr></tbody></table>

+   查询张三和李四的好友总共有哪些人

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; sunion zhangsan lisi</span><br><span class="line">1) <span class="string">"wangwu"</span></span><br><span class="line">2) <span class="string">"zhaoliu"</span></span><br><span class="line">3) <span class="string">"ergou"</span></span><br><span class="line">4) <span class="string">"lisi"</span></span><br><span class="line">5) <span class="string">"mazi"</span></span><br></pre></td></tr></tbody></table>

+   判断李四是否是张三的好友

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; sismember zhangsan lisi</span><br><span class="line">(<span class="built_in">integer</span>) 1</span><br></pre></td></tr></tbody></table>

+   判断张三是否是李四的好友

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; sismember lisi zhangsan</span><br><span class="line">(<span class="built_in">integer</span>) 0</span><br></pre></td></tr></tbody></table>

+   将李四从张三的好友列表中移除

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; srem zhangsan lisi</span><br><span class="line">(<span class="built_in">integer</span>) 1</span><br></pre></td></tr></tbody></table>

### SortedSet类型

+   Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。
+   SortedSet具备下列特性：
    +   可排序
    +   元素不重复
    +   查询速度快
+   因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。
+   SortedSet的常见命令有：

| 命令                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| ZADD key score member        | 添加一个或多个元素到sorted set ，如果已经存在则更新其score值 |
| ZREM key member              | 删除sorted set中的一个指定元素                               |
| ZSCORE key member            | 获取sorted set中的指定元素的score值                          |
| ZRANK key member             | 获取sorted set 中的指定元素的排名                            |
| ZCARD key                    | 获取sorted set中的元素个数                                   |
| ZCOUNT key min max           | 统计score值在给定范围内的所有元素的个数                      |
| ZINCRBY key increment member | 让sorted set中的指定元素自增，步长为指定的increment值        |
| ZRANGE key min max           | 按照score排序后，获取指定排名范围内的元素                    |
| ZRANGEBYSCORE key min max    | 按照score排序后，获取指定score范围内的元素                   |
| ZDIFF、ZINTER、ZUNION        | 求差集、交集、并集                                           |

注意：所有的排名默认都是升序，如果要降序则在命令的Z后面添加REV即可，例如：

+   `升序`获取sorted set 中的指定元素的排名：ZRANK key member
+   `降序`获取sorted set 中的指定元素的排名：ZREVRANK key memeber

+   练习题：
    
    +   将班级的下列学生得分存入Redis的SortedSet中：
    +   Jack 85, Lucy 89, Rose 82, Tom 95, Jerry 78, Amy 92, Miles 76
    
    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zadd stu 85 Jack 89 Lucy 82 Rose 95 Tom 78 Jerry 92 Amy 76 Miles</span><br><span class="line">(<span class="built_in">integer</span>) 7</span><br></pre></td></tr></tbody></table>
    
    +   并实现下列功能：
        
        +   删除Tom同学
        
        <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zrem stu Tom</span><br><span class="line">(<span class="built_in">integer</span>) 1</span><br></pre></td></tr></tbody></table>
        
        +   获取Amy同学的分数
        
        <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zscore stu Amy</span><br><span class="line"><span class="string">"92"</span></span><br></pre></td></tr></tbody></table>
        
        +   获取Rose同学的排名
        
        <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zrank stu Rose</span><br><span class="line">(<span class="built_in">integer</span>) 2</span><br></pre></td></tr></tbody></table>
        
        +   查询80分以下有几个学生
        
        <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zcount stu 0 80</span><br><span class="line">(<span class="built_in">integer</span>) 2</span><br></pre></td></tr></tbody></table>
        
        +   给Amy同学加2分
        
        <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zincrby stu 2 Amy</span><br><span class="line"><span class="string">"94"</span></span><br></pre></td></tr></tbody></table>
        
        +   查出成绩前3名的同学
        
        <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zrange stu 0 2</span><br><span class="line">1) <span class="string">"Miles"</span></span><br><span class="line">2) <span class="string">"Jerry"</span></span><br><span class="line">3) <span class="string">"Rose"</span></span><br></pre></td></tr></tbody></table>
        
        +   查出成绩80分以下的所有同学
        
        <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">127.0.0.1:6379&gt; zrangebyscore stu 0 80</span><br><span class="line">1) <span class="string">"Miles"</span></span><br><span class="line">2) <span class="string">"Jerry"</span></span><br></pre></td></tr></tbody></table>
        

## Redis的Java客户端

+   目前主流的Redis的Java客户端有三种
    +   Jedis和Lettuce：这两个主要是提供了Redis命令对应的API，方便我们操作Redis，而SpringDataRedis又对这两种做了抽象和封装，因此我们后期会直接以SpringDataRedis来学习。
    +   Redisson：是在Redis基础上实现了分布式的可伸缩的java数据结构，例如Map、Queue等，而且支持跨进程的同步机制：Lock、Semaphore等待，比较适合用来实现特殊的功能需求。

### Jedis客户端

#### 快速入门

+   使用Jedis的步骤
    
    1.  导入Jedis的maven坐标
    
    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br></pre></td><td class="code"><pre><span class="line"><span class="comment">&lt;!--jedis--&gt;</span></span><br><span class="line"><span class="tag">&lt;<span class="name">dependency</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">groupId</span>&gt;</span>redis.clients<span class="tag">&lt;/<span class="name">groupId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">artifactId</span>&gt;</span>jedis<span class="tag">&lt;/<span class="name">artifactId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">version</span>&gt;</span>3.7.0<span class="tag">&lt;/<span class="name">version</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">dependency</span>&gt;</span></span><br><span class="line"><span class="comment">&lt;!--单元测试--&gt;</span></span><br><span class="line"><span class="tag">&lt;<span class="name">dependency</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">groupId</span>&gt;</span>org.junit.jupiter<span class="tag">&lt;/<span class="name">groupId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">artifactId</span>&gt;</span>junit-jupiter<span class="tag">&lt;/<span class="name">artifactId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">version</span>&gt;</span>5.7.0<span class="tag">&lt;/<span class="name">version</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">scope</span>&gt;</span>test<span class="tag">&lt;/<span class="name">scope</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">dependency</span>&gt;</span></span><br></pre></td></tr></tbody></table>
    
    2.  建立连接  
        新建一个单元测试类
    
    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br></pre></td><td class="code"><pre><span class="line"><span class="keyword">private</span> Jedis jedis;</span><br><span class="line"></span><br><span class="line"><span class="meta">@BeforeEach</span></span><br><span class="line"><span class="keyword">void</span> <span class="title function_">setUp</span><span class="params">()</span> {</span><br><span class="line">    <span class="comment">//1. 建立连接</span></span><br><span class="line">    jedis = <span class="keyword">new</span> <span class="title class_">Jedis</span>(<span class="string">"101.42.225.160"</span>, <span class="number">6379</span>);</span><br><span class="line">    <span class="comment">//2. 设置密码</span></span><br><span class="line">    jedis.auth(<span class="string">"root"</span>);</span><br><span class="line">    <span class="comment">//3. 选择库</span></span><br><span class="line">    jedis.select(<span class="number">0</span>);</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>
    
    3.  测试
    
    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@Test</span></span><br><span class="line"><span class="keyword">void</span> <span class="title function_">testString</span><span class="params">()</span>{</span><br><span class="line">    jedis.set(<span class="string">"name"</span>,<span class="string">"Kyle"</span>);</span><br><span class="line">    <span class="type">String</span> <span class="variable">name</span> <span class="operator">=</span> jedis.get(<span class="string">"name"</span>);</span><br><span class="line">    System.out.println(<span class="string">"name = "</span> + name);</span><br><span class="line">}</span><br><span class="line"></span><br><span class="line"><span class="meta">@Test</span></span><br><span class="line"><span class="keyword">void</span> <span class="title function_">testHash</span><span class="params">()</span>{</span><br><span class="line">    jedis.hset(<span class="string">"reggie:user:1"</span>,<span class="string">"name"</span>,<span class="string">"Jack"</span>);</span><br><span class="line">    jedis.hset(<span class="string">"reggie:user:2"</span>,<span class="string">"name"</span>,<span class="string">"Rose"</span>);</span><br><span class="line">    jedis.hset(<span class="string">"reggie:user:1"</span>,<span class="string">"age"</span>,<span class="string">"21"</span>);</span><br><span class="line">    jedis.hset(<span class="string">"reggie:user:2"</span>,<span class="string">"age"</span>,<span class="string">"18"</span>);</span><br><span class="line">    Map&lt;String, String&gt; map = jedis.hgetAll(<span class="string">"reggie:user:1"</span>);</span><br><span class="line">    System.out.println(map);</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>
    
    4.  释放资源
    
    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@AfterEach</span></span><br><span class="line"><span class="keyword">void</span> <span class="title function_">tearDown</span><span class="params">()</span>{</span><br><span class="line">    <span class="keyword">if</span> (jedis != <span class="literal">null</span>){</span><br><span class="line">        jedis.close();</span><br><span class="line">    }</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>
    

#### 连接池

+   `Jedis`本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，因此我们推荐大家使用Jedis连接池代替Jedis的直连方式。
+   新建一个`com.blog.util`，用于存放我们编写的工具类
+   但后面我们使用`SpringDataRedis`的时候，可以直接在`yml`配置文件里配置这些内容

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br></pre></td><td class="code"><pre><span class="line"><span class="keyword">public</span> <span class="keyword">class</span> <span class="title class_">JedisConnectionFactory</span> {</span><br><span class="line"></span><br><span class="line">    <span class="keyword">private</span> <span class="keyword">static</span> JedisPool jedisPool;</span><br><span class="line"></span><br><span class="line">    <span class="keyword">static</span> {</span><br><span class="line">        <span class="comment">// 配置连接池</span></span><br><span class="line">        <span class="type">JedisPoolConfig</span> <span class="variable">poolConfig</span> <span class="operator">=</span> <span class="keyword">new</span> <span class="title class_">JedisPoolConfig</span>();</span><br><span class="line">        poolConfig.setMaxTotal(<span class="number">8</span>);</span><br><span class="line">        poolConfig.setMaxIdle(<span class="number">8</span>);</span><br><span class="line">        poolConfig.setMinIdle(<span class="number">0</span>);</span><br><span class="line">        poolConfig.setMaxWaitMillis(<span class="number">1000</span>);</span><br><span class="line">        <span class="comment">// 创建连接池对象，参数：连接池配置、服务端ip、服务端端口、超时时间、密码</span></span><br><span class="line">        jedisPool = <span class="keyword">new</span> <span class="title class_">JedisPool</span>(poolConfig, <span class="string">"101.42.225.160"</span>, <span class="number">6379</span>, <span class="number">1000</span>, <span class="string">"root"</span>);</span><br><span class="line">    }</span><br><span class="line"></span><br><span class="line">    <span class="keyword">public</span> <span class="keyword">static</span> Jedis <span class="title function_">getJedis</span><span class="params">()</span>{</span><br><span class="line">        <span class="keyword">return</span> jedisPool.getResource();</span><br><span class="line">    }</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

+   之后我们的测试类就可以修改为如下

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br><span class="line">22</span><br><span class="line">23</span><br><span class="line">24</span><br><span class="line">25</span><br><span class="line">26</span><br><span class="line">27</span><br><span class="line">28</span><br><span class="line">29</span><br><span class="line">30</span><br><span class="line">31</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@SpringBootTest</span></span><br><span class="line"><span class="keyword">class</span> <span class="title class_">RedisTestApplicationTests</span> {</span><br><span class="line"></span><br><span class="line">    <span class="keyword">private</span> <span class="type">Jedis</span> <span class="variable">jedis</span> <span class="operator">=</span> JedisConnectionFactory.getJedis();</span><br><span class="line"></span><br><span class="line">    <span class="meta">@Test</span></span><br><span class="line">    <span class="keyword">void</span> <span class="title function_">testString</span><span class="params">()</span>{</span><br><span class="line">        jedis.set(<span class="string">"name"</span>,<span class="string">"Kyle"</span>);</span><br><span class="line">        <span class="type">String</span> <span class="variable">name</span> <span class="operator">=</span> jedis.get(<span class="string">"name"</span>);</span><br><span class="line">        System.out.println(<span class="string">"name = "</span> + name);</span><br><span class="line">    }</span><br><span class="line"></span><br><span class="line">    <span class="meta">@Test</span></span><br><span class="line">    <span class="keyword">void</span> <span class="title function_">testHash</span><span class="params">()</span>{</span><br><span class="line">        jedis.hset(<span class="string">"reggie:user:1"</span>,<span class="string">"name"</span>,<span class="string">"Jack"</span>);</span><br><span class="line">        jedis.hset(<span class="string">"reggie:user:2"</span>,<span class="string">"name"</span>,<span class="string">"Rose"</span>);</span><br><span class="line">        jedis.hset(<span class="string">"reggie:user:3"</span>,<span class="string">"name"</span>,<span class="string">"Kyle"</span>);</span><br><span class="line">        jedis.hset(<span class="string">"reggie:user:1"</span>,<span class="string">"age"</span>,<span class="string">"21"</span>);</span><br><span class="line">        jedis.hset(<span class="string">"reggie:user:2"</span>,<span class="string">"age"</span>,<span class="string">"18"</span>);</span><br><span class="line">        jedis.hset(<span class="string">"reggie:user:3"</span>,<span class="string">"age"</span>,<span class="string">"18"</span>);</span><br><span class="line">        Map&lt;String, String&gt; map = jedis.hgetAll(<span class="string">"reggie:user:1"</span>);</span><br><span class="line">        System.out.println(map);</span><br><span class="line">    }</span><br><span class="line"></span><br><span class="line">    <span class="meta">@AfterEach</span></span><br><span class="line">    <span class="keyword">void</span> <span class="title function_">tearDown</span><span class="params">()</span>{</span><br><span class="line">        <span class="keyword">if</span> (jedis != <span class="literal">null</span>){</span><br><span class="line">            jedis.close();</span><br><span class="line">        }</span><br><span class="line">    }</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

### SpringDataRedis客户端

+   SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis
    
+   官网地址：[https://spring.io/projects/spring-data-redis](https://spring.io/projects/spring-data-redis)
    
    +   提供了对不同Redis客户端的整合（Lettuce和Jedis）
    +   提供了RedisTemplate统一API来操作Redis
    +   支持Redis的发布订阅模型
    +   支持Redis哨兵和Redis集群
    +   支持基于Lettuce的响应式编程
    +   支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
    +   支持基于Redis的JDKCollection实现
+   SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：
    

| API                         | 返回值类型      | 说明                  |
| --------------------------- | --------------- | --------------------- |
| redisTemplate.opsForValue() | ValueOperations | 操作String类型数据    |
| redisTemplate.opsForHash()  | HashOperations  | 操作Hash类型数据      |
| redisTemplate.opsForList()  | ListOperations  | 操作List类型数据      |
| redisTemplate.opsForSet()   | SetOperations   | 操作Set类型数据       |
| redisTemplate.opsForzSet()  | ZSetOperations  | 操作SortedSet类型数据 |
| redisTemplate               |                 | 通用的命令            |

#### 快速入门

SpringBoot已经提供了对SpringDataRedis的支持，使用起来非常简单

1.  导入依赖，

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br><span class="line">22</span><br><span class="line">23</span><br><span class="line">24</span><br><span class="line">25</span><br><span class="line">26</span><br></pre></td><td class="code"><pre><span class="line"><span class="comment">&lt;!--redis依赖--&gt;</span></span><br><span class="line"><span class="tag">&lt;<span class="name">dependency</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">groupId</span>&gt;</span>org.springframework.boot<span class="tag">&lt;/<span class="name">groupId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">artifactId</span>&gt;</span>spring-boot-starter-data-redis<span class="tag">&lt;/<span class="name">artifactId</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">dependency</span>&gt;</span></span><br><span class="line"><span class="comment">&lt;!--common-pool--&gt;</span></span><br><span class="line"><span class="tag">&lt;<span class="name">dependency</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">groupId</span>&gt;</span>org.apache.commons<span class="tag">&lt;/<span class="name">groupId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">artifactId</span>&gt;</span>commons-pool2<span class="tag">&lt;/<span class="name">artifactId</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">dependency</span>&gt;</span></span><br><span class="line"><span class="comment">&lt;!--Jackson依赖--&gt;</span></span><br><span class="line"><span class="tag">&lt;<span class="name">dependency</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">groupId</span>&gt;</span>com.fasterxml.jackson.core<span class="tag">&lt;/<span class="name">groupId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">artifactId</span>&gt;</span>jackson-databind<span class="tag">&lt;/<span class="name">artifactId</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">dependency</span>&gt;</span></span><br><span class="line"><span class="comment">&lt;!--lombok--&gt;</span></span><br><span class="line"><span class="tag">&lt;<span class="name">dependency</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">groupId</span>&gt;</span>org.projectlombok<span class="tag">&lt;/<span class="name">groupId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">artifactId</span>&gt;</span>lombok<span class="tag">&lt;/<span class="name">artifactId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">optional</span>&gt;</span>true<span class="tag">&lt;/<span class="name">optional</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">dependency</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;<span class="name">dependency</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">groupId</span>&gt;</span>org.springframework.boot<span class="tag">&lt;/<span class="name">groupId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">artifactId</span>&gt;</span>spring-boot-starter-test<span class="tag">&lt;/<span class="name">artifactId</span>&gt;</span></span><br><span class="line">    <span class="tag">&lt;<span class="name">scope</span>&gt;</span>test<span class="tag">&lt;/<span class="name">scope</span>&gt;</span></span><br><span class="line"><span class="tag">&lt;/<span class="name">dependency</span>&gt;</span></span><br></pre></td></tr></tbody></table>

2.  配置Redis

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br></pre></td><td class="code"><pre><span class="line"><span class="attr">spring:</span></span><br><span class="line">  <span class="attr">redis:</span></span><br><span class="line">    <span class="attr">host:</span> <span class="number">101.42</span><span class="number">.225</span><span class="number">.160</span></span><br><span class="line">    <span class="attr">port:</span> <span class="number">6379</span></span><br><span class="line">    <span class="attr">password:</span> <span class="string">root</span></span><br><span class="line">    <span class="attr">lettuce:</span></span><br><span class="line">      <span class="attr">pool:</span></span><br><span class="line">        <span class="attr">max-active:</span> <span class="number">8</span></span><br><span class="line">        <span class="attr">max-idle:</span> <span class="number">8</span></span><br><span class="line">        <span class="attr">min-idle:</span> <span class="number">0</span></span><br><span class="line">        <span class="attr">max-wait:</span> <span class="string">100ms</span></span><br></pre></td></tr></tbody></table>

3.  注入RedisTemplate  
    因为有了SpringBoot的自动装配，我们可以拿来就用

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@Autowired</span></span><br><span class="line"><span class="keyword">private</span> RedisTemplate redisTemplate;</span><br></pre></td></tr></tbody></table>

4.  编写测试方法

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@Test</span></span><br><span class="line"><span class="keyword">void</span> <span class="title function_">stringTest</span><span class="params">()</span>{</span><br><span class="line">    redisTemplate.opsForValue().set(<span class="string">"username"</span>,<span class="string">"David"</span>);</span><br><span class="line">    <span class="type">String</span> <span class="variable">username</span> <span class="operator">=</span> (String) redisTemplate.opsForValue().get(<span class="string">"username"</span>);</span><br><span class="line">    System.out.println(username);</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

#### 自定义序列化

+   RedisTemplate可以接收任意Object作为值写入Redis
+   只不过写入前会把Object序列化为字节形式，默认是采用JDK序列化，得到的结果是这样的

> \\xAC\\xED\\x00\\x05t\\x00\\x06\\xE5\\xBC\\xA0\\xE4\\xB8\\x89

+   缺点：
    
    +   可读性差
    +   内存占用较大
+   我们可以自定义RedisTemplate的序列化方式，代码如下  
    在`com.blog.config`包下编写对应的配置类
    

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br><span class="line">22</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@Configuration</span></span><br><span class="line"><span class="keyword">public</span> <span class="keyword">class</span> <span class="title class_">RedisConfig</span> {</span><br><span class="line"></span><br><span class="line">    <span class="meta">@Bean</span></span><br><span class="line">    <span class="keyword">public</span> RedisTemplate&lt;String, Object&gt; <span class="title function_">redisTemplate</span><span class="params">(RedisConnectionFactory connectionFactory)</span> {</span><br><span class="line">        <span class="comment">// 创建RedisTemplate对象</span></span><br><span class="line">        RedisTemplate&lt;String, Object&gt; template = <span class="keyword">new</span> <span class="title class_">RedisTemplate</span>&lt;&gt;();</span><br><span class="line">        <span class="comment">// 设置连接工厂</span></span><br><span class="line">        template.setConnectionFactory(connectionFactory);</span><br><span class="line">        <span class="comment">// 创建JSON序列化工具</span></span><br><span class="line">        <span class="type">GenericJackson2JsonRedisSerializer</span> <span class="variable">jsonRedisSerializer</span> <span class="operator">=</span></span><br><span class="line">                <span class="keyword">new</span> <span class="title class_">GenericJackson2JsonRedisSerializer</span>();</span><br><span class="line">        <span class="comment">// 设置Key的序列化</span></span><br><span class="line">        template.setKeySerializer(RedisSerializer.string());</span><br><span class="line">        template.setHashKeySerializer(RedisSerializer.string());</span><br><span class="line">        <span class="comment">// 设置Value的序列化</span></span><br><span class="line">        template.setValueSerializer(jsonRedisSerializer);</span><br><span class="line">        template.setHashValueSerializer(jsonRedisSerializer);</span><br><span class="line">        <span class="comment">// 返回</span></span><br><span class="line">        <span class="keyword">return</span> template;</span><br><span class="line">    }</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

+   我们编写一个User类，并尝试将其创建的对象存入Redis，看看是什么效果

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@Data</span></span><br><span class="line"><span class="meta">@AllArgsConstructor</span></span><br><span class="line"><span class="meta">@NoArgsConstructor</span></span><br><span class="line"><span class="keyword">public</span> <span class="keyword">class</span> <span class="title class_">User</span> {</span><br><span class="line">    <span class="keyword">private</span> String name;</span><br><span class="line">    <span class="keyword">private</span> Integer age;</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

+   测试方法

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@Test</span></span><br><span class="line"><span class="keyword">void</span> <span class="title function_">stringTest</span><span class="params">()</span>{</span><br><span class="line">    redisTemplate.opsForValue().set(<span class="string">"userdata"</span>,<span class="keyword">new</span> <span class="title class_">User</span>(<span class="string">"张三"</span>,<span class="number">18</span>));</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

+   这里采用了JSON序列化来代替默认的JDK序列化方式。最终结果如下：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br></pre></td><td class="code"><pre><span class="line"><span class="punctuation">{</span></span><br><span class="line">  <span class="attr">"@class"</span><span class="punctuation">:</span> <span class="string">"com.blog.entity.User"</span><span class="punctuation">,</span></span><br><span class="line">  <span class="attr">"name"</span><span class="punctuation">:</span> <span class="string">"张三"</span><span class="punctuation">,</span></span><br><span class="line">  <span class="attr">"age"</span><span class="punctuation">:</span> <span class="number">18</span></span><br><span class="line"><span class="punctuation">}</span></span><br></pre></td></tr></tbody></table>

+   整体可读性有了很大提升，并且能将Java对象自动的序列化为JSON字符串，并且查询时能自动把JSON反序列化为Java对象。不过，其中记录了序列化时对应的class名称，目的是为了查询时实现自动反序列化。这会带来额外的内存开销。
+   所以肯定会有更好的方法

#### StringRedisTemplate

+   为了节省内存空间，我们可以不使用JSON序列化器来处理value，而是统一使用String序列化器，要求只能存储String类型的key和value。当需要存储Java对象时，手动完成对象的序列化和反序列化。
+   因为存入和读取时的序列化及反序列化都是我们自己实现的，SpringDataRedis就不会将class信息写入Redis了
+   这种用法比较普遍，因此SpringDataRedis就提供了RedisTemplate的子类：StringRedisTemplate，它的key和value的序列化方式默认就是String方式。源码如下

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br></pre></td><td class="code"><pre><span class="line"><span class="keyword">public</span> <span class="keyword">class</span> <span class="title class_">StringRedisTemplate</span> <span class="keyword">extends</span> <span class="title class_">RedisTemplate</span>&lt;String, String&gt; {</span><br><span class="line">    <span class="keyword">public</span> <span class="title function_">StringRedisTemplate</span><span class="params">()</span> {</span><br><span class="line">        <span class="built_in">this</span>.setKeySerializer(RedisSerializer.string());</span><br><span class="line">        <span class="built_in">this</span>.setValueSerializer(RedisSerializer.string());</span><br><span class="line">        <span class="built_in">this</span>.setHashKeySerializer(RedisSerializer.string());</span><br><span class="line">        <span class="built_in">this</span>.setHashValueSerializer(RedisSerializer.string());</span><br><span class="line">    }</span><br></pre></td></tr></tbody></table>

+   省去了我们自定义RedisTemplate的序列化方式的步骤（可以将之前配置的RedisConfig删除掉），而是直接使用：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">@Test</span></span><br><span class="line"><span class="keyword">void</span> <span class="title function_">stringTest</span><span class="params">()</span> <span class="keyword">throws</span> JsonProcessingException {</span><br><span class="line">    <span class="comment">//创建对象</span></span><br><span class="line">    <span class="type">User</span> <span class="variable">user</span> <span class="operator">=</span> <span class="keyword">new</span> <span class="title class_">User</span>(<span class="string">"张三"</span>, <span class="number">18</span>);</span><br><span class="line">    <span class="comment">//手动序列化</span></span><br><span class="line">    <span class="type">String</span> <span class="variable">json</span> <span class="operator">=</span> mapper.writeValueAsString(user);</span><br><span class="line">    <span class="comment">//写入数据</span></span><br><span class="line">    stringRedisTemplate.opsForValue().set(<span class="string">"userdata"</span>, json);</span><br><span class="line">    <span class="comment">//获取数据</span></span><br><span class="line">    <span class="type">String</span> <span class="variable">userdata</span> <span class="operator">=</span> stringRedisTemplate.opsForValue().get(<span class="string">"userdata"</span>);</span><br><span class="line">    <span class="comment">//手动反序列化</span></span><br><span class="line">    <span class="type">User</span> <span class="variable">readValue</span> <span class="operator">=</span> mapper.readValue(userdata, User.class);</span><br><span class="line">    System.out.println(readValue);</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

+   存入Redis中是这样的

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line"><span class="punctuation">{</span></span><br><span class="line">  <span class="attr">"name"</span><span class="punctuation">:</span> <span class="string">"张三"</span><span class="punctuation">,</span></span><br><span class="line">  <span class="attr">"age"</span><span class="punctuation">:</span> <span class="number">18</span></span><br><span class="line"><span class="punctuation">}</span></span><br></pre></td></tr></tbody></table>