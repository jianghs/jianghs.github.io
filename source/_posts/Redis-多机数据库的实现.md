---
title: Redis-多机数据库的实现
date: 2019-07-29 15:02:36
tags: Redis
categories: coding
---
## 复制

通过 `SLAVEOF` 命令或者设置 slaveof 选项，启动复制功能。

### 旧版复制功能的实现

* 同步（SYNC）：将从服务器数据更新至主服务器当前所处的数据状态。
* 命令传播（command propagate）:在主服务器数据发生修改，导致主从服务器不一致时，让主从服务器回到一致状态。

#### 同步

1. 从服务器向主服务器发送 SYNC 命令。
2. 收到 SYNC 命令的主服务器执行 BGSAVE 命令，在后台生成一个 RDB 文件，并使用一个缓冲区记录从现在开始执行的所有写命令。
3. 当主服务器的 BGSAVE 命令执行完毕时，主服务器会将生成的 RDB 文件发送给从服务器，从服务器接收并载入这个 RDB 文件，将自己的数据库状态更新至主服务器执行 BGSAVE 时的数据库状态。
4. 主服务器将记录在缓冲区的所有写命令发送给从服务器，从服务器执行这些写命令，将数据库状态更新到主服务器数据库当前所处的状态。

#### 命令传播

主服务器对从服务器执行命令传播操作。

### 旧版复制功能的缺陷

断线后重新复制，主服务器接收到 SYNC 命令后会将所有的数据生成 RDB 文件，导致效率低下。

### 新版复制功能的实现

`PSYNC` 命令具备完整重同步和部分重同步两种模式。

完整重同步用于初次复制情况，与 SYNC 功能一致。

部分重同步用于断线后重新复制的情况，断线后，如果条件允许，只复制断线期间的写命令。

1. 从服务器向主服务器发送 PSYNC 命令。
2. 主服务器向从服务器回复 +CONTINUE 回复，表示执行部分重同步。
3. 从服务器接收 +CONTINUE 回复，准备执行部分重同步。

### 部分重同步的实现

#### 主服务器的复制偏移量和从服务器的复制偏移量

1. 主服务器每次向从服务器传播 N 个字节数据时，就将自己的复制偏移量加 N。
2. 从服务器接收到 N 个字节数据时，就将自己的复制偏移量加 N。

如果主从服务器的复制偏移量相等，则数据库状态一致。否则处于不一致的状态。

#### 主服务器的复制积压缓冲区

复制积压缓冲区是主服务器维护的一个固定长度、先进先出的队列。默认大小 1 MB。

积压缓冲区会记录每个字节对应的偏移量。

1. 当主服务器进行命令传播时，不仅会把所有写命令发给所有从服务器，还会将写命令入队到复制积压缓冲区。
2. 主服务器接收到从服务器的 PSYNC 命令后，如果从服务器复制偏移量之后的数据仍然存在于积压缓冲区，那么主服务器会对从服务器执行部分重同步。
3. 否则执行完整重同步。

#### 服务器的运行 ID

从服务器进行初次复制时，主服务器会将自己的服务器 ID 发送给从服务器，从服务器会保存这个 ID。

如果从服务器保存的 ID 与 重新连接的服务器 ID 相同，主服务器可以继续尝试部分重同步。否则进行完整重同步。

### PSYNC 命令的实现

![PSYNC 情况](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/PSYNC_PROCESS.png)

### 复制的实现

1. 设置主服务器的地址和端口

    从服务器的 redisServer 的 masterhost 属性和 masterport 属性记录以上信息。

2. 建立套接字连接

    从服务器是主服务器的客户端。

3. 发送 PING 命令

    * 检查套接字是否读写正常。
    * 检查主服务器是否可以正常处理命令请求。

    ![ping](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/PING.png)

4. 身份验证

    ![verify](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/verify.png)

5. 发送端口信息

    从服务器向主服务器发送自己的端口信息，主服务器将从服务器的端口记录在 redisClient 中。

6. 进行同步

    从服务器向主服务器发送 PSYNC 命令，执行同步操作。

    执行同步前，只有从服务器是主服务器是客户端，执行同步后，主从服务器互为客户端。

7. 进行命令传播

    完成同步后，主从服务器就会进入命令传播阶段。

### 心跳检测

命令传播阶段，从服务器默认每秒一次的频率，向主服务器发送以下命令：

```redis
REPLCONF ACK <replication_offset>
```

作用：

* 检测主从服务器网络连接状态
* 辅助实现 min-slaves 选项
* 检测命令丢失

#### 检测主从服务器网络连接状态

如果主服务器一秒内没有收到从服务器发送的命令，认为网络有问题。

#### 辅助实现 min-slaves 选项

min-salves-to-write 和 min-slaves-max-lag 防止主服务器在不安全的情况下执行写操作。

#### 检测命令丢失

与部分重同步原理类似，区别在于检测命令丢失不是在断线的情况下。

## Sentinel

哨兵解决方案：由一个或者多个 sentinel 实例组成的 sentinel 系统可以监视任意多个主服务器以及这些主服务器属下的所有从服务器。

### 启动并初始化 Sentinel

#### 初始化服务器

Sentinel 服务器本质上是一个 Redis 服务器，但是初始化过程不完全一致。

![Sentinel-init](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/Sentinel-init.png)

#### 将普通 Redis 服务器使用的代码替换成 Sentinel 专用代码

指定自己的端口和载入自己的命令。

#### 初始化 Sentinel 状态

```c
struct sentinelState {

  // 当前纪元，用于实现故障转移
  unit64_t current_epoch;

  // 保存了所有被这个 sentinel 监视的主服务器
  // 字典的键是主服务器的名字
  // 字典的值时指向 sentinelRedisInstance 结构的指针
  dict *masters;

  // 是否进入 TILT 模式
  int tilt;

  // 目前正在执行的脚本的数量
  int running_scripts;

  // 进入 TILT 模式的时间
  mstime_t tilt_start_time;

  // 最后一次执行时间处理器的时间
  mstime_t previous_time;

  // 一个 FIFO 队列，包含了所有需要执行的用户脚本
  list *scripts_queue;
} sentinel;
```

#### 初始化 Sentinel 状态的 masters 属性

* 字典的键是被监视的主服务器的名字
* 字典的值指向被监视主服务器对应的 sentinelRedisInstance 结构

每个实例结构代表一被 sentinel 监控的 Redis 服务器实例，这个实例可以是主服务器、从服务器或者另一个 Sentinel。

```c
typedef struct sentinelRedisInstance {

  // 标识值，记录了实例的类型，以及该实例的当前状态
  int flags;

  // 实例的名字
  // 主服务器的名字由用户在配置文件中设置
  // 从服务器及 Sentinel 的名字由 Sentinel 自动设置
  // 格式为 ip:port
  char *name;

  // 实例的运行 ID
  char *runid;

  // 配置纪元，用于实现故障转移
  uint_64_t config_epoch;

  // 实例的地址
  sentinelAddr *addr;

  // 实例无响应多少毫秒后才会被判断为主观下线
  mstime_t down_after_period;

  // 判断客观下线所需的支持投票数量
  int quorum;

  // 在执行故障转移时，可以同时对新的主服务器进行同步的从服务器数量
  int parallel_syncs;

  // 刷新故障转移状态的最大时限
  mstime_t failover_timeout;

  // ...
} sentinelRedisInstance;
```

#### 创建连向主服务器的网络连接

创建两个连向主服务器的异步连接：

* 命令连接，用于向主服务器发送命令，并接受命令回复。
* 订阅连接，用于订阅主服务器的 _sentinel_:hello 频道。

### 获取主服务器信息

默认以每十秒一次的频率，通过命令连接向被监视的主服务器发送 INFO 命令，并通过分析 INFO 命令的回复来获取主服务器的当前信息。

* 主服务器本身信息：run_id 域记录的服务器运行 ID，服务器角色。
* 主服务器属下所有从服务器信息：slaves 字段。

### 获取从服务器信息

Sentinel 会创建连接到从服务器的命令连接和订阅连接。

会以每十秒一次的频率通过命令连接向从服务器发送 INFO 命令。

命令回复如下：

* 从服务器的运行 ID run_id。
* 从服务器的角色 role。
* 主服务器的 IP 地址 master_host，以及主服务器的端口号 master_port。
* 主从服务器的连接状态 master_link_status。
* 从服务器的优先级 slave_priority。
* 从服务器的复制偏移量 slave_repl_offset。

### 向主服务器和从服务器发送信息

默认情况下，Sentinel 会以每两秒一次的频率，通过命令连接向所有被监视的主服务器和从服务器发送命令。

s_ 开头：记录的是 Sentinel 本身信息。

m_ 开头：记录的是主服务器的信息。

### 接收来自主服务器和从服务器的频道信息

对于每个与 Sentinel 连接的服务器，Sentinel 既通过命令连接向服务器的 \_sentinel\_:hello 频道发送信息，又通过订阅连接从服务器的 \_sentinel\_:hello 频道接收信息。

对于监视同一个服务器的多个 Sentinel 来说，一个 Sentinel 发送的信息会被其他 Sentinel 接收到，包括发送的 Sentinel 自身。

#### 更新 sentinels 字典

Sentinel 还会记录除了自身之外的其他 Sentinels 信息。

#### 创建连向其他 Sentinel 的命令连接

Sentinel 之间只会创建命令连接，不会创建订阅连接。

### 检测主观下线状态

默认情况下，Sentinel 会以每秒一次的频率向所有与它创建了命令连接的实例（包括主服务器、从服务器、其他 Sentinel 在内）发送 PING 命令，并通过实例返回的 PING 命令回复来判断实例是否在线。

down-after-milliseconds 选项指定了 Sentinel 判断实例进入主观下线所需的时间长度，同样作用于监视 master 的其他 Sentinels。

### 检测客观下线状态

当 Sentinel 将一个主服务器判断为主观下线后，为了确认这个主服务器是否真的下线，它会向同样监视这一主服务器的其他 Sentinel 进行询问，看他们是否也认为主服务器已经进入下线状态。当接收的数量足够多时，Sentinel 就会将主服务器判定为客观下线，并执行故障转移。

#### 发送 SENTINEL is-master-down-by-addr 命令

源 SENTINEL 服务器询问其他 SENTINEL 服务器主服务器是否已下线。

#### 接收 SENTINEL is-master-down-by-addr 命令

目标 SENTINEL 接收到命令后，分析参数，进行命令回复。返回对主服务器的检查结果。

#### 接收 SENTINEL is-master-down-by-addr 命令的回复

根据其他 Sentinel 的命令回复， Sentinel 将统计其他 Sentinel 同意主服务器已下线的数量，当这一数量达到配置指定的客观下线所需数量时，会将祝福器进入客观下线状态。

### 选举领头 Sentinel

当一个主服务器被判断为客观下线时，监视这个下线主服务器的各个 Sentinel 会进行协商，选举出一个领头 Sentinel，并由领头 Sentinel 对下线主观服务器执行故障转移操作。

选举的规则和方法：

* 所有在线的 Sentinel 都有被选举为领头 Sentinel 的资格。
* 每次进行领头选举后，不论选举是否成功，所有 Sentinel 配置的纪元值都会自增一次。
* 在一个配置纪元里面，所有 Sentinel 都有一次将某个 Sentinel 设置为局部领头 Sentinel 的机会，并且局部领头一旦设置完成，在这个配置纪元里面就不能修改。
* 每个发现主服务器客观下线的 Sentinel 都会要求其他 Sentinel 将自己设置为局部领头 Sentinel。
* Sentinel 设置局部领头 Sentinel 的规则是先到先得。
* 目标 Sentinel 接收到命令后，会向源 Sentinel 发送一条命令回复，回复中的 leader_runid 参数和 leader_epoch 参数分别记录了目标 Sentinel 的局部领头 Sentinel 的运行 ID 和配置纪元。
* 源 Sentinel 接收到目标 Sentinel 的命令回复后，检查配置纪元和自己的是否一致，如果相同，判断运行 ID 是否和自己一致，两者都一致表明目标 Sentinel 将源 Sentinel 设置为局部领头 Sentinel。
* 如果某个 Sentinel 被半数以上的 Sentinel 设置成了局部领头 Sentinel，那么这个 Sentinel 称为领头 Sentinel。
* 因为领头 Sentinel 需要半数以上的 Sentinel 的支持，并且每个 Sentinel 在每个配置纪元里面只能产生一次局部领头 Sentinel，所以在一次配置纪元里面只会出现一个领头 Sentinel。
* 如果在给定的时间内，没有一个 Sentinel 被选举为领头 Sentinel，那么会进行重新选举，直到选出领头 Sentinel 为止。

### 故障转移

选举出的领头 Sentinel 对已下线的主服务器进行故障转移

1. 在已下线的主服务器属下的所有从服务器里面，挑选一个从服务器，并将其转换为主服务器。

    挑选出一个状态良好、数据完整的从服务器，发送 SLAVEOF no one 命令。

2. 让已下线的主服务器属下的所有从服务器改为复制新的主服务器。

    向其他从服务器发送 SLAVEOF 命令。

3. 将已下线的主服务器设置为新主服务器的从服务器。

    已线下的服务器重新上线后，领头 Sentinel 发送 SLAVEOF 命令。

## 集群

### 节点

刚开始的时候，每个节点是相互独立的，他们都处于只包含自己的集群中。

通过 `CLUSTER MEET` 命令完成。

#### 启动节点

一个节点就是一个运行在集群模式下的 Redis 服务器。节点会继续使用单机模式下的服务器组件。

#### 集群数据结构

每个节点都会使用一个 clusterNode 结构来记录自己的状态，并未集群中的所有其他节点（包括主节点和从节点）都创建一个相应的 clusterNode 结构。

```c
struct clusterNode {
    // 创建节点的时间
    mstime_t ctime;

    // 节点的名字
    char name[REDIS_CLUSTER_NAMELEN];

    // 节点标识
    // 使用各种不同的标识值记录节点的角色（比如主节点或者从节点）
    // 以及节点目前所处的状态（比如在线或者下线）
    int flags;

    // 节点当前的配置纪元，用于实现故障转义
    uint64_t configEpoch;

    // 节点的 IP 地址
    char ip[REDIS_IP_LEN];

    // 节点的端口
    int port;

    // 保存连接节点所需的有关信息
    clusterLink *link;

    // ...
}
```

clusterNode 结构的 link 属性是一个 clusterLink 结构，该结构保存了连接节点所需的有关信息，比如套接字描述符，输入缓冲区和输出缓冲区。

```c
typedef sturct clusterLink {

    // 连接的创建时间
    mstime_t ctime;

    // TCP 套接字描述符
    int fd;

    // 输出缓冲区，保存着等待发送给其他节点的消息
    sds sndbuf;

    // 输入缓冲区，保存着从其他节点接收到的消息
    sds rcvbuf;

    // 与这个连接相关联的节点，如果没有的话就为 NULL
    struct clusterNode *node;
} clusterLink;
```

最后，每个节点都保存着一个 clusterState 结构，这个结构记录了在当前节点的视角下，集群目前所处的状态。

```c
typedef struct clusterState {

    // 指向当前节点的指针
    clusterNode *myself;

    // 集群当前的配置纪元，用于实现故障转移
    uint64_t currentEpoch;

    // 集群当前的状态：上线还是下线
    int state;

    // 集群中至少处理这一个槽的节点的数量
    int size;

    // 集群节点名单
    // 字典的键为节点的名字，字典的值为节点对应的 clusterNode 结构
    dict *nodes;
}
```

#### CLUSTER MEET 命令的实现

客户端向节点 A 发送 CLUSTER MEET 命令，收到命令的节点 A 将于节点 B 进行握手。

1. 节点 A 会为节点 B 创建一个 clusterNode 结构，并将该结构添加到自己的 clusterState.nodes 字典里面。

2. 之后，节点 A 将根据 CLUSTER MEET 命令给定的 IP 地址和端口号，向节点 B 发送一条 MEET 消息。

3. 如果一切顺利，节点 B 将接受到 节点 A 发送的 MEET 消息，节点 B 会为节点 A 创建一个 clusterNode 结构，并将该结构添加到自己的 clusterState.nodes 字典里面。

4. 之后，节点 B 将向节点 A 返回一条 PONG 消息。

5. 节点 A 接收到节点 B 返回的 PONG 消息后，就知道节点 B 成功接收到自己发送的 MEET 消息。

6. 之后，节点 A 将向节点 B 返回一条 PING 消息。

7. 节点 B 将接收到节点 A 返回的 PING 消息，通过这条消息节点 B 可以知道节点 A 已经成功接到自己返回的 PONG 消息，握手完成。

握手完成后，节点 A 会将节点 B 的信息传播到集群中的其他节点，让其他节点与节点 B 进行握手，最终，节点 B 会被集群中的所有节点认识。

### 指派槽

Redis 集群通过分片的方式来保存数据库中的键值对：集群的整个数据库被分为16384 个槽，数据库中的每个键都属于这16384个槽的其中一个，集群中的每个节点可以处理0个或者最多16384个槽。

```redis
CLUSTER ADDSLOTS <slot> [slot ...]
```

#### 记录节点的槽指派信息

clusterNode 结构的 slots 属性和 numslot 属性记录了节点负责处理那些槽。

```c
struct clusterNode {
    // ...

    unsigned char slots[16384/8];

    int numslots;
    // ...
}
```

#### 传播节点的槽指派信息

一个节点除了将自己负责的槽记录在 clusterNode 结构的 slots 属性和 numslots 属性之外，还会将自己的 slots 数组通过消息发送给集群中的其他节点。

其他节点接收到数组后，会从自己的 clusterState.nodes 字典中查找对应的 clusterNode 结构，并对结构中的 slots 数组进行保存或者更新。

#### 记录所有槽的指派信息

clusterState 结构中的 slots 数组记录了集群中所有 16384 个槽的指派信息：

```c
typedef struct clusterState {
    // ...

    clusterNode *slots[16384];

    // ...
}
```

* 如果 slots[i] 指针指向 NULL，表示槽 i 尚未指派给任何节点。
* 如果 slots[i] 指针指向一个 clusterNode 结构，那么表示槽 i 已经指派给了 clusterNode 结构所代表的节点。

### 在集群中执行命令

当客户端向节点发送与数据库键有关的命令时，接收命令的节点会计算出命令要处理的数据库键属于哪个槽，并检查这个槽是否指派给了自己：

* 如果键所在的槽正好就指派给了当前节点，那么节点直接执行这个命令。
* 如果没有指派给当前节点，那么节点会向客户端返回一个 MOVED 错误，指引客户端 redirect 至正确的节点，并在此发送之前的命令。

#### 计算键属于哪个槽

使用 `CLUSTER KEYSLOT <key>` 命令可以查看一个给定键属于哪个槽。

#### 判断槽是否由当前节点负责处理

如果 clusterState.slots[i] 等于 clusterState.myself，那么说明槽 i 由当前节点负责，节点可以执行命令。

否则，返回 MOVED 错误，并指引客户端转向至正在处理槽 i 的节点。

#### MOVED 错误

错误格式：`MOVED <slot> <ip>:<port>`

#### 节点数据库的实现

节点只能使用 0 号数据库。

### 重新分片

可以将任意数量已经指派给某个节点的槽改为指派给另一个节点，并且相关槽所属的键值对也会转移。

重写分片操作可以在线进行，并且指派的两个节点都可以继续处理命令请求。

### ASK 错误

当客户端向源节点发送一个与数据库键相关

### 复制与故障转移

### 消息
