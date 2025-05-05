# Redis

---

## Redis数据类型
参考：https://redis.com.cn/commands.html

---

### BitMap

---

#### 介绍
BitMap （位图）的底层数据结构使用的是String类型的的 SDS 数据结构来保存。因为一个字节8个bit位，为了有效的将字节的8个bit都利用到位，使用数组模式存储。  
并且每个bit都使用二值状态表示，要么0，要么1。  
所以，BitMap 是通过一个 bit 位来表示某个元素对应的值或者状态, 它的结构如下，key 对应元素本身；offset即是偏移量，固定整型，一般存数组下表或者唯一值；value存储的是二值（要么0要么1），一般用来表示状态，如性别、是否登录、是否打卡等。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-13/44915092246083.png?Expires=4898109516&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=NhXyIKHa6JEYc3eDxainZy%2FgbJQ%3D)

从上面可以看出这边使用一个字节表示1行，每1行存储8个bit，就是可以存储8个状态位，极大的提高了空间利用。这也是BitMap的优势，我们可以使用很少的字节，存储大量的在线状态、打卡标记等状态信息，非常有效果。

---

#### 使用
我们可以使用 setbit, getbit, bitcount 等几个相关命令来管理BitMap。语法如下：

```redis
SETBIT key offset value
```
* `key` 是要操作的键名。
* `offset` 是位偏移量。
* `value` 是要设置的位值（0 或 1）。

上面说过了，key是元素名称， offset 必须是数值类型，value 只能是 0 或者 1，如果我们存储一个用户的在线状态，用户，代码如下：

```redis
//设置在线状态 
// setBit('online_statu', $uid, 1);

SETBIT online_statu 5 1;
SETBIT online_statu 9 1;
```

| byte   | bit0 | bit1 | bit2 | bit3 | bit4 | bit5 | bit6 | bit7 |
|--------|------|------|------|------|------|------|------|------|
| buf[0] | 0    | 0    | 0    | 0    | 0    | 1    | 0    | 0    |
| buf[1] | 0    | 1    | 0    | 0    | 0    | 0    | 0    | 0    |

可以看出用户ID为5和9被打上1的标志，代表在线状态，其他未设置值默认为0，是离线状态。
除了Set之外，还有getBit、bitCount等语法，如下：

```redis
// 获取是否在线的状态 
getbit online_statu 9; 
 
// 获取在线人数 统计
bitcount online_statu
```

---

#### 主要应用场景

---

##### 状态统计
我们以用户 离线/在线 为例子，看看如何使用 Bitmap 在海量的用户数据之中判断某个用户是否是在线状态。
假设我们使用一个 `online_statu` 来作为key，用来存储 用户登录后的状态集合，而用户的ID则为offset，online的状态就用1表示，offline的状态就用0表示。

如果1024用户登录系统，那么设置ID为1024的用户为在线的代码如下：
```redis
SETBIT online_statu 1024 1
```
如果想看1024的用户是否是在线状态（这边注意，key可能不存在，代表没有这个用户，这时候默认返回0），代码如下：
```redis
GETBIT online_statu 1024
```
如果1024的用户退出系统，则为他执行下线，代码如下：
```redis
SETBIT online_statu 1024 0
```
统计当前多少人在线
```redis
bitcount online_statu
```
空间上的有效利用，1亿 人的状态存储只需要 100000000/8/1024/1024 = 11.92 M，简单的数据结构也保证了性能上的优势。
> 基于上面的讨论，我们可以总结出一个预评估公式，根据实际的数据量获取存储空间：( offset / 8 / 1024 / 1024 ) M

---

##### 固定周期的签到情况统计（周/月/年）
固定周期可能是年/月/周，按照不同维度，可能有 365，31，7的bit位的统计周期。
假设这时候我们如果对于某个用户（如1024）全年的签到情况做统计，可以这么设计:

设计key 为 {bus_type}{uid} ，及业务类型+用户id+年份
比如 sign_alex_2025

签到则执行对应代码
举例，1024用户在2022 年的第1天和最后一天如果有签到，那就是：
```redis
# 22年第一天
SETBIT sign_alex_2025 0 1

# 22年最后一天
SETBIT sign_alex_2025 364 1
```
判断某用户（1024）在某一天（150）是否有签到
```redis
GETBIT sign_alex_2025 150
```
`BITCOUNT`命令用于计算字符串中被设置为1的比特位的数量。它的基本语法是 `BITCOUNT key [start] [end]`，其中 `key` 是必须的参数，而 `start` 和 `end` 是可选的，用来指定要检查的字节范围。
* `key`：要搜索的键。
* `[start]`（可选）：开始搜索的`字节`偏移量。
* `[end]`（可选）：结束搜索的`字节`偏移量。
统计某用户（1024） 全面的签到次数，使用 BITCOUNT 指令，统计给定的 bit 数组中，值 = 1 的所有bit位数量。
```redis
BITCOUNT sign_alex_2025
```
当然也可以查询某个时间段的签到次数（如第64天到72天）：
> 因为参数是字节偏移量，不是位偏移量，所以只能统计8的倍数。
```redis
BITCOUNT sign_alex_2025 8 9
```

统计某个月的第一次打卡记录，这时候就要使用BITPOS了。  
`BITPOS`命令用于查找字符串中第一个出现的指定位值的位置。
```redis
BITPOS key bit [start] [end]
```
* `key`：要搜索的键。
* `bit`：要查找的位值，必须是0或1。
* `[start]`（可选）：开始搜索的字节偏移量。
* `[end]`（可选）：结束搜索的字节偏移量。

通过 `BITPOS key value [start] [end]` 指令，返回数据表示 Bitmap 中第一个值为 给定value 的 offset 位置。
在默认情况下，命令会检测整个位图，但用户也可以通过可选的start参数和end参数指定要检测的范围。
比如要查询第64天后的第一个打卡日，那么start设为8：
> 因为参数是字节偏移量，不是位偏移量，所以只能统计8的倍数。
```redis
BITPOS sign_alex_2025 1 8
```

---

##### 连续签到用户信息
如果一个平台有千万级别以上的大量用户，而我们需要统计每个用户连续签到的信息，那需要怎么设计呢？

可以把每天的日期当成位图（BitMap）的key，如 20250101
* 用户的唯一键当成（UserId）当成offset，如编号 1024 的用户
* 如果 1024 的用户在 2025.01.01 有签到，则位图的value为1，否则为0。
* 如果这时候我们要判断用户是否整周都有签到或者整个月都有签到就可以使用 【与】运算

只有指定周期内的所有值都是1（签到）的时候，结果才是1，否则是我们整周或者整个月都拿起来【与】运算，得到的结果是不是1就能确是否满勤。
```text
# 与运算：  0&0=0；0&1=0；1&0=0；1&1=1
# 下面为伪代码，类似：
(20221022 1024)  & ( 20221023 1024)  & ...
```

Redis 提供了 BITOP operation destkey key [key ...]这个指令用于对一个或者多个 键 = key 的 Bitmap 进行 位元 操作。
operation 可以是 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种：

* BITOP AND destkey key [key ...] ，对一个或多个 key 求逻辑并，并将结果保存到 destkey 。
* BITOP OR destkey key [key ...] ，对一个或多个 key 求逻辑或，并将结果保存到 destkey 。
* BITOP XOR destkey key [key ...] ，对一个或多个 key 求逻辑异或，并将结果保存到 destkey 。
* BITOP NOT destkey key ，对给定 key 求逻辑非，并将结果保存到 destkey 。
```redis
# 统计一周的值（7个BitMap，10.17 ~ 10.23 号）并将结果存入到新的BitMap （sign-result） 中
redis> BITOP AND sign-result 20221017 20221018 20221019 20221020 20221021 20221022 20221023
(integer) 1

# 新的BitMap 中，获取 1024的签到结果，如果为1，则本周全部签到
redis> GETBIT sign-result 1024
(integer) 1
```

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-13/51103548449750.png?Expires=4898115705&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=c3uOfdQmqXzowfT9zp%2FEUZ9yzzo%3D)


这边需要注意：当 BITOP 处理不同长度的字符串时，较短字符串所缺部分会被当作 0 对待。同样的，空 key 也被看作是 0 的字符串序列看待。

---

### HyperLogLog
某宝有数亿的用户，那么对于某个页面，怎么使用Redis来统计一个网站的用户访问数呢？
如果所需要的数量不用那么准确，可以使用概率算法。 事实上，我们对一个网站的UV的统计，1亿跟1亿零30万其实是差不多的。

在Redis中，已经封装了HyperLogLog算法，他是一种基数评估算法。这种算法的特征，一般都是数据不存具体的值，而是存用来计算概率的一些相关数据。

```redis
127.0.0.1:6379> pfadd html2025 111 222 333 444 555
(integer) 1
127.0.0.1:6379> pfcount html2025
(integer) 5
127.0.0.1:6379>
```


---

## RDB与AOF持久化机制

---

### RDB(Redis Data Base)
在指定的时间间隔内将内存中的数据集快照写入磁盘，RDB是内存快照（内存数据的二进制序列化形式）的方式持久化，每次都是从Redis中生成一个快照进行数据的全量备份。

> Redis持久化默认开启为RDB持久化

---

#### RDB触发规则
* save

  阻塞当前 Redis进程，直到RDB持久化过程完成，如果内存实例比较大会造成长时间阻塞，尽量不要使用这方式
* bgsave

  Redis主进程fork创建子进程，由子进程完成持久化，阻塞时间很短（微秒级）

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/5082231246625.png?Expires=4897957832&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=UpzB1fiGWo4H9PgH%2FdHlMdVOhHA%3D)

> 通过命令`CONFIG GET dir`查看执行SAVE命令之后，redis默认存放备份文件的目录；
> 
> 通过命令`CONFIG GET dbfilename`查看备份RDB文件的文件名称

* 配置触发

  在Redis安装目录下的redis.conf配置文件中搜索 /snapshot即可快速定位，配置文件默认注释了下面三行数据，通过配置规则来触发RDB的持久化，需要开启或者根据自己的需求按照规则来配置。

  ![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/4552882346541.png?Expires=4897957303&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=6kptNLBAo38%2Fy1cMrMyEmR84TU4%3D)

* shutdown触发
  
  shutdown触发Redis的RDB持久化机制非常简单，我们在客户端执行shutdown即可。
* flushall触发
  
  flushall清空Redis所有数据库的数据（16个库数据都会被删除）（等同于删库跑路）

---

#### 优缺点
优点：
* 恢复性能高：RDB持久化是通过生成一个快照文件来保存数据，因此在恢复数据时速度非常快。
* 文件紧凑：RDB文件是二进制格式的数据库文件，相对于AOF文件来说，文件体积较小。

缺点：
* 可能丢失数据：由于RDB是定期生成的快照文件，如果Redis意外宕机，最近一次的修改可能会丢失。

---

### AOF(Append Only File)
AOF持久化方案进行备份时，客户端所有请求的写命令都会被追加到AOF缓冲区中，缓冲区中的数据会根据Redis配置文件中配置的同步策略来同步到磁盘上的AOF文件中，同时当AOF的文件达到重写策略配置的阈值时，Redis会对AOF日志文件进行重写，给AOF日志文件瘦身。Redis服务重启的时候，通过加载AOF日志文件来恢复数据。

> AOF持久化需要手动修改conf配置开启。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/5189998575166.png?Expires=4897957940&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=EdIPzLPblLF18SMapLJOuoD5peA%3D)

---

#### 配置
AOF默认不开启，默认为appendonly no，开启则需要修改为appendonly yes

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/5452488982750.png?Expires=4897958202&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=coYA64%2BLLWUrRJZtUVA67XoIqfo%3D)

关闭AOF+RDB混合模式，设为no

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/5549839106000.png?Expires=4897958299&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=G5rOWsB8b2J0dC88IbItl4KlpTE%3D)

---

#### AOF同步策略

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/5643786526875.png?Expires=4897958393&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=7UCAFxaQTwYwghJ0mFRz1yqLBpU%3D)

* appendfsync always  
  每次Redis写操作，都写入AOF日志，非常耗性能。
* appendfsync everysec  
  每秒刷新一次缓冲区中的数据到AOF文件，这个Redis配置文件中默认的策略，兼容了性能和数据完整性的折中方案，这种配置，理论上丢失的数据在一秒钟左右
* appendfsync no  
  Redis进程不会主动的去刷新缓冲区中的数据到AOF文件中，而是直接交给操作系统去判断，这种操作也是不推荐的，丢失数据的可能性非常大。

---

#### AOF重写(rewrite)
重写其实是针对AOF存储的重复性冗余指令进行整理，比如有些key反复修改，又或者key反复修改后最终被删除，这些过程中的指令都是冗余且不需要存储的。

当AOF日志文件达到阈值时会触发自动重写。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/5917605504833.png?Expires=4897958667&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=OTSCDob6r%2BFvJFYMugjvqsuo1m0%3D)

`auto-aof-rewrite-percentage 100`：当AOF文件体积达到上次重写之后的体积的100%时，会触发AOF重写。  
`auto-aof-rewrite-min-size 64mb`：当AOF文件体积超过这个阈值时，会触发AOF重写。

手动重写：`bgrewriteaof`

#### 优缺点
优点：
* 数据更加可靠：AOF持久化记录了每个写命令的操作，因此在出现故障时，可以通过重新执行AOF文件来保证数据的完整性。
* 可以保留写命令历史：AOF文件是一个追加日志文件，可以用于回放过去的写操作。

缺点：
* 文件较大：由于记录了每个写命令，AOF文件体积通常比RDB文件要大。
* 恢复速度较慢：当AOF文件较大时，Redis重启时需要重新执行整个AOF文件，恢复速度相对较慢。

---

### 混合持久化
Redis4.0版本开始支持混合持久化，因为RDB虽然加载快但是存在数据丢失，AOF数据安全但是加载缓慢。
混合持久化通过aof-use-rdb-preamble yes开启，Redis 4.0以上版本默认开启(_开启AOF即为混合持久化，开启AOF并关闭混合持久化才为单独AOF_)

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-11/5549839106000.png?Expires=4897958299&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=G5rOWsB8b2J0dC88IbItl4KlpTE%3D)


开启混合持久化之后：appendonlydir文件下存在一个rdb文件与一个aof文件

---

## Redis的过期策略

---

### 惰性删除（Lazy expiration）
当客户端尝试访问某个键时，Redis会先检查该键是否设置了过期时间，并判断是否过期。  
如果键已过期，则Redis会立即将其删除。这就是惰性删除策略。

> 该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。

---

### 定期删除（Active expiration）
Redis会每隔一段时间（默认100毫秒）随机检查一部分设置了过期时间的键。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/27549487813583.png?Expires=4898046660&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=%2Fe3F9%2FFaBXyf2CZVsNvXPs2CzSE%3D)

定期过期策略通过使用循环遍历方式，逐个检查键是否过期，并删除已过期的键值对。

---

### 同时使用
Redis中同时使用了惰性过期和定期过期两种过期策略。
* 假设Redis当前存放20万个key，并且都设置了过期时间，如果你每隔100ms就去检查这全部的key，CPU负载会特别高，最后可能会挂掉。
* 因此redis采取的是定期过期，每隔100ms就随机抽取一定数量的key来检查和删除的。使用了简单的贪心策略。
  1. 从过期字典中随机 20 个 key；
  2. 删除这 20 个 key 中已经过期的 key；
  3. 如果过期的 key 比率超过 1/4，那就重复步骤 1；
* 但是呢，最后可能会有很多已经过期的key没被删除。这时候，redis采用惰性删除。在你获取某个key的时候，redis会检查一下，这个key如果设置了过期时间并且已经过期了，此时就会删除。
* 需要注意如果定期删除漏掉了很多过期的key，然后也没走惰性删除。就会有很多过期key积在内存中，可能会导致内存溢出，或者是业务量太大，内存不够用然后溢出了，为了应对这个问题，Redis引入了内存淘汰策略进行优化。

---

## Redis的内存淘汰策略
有了以上过期策略的说明后，就很容易理解为什么需要淘汰策略了，因为不管是定期采样删除还是惰性删除都不是一种完全精准的删除，就还是会存在key没有被删除掉的场景，所以就需要内存淘汰策略进行补充。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/27963453731875.png?Expires=4898047074&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=dQ69HuWWqaIyluAz6QYynScY978%3D)

### Redis 最大运行内存
只有在 Redis 的运行内存达到了某个阀值，才会触发内存淘汰机制，这个阀值就是我们设置的最大运行内存。  
此值在 Redis 的配置文件 redis.conf 中可以找到，配置项为 maxmemory。  
默认为0表示无限制。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/28432262283875.png?Expires=4898047543&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Ys%2BGmSndluUvctQLQnmGYHFcn9g%3D)

使用命令 `config get maxmemory` 来查看设置的最大运行内存。

### 逐出策略

* noeviction（默认，直接拒绝）：不删除任何Key，当内存达到上限时，将无法写入新数据，数据库会返回错误信息。
* volatile-lru（过期LRU）：从已设置过期时间（Expire）的Key中，删除最近最少使用的Key（LRU算法），且不会考虑Key是否已经过期。
* allkeys-lru（全局LRU）：从所有Key中，删除最近最少使用的Key（LRU算法）。
* volatile-lfu（过期LFU）：从已设置过期时间（Expire）的Key中，删除最不常用的Key（LFU算法）。
* allkeys-lfu（全局LFU）：从所有Key中，删除最不常用的Key（LFU算法）。
* volatile-random（过期随机）：从已设置过期时间（Expire）的Key中，随机删除一些Key。
* allkeys-random（全局随机）：从所有Key中，随机删除一些Key。
* volatile-ttl（过期时间）：从已设置过期时间（Expire）的Key中，根据存活时间（TTL）从小到大排序进行删除。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/28723224133416.png?Expires=4898047834&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=0%2BH%2B%2BVzBHnM7jBCNRKImB2W0vfg%3D)

`LRU means Least Recently Used`:选择最近最久未使用的页面予以淘汰。
LRU 算法需要基于链表结构，链表中的元素按照`操作顺序`从前往后排列，**最新操作的键**会被移动到表头，当需要内存淘汰时，只需要删除链表尾部的元素即可。

LRU 算法有一个缺点，比如说很久没有使用的一个键值，如果最近被访问了一次，那么它就不会被淘汰，即使它是使用次数最少的缓存，那它也不会被淘汰。

`LFU means Least Frequently Used`:根据总访问次数来淘汰数据。

在 Redis 中每个对象头中记录着 LFU 的信息。
```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;
    void *ptr;
} robj;
```

`Both LRU, LFU and volatile-ttl are implemented using approximated randomized algorithms.`
Redis 使用的是一种近似 LRU 算法，目的是为了更好的节约内存，它的实现方式是给现有的数据结构添加一个额外的字段，用于记录此键值的最后一次访问时间，Redis 内存淘汰时，会使用随机采样的方式来淘汰数据，它是随机取 5 个值（此值可配置），然后淘汰最久没有使用的那个。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/29768897648250.png?Expires=4898048880&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=epg%2FKByyIs%2BJSv43WUaH2JgtwJI%3D)

---

### 如何保证 redis 中的数据都是热点数据
回顾内存逐出策略，很简单了，使用LFU算法，选择volatile还是allkeys就要根据具体的业务需求了。

---

## 缓存击穿（Cache Miss）
缓存击穿是指在高并发访问下，`一个热点数据失效`时，大量请求会直接绕过缓存，直接查询数据库，导致数据库压力剧增。
通常情况下，缓存是为了减轻数据库的负载，提高读取性能而设置的。当某个特定的缓存键（key）失效后，在下一次请求该缓存时，由于缓存中没有对应的数据，因此会去数据库中查询，这就是缓存击穿。

* 合理的过期时间：设置热点数据永不过期，或者设置较长的过期时间，以免频繁失效。
* 使用互斥锁：保证同一时间只有一个线程来查询数据库，其他线程等待查询结果。

## 缓存雪崩（Cache Avalanche）
缓存雪崩是指在`大规模缓存失效`或者`缓存宕机`的情况下，大量请求同时涌入数据库，导致数据库负载过大甚至崩溃的情况。
正常情况下，缓存中的数据会根据过期时间进行更新，当大量数据同时失效时，下一次请求就会直接访问数据库，给数据库带来巨大压力。


* 合理的过期时间：为缓存的过期时间引入随机值，分散缓存过期时间，避免大规模同时失效。或者是粗暴的设置热点数据永不过期
* 多级缓存：使用多级缓存架构，如本地缓存 + 分布式缓存，提高系统的容错能力。
* 使用互斥锁：保证同一时间只有一个线程来查询数据库，其他线程等待查询结果。
* 高可用架构：使用Redis主从复制或者集群来增加缓存的可用性，避免单点故障导致整个系统无法使用。

## 缓存穿透（Cache Penetration）
缓存穿透是指`恶意请求查询一个不存在于缓存和数据库中的数据`，导致每次请求都直接访问数据库，从而增加数据库的负载。
攻击者可以通过故意构造不存在的 Key 来进行缓存穿透攻击。

* 缓存空对象：对于查询结果为空的情况，也将其缓存起来，但使用较短的过期时间，防止攻击者利用同样的 key 进行频繁攻击。
* 参数校验：在接收到请求之前进行参数校验，判断请求参数是否合法。
* 布隆过滤器：判断请求的参数是否存在于缓存或数据库中。

## 布隆过滤器（Bloom Filter）
本质上布隆过滤器是一种数据结构，比较巧妙的概率型数据结构（probabilistic data structure），特点是高效地插入和查询，可以用来告诉你 “某样东西一定不存在或者可能存在”。  
相比于传统的 List、Set、Map 等数据结构，它更高效、占用空间更少，但是缺点是其返回的结果是概率性的，而不是确切的。

布隆过滤器的原理是，当一个元素被加入集合时，通过K个散列函数将这个元素映射成一个位数组中的K个点，把它们置为1。检索时，我们只要看看这些点是不是都是1就（大约）知道集合中有没有它了：如果这些点`有任何一个0`，则被检元素`一定不在`；如果`都是1`，则被检元素`很可能在`。这就是布隆过滤器的基本思想。所以布隆过滤器可能会产生假阳性（误报），但不会产生假阴性（漏报）。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/30713164301416.png?Expires=4898049824&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=Bcrd%2FoGS14tD4y0ym1%2BW6X5KgwE%3D)

但是布隆过滤器的缺点和优点一样明显。误算率是其中之一。随着存入的元素数量增加，误算率随之增加。但是如果元素数量太少，则使用散列表足矣。

---

## Redis集群

---

### 主从复制模式（Master-Slave）

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/32051231458583.png?Expires=4898051162&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=PXLr%2FUJizO66Lh550mR8KX0Ro%2Fw%3D)

主从复制是Redis的一种基本集群模式，它通过将一个Redis节点（主节点）的数据复制到一个或多个其他Redis节点（从节点）来实现数据的冗余和备份。

主节点负责处理客户端的写操作，同时从节点会实时同步主节点的数据。客户端可以从从节点读取数据，实现读写分离，提高系统性能。

#### 主从复制配置和实现
1. 配置主节点：在主节点的redis.conf配置文件中，无需进行特殊配置，主节点默认监听所有客户端请求。
    ```text
    # 主节点默认端口号6379
    port 6379
    ```
2. 配置从节点：在从节点的redis.conf配置文件中，添加如下配置，指定主节点的地址和端口：
    ```text
    # 从节点设置端口号6380
    port 6380
    
    # replicaof 主节点IP 主节点端口
    replicaof 127.0.0.1 6379
    ```
    或者，通过Redis命令行在从节点上执行如下命令：
    ```redis
    replicaof 127.0.0.1 6379
    ```


#### 主从复制的优缺点
优点：
* 配置简单，易于实现。
* 实现数据冗余，提高数据可靠性。
* 读写分离，提高系统性能。

缺点：
* 主节点故障时，需要手动切换到从节点，故障恢复时间较长。
* 主节点承担所有写操作，可能成为性能瓶颈。
* 无法实现数据分片，受单节点内存限制。

#### 适用场景
* 数据备份和容灾恢复：通过从节点备份主节点的数据，实现数据冗余。
* 读写分离：将读操作分发到从节点，减轻主节点压力，提高系统性能。
* 在线升级和扩展：在不影响主节点的情况下，通过增加从节点来扩展系统的读取能力。

---

### 哨兵模式（Sentinel）

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/32356020712375.png?Expires=4898051467&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=nKbH%2BalzR5OqueSoDyN2%2FOo4rzs%3D)

哨兵模式是在主从复制基础上加入了哨兵节点，实现了自动故障转移。哨兵节点是一种特殊的Redis节点，它会监控主节点和从节点的运行状态。当主节点发生故障时，哨兵节点会自动从从节点中选举出一个新的主节点，并通知其他从节点和客户端，实现故障转移。

#### 哨兵模式配置和实现
1. 配置主从复制：首先按照主从复制模式的配置方法，搭建一个主从复制集群。
2. 配置哨兵节点：在哨兵节点上创建一个新的哨兵配置文件（如：sentinel.conf），并添加如下配置：
    ```text
    # sentinel节点端口号
    port 26379
    
    # sentinel monitor 被监控主节点名称 主节点IP 主节点端口 quorum
    sentinel monitor mymaster 127.0.0.1 6379 2
    
    # sentinel down-after-milliseconds 被监控主节点名称 毫秒数
    sentinel down-after-milliseconds mymaster 60000
    
    # sentinel failover-timeout 被监控主节点名称 毫秒数
    sentinel failover-timeout mymaster 180000
    ```
    > 其中，quorum是指触发故障转移所需的最小哨兵节点数。down-after-milliseconds表示主节点被判断为失效的时间。failover-timeout是故障转移超时时间。

3. 启动哨兵节点：使用如下命令启动哨兵节点：
    ```redis
    redis-sentinel /path/to/sentinel.conf
    ```

#### 哨兵模式的优缺点
优点：
* 自动故障转移，提高系统的高可用性。
* 具有主从复制模式的所有优点，如数据冗余和读写分离。

缺点：
* 配置和管理相对复杂。
* 依然无法实现数据分片，受单节点内存限制。

#### 适用场景
* 高可用性要求较高的场景：通过自动故障转移，确保服务的持续可用。
* 数据备份和容灾恢复：在主从复制的基础上，提供自动故障转移功能。

---

### Cluster模式

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-12/32636577201333.png?Expires=4898051747&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=MMa%2FiYdRmF1LZ1oVJG%2FQ%2BUVtU2I%3D)

Cluster模式是Redis的一种高级集群模式，它通过数据分片和分布式存储实现了负载均衡和高可用性。在Cluster模式下，Redis将所有的键值对数据分散在多个节点上。每个节点负责一部分数据，称为槽位。通过对数据的分片，Cluster模式可以突破单节点的内存限制，实现更大规模的数据存储。

Redis Cluster将数据分为16384个槽位，每个节点负责管理一部分槽位。当客户端向Redis Cluster发送请求时，Cluster会根据键的哈希值将请求路由到相应的节点。具体来说，Redis Cluster使用CRC16算法计算键的哈希值，然后对16384取模，得到槽位编号。

#### Cluster模式配置和实现
1. 配置Redis节点：为每个节点创建一个redis.conf配置文件，并添加如下配置：
    ```text
    # cluster节点端口号
    port 7001
    
    # 开启集群模式
    cluster-enabled yes
    
    # 节点超时时间
    cluster-node-timeout 15000
    ```
2. 启动Redis节点：使用如下命令启动6个节点：
    ```redis
    redis-server redis_7001.conf
    ```
3. 创建Redis Cluster：使用Redis命令行工具执行如下命令创建Cluster
    ```redis
    redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
    ```
    > cluster-replicas 表示从节点的数量，1代表每个主节点都有一个从节点。

#### Cluster模式的优缺点
优点：
* 数据分片，实现大规模数据存储。
* 负载均衡，提高系统性能。
* 自动故障转移，提高高可用性。

缺点：
* 配置和管理较复杂。
* 一些复杂的多键操作可能受到限制。

#### 适用场景
* 大规模数据存储：通过数据分片，突破单节点内存限制。
* 高性能要求场景：通过负载均衡，提高系统性能。
* 高可用性要求场景：通过自动故障转移，确保服务的持续可用。

---

## 数据库和缓存一致性问题
不管是先写MySQL数据库，再删除Redis缓存；还是先删除缓存，再写库，都有可能出现数据不一致的情况。

### 延时双删
* 删除缓存
* 缓存删除完成之后，更新数据库
* 数据库更新完成之后，休眠 n ms，二次删除缓存

这时候唯一存在的一个问题就是，在（更新据库 + 休眠 n ms） 这个时间窗口中，依旧能读取到旧值，而这个短暂时间控制的好的话，是可以接受的。

### 事务保证
无论是先更新数据库，再删除缓存；还是先删除缓存，在更新数据库。
保持事务性都是一种方案，如果删除缓存失败，则数据库更新会被回滚；如果更新数据库失败，则缓存也不会被删除。这个需要一致性策略接入。不过无论怎么做，这个都会在一定程度上影响执行完成的性能。


### 队列 + 重试机制
如果双删还是失败，还是会产生缓存和数据库数据不一致的现象。
一般的做法是做一层兜底，比如记录日志，人工来处理；或者通过MQ来发布消息，然后开发一个独立的服务来订阅，专门用于数据清理，这就将操作异步化了。

### binlog独立删除能力
* 更新数据库数据
* 数据库更新完成之后会把变更记录在 binlog 中
* 使用 canal 订阅 binlog 日志获取待删除的key（或者更完整的数据对象）
* 消费者（缓存删除服务）获取到 canal 数据，获得待删除的key，并删除缓存

## Redis分布式锁

---

### SETNX + EXPIRE
先用setnx来抢锁，如果抢到之后，再用expire给锁设置一个过期时间，防止锁忘记了释放。
```java
if（jedis.setnx(key_resource_id,lock_value) == 1）{ //加锁
    expire（key_resource_id，100）; //设置过期时间
    try {
        do something  //业务请求
    }catch() {
        
    }finally {
       jedis.del(key_resource_id); //释放锁
    }
}
```
> 缺陷：加锁与设置过期时间是非原子操作，如果加锁后未来得及设置过期时间系统异常等，会导致其他线程永远获取不到锁。

---

### 使用Lua脚本（包含setnx和expire两条指令）

```lua
if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then
   redis.call('expire',KEYS[1],ARGV[2])
else
   return 0
end;
```
```java
String lua_scripts = "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then" +
            " redis.call('expire',KEYS[1],ARGV[2]) return 1 else return 0 end";   
Object result = jedis.eval(lua_scripts, Collections.singletonList(key_resource_id), Collections.singletonList(UUID, String.valueOf(expireTimeInSeconds))));
//判断是否成功
return result.equals(1L);
```

---

### SET的扩展命令（SET EX PX NX）
使用Lua脚本，保证SETNX + EXPIRE两条指令的原子性，我们还可以巧用Redis的SET指令扩展参数！（`SET key value[EX seconds] [PX milliseconds] [NX|XX]`），它也是原子性的
```redis
SET key value[EX seconds][PX milliseconds][NX|XX]
```
* EX seconds: 设定过期时间，单位为秒
* PX milliseconds: 设定过期时间，单位为毫秒
* NX: 仅当key不存在时设置值
* XX: 仅当key存在时设置值

```java
if（jedis.set(key_resource_id, lock_value, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch() {
        
    }finally {
        jedis.del(key_resource_id); //释放锁
    }
}
```

> * 问题一：「锁过期释放了，业务还没执行完」。假设线程a获取锁成功，一直在执行临界区的代码。但是100s过去后，它还没执行完。但是，这时候锁已经过期了，此时线程b又请求过来。显然线程b就可以获得锁成功，也开始执行临界区的代码。那么问题就来了，临界区的业务代码都不是严格串行执行的啦。
> * 问题二：「锁被别的线程误删」。假设线程a执行完后，去释放锁。但是它不知道当前的锁可能是线程b持有的（线程a去释放锁时，有可能过期时间已经到了，此时线程b进来占有了锁）。那线程a就把线程b的锁释放掉了，但是线程b临界区业务代码可能都还没执行完呢。

释放锁时需要验证value值，也就是说我们在获取锁的时候需要设置一个value，不能直接用del key这种粗暴的方式，因为直接del key任何客户端都可以进行解锁了，所以解锁时，我们需要判断锁是否是自己的，基于value值来判断，代码如下：
```java
if（jedis.set(key_resource_id, uni_request_id, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch() {
        
    }finally {
        //判断是不是当前线程加的锁,是才释放
        if (uni_request_id.equals(jedis.get(key_resource_id))) {
            jedis.del(lockKey); //释放锁
        }
    }
}
```
为了更严谨，一般也是用lua脚本代替。lua脚本如下：
```lua
if redis.call('get',KEYS[1]) == ARGV[1] then 
   return redis.call('del',KEYS[1]) 
else
   return 0
end;
```

至于问题一可以将过期时间设置的相对长一点，当然也可以另开一个线程续期，不过这种就不用我们去实现了，利用 Redisson 进行处理。

> 实际上在Redis集群的时候也会出现问题，比如说A客户端在Redis的master节点上拿到了锁，但是这个加锁的key还没有同步到slave节点，master故障，发生故障转移，一个slave节点升级为master节点，B客户端也可以获取同个key的锁，但客户端A也已经拿到锁了，这就导致多个客户端都拿到锁。

---

### Redisson框架
设想一下，是否可以给获得锁的线程，开启一个定时守护线程，每隔一段时间检查锁是否还存在，存在则对锁的过期时间延长，防止锁过期提前释放。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-13/53676820666166.png?Expires=4898128777&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=gRmx%2BDF5YGe%2B362ITOR0%2FJj34Hg%3D)

只要线程一加锁成功，就会启动一个watch dog看门狗，它是一个后台线程，会每隔10秒检查一下，如果线程1还持有锁，那么就会不断的延长锁key的生存时间。因此，Redisson就是使用Redisson解决了「锁过期释放，业务没执行完」问题。

---

### 多机实现的分布式锁Redlock+Redisson

如果线程一在Redis的master节点上拿到了锁，但是加锁的key还没同步到slave节点。

恰好这时，master节点发生故障，一个slave节点就会升级为master节点。线程二就可以获取同个key的锁啦，但线程一也已经拿到锁了，锁的安全性就没了。

为了解决这个问题，Redis作者 antirez提出一种高级的分布式锁算法：Redlock。Redlock核心思想是这样的：

搞多个Redis master部署，以保证它们不会同时宕掉。并且这些master节点是完全相互独立的，相互之间不存在数据同步。同时，需要确保在这多个master实例上，是与在Redis单实例，使用相同方法来获取和释放锁。

我们假设当前有5个Redis master节点，在5台服务器上面运行这些Redis实例。


![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-13/53920271099333.png?Expires=4898129021&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=EXtK3fqKID6%2BzgvUGNJRc2p1jvs%3D)

* 按顺序向5个master节点请求加锁
* 根据设置的超时时间来判断，是不是要跳过该master节点。
* 如果大于等于3个节点加锁成功，并且使用的时间小于锁的有效期，即可认定加锁成功啦。
* 如果获取锁失败，解锁！

---

## Redis事务
Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

* MULTI ：开启事务，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令系列。
* EXEC：执行事务中的所有操作命令。
* DISCARD：取消事务，放弃执行事务块中的所有命令。
* WATCH：监视一个或多个key,如果事务在执行前，这个key(或多个key)被其他命令修改，则事务被中断，不会执行事务中的任何命令。
* UNWATCH：取消WATCH对所有key的监视。

### ACID
* 原子性 atomicity
> 首先通过上文知道 运行期的错误是不会回滚的，很多文章由此说Redis事务违背原子性的；而官方文档认为是遵从原子性的。Redis官方文档给的理解是，Redis的事务是原子性的：所有的命令，要么`全部执行`，要么`全部不执行`。而`不是完全成功`。
* 一致性 consistencyredis
> 事务可以保证命令失败的情况下得以回滚，数据能恢复到没有执行之前的样子，是保证一致性的，除非redis进程意外终结。
* 隔离性 Isolationredis
> 事务是严格遵守隔离性的，原因是redis是单进程单线程模式(v6.0之前），可以保证`命令执行过程中不会被其他客户端命令打断`。但是，Redis不像其它结构化数据库有隔离级别这种设计。
* 持久性 Durabilityredis
> 事务是`不保证持久性`的，这是因为redis持久化策略中不管是RDB还是AOF都是异步执行的，不保证持久性是出于对性能的考虑。


### 标准的事务执行
```redis
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> set k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) OK
```

### 事务取消
```redis
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 33
QUEUED
127.0.0.1:6379> set k2 34
QUEUED
127.0.0.1:6379> DISCARD
OK
```

### 语法错误（编译器错误）
在开启事务后，修改k1值为11，k2值为22，但k2语法错误，最终导致事务提交失败，k1、k2保留原值。
```redis
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> sets k2 22
(error) ERR unknown command `sets`, with args beginning with: `k2`, `22`, 
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
```

### Redis类型错误（运行时错误）
在开启事务后，修改k1值为11，k2值为22，但将k2的类型作为List，在运行时检测类型错误，最终导致事务提交失败，此时事务并没有回滚，而是跳过错误命令继续执行， 结果k1值改变、k2保留原值
```redis
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k1 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> lpush k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1
"11"
127.0.0.1:6379> get k2
"v2"
```

### WATCH 命令
WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。

假设我们需要原子性地为某个值进行增 1 操作（假设 INCR 不存在）。
```redis
val = GET mykey
val = val + 1
SET mykey $val
```
上面的这个实现在只有一个客户端的时候可以执行得很好。 但是， 当多个客户端同时对同一个键进行这样的操作时， 就会产生竞争条件。举个例子， 如果客户端 A 和 B 都读取了键原来的值， 比如 10 ， 那么两个客户端都会将键的值设为 11 ， 但正确的结果应该是 12 才对。有了 WATCH ，我们就可以轻松地解决这类问题了：
```redis
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```
Redis使用WATCH命令来决定事务是继续执行还是回滚，那就需要在MULTI之前使用WATCH来监控某些键值对，然后使用MULTI命令来开启事务，执行对数据结构操作的各种命令，此时这些命令入队列。当使用EXEC执行事务时，首先会比对WATCH所监控的键值对，如果没发生改变，它会执行事务队列中的命令，提交事务；如果发生变化，将不会执行事务中的任何命令，同时事务回滚。当然无论是否回滚，Redis都会取消执行事务前的WATCH命令。

### UNWATCH取消监视
```redis
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> WATCH k1
OK
127.0.0.1:6379> set k1 11
OK
127.0.0.1:6379> UNWATCH
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 12
QUEUED
127.0.0.1:6379> set k2 22
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
127.0.0.1:6379> get k1
"12"
127.0.0.1:6379> get k2
"22"
```

### Redis事务执行步骤
* 开启：以MULTI开始一个事务

* 如果客户端发送的命令为 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令的其中一个， 那么服务器立即执行这个命令。
* 与此相反， 如果客户端发送的命令是 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令以外的其他命令， 那么服务器并不立即执行这个命令， 而是将这个命令放入一个事务队列里面， 然后向客户端返回 QUEUED 回复。

* 执行：由EXEC命令触发事务

### 为什么 Redis 不支持回滚
* Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：
* 这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
* 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

---

## Redis相比memcached有哪些优势
* 从`数据结构`来说，memcached仅支持value为string类型，而我们redis支持的类型是相当丰富的，有string、hash、list、set、sort set等等，所以在功能上redis是比我们memcached支持的更好的。
* memcached的单`value值容量`只有1M，而我们的redis则最大支持至512M。
* 从`数据持久化`来说，memcached只做缓存，没有可靠性的需求，所以是不支持的，只要断电或者服务关闭之后那么就会丢失内存中的数据，而redis更倾向于内存数据库，如果我们有持久化需求的话可以优先考虑redis。
* redis还支持lua脚本，脚本提交是`原子执行`的，我们在面对复杂业务场景中，需要保证按照我们所需的顺序一步步执行就可以通过我们的lua脚本来解决。

---

## Redis6为何引入多线程
redis6中引入的多线程是正对于网络IO模块进行了多线程改造，因为多路复用的IO模型本质上来说还是同步阻塞型IO模型，在调用epoll的过程是阻塞的，并发量极高的场景就成为了性能瓶颈，那么在碰到这类问题上，就可以通过多线程来解决。它通过多线程解决了网络IO等待造成的影响，还可以充分利用CPU的多核优势。对于我们读写模块依旧还是采用的单线程模型，避免了多线程环境下并发访问带来的很多问题。在简单的get/set命令性能上多线程IO模型提升了有接近一倍。

---

## Redis是如何解决Hash冲突的
redis是通过我们的链式hash来解决我们的hash冲突问题，哈希算法产生的哈希值的长度是固定并且是有限的，比如说我们通过MD5算法生成32位的散列值，那么它能生成出来的长度则是有限的，我们的数据如果大于32位是不是就可能存在不同数据生成同一个散列值，那么redis通过链式hash，以不扩容的前提下把有相同值的数据链接起来，但是如果链表变得很长就会导致性能下降，那么redis就采用了rehash的机制来解决，类似于hashmap里面的扩容机制，但是redis中的rehash并不是一次把hash表中的数据映射到另外一张表，而是通过了一种渐进式的方式来处理，将rehash分散到多次请求过程中，避免阻塞耗时。

---

## Redis性能调优

---

### Redis真的变慢了吗
首先，在开始之前，你需要弄清楚 Redis 是否真的变慢了？

如果你发现你的业务服务 API 响应延迟变长，首先你需要先排查服务内部，究竟是哪个环节拖慢了整个服务。比较高效的做法是，在服务内部集成链路追踪，也就是在服务访问外部依赖的出入口，记录下每次请求外部依赖的响应延时。

从你的业务服务到 Redis 这条链路变慢的原因可能也有 2 个：
* 业务服务器到 Redis 服务器之间的网络存在问题，例如网络线路质量不佳，网络数据包在传输时存在延迟、丢包等情况
* Redis 本身存在问题，需要进一步排查是什么原因导致 Redis 变慢

排除网络原因，如何确认你的 Redis 是否真的变慢了？

---

### 什么是基准性能
Redis 在不同的软硬件环境下，它的性能是各不相同的。

所以，你只有了解了你的 Redis 在生产环境服务器上的基准性能，才能进一步评估，当其延迟达到什么程度时，才认为 Redis 确实变慢了。

执行以下命令，就可以测试出这个实例 60 秒内的最大响应延迟：
```shell
redis-cli -h 127.0.0.1 -p 6379 --intrinsic-latency 60
```

```shell
[root@VM-8-17-centos ~]# redis-cli -h 127.0.0.1 -p 6379 --intrinsic-latency 60
Max latency so far: 1 microseconds.
Max latency so far: 13 microseconds.
Max latency so far: 14 microseconds.
Max latency so far: 55 microseconds.
Max latency so far: 78 microseconds.
Max latency so far: 710 microseconds.
Max latency so far: 3258 microseconds.
Max latency so far: 3411 microseconds.
Max latency so far: 3550 microseconds.
Max latency so far: 3750 microseconds.

1041054025 total runs (avg latency: 0.0576 microseconds / 57.63 nanoseconds per run).
Worst run took 65066x longer than the average latency.
```
从输出结果可以看到，这 60 秒内的最大响应延迟为 3750 微秒（3.75毫秒）。

你还可以使用以下命令，查看一段时间内 Redis 的最小、最大、平均访问延迟：
```shell
redis-cli -h 127.0.0.1 -p 6379 --latency-history -i 1
```
```shell
[root@VM-8-17-centos ~]# redis-cli -h 127.0.0.1 -p 6379 --latency-history -i 1
min: 0, max: 2, avg: 0.10 (100 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.08 (99 samples) -- 1.00 seconds range
min: 0, max: 1, avg: 0.06 (99 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.09 (99 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.09 (99 samples) -- 1.00 seconds range
min: 0, max: 1, avg: 0.11 (99 samples) -- 1.01 seconds range
min: 0, max: 1, avg: 0.10 (99 samples) -- 1.01 seconds range
......
```
以上输出结果是，每间隔 1 秒，采样 Redis 的平均操作耗时，其结果分布在 0.08 ~ 0.11 毫秒之间。

了解了基准性能测试方法，那么你就可以按照以下几步，来判断你的 Redis 是否真的变慢了：
* 在相同配置的服务器上，测试一个正常 Redis 实例的基准性能
* 找到你认为可能变慢的 Redis 实例，测试这个实例的基准性能
* 如果你观察到，这个实例的运行延迟是正常 Redis 基准性能的 2 倍以上，即可认为这个 Redis 实例确实变慢了

---

### Redis 的慢日志（slowlog）
第一步，你需要去查看一下 Redis 的慢日志（slowlog）。

Redis 提供了慢日志命令的统计功能，它记录了有哪些命令在执行时耗时比较久。
查看 Redis 慢日志之前，你需要设置慢日志的阈值。例如，设置慢日志的阈值为 5 毫秒，并且保留最近 500 条慢日志记录：

```redis
CONFIG SET slowlog-log-slower-than 5000

CONFIG SET slowlog-max-len 500
```

此时，你可以执行以下命令，就可以查询到最近记录的慢日志：
```shell
127.0.0.1:6379> SLOWLOG get 5
1) 1) (integer) 32693       
   2) (integer) 1593763337  
   3) (integer) 5299        
   4) 1) "LRANGE"           
      2) "user_list:2000"
      3) "0"
      4) "-1"
2) 1) (integer) 32692
   2) (integer) 1593763337
   3) (integer) 5044
   4) 1) "GET"
      2) "user_info:1000"
...
```

如果你的应用程序执行的 Redis 命令有以下特点，那么有可能会导致操作延迟变大：
* 经常使用 O(N) 以上复杂度的命令，例如 SORT、SUNION、ZUNIONSTORE 聚合类命令
  > Redis 在操作内存数据时，时间复杂度过高，要花费更多的 CPU 资源。
* 使用 O(N) 复杂度的命令，但 N 的值非常大
  > Redis 一次需要返回给客户端的数据过多，更多时间花费在数据协议的组装和网络传输过程中。

如果你的应用程序操作 Redis 的 OPS 不是很大，但 Redis 实例的 CPU 使用率却很高，那么很有可能是使用了复杂度过高的命令导致的。
Redis 是单线程处理客户端请求的，如果你经常使用以上命令，那么当 Redis 处理客户端请求时，一旦前面某个命令发生耗时，就会导致后面的请求发生排队，对于客户端来说，响应延迟也会变长。

---

### 禁止操作bigkey
如果你查询慢日志发现，并不是复杂度过高的命令导致的，而都是 SET / DEL 这种简单命令出现在慢日志中，那么你就要怀疑你的实例否写入了 bigkey。

> Redis 在写入数据时，需要为新的数据分配内存，相对应的，当从 Redis 中删除数据时，它会释放对应的内存空间。
> 如果一个 key 写入的 value 非常大，那么 Redis 在分配内存时就会比较耗时。同样的，当删除这个 key 时，释放内存也会比较耗时，这种类型的 key 我们一般称之为 bigkey。

扫描出实例中 bigkey 的分布情况:
```shell
$ redis-cli -h 127.0.0.1 -p 6379 --bigkeys -i 0.01
 
...
-------- summary -------
 
Sampled 829675 keys in the keyspace!
Total key length in bytes is 10059825 (avg len 12.13)
 
Biggest string found 'key:291880' has 10 bytes
Biggest   list found 'mylist:004' has 40 items
Biggest    set found 'myset:2386' has 38 members
Biggest   hash found 'myhash:3574' has 37 fields
Biggest   zset found 'myzset:2704' has 42 members
 
36313 strings with 363130 bytes (04.38% of keys, avg size 10.00)
787393 lists with 896540 items (94.90% of keys, avg size 1.14)
1994 sets with 40052 members (00.24% of keys, avg size 20.09)
1990 hashs with 39632 fields (00.24% of keys, avg size 19.92)
1985 zsets with 39750 members (00.24% of keys, avg size 20.03)
```
从输出结果我们可以很清晰地看到，每种数据类型所占用的最大内存 / 拥有最多元素的 key 是哪一个，以及每种数据类型在整个实例中的占比和平均大小 / 元素数量。

其实，使用这个命令的原理，就是 Redis 在内部执行了 SCAN 命令，遍历整个实例中所有的 key，然后针对 key 的类型，分别执行 STRLEN、LLEN、HLEN、SCARD、ZCARD 命令，来获取 String 类型的长度、容器类型（List、Hash、Set、ZSet）的元素个数。

当执行这个命令时，要注意 2 个问题：
* 对线上实例进行 bigkey 扫描时，Redis 的 OPS 会突增，为了降低扫描过程中对 Redis 的影响，最好控制一下扫描的频率，指定 -i 参数即可，它表示扫描过程中每次扫描后休息的时间间隔，单位是秒
* 扫描结果中，对于容器类型（List、Hash、Set、ZSet）的 key，只能扫描出元素最多的 key。但一个 key 的元素多，不一定表示占用内存也多，你还需要根据业务情况，进一步评估内存占用情况

优化方法
* 业务应用尽量避免写入 bigkey
* 如果你使用的 Redis 是 4.0 以上版本，用 UNLINK 命令替代 DEL，此命令可以把释放 key 内存的操作，放到后台线程中去执行，从而降低对 Redis 的影响
* 如果你使用的 Redis 是 6.0 以上版本，可以开启 lazy-free 机制（lazyfree-lazy-user-del = yes），在执行 DEL 命令时，释放内存也会放到后台线程中执行

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-15/105911515889916.png?Expires=4898316664&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=nLHIWcDDfwocWYJI4F56lfA1nEA%3D)


---

### 禁止集中过期
如果你发现，平时在操作 Redis 时，并没有延迟很大的情况发生，但在某个时间点突然出现一波延时，其现象表现为：变慢的时间点很有规律，例如某个整点，或者每间隔多久就会发生一波延迟。如果是出现这种情况，那么你需要排查一下，业务代码中是否存在设置大量 key 集中过期的情况。

Redis 的过期数据采用被动过期 + 主动过期两种策略：
* 被动过期：只有当访问某个 key 时，才判断这个 key 是否已过期，如果已过期，则从实例中删除
* 主动过期：Redis 内部维护了一个定时任务，默认每隔 100 毫秒（1秒10次）就会从全局的过期哈希表中随机取出 20 个 key，然后删除其中过期的 key，如果过期 key 的比例超过了 25%，则继续重复此过程，直到过期 key 的比例下降到 25% 以下，或者这次任务的执行耗时超过了 25 毫秒，才会退出循环

注意，这个主动过期 key 的定时任务，是在 Redis 主线程中执行的。

也就是说如果在执行主动过期的过程中，出现了需要大量删除过期 key 的情况，那么此时应用程序在访问 Redis 时，必须要等待这个过期任务执行结束，Redis 才可以服务这个客户端请求。

此时就会出现，应用访问 Redis 延时变大。也就是缓存雪崩。

如果此时需要过期删除的是一个 bigkey，那么这个耗时会更久。而且，这个操作延迟的命令并不会记录在慢日志中。
所以，此时你会看到，慢日志中没有操作耗时的命令，但我们的应用程序却感知到了延迟变大，其实时间都花费在了删除过期 key 上，这种情况我们需要尤为注意。

如何分析和排查？
* 检查你的业务代码，是否存在集中过期 key 的逻辑。
* 集中过期 key 增加一个随机过期时间，把集中过期的时间打散，降低 Redis 清理过期 key 的压力
* 如果你使用的 Redis 是 4.0 以上版本，可以开启 lazy-free 机制，当删除过期 key 时，把释放内存的操作放到后台线程中执行，避免阻塞主线程

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-15/105911515889916.png?Expires=4898316664&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=nLHIWcDDfwocWYJI4F56lfA1nEA%3D)

运维层面，你需要把 Redis 的各项运行状态数据监控起来，在 Redis 上执行 INFO 命令就可以拿到这个实例所有的运行状态数据。
在这里我们需要重点关注 expired_keys 这一项，它代表整个实例到目前为止，累计删除过期 key 的数量。

---

### 检查实例内存达到上限
如果你的 Redis 实例设置了内存上限 maxmemory，那么也有可能导致 Redis 变慢。

当实例的内存达到了 maxmemory 后，你可能会发现，在此之后每次写入新数据，操作延迟变大了。

原因在于，当 Redis 内存达到 maxmemory 后，每次写入新的数据之前，Redis 必须先从实例中踢出一部分数据，让整个实例的内存维持在 maxmemory 之下，然后才能把新数据写进来。
这个踢出旧数据的逻辑也是需要消耗时间的，而具体耗时的长短，要取决于你配置的淘汰策略。

需要注意的是，Redis 的淘汰数据的逻辑与删除过期 key 的一样，也是在命令真正执行之前执行的，也就是说它也会增加我们操作 Redis 的延迟，而且，写 OPS 越高，延迟也会越明显。

如何避免？
* 避免存储 bigkey，降低释放内存的耗时
* 淘汰策略改为随机淘汰，随机淘汰比 LRU 要快很多（视业务情况调整）
* 拆分实例，把淘汰 key 的压力分摊到多个实例上
* 如果使用的是 Redis 4.0 以上版本，开启 layz-free 机制，把淘汰 key 释放内存的操作放到后台线程中执行（配置 lazyfree-lazy-eviction = yes）

---

### 检查fork耗时严重
为了保证 Redis 数据的安全性，我们可能会开启后台定时 RDB 和 AOF rewrite 功能。但如果你发现，操作 Redis 延迟变大，都发生在 Redis 后台 RDB 和 AOF rewrite 期间，那你就需要排查，在这期间有可能导致变慢的情况。

当 Redis 开启了后台 RDB 和 AOF rewrite 后，在执行时，它们都需要主进程创建出一个子进程进行数据的持久化。  
主进程创建子进程，会调用操作系统提供的 fork 函数。

而 fork 在执行过程中，主进程需要拷贝自己的内存页表给子进程，如果这个实例很大，那么这个拷贝的过程也会比较耗时。  
而且这个 fork 过程会消耗大量的 CPU 资源，在完成 fork 之前，整个 Redis 实例会被阻塞住，无法处理任何客户端请求。  
如果此时你的 CPU 资源本来就很紧张，那么 fork 的耗时会更长，甚至达到秒级，这会严重影响 Redis 的性能。

如何确认确实是因为 fork 耗时导致的 Redis 延迟变大呢？

你可以在 Redis 上执行 `INFO` 命令，查看 `latest_fork_usec` 项，单位微秒。
这个时间就是主进程在 fork 子进程期间，整个实例阻塞无法处理客户端请求的时间。
如果你发现这个耗时很久，就要警惕起来了，这意味在这期间，你的整个 Redis 实例都处于不可用的状态。

要想避免这种情况，你可以采取以下方案进行优化：
* 控制 Redis 实例的内存：尽量在 10G 以下，执行 fork 的耗时与实例大小有关，实例越大，耗时越久
* 合理配置数据持久化策略：在 slave 节点执行 RDB 备份，推荐在低峰期执行，而对于丢失数据不敏感的业务（例如把 Redis 当做纯缓存使用），可以关闭 AOF 和 AOF rewrite
* Redis 实例不要部署在虚拟机上：fork 的耗时也与系统也有关，虚拟机比物理机耗时更久
* 降低主从库全量同步的概率：适当调大 repl-backlog-size 参数，避免主从全量同步

---

### 禁止开启内存大页
我们都知道，应用程序向操作系统申请内存时，是按内存页进行申请的，而常规的内存页大小是 4KB。  
Linux 内核从 2.6.38 开始，支持了内存大页机制，该机制允许应用程序以 2MB 大小为单位，向操作系统申请内存。  
应用程序每次向操作系统申请的内存单位变大了，但这也意味着申请内存的耗时变长。

这对 Redis 会有什么影响呢？

当 Redis 在执行后台 RDB 和 AOF rewrite 时，采用 fork 子进程的方式来处理。但主进程 fork 子进程后，此时的主进程依旧是可以接收写请求的，而进来的写请求，会采用 `Copy On Write`（写时复制）的方式操作内存数据。  
也就是说，主进程一旦有数据需要修改，Redis 并不会直接修改现有内存中的数据，而是先将这块内存数据拷贝出来，再修改这块新内存的数据，这就是所谓的「写时复制」。  
写时复制你也可以理解成，谁需要发生写操作，谁就需要先拷贝，再修改。  
这样做的好处是，父进程有任何写操作，并不会影响子进程的数据持久化（子进程只持久化 fork 这一瞬间整个实例中的所有数据即可，不关心新的数据变更，因为子进程只需要一份内存快照，然后持久化到磁盘上）。  
但是请注意，主进程在拷贝内存数据时，这个阶段就涉及到新内存的申请，如果此时操作系统开启了内存大页，那么在此期间，客户端即便只修改 10B 的数据，Redis 在申请内存时也会以 2MB 为单位向操作系统申请，申请内存的耗时变长，进而导致每个写请求的延迟增加，影响到 Redis 性能。

如何解决这个问题？

只需要关闭内存大页机制就可以了。

查看 Redis 机器是否开启了内存大页:
```shell
[root@VM-8-17-centos ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
```
如果输出选项是 always，就表示目前开启了内存大页机制，我们需要关掉它：
```shell
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```
操作系统提供的内存大页机制，其优势是，可以在一定程序上降低应用程序申请内存的次数。  
但是对于 Redis 这种对性能和延迟极其敏感的数据库来说，我们希望 Redis 在每次申请内存时，耗时尽量短，所以我不建议你在 Redis 机器上开启这个机制。

---

### 注意开启AOF
当 Redis 开启 AOF 后，其工作原理如下：
1. Redis 执行写命令后，把这个命令写入到 AOF 文件内存中（write 系统调用）
2. Redis 根据配置的 AOF 刷盘策略，把 AOF 内存数据刷到磁盘上（fsync 系统调用）

为了保证 AOF 文件数据的安全性，Redis 提供了 3 种刷盘机制：
* appendfsync always：主线程每次执行写操作后立即刷盘，此方案会占用比较大的磁盘 IO 资源，但数据安全性最高
* appendfsync no：主线程每次写操作只写内存就返回，内存数据什么时候刷到磁盘，交由操作系统决定，此方案对性能影响最小，但数据安全性也最低，Redis 宕机时丢失的数据取决于操作系统刷盘时机
* appendfsync everysec：主线程每次写操作只写内存就返回，然后由后台线程每隔 1 秒执行一次刷盘操作（触发fsync系统调用），此方案对性能影响相对较小，但当 Redis 宕机时会丢失 1 秒的数据

如果你的 AOF 配置为 appendfsync always，这个过程必然会加重 Redis 写负担。  
如果你的 AOF 配置为 appendfsync no，此方案对 Redis 的性能影响最小，但当 Redis 宕机时，会丢失一部分数据，为了数据的安全性，一般我们也不采取这种配置。

appendfsync everysec这种方案还是存在导致 Redis 延迟变大的情况发生，甚至会阻塞整个 Redis。

试想这样一种情况：当 Redis 后台线程在执行 AOF 文件刷盘时，如果此时磁盘的 IO 负载很高，那这个后台线程在执行刷盘操作（fsync系统调用）时就会被阻塞住。

此时的主线程依旧会接收写请求，紧接着，主线程又需要把数据写到文件内存中（write 系统调用），但此时的后台子线程由于磁盘负载过高，导致 fsync 发生阻塞，迟迟不能返回，那主线程在执行 write 系统调用时，也会被阻塞住，直到后台线程 fsync 执行完成后，主线程执行 write 才能成功返回。

![](http://alexali.oss-cn-guangzhou.aliyuncs.com/pasteimageintomarkdown/2025-04-15/107066709026791.png?Expires=4898317820&OSSAccessKeyId=LTAI5tBX2zkmA8G3Aw5HNqtH&Signature=bEq2S9xsD2R0D8yKLLJK6d%2FgBHs%3D)

那什么情况下会导致磁盘 IO 负载过大？
* 子进程正在执行 AOF rewrite，这个过程会占用大量的磁盘 IO 资源
  > 说白了就是，Redis 的 AOF 后台子线程刷盘操作，撞上了子进程 AOF rewrite！  
  > Redis 提供了一个配置项，当子进程在 AOF rewrite 期间，可以让后台子线程不执行刷盘（不触发 fsync 系统调用）操作。  
  > 这相当于在 AOF rewrite 期间，临时把 appendfsync 设置为了 none，配置如下：
  > ```redis
  > # AOF rewrite 期间，AOF 后台子线程不进行刷盘操作
  > # 相当于在这期间，临时把 appendfsync 设置为了 none
  > no-appendfsync-on-rewrite yes
  > ```
  > 当然，开启这个配置项，在 AOF rewrite 期间，如果实例发生宕机，那么此时会丢失更多的数据，性能和数据安全性，你需要权衡后进行选择。
* 有其他应用程序在执行大量的写文件操作，也会占用磁盘 IO 资源
  > 需要定位到是哪个应用程序在大量写磁盘，然后把这个应用程序迁移到其他机器上执行就好了，避免对 Redis 产生影响。

---

### 绑定CPU
很多时候，我们在部署服务时，为了提高服务性能，降低应用程序在多个 CPU 核心之间的上下文切换带来的性能损耗，通常采用的方案是进程绑定 CPU 的方式提高性能。

一般现代的服务器会有多个 CPU，而每个 CPU 又包含多个物理核心，每个物理核心又分为多个逻辑核心，每个物理核下的逻辑核共用 L1/L2 Cache。

而 Redis Server 除了主线程服务客户端请求之外，还会创建子进程、子线程。
其中子进程用于数据持久化，而子线程用于执行一些比较耗时操作，例如异步释放 fd、异步 AOF 刷盘、异步 lazy-free 等等。

如果你把 Redis 进程只绑定了一个 CPU 逻辑核心上，那么当 Redis 在进行数据持久化时，fork 出的子进程会继承父进程的 CPU 使用偏好。

而此时的子进程会消耗大量的 CPU 资源进行数据持久化（把实例数据全部扫描出来需要耗费CPU），这就会导致子进程会与主进程发生 CPU 争抢，进而影响到主进程服务客户端请求，访问延迟变大。

> 那如何解决这个问题呢？

如果你确实想要绑定 CPU，可以优化的方案是，不要让 Redis 进程只绑定在一个 CPU 逻辑核上，而是绑定在多个逻辑核心上，而且，绑定的多个逻辑核心最好是同一个物理核心，这样它们还可以共用 L1/L2 Cache。

当然，即便我们把 Redis 绑定在多个逻辑核心上，也只能在一定程度上缓解主线程、子进程、后台线程在 CPU 资源上的竞争。
因为这些子进程、子线程还是会在这多个逻辑核心上进行切换，存在性能损耗。

> 如何再进一步优化？

我们是否可以让主线程、子进程、后台线程，分别绑定在固定的 CPU 核心上，不让它们来回切换，这样一来，他们各自使用的 CPU 资源互不影响。

其实，这个方案 Redis 官方已经想到了。

Redis 在 6.0 版本已经推出了这个功能，我们可以通过以下配置，对主线程、后台线程、后台 RDB 进程、AOF rewrite 进程，绑定固定的 CPU 逻辑核心：

```redis
# Redis Server 和 IO 线程绑定到 CPU核心 0,2,4,6
server_cpulist 0-7:2

# 后台子线程绑定到 CPU核心 1,3
bio_cpulist 1,3

# 后台 AOF rewrite 进程绑定到 CPU 核心 8,9,10,11
aof_rewrite_cpulist 8-11

# 后台 RDB 进程绑定到 CPU 核心 1,10,11
bgsave_cpulist 1,10-1
```

一般来说，Redis 的性能已经足够优秀，除非你对 Redis 的性能有更加严苛的要求，否则不建议你绑定 CPU。

从上面的分析你也能看出，绑定 CPU 需要你对计算机体系结构有非常清晰的了解，否则谨慎操作。

---

### 禁止使用Swap
什么是 Swap？为什么使用 Swap 会导致 Redis 的性能下降？

如果你对操作系统有些了解，就会知道操作系统为了缓解内存不足对应用程序的影响，允许把一部分内存中的数据换到磁盘上，以达到应用程序对内存使用的缓冲，这些内存数据被换到磁盘上的区域，就是 Swap。

问题就在于，当内存中的数据被换到磁盘上后，Redis 再访问这些数据时，就需要从磁盘上读取，访问磁盘的速度要比访问内存慢几百倍！

需要检查 Redis 机器的内存使用情况，确认是否存在使用了 Swap。
```shell
# 先找到 Redis 的进程 ID
ps -aux | grep redis-server

# 查看 Redis Swap 使用情况
cat /proc/$pid/smaps | egrep '^(Swap|Size)'
```
这个结果会列出 Redis 进程的内存使用情况。
```shell
[root@VM-8-17-centos ~]# cat /proc/2605/smaps | egrep '^(Swap|Size)'
Size:               2856 kB
Swap:                  0 kB
Size:                  4 kB
Swap:                  0 kB
Size:                276 kB
Swap:                  0 kB
Size:               2224 kB
Swap:                  0 kB
Size:                132 kB
Swap:                  0 kB
Size:                132 kB
Swap:                  0 kB
Size:              65404 kB
Swap:                  0 kB
Size:                  4 kB
Swap:                  0 kB
Size:               8192 kB
Swap:                  0 kB
```
每一行 Size 表示 Redis 所用的一块内存大小，Size 下面的 Swap 就表示这块 Size 大小的内存，有多少数据已经被换到磁盘上了，如果这两个值相等，说明这块内存的数据都已经完全被换到磁盘上了。

如果只是少量数据被换到磁盘上，例如每一块 Swap 占对应 Size 的比例很小，那影响并不是很大。如果是几百兆甚至上 GB 的内存被换到了磁盘上，那么你就需要警惕了，这种情况 Redis 的性能肯定会急剧下降。

解决方案是：
* 增加机器的内存，让 Redis 有足够的内存可以使用
* 整理内存空间，释放出足够的内存供 Redis 使用，然后释放 Redis 的 Swap，让 Redis 重新使用内存

释放 Redis 的 Swap 过程通常要重启实例，为了避免重启实例对业务的影响，一般会先进行主从切换，然后释放旧主节点的 Swap，重启旧主节点实例，待从库数据同步完成后，再进行主从切换即可。

可见，当 Redis 使用到 Swap 后，此时的 Redis 性能基本已达不到高性能的要求（你可以理解为武功被废），所以你也需要提前预防这种情况。

预防的办法就是，你需要对 Redis 机器的内存和 Swap 使用情况进行监控，在内存不足或使用到 Swap 时报警出来，及时处理。

---

### 碎片整理
Redis 的数据都存储在内存中，当我们的应用程序频繁修改 Redis 中的数据时，就有可能会导致 Redis 产生内存碎片。

内存碎片会降低 Redis 的内存使用率，我们可以通过执行 INFO 命令，得到这个实例的内存碎片率：
```redis
Memory
used_memory:5709194824
used_memory_human:5.32G
used_memory_rss:8264855552
used_memory_rss_human:7.70G
...
mem_fragmentation_ratio:1.45
```

这个内存碎片率是怎么计算的？

很简单，mem_fragmentation_ratio = used_memory_rss / used_memory。

其中 used_memory 表示 Redis 存储数据的内存大小，而 used_memory_rss 表示操作系统实际分配给 Redis 进程的大小。

如果 mem_fragmentation_ratio > 1.5，说明内存碎片率已经超过了 50%，这时我们就需要采取一些措施来降低内存碎片了。

解决的方案一般如下：
* 如果你使用的是 Redis 4.0 以下版本，只能通过重启实例来解决
* 如果你使用的是 Redis 4.0 版本，它正好提供了自动碎片整理的功能，可以通过配置开启碎片自动整理
  > 但是，开启内存碎片整理，它也有可能会导致 Redis 性能下降。  
  > 原因在于，Redis 的碎片整理工作是也在主线程中执行的，当其进行碎片整理时，必然会消耗 CPU 资源，产生更多的耗时，从而影响到客户端的请求。

你需要结合 Redis 机器的负载情况，以及应用程序可接受的延迟范围进行评估，合理调整碎片整理的参数，尽可能降低碎片整理期间对 Redis 的影响。

Redis 碎片整理的参数配置如下：
```redis
# 开启自动内存碎片整理（总开关）
activedefrag yes

# 内存使用 100MB 以下，不进行碎片整理
active-defrag-ignore-bytes 100mb

# 内存碎片率超过 10%，开始碎片整理
active-defrag-threshold-lower 10
# 内存碎片率超过 100%，尽最大努力碎片整理
active-defrag-threshold-upper 100

# 内存碎片整理占用 CPU 资源最小百分比
active-defrag-cycle-min 1
# 内存碎片整理占用 CPU 资源最大百分比
active-defrag-cycle-max 25

# 碎片整理期间，对于 List/Set/Hash/ZSet 类型元素一次 Scan 的数量
active-defrag-max-scan-fields 1000
```

---

### 网络带宽过载
如果以上产生性能问题的场景，你都规避掉了，而且 Redis 也稳定运行了很长时间，但在某个时间点之后开始，操作 Redis 突然开始变慢了，而且一直持续下去，这种情况又是什么原因导致？

此时你需要排查一下 Redis 机器的网络带宽是否过载，是否存在某个实例把整个机器的网路带宽占满的情况。

网络带宽过载的情况下，服务器在 TCP 层和网络层就会出现数据包发送延迟、丢包等情况。

Redis 的高性能，除了操作内存之外，就在于网络 IO 了，如果网络 IO 存在瓶颈，那么也会严重影响 Redis 的性能。

如果确实出现这种情况，你需要及时确认占满网络带宽 Redis 实例，如果属于正常的业务访问，那就需要及时扩容或迁移实例了，避免因为这个实例流量过大，影响这个机器的其他实例。

运维层面，你需要对 Redis 机器的各项指标增加监控，包括网络流量，在网络流量达到一定阈值时提前报警，及时确认和扩容。

---

### 频繁短连接
你的业务应用，应该使用长连接操作 Redis，避免频繁的短连接。

频繁的短连接会导致 Redis 大量时间耗费在连接的建立和释放上，TCP 的三次握手和四次挥手同样也会增加访问延迟。

---

### 运维监控
监控其实就是对采集 Redis 的各项运行时指标，通常的做法是监控程序定时采集 Redis 的 INFO 信息，然后根据 INFO 信息中的状态数据做数据展示和报警。

在写监控脚本访问 Redis 时，尽量采用长连接的方式采集状态信息，避免频繁短连接。同时，你还要注意控制访问 Redis 的频率，避免影响到业务请求。

在使用一些开源的监控组件时，最好了解一下这些组件的实现原理，以及正确配置这些组件，防止出现监控组件发生 Bug，导致短时大量操作 Redis，影响 Redis 性能的情况发生。

我们当时就发生过，DBA 在使用一些开源组件时，因为配置和使用问题，导致监控程序频繁地与 Redis 建立和断开连接，导致 Redis 响应变慢。

---

### 其它程序争抢资源
你的 Redis 机器最好专项专用，只用来部署 Redis 实例，不要部署其他应用程序，尽量给 Redis 提供一个相对「安静」的环境，避免其它程序占用 CPU、内存、磁盘资源，导致分配给 Redis 的资源不足而受到影响。

---

### 总计
* CPU 相关：使用复杂度过高命令、数据的持久化，都与耗费过多的 CPU 资源有关
* 内存相关：bigkey 内存的申请和释放、数据过期、数据淘汰、碎片整理、内存大页、内存写时复制都与内存息息相关
* 磁盘相关：数据持久化、AOF 刷盘策略，也会受到磁盘的影响
* 网络相关：短连接、实例流量过载、网络流量过载，也会降低 Redis 性能
* 计算机系统：CPU 结构、内存分配，都属于最基础的计算机系统知识
* 操作系统：写时复制、内存大页、Swap、CPU 绑定，都属于操作系统层面的知识












