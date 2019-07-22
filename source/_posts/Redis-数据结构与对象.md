---
title: Redis 数据结构与对象
date: 2019-07-20 21:55:09
tags: Redis
categories: coding
---
## 数据结构与对象

### 简单动态字符串

#### 结构

* len : buf 数组已使用字节数量
* free : buf 数组未使用字节数量
* buf[] : 用于保存字符串

#### 优点

1. 常数复杂度获取字符串长度。
 O(1)的时间复杂度
2. 杜绝缓冲区溢出。
当 SDS API 对 SDS 进行修改时，API 会先检查空间是否满足修改的需求。如果不满足 API 会自动扩展空间，然后执行实际修改。
3. 减少修改字符串长度所需的内存重分配次数。
通过未使用空间解除了字符串长度和底层数组长度的关联。
空间预分配：用于优化 SDS 字符串增长。当 SDS 的 API 对一个 SDS 进行修改，并且需要对 SDS 进行空间扩展的时候，程序不进会为 SDS 分配修改所必须要的空间，还会分配额外的未使用空间。
惰性空间：由于优化 SDS 字符串缩短。当 SDS 的 API 需要缩短 SDS 保存的字符串时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用 free 属性将这些字节的数量记录起来，并等待将来使用。
4. 二进制安全。
5. 兼容部分 C 字符串函数。

### 链表

#### 链表结构

链表节点结构

```c
typedef struct listNode {
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;
}
```

list 结构

```c
typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 链表所包含的节点数量
    unsigned long len;
    // 节点值复制函数
    void *(*dup) (void *ptr);
    // 节点值释放函数
    void *(*free) (void *ptr);
    // 节点值对比函数
    int (*match) (void *ptr,void *key);
} list;
```

由 list 结构和 listNode 结构组成的链表

![list](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/redis-list.png)

特点：

1. 双端
2. 无环
3. 带表头指针和表尾指针
4. 带链表长度计数器
5. 多态：链表可以存放各种类型的值。

### 字典结构

又称符号表、关联数组、映射，用于保存键值对。

底层使用哈希表实现，一个哈希表有多个哈希节点，每个哈希节点保存了字典中的一个键值对。

#### 哈希表

```c
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值，= size - 1
    unsigned long sizemask;
    // 该哈希表已有节点数量
    unsigned long used;
} dictht;
```

#### 哈希表节点

```c
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union{
        void *val;
        uint64_tu64;
        int64_ts64;
    } v;
    // 指向下一个哈希节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

`*next` 可以将多个哈希值相同的键值对连接在一起。

![hashtable](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/%E5%93%88%E5%B8%8C%E8%A1%A8.png)

#### 字典

```c
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为-1
    in trehashidx;
} dict;
```

type 属性是一个指向 dictType 结构的指针，每个 dicType 保存了用于操作特定类型键值对的函数。

privdata 属性则保存了需要传递给特定函数的可选参数。

ht 属性是包含两个项的数组，字典只使用 ht[0] 哈希表，ht[1] 哈希表只会在对 ht[0] 哈希表 rehash 时使用。

![dict](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/dict.png)

#### 哈希算法

Redis 计算哈希值和索引值如下：

```c
# 使用字典设置的哈希函数计算出键对应的哈希值
hash = dict -> type -> hashFunction(key)
# 使用哈希表的 sizemask 和哈希值计算出索引值
index = hash & dict -> ht[x].sizemask
```

#### 解决键冲突

使用链地址法，每个哈希表节点都有一个 next 指针。

程序将新节点添加到链表的表头位置。

#### rehash

扩展或收缩哈希表的工作由 rehash（重新散列）完成。

1. 为字典的 ht[1] 哈希表分配空间，大小取决于要执行的操作，以及 ht[0] 当前包含的键值对。
2. 将保存在 ht[0] 上的所有键值对 rehash 到 ht[1] 上。
3. 迁移后，释放 ht[0]，将 ht[1] 设置成 ht[0], 并在 ht[1] 创建一个空白哈希表。

#### 渐进式 rehash

采用分而治之的方式，将 rehash 键值对所需的计算工作均摊到对字典的每个添加、删除、查找和更新上，避免了集中式 rehash 带来的庞大计算量。

渐进式 rehash 期间，删除、查找、更新会在两个哈希表上进行，插入只会在 ht[1] 上进行。

### 跳跃表

#### 跳跃表的实现

跳跃表是一种有序数据结构。是有序集合键的底层实现之一。Redis 只在两个地方用到了跳跃表，一个实现有序集合键，另一个在集群节点中用作内部数据结构。

跳跃表包括一个 zskiplist 和 若干个 zskiplistNode。

![skiplist](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/skiplist.png)

zskiplist 结构包含

* header : 指向跳跃表的表头节点。
* tail : 指向跳跃表的表尾节点。
* level : 记录跳跃表中层数最大的节点的层数（表头节点除外）。
* length : 记录跳跃表目前包含的节点的数量（表头节点除外）。

```c
typedef struct zskiplist {
// 表头节点和表尾节点
structz skiplistNode *header, *tail;
// 表中节点的数量
unsigned long length;
// 表中层数最大的节点的层数
int level;
} zskiplist;
```

zskiplistNode 结构包含：

* level : L1\L2\L3...，每个层有两个属性，前进指针和跨度，1-32的随机数。
* 后退指针 : BW 表示后退指针。
* 分值 : 跳跃表中各个节点按照分值从小到大排列。
* 成员对象 : o1\o2\o3...

```c
typedef struct zskiplistNode {
    // 层
    struct zskiplistLevel {
        // 前进节点
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
    // 后退指针
    struct zskiplistNode *backward;
    // 分值
    double score;
    // 成员对象
    robj *obj;
} zskiplistNode;
```

前进指针用于遍历，跨度用于计算排位。

### 整数集合

#### 整数集合的实现

可以保存类型为 int16_t、int32_t 或者 int64_t 的整数值，并且保证集合中不出现重复元素。

```c
typedef struct intset {
// 编码方式
uint32_t encoding;
// 集合包含的元素数量
uint32_t length;
// 保存元素的数组
int8_t contents[];
} intset;
```

`contents[]` 每项按值的大小从小到大排列，且不包含重复项。

#### 升级

升级步骤：

1. 根据新元素类型，扩展整数集合底层数组的空间大小，并为新元素分配空间。
2. 将数组现有元素转换成与新元素相同的类型，并将转换后的元素放置在正确的位上，保持有序。
3. 将新元素添加到底层数组。

好处：

1. 提升灵活性
2. 节约内存

#### 降级

不支持降级。

### 压缩列表

压缩列表是为了节约内存而设计的。是由一系列特殊编码的连续内存组成的顺序性数据结构。

#### 压缩列表的组成

一个压缩列表由任意多个节点组成，每个节点可以保存一个字节数组或者一个整数值。

1. zlbytes：记录整个压缩列表占用的内存字节数。
2. zltail：记录压缩列表表尾距离压缩列表的起始地址有多少字节。
3. zllen：记录了压缩列表包含的节点数量。
4. entryX：各个节点。
5. zlend：标记压缩列表末端。

![ziplist](https://raw.githubusercontent.com/jianghs/myBlogPicBed/master/redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0%E6%88%AA%E5%9B%BE/ziplist.png)

#### 压缩列表节点

可以保存一个字节数组或者整数值。

1. previous_entry_length：前一个节点的长度。
2. encoding：记录 content 属性保存的数据类型和长度。
3. content：保存节点的值。

#### 连锁更新

连锁更新的最坏复杂度是O(n2)。

### 对象

Redis 没有直接使用上述数据结构来实现键值对数据库，而是基于这些数据结构创建了一个对象系统。

* 字符串对象
* 列表对象
* 哈希对象
* 集合对象
* 有序集合对象

#### 对象的类型和编码

每个对象的结构如下：

```c
typedef struct redisObjec {
    // 类型
    unsigned type : 4;
    // 编码
    unsigned encoding : 4;
    // 指向底层实现数据结构的指针
    void *ptr;
    ...
}
```

对象的类型如下：

|类型常量|对象的名称
| --- | --- |
| REDIS_STRING|字符串常量
|REDIS_LIST|列表常量
| REDIS_HASH |哈希常量
|REDIS_SET  |集合常量
|REDIS_ZSET  |有序集合常量

对于 Redis 数据库保存的键值来说。键总是字符串对象，值可能是字符串对象、列表对象、哈希对象、集合对象或有序集合对象。

对象的编码

|编码常量  |编码所对应的底层数据结构  |
| --- | --- |
|REDIS_ENCODING_INT  | long 型整数 |
|REDIS_ENCODING_EMBSTR  |  embstr 编码的简单动态字符串|
|REDIS_ENCODING_RAW  |  简单动态字符串|
|REDIS_ENCODING_HT  |  字典|
|REDIS_ENCODING_LINKEDLIST  | 双端链表 |
|REDIS_ENCODING_ZIPLIST  |  压缩列表|
|REDIS_ENCODING_INTSET  |  整数集合|
|REDIS_ENCODING_SKIPLIST  |  跳跃表和字典|

每个类型的对象至少使用了两种不同的编码。

Redis 可以根据不同的场景来为同一对象设置不同的编码，从而优化对象在某一场景的效率。

#### 字符串对象

字符串对象的编码可以用 int\raw\embstr。

1. 一个字符串对象保存的是整数值，并且这个整数可以用 long 类型表示，则用 int 类型表示。

2. embstr 专门用于优化短字符串，字符串长度下于等于32个字节。

|类型|内存分配次数|内存释放次数|内存空间|
| --- | --- | --- | --- |
|raw|2|2|分两次创建|
|embstr|1|1|一块连续的内存空间|

#### 列表对象

列表对象的编码可以是 ziplist 或 linkedlist。

1. ziplist 底层使用压缩列表实现。

   使用 ziplist 的条件：

    * 列表对象保存的所有字符串元素的长度都小于64字节。

    * 列表对象保存的元素数量小于512个。

    如果有一个条件不满足则使用 linkedlist。

2. linkedlist 底层使用双向链表实现。

#### 哈希对象

哈希对象的编码可以是 ziplist 或 hashtable。

1. ziplist 编码的哈希对象使用压缩列表作为底层实现。

    * 保存了同一键值对的两个节点总是紧挨在一起，保存键的节点在前，保存值的节点在后。
    * 先添加到哈希对象中的键值对会被放在压缩列表的表头方向，后添加到哈希对象中的键值对会被放在压缩列表的表尾方向。

2. hashtable 编码的哈希对象使用字典座位底层实现。
    * 字典中的每个键都是一个字符串对象，对象中保存了键值对的键。
    * 字典中的每个值都是一个字符串对象，对象中保存了键值对的值。

3. 编码转换
    * 哈希对象保存的所有键值对的键和值的字符串长度小于64字节。
    * 哈希对象保存的键值对数量小于512个。

    两个条件同时满足时，使用 ziplist 编码。

#### 集合对象

集合对象的编码可以是 intset 或 hashtable。

1. intset 编码的集合对象使用整数集合作为底层实现，集合对象包含的所有元素都被保存在整数集合里面。

2. hashtable 编码的集合对象使用字典座位底层实现，字典每个键都是一个字符串对象，每个字符串对象包含了一个集合元素，而字典的值全部被设置为 NULL。

3. 编码转换
    * 集合对象保存的所有元素都是整数值。
    * 集合对象保存的元素数量小于512个。

    两个条件同时满足时，使用 intset 编码。

#### 有序集合对象

有序集合的编码可以是 ziplist 或 skiplist。

1. ziplist 编码的压缩列表对象使用压缩列表座位底层实现，每个元素使用两个紧挨一起的压缩列表节点来保存，第一个节点保存元素的成员，第二个点解保存元素的分值。

    按照分值从小到大排列。分值较小的元素靠近表头位置，分值较大的元素靠近表尾位置。

2. skiplist 编码的有序集合对象使用 zset 结构作为底层实现，一个 zset 结构包含一个字典和一个跳跃表。

    ```c
    typedef struct zset {
        zskiplist *zsl;
        dict *dict;
    } zset;
    ```

    * zsl 跳跃表按照分值从小到大保存了所有集合元素，每个跳跃表节点都保存了一个集合元素：跳跃表节点的 object 属性保存了元素成员，score 属性则保存分值。
    * dict 字典为有序集合创建看一个从成员到分值的映射，字典中的每个键值对都保存了一个集合元素：键保存了元素，值保存了分值。

3. 编码转换
    * 有序集合保存的元素数量小于128个。
    * 有序集合保存的所有元素成员的长度都小于64字节。

    两个条件同时满足时，使用 ziplist 编码。

#### 类型检查与命令多态

* 类型检查的实现

    1. 执行特定类型特定命令之前，服务器先检查输入数据库键的值对象是否为执行命令的所需类型，如果是，则执行。
    2. 否则，服务器拒绝执行，并返回一个类型错误。

* 命令多态的实现

    Redis 除了会根据值对象判断键是否执行命令外，还会根据值对象的编码方式，选择正确的命令实现代码来执行命令。

#### 内存回收

Redis 构建了一个引用计数计数实现的内存回收机制。

1. 创建一个新对象时，引用计数的值初始化为1。
2. 当对象被一个新程序使用时，它的引用计数值会加1。
3. 当对象不再被一个程序使用时，它的引用计数值会减1。
4. 当对象的引用计数值为0时，对象所占用的内存会被释放。

#### 对象共享

1. 将数据库键的值指针指向一个现有的值对象；
2. 将被共享的值对象的引用计数加1。

Redis 初始化服务器时，创建0-9999一共一万个整数值。

#### 对象的空转时间

lru 属性记录了对象最后一次被命令程序访问的时间，可以用于计算空转时间。
