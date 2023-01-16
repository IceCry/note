### redis入门指南（第二版）

#### 介绍

> Redis （remote dictionary server 远程字典服务器）是一个开源的、高性能的、基于键值对的缓存与存储系统，通过提供多种键值数据类型来适应不同场景下的缓存和存储需求。同时redis的诸多高层级功能使其可以胜任消息队列、任务队列等不同的角色。

开发时间：2009，vmware自2010年开始赞助

TTL：time to live（生存时间到期后键会自动删除）

redis是单线程模型，memcached支持多线程，所以在多核服务器上memcached理论上性能更高。随着redis 3.0推出，标志memcached几乎所有功能都成为了redis的子集。同时，redis对集群的支持使得memcached原有的第三方集群工具不再成为优势。

redis版本约定：次版本为偶数则为稳定版；

| 文件名           | 说明                |
| ---------------- | ------------------- |
| redis-server     | 服务器              |
| redis-cli        | 命令行客户端        |
| redis-benchark   | 性能测试工具        |
| redis-check-aof  | aof文件修复工具     |
| redis-check-dump | rdb文件检查工具     |
| redis-sentinel   | sentinel服务器 2.8+ |

启动：redis-server [--port 6379]

> 6379 对应手机键盘MERZ（意大利歌女）

停止：redis-cli shutdown

> redis收到shutdown命令后，先断开客户端连接，后根据配置执行持久化，最后退出。
>
> redis可以妥善处理sigterm信号，故使用kill pid方式也可正常结束。

**redis返回值：**

- 状态回复
- 错误回复
- 整数回复
- 字符串回复
- 多行字符串回复

通过启动参数传递同名的配置项会覆盖配置文件中对应的参数：`redis-server /path/redis.conf --loglevel warning`

动态修改redis配置：`config set loglevel warning`

redis默认支持16个数据库；不支持修改名字；不支持为每个数据库设置不同密码；

> 多个数据库之间并不是完全隔离，比如flushall命令可以清空一个redis实例中所有数据库的数据。
>
> 数据库更像是命名空间，不适宜存储不同应用程序的数据。



#### 数据类型

1. keys pattern

   > pattern支持glob风格通配符格式：?匹配一个字符、*任意个字符、[]匹配括号内的任一字符，支持[a-z]范围、\x匹配字符x，用于转义；
   >
   > keys需遍历所有键，当键数量较多时影响性能；

2. set key value

3. exists key

4. del key [key ...]

   > del命令参数不支持通配符，但可结合linux管道和xargs命令实现批量删除。
   >
   > keys "user:*" | xargs redis-cli DEL
   >
   > del `keys "user:*"`

5. type key



##### 字符串类型

字符串类型是redis最基础的数据类型，它能存储任何形式的字符串，包括二进制数据。一个字符串类型键允许存储数据的最大容量是512M。

1. incr key

   > 竞态条件（race condition）：指一个系统或进程输出，依赖于不受控制的事件的出现顺序或出现时机。
   >
   > 所有redis命令都是原子操作（atomic operation）

2. incrby key increment

3. incrbyfloat bar 2.7 #增加双精度浮点数

4. append key value #追加

5. strlen key

6. mget key [key...] / mset key value [key value] #同时获取或设置多个值

7. 位操作：getbit key offset / setbit key offset value；bitcount key（获取字符串类型键中值位1的二进制位个数），bitcount foo 0 1（限制统计的字节范围）;bitop对多个字符串类型键进行位运算，并将结果存储在destkey参数指定的键：bitop OR res foo1 foo2；



> 键命名建议：对象类型:对象ID:对象属性，如user:1:friends存储用户1的好友列表。



##### 散列类型

散列类型适合存储对象：使用对象类别和ID构成键名，使用字段表示对象的属性，而字段值则存储属性值。

1. **赋值和取值**

   hset key field value

   hget key field

   hmset key  field value [field value ...]

   hmget key field [field ...]

   hgetall key

   > hset 不区分插入或更新，当执行插入时返回1，更新返回0

2. **判断字段是否存在**

   hexists key field #存在返回1，否则0

3. **当字段不存在时赋值**

   hsetnx key field value #与hset类型，如存在则不更新（NX：if not exists）

4. **增加数字**

   hincrby key field increment

5. **删除字段**

   hdel key field [field ...]

6. **其他命令**

   hkeys key / hvals key #只获取字段名或字段值

   hlen key #获取字段数量



##### 列表类型

列表类型list可以存储一个有序的字符串列表，常用的操作是向列表两端添加元素或者获取一个列表的某一个片段。

列表类型内部是使用双向链表实现的（double linked list），所以向列表两端添加元素的时间复杂度位O(1)，获取越接近两端的元素速度就越快。

1. lpush key value [value...] / rpush #向列表两端添加元素，lpush xx 1 2 3（3 2 1）
2. lpop key / rpop key
3. llen key
4. lrange key start stop #获取列表片段（**含两端**）
5. lrem key count value #删除列表中前count个值为value的元素，返回删除数。count>0左侧删除;count<0右侧；count=0删除所有
6. lindex key index / lset key index value #获取/设置指定索引的元素值
7. ltrim key start end #只保留列表指定片段，如保存近100条日志：lpush logs xxx; ltrim logs 0 99
8. linsert key before|after pivot value #向列表插入数据，自左向右查询pivot值并插入
9. rpoplpush source destination #将元素从一个列表转移到另一个列表

> 往返延时（round-trip delay time）



##### 集合类型

集合中每个元素都是不同的，且无序。

1. sadd key member [member ...] / srem key member [member ...] #增加/删除元素

2. smembers key #获取所有元素

3. sismember key member #判断元素是否在集合中

4. sdiff key [key...] / sinter key [key ...] / sunion key [key ...] #差集 / 交集 / 并集

5. scard key #获取集合中元素的个数

6. sdiffstore|sinterstore|sunionstore destionation key [key...] #进行集合运算并存储结果

7. srandmember key [count] #随机（非绝对随机）获取集合中的元素，count>0不重复；count<0可能重复；

   > 由于集合类型采用的存储结构（散列表）导致，散列表使用散列函数将元素映射到不同的存储位置（桶）上以实现O(1)时间复杂度的元素查找。当不同元素的散列值相同时会出现冲突，redis使用拉链法来解决冲突，即将散列值冲突的元素以链表的形式存入一个桶中，查找元素时先找到元素对应的桶，然后再从桶中的链表中找到对应的元素。
   >
   > 使用srandmemeber命令从集合中获得一个随机元素时，redis首先会从所有桶中随机选择一个桶，然后再从桶中的所有元素中随机选择一个元素，所以元素所在的桶中的元素数量越少，其被随机选中的可能性就越大。

8. spop key #随机弹出



##### 有序集合类型（sorted set）

> 列表类型是通过链表实现的，获取靠近两端的数据速度极快，而当元素增多后，访问中间数据的速度比较慢，所以更适合如“新鲜事”、“日志”这种很少访问中间元素的应用。
>
> 有序集合类型是使用散列表和跳跃表（skip list）实现的，所以即使读取中间部分数据也很快。
>
> 列表不能简单的调整某个元素的位置，但有序集合可以通过调整元素分数实现。
>
> 有序集合比列表更耗内存。

1. zdd key score member [score member ...] #支持整数、双精度浮点数、+inf/-inf

2. zscore key member #获取元素分数

3. zrange|zrevrange key start stop [withscores]  #获取排名在某个范围的元素列表，zrange从小到大，zrevrange从大到小；

   > 如果两个元素分数相同，redis会按字典顺序（0<9<A<Z<a<z）排序。中文排序依据中文的编码方式。

4. zrangebyscore|zrevrangebyscore key min max [withscores] [limit offset count] #获取指定分数范围的元素

   > 如果希望分数范围不含端值，可在分数前加“(”，zrangebyscore key 80 (100 #80<=x<100
   >
   > zrangebyscore key (80 +inf
   >
   > limit offset count 在获得元素列表的基础上向后偏移offset个元素，且只获取前count个元素。

5. zincrby key increment member #增加某个元素分数

6. zcard key #获取集合中元素数量

7. zcount key min max #获取指定分数范围的元素个数

8. zrem key member [member ...] #删除一个或多个元素

9. zremrangebyrank key start stop #按照排名范围删除元素

10. zremrangebyscore key min max #按分数范围删除

11. zrank|zrevrank key memeber #获取元素排名

12. zinterstore destination numkeys key [key...] [weights weight [weight ...] ] [aggregate sum|min|max] #计算有序集合的交集



#### 事务

> redis中的事务（transaction）是一组命令的集合。

**错误处理**

- 语法错误：任何错误将导致不执行（2.6.5之前会忽略有语法错误的命令，然后执行正确的命令）
- 运行错误：忽略错误，执行正确命令

> redis事务无回滚功能，需开发者自行恢复。



**watch**

> watch命令可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到exec命令。
>
> watch命令作用只是当被监控的键值被修改后阻止之后的一个事务的执行，而不能保证其他客户端不修改这一键值，所以我们需要在exec执行失败后重新执行整个函数。



#### **过期时间**

1. expire key secs #设置过期时间
2. ttl key #查看剩余时间，当键不存在时返回-2；无过期时间返回-1；（v2.6 无论键不存在还是没有过期都返回-1，v2.8+后才区分）
3. persist key #取消过期时间，使用 set 或 getset 同样；
4. pexpire key microsec #毫秒过期时间
5. pttl
6. expireat|pexpireat #指定过期unix时间戳



> 如果使用watch检测一个拥有过期时间的键，该键到期自动删除并不会被watch命令认为该键值被改变。

> 修改maxmemory参数，限制redis最大可用内存大小（字节），当超出限制时，redis会依据maxmemory-policy参数指定的策略来删除不需要的键直到redis占用的内存小于指定内存。

| 规则            | 说明                                            |
| --------------- | ----------------------------------------------- |
| volatile-lru    | 使用LRU算法删除一个键（只针对设置过期时间的键） |
| allkeys-lru     | 使用LRU算法删除一个键                           |
| volatile-random | 随机删除（只针对设置过期时间的键）              |
| allkeys-random  | 随机删除                                        |
| volatile-ttl    | 删除过期时间最近的一个键                        |
| noeviction      | 不删除，只返回错误                              |

> 事实上，redis并不会准确的将整个数据库中最久未被使用的键删除，而是每次从数据库中随机取出5个键并删除其中最久未被使用的。“5”可以通过redis配置文件中的“maxmemory-samples”设置。

#### 排序

7. sort key #排序 sort key desc limit 1 2
8. sort key alpha #按字典顺序排序非数字元素
9. 



#### 消息通知

**队列**



**节省空间**

- 精简键名和键值
- 内部编码优化：`object encoding foo`查看一个键内部的编码方式

| 数据类型     | 内部编码方式              | 命令结果   |
| ------------ | ------------------------- | ---------- |
| 字符串类型   | REDIS_ENCODING_RAW        | raw        |
|              | REDIS_ENCODING_INT        | int        |
|              | REDIS_ENCODING_EMBSTR     | embstr     |
| 散列类型     | REDIS_ENCODING_HT         | hashtable  |
|              | REDIS_ENCODING_ZIPLIST    | ziplist    |
| 列表类型     | REDIS_ENCODING_LINKEDLIST | linkedlist |
|              | REDIS_ENCODING_ZIPLIST    | ziplist    |
| 集合类型     | REDIS_ENCODING_HT         | hashtable  |
|              | REDIS_ENCODING_INTSET     | intset     |
| 有序集合类型 | REDIS_ENCODING_SKIPLIST   | skiplist   |
|              | REDIS_ENCODING_ZIPLIST    | ziplist    |



#### 脚本

redis在2.6版推出了脚本功能，允许开发者使用lua语言编写脚本传到redis中执行。使用脚本的好处：

- 减少网络开销
- 原子操作（redis将整个脚本作为一个整体执行，无需担心出现竞态条件，无需使用事务）
- 复用（客户端发送的脚本会永久存储在redis中，其他项目可复用）

访问频率限制：

```lua
local times = redis.call('incr', KEYS[1])

if times == 1 then
	redis.call('expire', KEYS[1], ARGV[1])
end

if times > tonumber(ARGV[2]) then
	return 0
end

return 1
```

执行方式：

`redis-cli --eval e:\test.lua rate.limiting:127.0.0.1 , 10 3`



**Lua语言**

lua（西班牙语：月亮）是一个高效的轻量级脚本语言。lua是一个“卫星语言”，能够方便的嵌入到其他语言中使用。

**数据类型**

lua是一个动态类型语言，一个变量可以存储任何类型的值。

| 类型名        | 取值                                                         |
| ------------- | ------------------------------------------------------------ |
| 空 nil        | nil表示空，所有没赋值的变量或表的字段都是nil                 |
| 布尔 boolean  | true/false                                                   |
| 数字 number   | 整数、浮点数                                                 |
| 字符串 string |                                                              |
| 表 table      | 表类型是lua语言中唯一的数据结构，既可以当数组又可以当字典，十分灵活 |
| 函数 function | 函数在lua中是一等值（first-class value），可以存储在变量中、作为函数的参数或返回结果 |

**变量**

全局变量 a = 1

局部变量 local b = 2

> redis脚本中不能使用全局变量，防止脚本直接相互影响。

**注释**

-- 单行注释 --[[ ... ]] 多行注释

**赋值**

lua支持多重赋值：

local a, b = 1, 2 -- a=1 b=2

local a, b = 1, 2, 3 -- 3被舍弃

local a, b = 1 -- a=1 b=nil

**操作符**

- 数学操作符：+ - * / % - ^

- 比较运算符：== ~= < > <= >=（==类型与值都相等）

- 逻辑操作符：not and or

  > lua中只有nil和false为假；即使是0或空字符串也被当作真，故：
  >
  > redis.call('exists', 'key') == 1 判断是否存在

- 连接操作符：..（'hello'..' '..'world!'）

- 取长度操作符：# （  print(#'hello')  --5  ）

**if语句**

```lua
if 条件表达式 then
    语句块
elseif 条件表达式 then
    语句块
else
    语句块
end
```

**循环语句**

lua支持while repeat for循环语句。

```lua
while xxx do
    ...
end

repeat
    ...
until xxx

--数字形式
for 变量 = 初值, 终值, 步长 do
    ...
end

--通用形式
for var1, var2 ... varN in 迭代器 do
    ...
end
```

**表类型**

表是lua中唯一的数据结构，可理解为关联数组，任何类型的值（除空类型）都可作为表的索引。

> lua约定数组的索引是从1开始的

> ipairs 是lua内置的函数，实现类似迭代器的功能。pairs和ipairs的区别在于前者会遍历所有值不为nil的索引，而后者只会从索引1开始递增遍历到最后一个值不为nil的整数索引。

**函数**

```lua
function (args)
    ...
end

local square = function (num)
    return num * num
end
```

**标准库**

redis支持的lua标准库

| 库名   | 说明                       |
| ------ | -------------------------- |
| Base   | 提供了一些基础函数         |
| String | 提供了用于字符串操作的函数 |
| Table  | 提供了用于表操作的函数     |
| Math   | 提供了数学计算函数         |
| Debug  | 提供了用于调试的函数       |

**String库**

string库的函数可以通过字符串类型的变量以面向对象的形式访问，如string.len(string_var)可以写成string_var:len()

- 获取字符串长度：string.len(string) / string.len() / #
- 转大小写：string.lower(string) / string.upper(string)
- 获取子字符串：string.sub(string, start [, end])

**Table库**

table库中大部分函数都需要表的形式是数组形式。

- 将数组转为字符串：table.concat(table [, sep [, i [, j]]]) #i和j用来限制要转换的表元素的索引范围，默认为1和表的长度，不支持负索引。如：`print( table.concat({1, 2, 3}, ',', 2, 2) ) #2`
- 向数组中插入元素：table.insert(table, [pos,] value) #pos默认为数组长度+1。
- 从数组中弹出一个元素：table.remove(table [,pos])，默认尾部删除

**Math库**

含：abs / sin / cos / tan / ceil / floor / max / min / pow / sqrt / random / randomseed

math.random([m [, n]])：未提供参数则返回[0, 1)的实数；只提供m返回[1, m]整数；同时提供m和n返回[m, n]整数。

**其他库**

除标准库外，redis还通过cjson库和cmsgpack库提供了对json和MessagePack的支持。redis自动加载了这两个库，在脚本中可通过cjson和cmspack两个全局变量来访问对应的库。



**redis与lua**

1. 在脚本中调用redis命令
   redis.call('set', 'foo', 'bar')
   local value = redis.call('get', 'foo')
   redis返回值类型与lua数据类型转换规则

   | redis返回值类型 | lua数据类型                           |
   | --------------- | ------------------------------------- |
   | 整数回复        | 数字类型                              |
   | 字符串回复      | 字符串类型                            |
   | 多行字符串回复  | 表类型（数组形式）                    |
   | 状态回复        | 表类型（只有一个ok字段存储状态信息）  |
   | 错误回复        | 表类型（只有一个err字段存储错误信息） |

   > redis还提供redis.pcall函数，当命令执行出错时会记录错误并继续执行，而redis.call会直接返回错误，不会继续执行。

2. 从脚本中返回值
   脚本中使用return返回给客户端，没有return则返回nil，redis会自动将脚本返回值的lua数据类型转为redis的返回值类型。（lua中的false会被转为空结果）

3. 脚本相关命令
   3.1 redis提供eval命令可以像内置命令一样调用脚本。
   格式为：eval 脚本内容 key参数的数量[key...] [arg...]，通过key和arg这两类参数向脚本传递数据，它们的值可以在脚本中分别使用KEYS和ARGV两个表类型的全局变量访问。如：`eval "return redis.call('set', keys[1], argv[1])" 1 foo bar`

   > eval命令依据第二个参数将后面的所有参数分别存入脚本中的keys和argv两个表的全局变量。当脚本不需要任何参数时需设为0

   3.2 evalsha命令
   考虑脚本比较长的情况，如果每次调用脚本都需要将整个脚本传给redis会占用较多的带宽。为了解决这个问题，redis提供了evalsha命令允许开发者通过脚本内容的sha1摘要来执行脚本。
   redis在执行eval命令时会计算脚本的sha1并记录在脚本缓存中，执行evalsha根据缓存查找脚本内容，未找到返回“noscript no match script. please use eval"

**应用实例**





#### 集群

**问题**

- 从结构上讲，单个redis发生单点故障，同一服务器需要承受所有的请求负载；
- 从容量上，单个redis服务器的内存非常容易成为存储瓶颈，所以需要进行数据分片。

##### 1. 复制

为避免单点故障，将数据库复制多个副本以部署到不同的服务器上。为此，redis提供了复制（replication）功能，实现当一台数据库中的数据更新后，自动同步到其他数据库。

**配置**

主库master可读写，从库slave一般为只读。

> 开启方式：从库配置文件中添加 `slaveof 主库地址 主库端口` 即可，主库无需配置。

查看相关信息：info replication

> 默认从库只读，不可修改从库数据，可通过设置从库的slave-read-only为no使从库可写。
>
> 但从库任何更改不会同步给其他数据库，主库更新相应数据会覆盖从库改动，故通常场景下不应该设置从库可写。

**原理**

当从库启动后，会向主库发送sync命令。主库收到sync命令后会开始在后台保存快照（即RDB持久化的过程），并将保存快照期间接收的命令缓存起来。

当快照完成后，redis会将快照文件和所有缓存的命令发送给从库，从库接收后载入缓存文件并执行接收的缓存的命令。以上过程称为复制初始化。

复制初始化完成后，主库每收到写命令就同步从库。

> 主从断开重连后，2.6及以前版本会重新进行复制初始化，即使从库仅有几条命令没有收到，主从断线重连效率低。
>
> redis2.8断线重连支持有条件的增量数据传输，仅传送断线期间命令。

同步过程中从数据库不会阻塞，默认从数据库会用同步前的数据对命令响应。可配置slave-server-stale-data参数为no来使从数据库在同步完成前对所有命令（除info和slaveof）都回复错误：sync with master in progress

> **乐观复制**
>
> redis采用乐观复制（optimistic replication）的复制策略，容忍在一定时间内主从数据库的内容是不同的，但是两者的数据会最终同步。主库收到命令后直接返回结果并异步通知从库。如果网络断开，主库数据已变动，此时两者数据不一致。从这个角度看，主数据库无法得知某个命令最终同步给了多少个从数据库，不过redis提供两个配置项来限制只有当数据至少同步给指定数量的从数据库时，主库才是可写的：
>
> min-slaves-to-write 3 #只有当>=3个从库连接到主库时，主库才可写
>
> min-slaves-max-lag 10 #允许从库最长失去连接的时间

**从数据库持久化**

持久化相对耗时，为提高性能可禁用主库持久化，通过从库操作。

- 从库崩溃重启后主库自动同步数据到从库，无需担心数据丢失
- 主库崩溃，需手动通过从库恢复主库数据，需严格按以下两步：
  - 在从库使用slaveof no one命令将从库提升为主库继续服务
  - 启动之前崩溃的主库，使用slaveof命令将其设置成新的主库的从库，即可将数据同步过来。

> 当开启复制且主库关闭持久化功能时，一定不要使用supervisor及其他类似的进程管理工具令主库崩溃后自动重启。同样，当主库所在服务器因故关闭重启后默认禁止直接启动redis。
>
> 因为当主库重启后，由于未开启持久化，所以库中数据被清空，并同时清空从库，导致从数据库持久化失去意义。

**无硬盘复制**

redis复制的工作原理是基于RDB方式的持久化实现的，优点是显著简化逻辑，复用已有代码，但缺点也很明显：

- 当主库禁用rdb快照时（即删除了所有配置文件中的save语句），如果执行了复制初始化操作，redis依然会生成rdb快照，所以下次启动后主库会以该快照恢复数据。因为复制发生的时间不能确定，这使得恢复的数据可能是任何时间点的。
- 复制初始化需要在硬盘中创建rdb快照文件，如硬盘性能很难（如网络硬盘）时这一过程会对性能产生影响。

> 2.8.18开始，redis引入了无硬盘复制选项 repl-diskless-sync yes ，开启后，redis在与从库进行复制初始化时将不会将快照内容存储到硬盘上，而是直接通过网络发送给从数据库，避免硬盘的性能瓶颈。

**增量复制**

复制时当主从数据库断开后，从库会发送sync命令来重新进行一次完整复制操作。这样即使断开期间数据库的变化很小（甚至没有），也需要将数据库中的所有数据重新快照并传送一次。

redis2.8最重要的更新之一就是实现了主从断线重连情况下的增量复制。

增量复制是基于以下实现的：

- 从库会存储主库的运行ID，每个redis运行实例均会拥有一个唯一的运行ID，每当实例重启后，就会自动生成一个新的运行ID。
- 在复制同步阶段，主库将每一个命令传送给从库时，都会同时把该命令存放到一个积压队列（backlog）中，并记录下当前积压队列中存放的命令的偏移量。
- 同时，从库接收到主库传来的命令时，会记录下该命令的偏移量。

当主从准备就绪后，从库会发送一条sync命令来告诉主库可以开始把所有数据库同步过来了，而2.8之后，取而代之的是发送psync，格式为“psync 主库运行ID 断开前最新的命令偏移量”。主库收到psync命令后，会执行以下判断来决定此次重连是否可以执行增量复制：

- 主库判断从库传来的运行ID是否与自己的运行ID相同，避免主库断线期间重启过，造成数据不一致。
- 判断从库最后同步成功的命令偏移量是否在积压队列中，如在则执行增量复制，并将积压队列中相对应的命令发送给从库。

> 积压队列本质上是一个固定长度的循环队列，默认情况下积压队列大小为1M，可通过`repl-backlog-size`来调整。
>
> `repl-backlog-ttl` 即当所有从库与主库断开后，经过多久可以释放积压队列的内存空间，默认为1小时。



##### 2. 哨兵

一主多从的redis系统中，从库在整个系统中起到了数据冗余备份和读写分离的作用。当主库遇到异常中断服务后，开发者可通过手动的方式选择一个从库来升级为主库，使系统能继续提供服务。而这个过程相对麻烦且需人工接入，难以实现自动化。

为此，redis2.8中提供了哨兵v2工具来实现自动化的系统监控和故障恢复功能。

**哨兵简介**

监控redis运行状态：

- 监控主库和从库是否正常运行
- 主库出现故障时自动将从库转为主库

哨兵是一个独立的进程，典型架构图：

![1671169078358](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1671169078358.png)

在一个一主多从的redis系统中，可以使用多个哨兵进行监控任务以保证系统足够稳健，如下图：

注：此时不仅哨兵会同时监控主库和从库，哨兵之间也会相互监控。

![1671169233391](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1671169233391.png)

**使用**

1. 建立3个实例：redis-server --port 6381 --slaveof 127.0.0.1 6379

2. 创建配置文件sentinel.conf，内容如下：sentinel monitor myMaster 127.0.0.1 6379 1

   myMaster为自定义要监控的主库名字，1表示最低通过票数。

3. 启动sentinel进程：redis-sentinel /path/sentinel.conf

4. 杀死主库，等待指定时间（默认30s）+sdown表示哨兵主观认为主库停服了，而+odown表示哨兵客观认为主库停服。此时哨兵开始执行故障恢复，即挑一个从库将其升格为主库。

   -sdown表示恢复服务；+convert-to-slave转为从库；

   +try-failover表示哨兵开始故障恢复；+failover-end表示完成故障恢复；+switch-master表示切换主库。

   > 哨兵并没有彻底清除停止服务的实例，因为停止服务的实例可能在某个时间恢复服务，这时哨兵会让其重新加入进来。

> 配置哨兵监控系统时，只需配置主库，哨兵会自动发现所有复制该主库的从库。

**实现原理**

一个哨兵进程启动时会读取配置文件的内容，通过如下的配置找出需要监控的主库：

sentinel monitor master-name ip redis-port quorum

因为考虑到故障恢复后当前监控的系统主库地址和端口会发生变化，所以哨兵提供命令可以通过主库名字获取当前主库的地址和端口号。

> 一个哨兵节点可以同时监控多个redis主从系统，只需要提供多个sentinel monitor配置即可：
>
> sentinel monitor myMaster 127.0.0.1 6379 2
>
> sentinel monitor otherMaster 192.168.1.3 6380 4

多个哨兵节点也可以监控一个redis主从系统。配置文件中还可以定义其他监控相关参数，每个配置项都包含主库的名字使得监控不同主库时可以使用不同的配置参数，如：

sentinel down-after-milliseconds myMaster 60000

sentinel down-after-milliseconds otherMaster 60000

哨兵启动后，会与监控的主库建立两条连接，一条连接用来订阅主数据的\_\_sentinel\_\_:hello 频道以获取其他同样监控该数据库的哨兵节点的信息；哨兵也需要定期向主库发送info等命令来获取主库本身的信息。因为当客户端的连接进入订阅模式时就不能再执行其他命令了，所以这时哨兵会使用另一条连接来发送命令。

和主库连接完成后，哨兵会定时执行：

- 每10s哨兵会向主库和从库发送info命令
- 每2s哨兵会向主库和从库的\_\_sentinel__:hello 频道发送自己的信息

  > 消息内容为：哨兵的地址，端口，运行ID，配置版本，主库名字，主库地址，主库端口，主库配置文件
- 每1s哨兵会向主库、从库和其他哨兵节点发送ping命令

这三个操作贯穿哨兵进程的整个生命周期。

当前哨兵节点发现主库客观下线，需进行故障恢复，但故障恢复需要由领头的哨兵完成，这样可以保证同一时间只有一个哨兵节点来执行故障恢复。选举领头哨兵的过程使用了raft算法，具体过程如下：

- 发现主库客观下线的哨兵节点（A）向每个哨兵节点发送命令，要求对方选自己为领头哨兵。
- 如果目标哨兵节点没有选过其他人，则会同意A设置为领头哨兵。
- 如果A发现有超过半数且超过quorum参数值的哨兵节点同意选自己成为领头哨兵，则A成功成为领头哨兵。
- 当有多个哨兵节点同时参选领头哨兵，则会出现没有任何节点当选的可能。此时每个参选节点将等待一个随机时间重新发起参选请求，进行下一轮选举，直到成功。

选出领头哨兵后，领头哨兵开始对主库进行恢复。

领头哨兵从停止服务的主库的从库中选择一个来充当新的主库，选出后，领头哨兵将向从库发送slaveof no one命令使其升格为主库。而后领头哨兵向其他从库发送slaveof命令来使其成为新主库的从库。最后更新内部记录，将已停止服务的主库更新为新主库的从库，使得当其恢复服务时以从库的身份继续服务。

**哨兵的部署**

哨兵以独立进程的方式对一个主从系统进行监控，监控的效果好坏与否取决于哨兵的视角是否有代表性。

相对稳妥的哨兵部署方案是使得哨兵的视角尽可能的与每个节点视角一致，即：

- 为每个节点部署一个哨兵
- 使每个哨兵与其对应的节点网络相同或相近



##### 3. 集群

即使使用哨兵，此时redis集群的每个数据库依然存有集群中的所有数据，从而导致集群的总数据存储量受限于可用存储内存最小的数据库节点，形成木桶效应。尤其是当使用redis做持久化存储服务使用时。

在旧版redis中通常使用客户端分片来解决这个问题，即启动多个redis数据库节点，由客户端决定每个键交由哪个数据库节点存储，下次客户端读取该键时直接到该节点读取。整个数据库分布存储在N个数据库节点中，每个节点只存放数据库量的1/N。

但对于需要扩容的场景，在客户端分片后，如果想增加更多的节点，就需要对数据进行手动迁移，同时在迁移的过程中为了保持数据的一致性，还需要将集群暂时下线，相对比较复杂。

redis3.0一大特性就是支持集群（cluster）功能，集群的特点在于拥有和单机实例同样的性能，同时在网络分区后能够提供一定的可访问性以及对主库故障恢复的支持。另外，集群支持几乎所有的单机实例支持的命令，对于涉及多键的命令（如MGET），如果每个键都位于同一个节点中，则可以正常支持，否则提示错误。除此之外，集群还有一个限制是只能使用默认的0号数据库。

哨兵和集群是两个独立的功能，但从特性上看哨兵可以视为集群的子集，当不需要数据分片或已在客户端分片的场景下哨兵就足够用了，但如需水平扩容，则集群是一个非常好的选择。

**配置集群**

使用集群，只需将每个数据库节点的cluster-enabled配置选项打开即可，每个集群中至少需要3个主库才能正常运行。集群会将当前节点记录的集群状态持久化地存储到指定文件中，这个文件默认为当前工作目录下的nodes.conf文件。每个节点对应的文件必须不同，否则会造成启动失败，所以启动节点时要注意最好为每个节点使用不同的工作目录，或通过cluster-config-file选项修改持久化文件的名称。

各节点启动后，需将它们加入到一个集群。

redis源代码提供了一个辅助工具redis-trib.rb可以非常方便的完成这一任务。redis-trib.rb使用ruby编写，所以需安装ruby程序。使用redis-trib.rb初始化集群：

/path/redis-trib.rb create --replicas 1 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385

--replicas 1表示每个主库拥有从库的个数为1，所有整个集群有3主3从。

1. 首先redis-trib.rb会以客户端形式尝试连接所有的节点，并发送ping命令确定节点能正常服务。如果有任何节点无法连接，则创建失败。同时发送info命令获取每个节点运行ID以及是否开启集群功能。

2. 准备就绪后集群会向每个节点发送cluster meet命令，格式为cluster meet ip port，这个命令用来告诉当前节点指定ip和port上在运行的节点也是集群的一部分，从而使得6个节点最终可以归入一个集群。

3. redis-trib.rb会分配主从数据库节点，分配的原则是尽量保证每个主库运行在不同的ip上，同时每个从库与主库均不运行在同一个ip地址上，保证系统的容灾能力。

4. 分配完成后，会为每个主库分配插槽，分配插槽的过程其实就是分配哪些键归哪些节点负责。之后对每个要成为子数据库的节点发送cluster replicate 主库运行ID来将当前节点转为从库并复制指定运行ID的节点。

   > 任意节点执行 cluster nodes 查看集群所有节点信息

**节点的增加**

加入新节点，只需要向新节点发送：cluster meet ip port

其中，ip和port是集群中任意节点的地址和端口号，新节点接收到客户端发来的命令后，会与该节点握手，握手成功后该节点使用Gossip协议将新节点的信息通知给集群中的每一个节点。

**插槽的分配**

新加入的节点有两种选择，要么使用cluster replicate命令复制每个主库来以从库的形式运行，要么向集群申请分配插槽（slot）来以主库形式运行。

在一个集群中，所有的键都会被分配16384个插槽，而每个主库会负责处理其中的一部分插槽。

键与插槽的对应关系：

redis将每个键键名的有效部分使用CRC16算法计算出散列值，然后取对16384的余数。这样使得每个键都可以分配到16384个插槽中，进而分配指定的一个节点中处理。

键名的有效部分是指：

1. 如果键名包含 { 符号，且在{符号后面存在}符号，并且{和}之间至少一个字符，则有效部分是指{和}之间的内容。
2. 如不满足上一条规则，那么整个键名为有效部分。

如{user102}:last.name 有效部分为user102。

如命令涉及多个键，只有当所有键都位于同一个节点时redis才能正常支持。利用键的分配规则，可以将所有相关的键的有效部分设置成同样的值使得相关键都能分配到同一节点以支持多键操作。

插槽的分配分为如下情况：

1. 插槽之前没分配过，现在想分配给指定节点
2. 插槽之前被分配过，现在想移动到指定节点

如果想将100和101两个插槽分配给某个节点，只需要在该节点执行：cluster addslots 100 101即可，如指定插槽已分配过，则提示：err slot 100 is already busy

可通过cluster slots查看插槽的分配情况。

使用redis-trib.rb将一个插槽从6380迁移到6381：

执行：/path/redis-trib.rb reshard 127.0.0.1:6380



#### 管理

**可信的环境**

redis的安全设计是在“redis运行在可信环境”这个前提下做出的。

bind 127.0.0.1

**数据库密码**

redis> auth pwd

> 由于redis性能极高，并且输入错误密码后redis并不会进行主动延迟，所以攻击者可通过穷举法破解redis密码（1s尝试十几万个），故密码一定要选择复杂密码。

> 配置redis复制时，如果主库设置了密码，需要在从库配置文件中通过masterauth参数设置主库的密码，使从库连接主库时自动使用auth命令认证。

**命令命名**

redis支持在配置文件中将命令重命名，比如讲flushall重命名成一个比较复杂的名字，以保证只有自己的应用可以使用该命令，如：

rename-command FLUSHALL sdfsdfuowerwdafsdfsdfdsf

如需禁用某个命令则可将命令重命名为空字符串：

rename-command FLUSHALL ''

**通信协议**

redis通信协议是redis客户端与redis之间交流的语言，通信协议规定了命令和返回值的格式。



**管理工具**






