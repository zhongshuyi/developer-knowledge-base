---
order: 2
---

# Redis Stream

内容来自：

- <https://juejin.cn/post/7112825943231561741>
- <https://www.runoob.com/redis/redis-stream.html>
- <https://www.pdai.tech/md/db/nosql-redis/db-redis-data-type-stream.html>
- <https://blog.csdn.net/weixin_50002038/article/details/131578575>
- <https://zhuanlan.zhihu.com/p/60501638>
- ChatGpt

Redis Stream 是 Redis 5.0 版本新增加的数据结构。

但它有个缺点就是消息无法持久化，如果出现网络断开、Redis 宕机等，消息就会被丢弃。

简单来说发布订阅 (pub/sub) 可以分发消息，但无法记录历史消息。

而 Redis Stream 提供了消息的持久化和主备复制功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。

Redis Stream 具有以下特点：

- 可持久化：Redis Stream 中的数据可以被持久化保存，保证了数据的可靠性。
- 可追溯：Redis Stream 提供了一种消息的追溯机制，可以追踪到消息的来源和处理过程。
- 高性能：Redis Stream 支持高性能的读写操作，可以处理大量数据。
- 灵活性：Redis Stream 提供了丰富的操作命令，可以对消息进行各种操作，如读取、写入、删除、标记等。

Redis Stream 在很多场景中都有广泛的应用如：

- 消息队列：Redis Stream 可以作为一个高性能的消息队列，用于处理大量的消息。
- 事件流：Redis Stream 可以用于记录和处理各种事件，如系统日志、用户行为等。
- 通知系统：Redis Stream 可以作为一个通知系统，用于将各种通知推送给用户或者系统。
- 社交网络：Redis Stream 可以用于实现社交网络中的好友关系、消息传递等功能。

Redis Stream 的结构如下所示，它有一个消息链表，将所有加入的消息都串起来，每个消息都有一个唯一的 ID 和对应的内容：

![Redis Stream 结构](https://xn--5nx.top:9000/image/202309021610596.png)

每个 Stream 都有唯一的名称 (上图代表一个 Stream)，它就是 Redis 的 key，在我们首次使用 xadd 指令追加消息时自动创建。

每个 Stream 都是一个消息链表，将所有的消息都串起来了。

每个消息都有一个唯一的 id，一般由 Redis 直接生成，形式是 `timestampInMillis-sequence` 也就是 `时间戳-序号`(序号从 0 开始)，例如 1527846880572-5，它表示当前的消息在毫米时间戳 1527846880572 时产生，并且是该毫秒内产生的第 5 条消息。也可以由客户端自己指定，但是形式必须是`整数 - 整数`，最小 ID 为 0-1 而且必须是后面加入的消息的 ID 要大于前面的消息 ID。如果不大于，那么会返回异常：

```bash
127.0.0.1:6379> XADD xx 123-123 name xiaoming age 22
(error) ERR The ID specified in XADD is equal or smaller than the target stream top item
```

为了保证消息是有序的，因此 Redis 生成的 ID 是单调递增有序的。由于 ID 中包含时间戳部分，为了避免服务器时间错误而带来的问题（例如服务器时间延后了），Redis 的每个 Stream 类型数据都维护一个 `latest_generated_id` 属性，用于记录最后一个消息的 ID。若发现当前时间戳退后（小于 `latest_generated_id` 所记录的），则采用时间戳不变而序号递增的方案来作为新消息 ID（这也是序号为什么使用 int64 的原因，保证有足够多的的序号），从而保证 ID 的单调递增性质。

强烈建议使用 Redis 的方案生成消息 ID，因为这种时间戳 + 序号的单调递增的 ID 方案，几乎可以满足你全部的需求。但同时，记住 ID 是支持自定义的，别忘了！

消息内容就是键值对，形如 hash 结构的键值对。

图内其他信息：

- `Consumer Group`：消费组，使用 XGROUP CREATE 命令创建，一个消费组有多个消费者 (Consumer), 这些消费者之间是竞争关系。
- `last_delivered_id`：游标，每个消费组会有个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。
- `pending_ids`：消费者 (Consumer) 的状态变量，作用是维护消费者的未确认的 id。pending_ids 记录了当前已经被客户端读取的消息，但是还没有 ack (Acknowledge character：确认字符）。如果客户端没有 ack，这个变量里面的消息 ID 会越来越多，一旦某个消息被 ack，它就开始减少。这个 pending_ids 变量在 Redis 官方被称之为 PEL，也就是 Pending Entries List，这是一个很核心的数据结构，它用来确保客户端至少消费了消息一次，而不会在网络传输的中途丢失了没处理。

常用命令

```bash
# 创建 Redis Stream
XADD mystream * field1 value1 field2 value2 ...

# 读取指定 Stream ID 的消息（ID是动态变化的）
XREAD COUNT count STREAMS key start end

# 读取指定 Stream Group 的消费者的消息
XREADGROUP GROUP group_name consumer_name COUNT count STREAMS key > 

# 将消息加入指定 Stream
XADD stream_key MAXLEN ~ length * field1 value1 field2 value2 ...

# 获取 Stream 中的元素数量
XLEN stream_key

# 以阻塞的方式获取指定 Stream 中的消息（等待新消息到达）
XREAD BLOCK timeout_ms STREAMS key id

# 将 Stream 分组
XGROUP CREATE key group_name id [MKSTREAM]

# 将消息标记为已处理
XACK key group_name id [id ...]

# 获取指定区间内的消息
XRANGE key start end [COUNT count]

# 获取指定 ID 的消息
XREADGROUP GROUP group_name consumer_name COUNT count STREAMS key start
```

## 获取长度

可以获取一个 Stream 的长度

语法为：`XLEN key`

示例：

```bash
redis> XLEN mystream
(integer) 3
```

## 数量限制

如果消息积累太多，那么 Stream 链表会很长，对内存来说是一个大问题。而 XDEL 指令又不会真正的删除消息，它只是给消息做了个标志位。

我们可以通过一些指定对 Stream 进行真正的修剪，限制其最大长度。单独使用 XTRIM 指令也能对 Stream 进行限制，它能指定 MAXLEN 参数，用于指定 Stream 的最大长度，消息之后长度超过 MAXLEN 时，会自动将最老的消息清除，确保最多不超过指定长度。

XTRIM 命令用于裁剪（Trim）一个 Stream 中的消息，删除指定范围之外的消息。

语法：

```bash
XTRIM <key> MAXLEN [~] <count>
```

参数说明：

`<key>`：指定要裁剪的 Stream 的键名。
`[~]`：使用`~`来表示非精确修剪，它会基保证至少会有指定的 N 条数据，也可能会多一些。
`<count>`：指定要保留或丢弃的消息数量。

使用 MAXLEN 选项精确修剪的花销是很大的，Stream 为了节省内存空间，采用了一种特殊的结构表示，而这种结构的调整是需要额外的花销的。所以我们可以使用“~”来表示非精确修剪，它会基保证至少会有指定的 N 条数据，也可能会多一些。

## 范围查询

使用 `XRANGE` 和 `XREVRANGE` 命令实现消息的正向和逆向的范围查询。

要按范围查询 `Stream`，我们只需要指定两个 ID，一个开始和一个结束。还有两个特殊的 ID：`-` 和 `+` ，分别表示可能的最小和最大 ID。

语法：

```bash
XRANGE <key> <start> <end> [COUNT <count>]
```

参数说明：

- `<key>`：指定要获取消息的 Stream 的键名。
- `<start>`：指定消息的起始 ID，可以使用 - 表示最小的 ID 或 + 表示最大的 ID。如果 Stream 中没有对应的 ID，命令将返回空结果。
- `<end>`：指定消息的结束 ID，同样可以使用 - 表示最小的 ID 或 + 表示最大的 ID。如果 Stream 中没有对应的 ID，命令将返回空结果。
- `COUNT <count>`（可选）：指定要返回的消息数量，默认为返回所有消息。`<count>` 参数必须是正整数。

如果 `<start>` 和 `<end>` 参数相同，且在 Stream 中存在对应的消息，那么只会返回一条消息。

`XREVRANGE` 的使用方法与 `XRANGE` 完全一样，只是顺序为倒叙

## 独立消费

简单的使用就是使用 XADD 生产消息，然后使用 XREAD 来消费消息

XADD，命令用于在某个 stream（流数据）中追加消息，其中语法格式为：

```bash
XADD key ID field string [field string ...]
```

其中 key 是 Redis 的 key

ID，最常使用 `*`，表示由 Redis 生成消息 ID，这也是强烈建议的方案。

field string [field string], 就是当前消息内容，由 1 个或多个 key-value 构成。

使用示例

```bash
127.0.0.1:6379> XADD memberMessage * user kang msg Hello
"1553439850328-0"
127.0.0.1:6379> XADD memberMessage * user zhong  msg nihao
"1553439858868-0"
```

上面的例子中，在 memberMemsages 这个 key 中追加了 user kang msg Hello 这个消息。Redis 使用毫秒时间戳和序号生成了消息 ID。此时，消息队列中就有一个消息可用了。

然后就是消费消息

XREAD 命令可以从 Stream 中读取消息，语法格式为：

```bash
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]
```

其中各部分的含义如下：

- `COUNT count`: 可选参数，指定要返回的最大消息数量。默认情况下，返回所有可用的消息。如果指定了 COUNT 参数，命令将返回不超过指定数量的消息。
- `BLOCK milliseconds`: 可选参数，指定阻塞的超时时间，单位为毫秒。客户端在执行命令后会进入阻塞状态，直到满足以下两个条件之一：
  - 1）有新消息可供读取；
  - 2）超过了指定的超时时间。如果未设置 BLOCK 参数，则 XREAD 命令将立即返回空结果 (nil)。如果阻塞时间设置为 0，则表示永远阻塞，直到消息到来
- `STREAMS key [key ...]`: 关键字 STREAMS 后面跟着一个或多个 Stream 的键名。指定要从哪些 Stream 中读取消息。
- `ID [ID ...]`: 用于设置由哪个消息 ID 开始读取。使用 0 表示从第一条消息开始。此处需要注意，消息队列 ID 是单调递增的，所以通过设置起点，可以向后读取。在阻塞模式中，可以使用`$`，表示最新的消息 ID。（在非阻塞模式下`$`无意义）。

XRED 读消息时分为阻塞和非阻塞模式，使用 BLOCK 选项可以表示阻塞模式，需要设置阻塞时长。非阻塞模式下，读取完毕（即使没有任何消息）立即返回，而在阻塞模式下，若读取不到内容，则阻塞等待。

一个典型的阻塞模式用法为：

```bash
127.0.0.1:6379> XREAD block 1000 streams memberMessage $
(nil)
(1.07s)
```

我们使用 Block 模式，配合$作为 ID，表示读取最新的消息，若没有消息，命令阻塞！等待过程中，其他客户端向队列追加消息，则会立即读取到。

因此，典型的队列就是 XADD 配合 XREAD Block 完成。XADD 负责生成消息，XREAD 负责消费消息。

## 消费者组模式

如之前那个图所示

![Redis Stream 结构](https://xn--5nx.top:9000/image/202309021610596.png)

每个 Stream 可以存在多个消费组，同一个 Stream 的多个消费组之间是独立的，每个消费组都可以从 Stream 读取数据而不会影响其他消费组，也就是说你在这个消费组中读取了消息，在另一个消费组中还会读取到这条一样的消息。

消费组中有多个消费者，这些消费者之间是竞争关系，也就是说一个消息在同一个消费组中只能由一个被一个消费者消费

测试不同组消费

```bash
# 给名为 mq 的 Stream 添加消息
XADD mq * msg 1
> 1693656423988-0

# 在名为 mq 的 Stream 创建消费者组 mqGroup，从第一个消息开始读取
XGROUP CREATE mq mqGroup 0
> OK

# 在名为 mq 的 Stream 创建消费者组 mqGroup1，从第一个消息开始读取
XGROUP CREATE mq mqGroup1 0
> OK

# mqGroup 组中的消费者 consumerA 消费一条消息 ，参数 > 表示未被组内消费的起始消息
XREADGROUP group mqGroup consumerA count 1 streams mq >
> mq
> 1693656423988-0
> msg
> 1

# mqGroup1 组中的消费者 consumerA 消费一条消息 ，参数 > 表示未被组内消费的起始消息
XREADGROUP group mqGroup consumerA count 1 streams mq >
> mq
> 1693656423988-0
> msg
> 1
```

测试同组多个消费者

```bash
# 开启事务，MULTI 命令用于开启一个事务。在执行 MULTI 命令之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中。当 EXEC 命令被调用时，所有队列中的命令才会被执行
127.0.0.1:6379> MULTI
 # 生成一个消息：msg 1
127.0.0.1:6379> XADD mq * msg 1
127.0.0.1:6379> XADD mq * msg 2
127.0.0.1:6379> XADD mq * msg 3
127.0.0.1:6379> XADD mq * msg 4
127.0.0.1:6379> XADD mq * msg 5
# 执行事务
127.0.0.1:6379> EXEC
 1) "1553585533795-0"
 2) "1553585533795-1"
 3) "1553585533795-2"
 4) "1553585533795-3"
 5) "1553585533795-4"

# 创建消费组 mqGroup
127.0.0.1:6379> XGROUP CREATE mq mqGroup 0 # 为消息队列 mq 创建消费组 mgGroup
OK

# 消费者A，消费第1条
127.0.0.1:6379> XREADGROUP group mqGroup consumerA count 1 streams mq > #消费组内消费者A，从消息队列mq中读取一个消息
1) 1) "mq"
   2) 1) 1) "1553585533795-0"
         2) 1) "msg"
            2) "1"
# 消费者A，消费第2条
127.0.0.1:6379> XREADGROUP GROUP mqGroup consumerA COUNT 1 STREAMS mq > 
1) 1) "mq"
   2) 1) 1) "1553585533795-1"
         2) 1) "msg"
            2) "2"
# 消费者B，消费第3条
127.0.0.1:6379> XREADGROUP GROUP mqGroup consumerB COUNT 1 STREAMS mq > 
1) 1) "mq"
   2) 1) 1) "1553585533795-2"
         2) 1) "msg"
            2) "3"
# 消费者A，消费第4条
127.0.0.1:6379> XREADGROUP GROUP mqGroup consumerA count 1 STREAMS mq > 
1) 1) "mq"
   2) 1) 1) "1553585533795-3"
         2) 1) "msg"
            2) "4"
# 消费者C，消费第5条
127.0.0.1:6379> XREADGROUP GROUP mqGroup consumerC COUNT 1 STREAMS mq > 
1) 1) "mq"
   2) 1) 1) "1553585533795-4"
         2) 1) "msg"
            2) "5"
```

上面的例子中，三个在同一组 mpGroup 消费者 A、B、C 在消费消息时（消费者在消费时指定即可，不用预先创建），有着互斥原则，消费方案为，A->1, A->2, B->3, A->4, C->5

### 相关命令

#### 创建消费组

命令格式为：

```bash
XGROUP CREATE <key> <groupname> <id or $> [MKSTREAM]
```

参数说明

- `<key>`：指定要创建消费组的 Stream（流）的键名。
- `<groupname>`：指定要创建的消费组的名称。
- `<id or $>`：指定消费组的起始 ID。可以使用一个现有的消息 ID 作为起点，或者使用特殊符号 $ 表示从最新的消息开始消费。当想从第一个开始消费时可以设置为 `0-0`，也可以使用不完整 ID，例如 `0` 这将被解释为 `0-0`
- `[MKSTREAM]`：可选参数，如果使用该参数，Redis 会在指定的键不存在时自动创建一个空的 Stream。

示例：

```bash
XGROUP CREATE mystream mygroup $
```

#### 设置消费组的 ID

XGROUP SETID 命令用于设置消费组的 ID，即将消费组的消费偏移量设置为指定的 ID。

语法：

```bash
XGROUP SETID <key> <groupname> <id or $>
```

参数说明：

- `<key>`：指定消费组关联的 Stream（流）的键名。
- `<groupname>`：指定要设置 ID 的消费组的名称。
- `<id or $>`：指定要设置的消费组的消费偏移量。可以使用一个现有的消息 ID 作为消费偏移量，或者使用特殊符号 $ 表示从最新的消息开始消费。

示例：

```bash
XGROUP SETID mystream mygroup 1599528962060-0
```

以上示例将名为 mygroup 的消费组的消费偏移量设置为 1599528962060-0。

通过 XGROUP SETID 命令，可以控制消费组从指定的位置开始消费消息，用于消费组中的消费者恢复或重新加入消费流程。

#### 删除消费组

XGROUP DESTROY 命令用于销毁一个消费组（Consumer Group），将其从相关的 Stream 中移除并删除。

语法：

```bash
XGROUP DESTROY <key> <groupname>
```

参数说明：

- `<key>`：指定关联的 Stream（流）的键名。
- `<groupname>`：指定要销毁的消费组的名称。

示例：

```bash
XGROUP DESTROY mystream mygroup
```

以上示例将名为 mygroup 的消费组从 mystream 的关联中删除和销毁。

注意事项：

- 使用 XGROUP DESTROY 命令会完全删除指定的消费组，并且无法恢复。消费组删除后，与之关联的消费者信息、消费偏移量等也会被删除。
- 在执行 XGROUP DESTROY 命令之前，请确保消费组中的所有任务都已经完成，并且不再需要这个消费组来协同消费 Stream 中的消息。
- 如果只是想暂停消费组的消费，可以通过暂停相应的消费者来实现，并不需要销毁整个消费组。

在 Redis 中，XGROUP DESTROY 命令用于清理不再需要的消费组，以便有效地管理消费者和 Stream 的关联关系。

#### 删除指定消费者

XGROUP DELCONSUMER 命令用于从消费组中删除指定的消费者。

语法：

```bash
XGROUP DELCONSUMER <key> <groupname> <consumername>
```

参数说明：

- `<key>`：指定消费组关联的 Stream（流）的键名。
- `<groupname>`：指定要操作的消费组的名称。
- `<consumername>`：指定要删除的消费者的名称。

示例：

```bash
XGROUP DELCONSUMER mystream mygroup consumer1
```

以上示例将名为 consumer1 的消费者从 mygroup 消费组中删除。

注意事项：

- 使用 XGROUP DELCONSUMER 命令可以在不销毁整个消费组的情况下，从消费组中移除某个特定的消费者。
- 被删除的消费者将不再接收来自消费组的任务，也不再更新消费偏移量。
- 如果需要重新加入消费组，被删除的消费者可以使用 XREADGROUP 命令重新注册到指定的消费组，并继续处理消息。

XGROUP DELCONSUMER 命令使得在消费组中动态管理消费者变得可能，可以根据需求添加或删除消费者，实现更灵活的消费组维护。

#### 消费消息

XREADGROUP 命令用于从指定的消费组中读取消息，并对消息进行处理。

语法：

```bash
XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] id [id ...]
```

参数说明：

- `<groupname>`：指定要读取的消费组的名称。
- `<consumername>`：指定当前消费者的名称，用于标识消费者在消费组中的身份。消费者在第一次被提及时自动创建，无需显式创建。
- `[COUNT <count>]`：可选参数，表示每次读取的消息数量。可以通过指定 COUNT 参数一次性读取多条消息。如果不指定，则默认为 1
- `[BLOCK <milliseconds>]`：可选参数，表示在没有可消费的消息时，阻塞等待新消息的时间（以毫秒为单位），如果为 0 则表示一直阻塞，直到新消息到来
- [NOACK]: 如果不需要可靠性，偶尔的消息丢失是可以接受的，可以使用 NOACK 子命令避免将消息添加到 PEL 中。这相当于在读取消息时确认该消息。
- `<key>`：指定要读取的 Stream（流）的键名。
- `<id>`：指定读取消息的起始位置，可以是一个现有的消息 ID，或者使用特殊符号 `>` 表示从最新的消息开始读取。

示例：

```bash
XREADGROUP GROUP mygroup consumer1 COUNT 10 BLOCK 1000 STREAMS mystream >
```

以上示例将名为 mygroup 的消费组中的 consumer1 消费者以一次读取 10 条消息的方式，阻塞等待新消息的到来，在键为 mystream 的 Stream 中从最新的消息开始读取。

ID 也可以指定 0、其他 ID 或者不完整的 ID（时间戳部分），但这样的话，Stream 只会返回已传递给当前消费者并且没有被 XACK 确定的历史消息，即该消费者内部的 pending_ids 集合，在这种情况下，BLOCK 和 NOACK 都被忽略。

每个消费组会有个游标 `last_delivered_id`，任意一个消费者读取了消息都会使游标 `last_delivered_id` 往前移动。

### Pending 等待列表

为了解决组内消息读取但处理期间消费者崩溃带来的消息丢失问题，STREAM 设计了 Pending 列表，用于记录读取但并未处理完毕的消息。命令 XPENDIING 用来获消费组或消费内消费者的未处理完毕的消息。

演示如下：

```bash
127.0.0.1:6379> XPENDING mq mqGroup # mpGroup的Pending情况
1) (integer) 5 # 5个已读取但未处理的消息
2) "1553585533795-0" # 起始ID
3) "1553585533795-4" # 结束ID
4) 1) 1) "consumerA" # 消费者A有3个
      2) "3"
   2) 1) "consumerB" # 消费者B有1个
      2) "1"
   3) 1) "consumerC" # 消费者C有1个
      2) "1"

127.0.0.1:6379> XPENDING mq mqGroup - + 10 # 使用 start end count 选项可以获取详细信息
1) 1) "1553585533795-0" # 消息ID
   2) "consumerA" # 消费者
   3) (integer) 1654355 # 从读取到现在经历了1654355ms，IDLE
   4) (integer) 5 # 消息被读取了5次，delivery counter
2) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 1654355
   4) (integer) 4
# 共5个，余下3个省略 ...

127.0.0.1:6379> XPENDING mq mqGroup - + 10 consumerA # 在加上消费者参数，获取具体某个消费者的Pending列表
1) 1) "1553585533795-0"
   2) "consumerA"
   3) (integer) 1641083
   4) (integer) 5
# 共3个，余下2个省略 ...
```

每个 Pending 的消息有 4 个属性：

1. 消息 ID
2. 所属消费者
3. IDLE，已读取时长
4. delivery counter，消息被读取次数

上面的结果我们可以看到，我们之前读取的消息，都被记录在 Pending 列表中，说明全部读到的消息都没有处理，仅仅是读取了。那如何表示消费者处理完毕了消息呢？使用命令 XACK 完成告知消息处理完成

```bash
127.0.0.1:6379> XACK mq mqGroup 1553585533795-0 # 通知消息处理结束，用消息ID标识
(integer) 1

127.0.0.1:6379> XPENDING mq mqGroup # 再次查看Pending列表
1) (integer) 4 # 已读取但未处理的消息已经变为4个
2) "1553585533795-1"
3) "1553585533795-4"
4) 1) 1) "consumerA" # 消费者A，还有2个消息处理
      2) "2"
   2) 1) "consumerB"
      2) "1"
   3) 1) "consumerC"
      2) "1"
127.0.0.1:6379>
```

有了这样一个 Pending 机制，就意味着在某个消费者读取消息但未处理后，消息是不会丢失的。等待消费者再次上线后，可以读取该 Pending 列表，就可以继续处理该消息了，保证消息的有序和不丢失。

相关命令解析

XPENDING 查看 Pending 列表

```bash
XPENDING key group [[IDLE min-idle-time] start end count [consumer]]
```

参数说明：

- `<key>`：指定要获取消息的 Stream 的键名。
- `<group>`：指定消费者组的名称。
- `[IDLE min-idle-time]`:最小空闲时间
- `<start>`（可选）：指定消息的起始 ID，可以使用 - 表示最小的 ID 或 + 表示最大的 ID。默认为最小的 ID。
- `<end>`（可选）：指定消息的结束 ID，同样可以使用 - 表示最小的 ID 或 + 表示最大的 ID。默认为最大的 ID。
- `<count>`（可选）：指定要返回的消息数量，默认为返回所有消息。
- `<consumer>`（可选）：指定要获取待处理消息的消费者。如果不指定，则返回所有消费者的待处理消息。

XACK 消息确认，XACK 命令用于确认消费者已经处理完成指定的消息，并且将待处理状态从消息队列中移除。

```bash
XACK key group id [id ...]
```

参数说明：

- `<key>`：指定要获取消息的 Stream 的键名。
- `<group>`：指定消费者组的名称。
- `<id>`：指定要确认处理完成的消息的 ID。

一旦消费者成功处理了一条消息，它就应该调用 XACK，这样这条消息就不会再被处理，同时，关于这条消息的 PEL 条目也会被清除，从而释放 Redis 服务器的内存。

此时还有一个问题，就是若某个消费者宕机之后，没有办法再上线了，那么就需要将该消费者 Pending 的消息，转义给其他的消费者处理，就是消息转移。请继续。

### 消息转移

消息转移的操作时将某个消息转移到自己的 Pending 列表中。使用语法 XCLAIM 来实现，需要设置组、转移的目标消费者和消息 ID，同时需要提供 IDLE（已被读取时长），只有超过这个时长，才能被转移。演示如下：

```bash
# 当前属于消费者A的消息1553585533795-1，已经15907,787ms未处理了
127.0.0.1:6379> XPENDING mq mqGroup - + 10
1) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 15907787
   4) (integer) 4

# 转移超过3600s的消息1553585533795-1到消费者B的Pending列表
127.0.0.1:6379> XCLAIM mq mqGroup consumerB 3600000 1553585533795-1
1) 1) "1553585533795-1"
   2) 1) "msg"
      2) "2"

# 消息1553585533795-1已经转移到消费者B的Pending中。
127.0.0.1:6379> XPENDING mq mqGroup - + 10
1) 1) "1553585533795-1"
   2) "consumerB"
   3) (integer) 84404 # 注意IDLE，被重置了
   4) (integer) 5 # 注意，读取次数也累加了1次
```

以上代码，完成了一次消息转移。转移除了要指定 ID 外，还需要指定 IDLE，保证是长时间未处理的才被转移。被转移的消息的 IDLE 会被重置，用以保证不会被重复转移，以为可能会出现将过期的消息同时转移给多个消费者的并发操作，设置了 IDLE，则可以避免后面的转移不会成功，因为 IDLE 不满足条件。例如下面的连续两条转移，第二条不会成功。

```bash
127.0.0.1:6379> XCLAIM mq mqGroup consumerB 3600000 1553585533795-1
127.0.0.1:6379> XCLAIM mq mqGroup consumerC 3600000 1553585533795-1
```

这就是消息转移。至此我们使用了一个 Pending 消息的 ID，所属消费者和 IDLE 的属性，还有一个属性就是消息被读取次数，delivery counter，该属性的作用由于统计消息被读取的次数，包括被转移也算。这个属性主要用在判定是否为错误数据上。

### 自动转移

XPENDING 和 XCLAIM 为不同类型的恢复机制提供了基本的步骤。Redis 6.2 中添加的 XAUTOCLAIM 命令则通过让 Redis 管理它来优化通用过程，并为大多数恢复需求提供简单的解决方案。

XAUTOCLAIM 识别空闲的待处理消息并将它们的所有权转移给指定的消费者，XAUTOCLAIM 相当于先调用 XPENDING，然后调用 XCLAIM。

命令语法：

```bash
XAUTOCLAIM <key> <group> <consumer> <min-idle-time> [<start> <end> <count>]
```

参数说明：

- `<key>`：指定要获取消息的 Stream 的键名。
- `<group>`：指定消费者组的名称。
- `<consumer>`：指定消费者的名称。
- `<min-idle-time>`：指定消费者必须在空闲多长时间（以毫秒为单位）后才能声明消息的所有权。
- `<start>`（可选）：指定消息的起始 ID，可以使用 - 表示最小的 ID 或 + 表示最大的 ID。默认为最小的 ID。
- `<end>`（可选）：指定消息的结束 ID，同样可以使用 - 表示最小的 ID 或 + 表示最大的 ID。默认为最大的 ID。
- `<count>`（可选）：指定要自动声明所有权的消息数量，默认为自动声明所有满足条件的消息。

### 死信队列

正如上面所说，如果某个消息，不能被消费者处理，也就是不能被 XACK，这是要长时间处于 Pending 列表中，即使被反复的转移给各个消费者也是如此。此时该消息的 delivery counter 就会累加（上一节的例子可以看到），当累加到某个我们预设的临界值时，我们就认为是坏消息（也叫死信，DeadLetter，无法投递的消息）。因此，一旦转移计数器达到要给给定阈值，将此类消息放入另一个 Stream 并且发送一封邮件可能更明智。

## Stream 监控

Redis 支持各种命令来监控 Stream 的信息，此前我们已经介绍了 XPENDING 命令，它允许我们检查在给定时刻正在处理的消息列表，以及它们的空闲时间和转移次数。

当我们想要获取其他信息的时候，我们可以使用 XINFO 命令，XINFO 命令是一个可观察性接口，可以与子命令一起使用以获取有关 Stream 或消费者组的信息。

语法：

```bash
XINFO [CONSUMERS key groupname] [GROUPS key] [STREAM key] [HELP]
```

- `XINFO STREAM <key>`：显示有关 Stream 的信息。
- `XINFO STREAM <key> FULL [COUNT <count>]`：该命令返回 Stream 的整个状态的详细信息，包括消息、组、消费者和待处理消息列表 (PEL) 信息，类似于几个命令的组合。
- `XINFO GROUPS <key>`：获得与流关联的所有消费者组的信息。
- `XINFO CONSUMERS <key> <group>`：获取特定消费者组中每个消费者的信息。

## 删除消息

语法

```bash
XDEL key id [id ...]
```

从流中移除指定的条目，并返回已删除的个数数。

这个删除只是将信息标记为删除，并不会真正的删除数据，使用 XPENDING 也能够看到其还在存在于 PEL 中，当整个 Stream 上的消息都被标记为删除时，才会执行真正的

## 零长度 Stream

Stream 和其他 Redis 数据结构的区别在于，当其他数据结构不再有任何元素时，key 本身将被删除。例如，当调用 ZREM 将删除 ZSET 中的最后一个元素时，将完全删除该 ZSET。

由于使用计数为零的 MAXLEN 选项（XADD 和 XTRIM 命令），或者因为调用了 XDEL，Stream 被允许保持在零元素处。

之所以 Stream 这么特殊，是因为 Stream 可能有关联的 consumer group，我们不想因为 Stream 中不再有任何消息而失去 consumer group 定义的状态。

目前 Redis6.2 的版本中，即使没有关联的消费者组，Stream 也不会被删除，但这在未来可能会改变。

## 总结

Redis Stream 基于内存存储，其速度相比于真正的消息队列比如 kafka、rocketmq 等更快，但也是因为内存的原因，我们无法使用 Redis Stream 长时间的存储大量的数据，因为内存相比于磁盘来说要昂贵得多。另外，Redis Stream 也没有提供延时消息的能力。

虽然 Redis Stream 作为消息队列的功能已经很强大了，但是因为“基于内存”这个 Redis 最重要的优点，导致 Redis Stream 无法存储大量的数据（这需要很多的内存），因此到目前为止，Redis Stream 在生产环境的应用也并不多，它更适用于小型、廉价的应用程序，以及可以丢弃数据的场景（限制 Stream 长度），比如记录某些不重要的操作日志。

