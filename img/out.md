<a name="i3DyD"></a>
# 安装 与 配置
<a name="txvN7"></a>
## 使用 yum 安装
```bash
yum install centos-release-scl-rh
yum install rh-redis5-redis
```

SCL源SCL（Software Collections）是 CentOS 提供的一种机制，**用于并行安装和使用多个软件版本**。SCL 源（Software Collections Repository）是 CentOS 的一个额外软件仓库，包含了一系列的软件包和工具，可以满足特定的应用程序和开发需求。

要启用 SCL 源，你需要执行以下步骤：

1.  安装 `centos-release-scl` 软件包： 
```
sudo yum install centos-release-scl
```
 

2.  更新软件包缓存： 
```
sudo yum update
```
 

3.  安装所需的 SCL 软件包。例如，如果你想安装 Redis 5.x 版本，可以执行以下命令： 
```
sudo yum install rh-redis5
```
 

4.  启用 SCL 软件包。你可以通过两种方式来启用： 
   - 临时启用：在执行命令时使用 `scl enable` 前缀。例如，要运行 Redis 5.x 版本的 `redis-cli`，可以执行以下命令：
```
scl enable rh-redis5 'redis-cli'
```

   - 永久启用：编辑用户的 `.bashrc` 或 `.bash_profile` 文件，在文件末尾添加以下行（以 Redis 5.x 为例）：
```
source /opt/rh/rh-redis5/enable
```
 然后重新登录或执行 `source` 命令来加载修改后的环境变量。

启用 SCL 源后，你就可以在 CentOS 系统中同时安装和使用多个软件版本。请注意，具体的 SCL 软件包名称可能因软件版本和发行版而异，你可以根据自己的需求来选择正确的软件包。


<a name="WFfxF"></a>
## 创建符号链接
<a name="DybkU"></a>
### 软链接
**软链接相当于 Windows 中的快捷方式**，它们**指向的是源文件的路径，而不是文件本身**。<br />以`ln -s /opt/rh/rh-redis5/root/usr/bin/redis-server ./redis-server`为例，它将 `/opt/rh/rh-redis5/root/usr/bin/redis-server` 这个路径下的 `redis-server` 可执行文件**创建一个软链接（symbolic link）到当前目录下的 **`**redis-server**`** 文件中。**<br />具体来说，这个命令的参数解释如下：

- `ln`: **创建链接**的命令。
- `-s`: 表示**创建的链接为符号链接（symbolic link）**，也称**软链接**。
- `/opt/rh/rh-redis5/root/usr/bin/redis-server`: 指定**源文件的路径**。
- `./redis-server`: **指定链接的名称和路径**。这里的 `./` 表示当前目录，因此该命令将在当前目录下创建一个名为 `redis-server` 的软链接，指向 `/opt/rh/rh-redis5/root/usr/bin/redis-server` 可执行文件。

<a name="WIVuR"></a>
### 相关指令
```bash
cd /usr/bin
ln -s /opt/rh/rh-redis5/root/usr/bin/redis-server ./redis-server
ln -s /opt/rh/rh-redis5/root/usr/bin/redis-sentinel ./redis-sentinel
ln -s /opt/rh/rh-redis5/root/usr/bin/redis-cli ./redis-cli
```
![image.png](D:\test\image-0.png)

```bash
cd /etc/
ln -s /etc/opt/rh/rh-redis5/ ./redis
```

<a name="TqWsj"></a>
## 修改配置文件
```bash
mkdir -p /var/lib/redis # 先出去设置目录

vi redis.conf

# 设置 ip 地址
# 指定 Redis 监听的地址（本地回环地址 127.0.0.1），即【只允许本地访问】
bind 0.0.0.0

# 关闭保护模式
# 允许来自【任意 IP 地址的连接】
protected-mode no

# 启动守护进程
# 在【后台运行】，并且不会占用当前终端的控制权
daemonize yes

# 设置工作目录
dir /var/lib/redis

# 设置日志目录
logfile /var/log/redis/1 redis-server.log

/dir 回车 按N下一个
:wq # 保持
```
设置完之后需要重启 Redis 服务才能生效


<a name="oWd8F"></a>
## Redis 的启停
```bash
# 启动
redis-server /etc/redis/redis.conf

# 查看启动的PID
netstat -anp | grep redis

# 查看 redis-server 的PID
ps aux | grep redis

# 通过 kill 命令直接杀死 redis 进程
kill PID
```
![image.png](D:\test\image-1.png)



<a name="Mdh2t"></a>
# Redis 的基本介绍
<a name="ypjwQ"></a>
## 关于 Redis
简单来说 Redis 是一个在内存中存储数据的中间件。
<a name="BWMyl"></a>
### 特性
<a name="Oz7Es"></a>
#### （1）在内存中存储数据
使用键值对的方式存储数据，其中`**key**`**都是**`**string**`**，**`**value**`**都是则可以是不同数据结构的数据。**与 `MySQL`相比，`Redis`这种以“键值对”的方式组织数据的方式，称为：**非关系型数据库。**
<a name="KE8w5"></a>
#### （2）可编程
可以直接通过简单的 **交互式命令** 操作，也可以通过 `**Lua**`**脚本 **进行操作。
<a name="oKsPS"></a>
#### （3）可扩展
`Redis`提供了一组`API`，可以通过 `C`、`C++`、`Rust` 语言编写的**动态链接库（可以理解为：函数库。在 **`**windows**`**上是**`**.ddl**`** 文件，在**`**Linux**`**上是 **`**.so**`** 文件）**，在原有的功能基础上再进行扩展。
<a name="mLwTn"></a>
#### （4）持久化
明明`Redis`是在内存中存储数据，那么进程退出 或者 系统重启，都会导致数据的丢失，那怎么能说持久化呢？其实，这里的持久化是指：`**Redis**`**也会把数据存储在硬盘上**，也就说：硬盘上备份了`Redis`的数据。**重启的**`**Redis**`**只需要加载硬盘中的备份数据即可**，这样`Redis`的内存就能恢复到重启前的状态。
<a name="anQ5X"></a>
#### （5）多集群
一个 `Redis`存储的数据是有限的，为了尽可能多地使用到 `Redis`的特性，就需要引入多台主机，**将**`**Redis**`**部署到多个节点上**。这样每个 `Redis` 都能存储一部分数据。能够被部署在多台机器上的`Redis`也就拥有了多集群的特点。
<a name="qHnSO"></a>
#### （6）高可用
`Redis`支持“主从”结构，从节点就相当于主节点的备份。
<a name="vjNM3"></a>
#### （7）快速高效 *
为什么`Redis`会快呢？主要是一下四点原因：

- 从**访问硬件**上看，第一点也是`Redis`特性的第一点，`**Redis**`**数据存储在内存中**，访问内存的速度 比 访问硬盘和数据库的速度快得多。
- 从**业务角度**上看，`Redis`核心功能的逻辑简单，大部分是操作内存的数据结构。相比起`MySQL`还有聚合函数，链表查询等......`Redis`的应用场景比较简单。
- 从**网络角度**上看，`Redis`使用了 **IO多路复用 **的方式（epoll）,使用了一个线程管理多个 `socket`，即：同时去买三份小吃，哪一份完成就先响应哪一份，注意
- 从**实现模型**上看，`Redis`使用的是 **单线程 模型。**这减少了不必要的线程竞争开销。这里多解释一下，多线程能提高效率是有场景要求的，即 **CPU密集性任务 **的场景。此时，多个线程才能够**充分利用 CPU的多核资源**。

<a name="jpoFv"></a>
### 应用场景
<a name="AS0kQ"></a>
#### （1）把 `Redis`当数据库
这里指的是把 `Redis`当作全量数据库，也就是说 这些数据并不能丢失，而且这部分被存储的数据不仅多，而且需要快速被搜索到。想要这样使用`Redis`就需要不少的硬件资源，而这只能发挥公司的“钞”能力了。

<a name="EcGpL"></a>
#### （2）把`Redis`当`Caching`和 `session`
当作缓存（Caching）的话，其实就是**把热点数据提出来存储到**`**Redis**`**中**，也就是所谓的**冷热分离。**此时的`Redis`存储部分数据，当`Redis`重启之后，数据依然可以从`MySQL`中读取回来。

当作会话（Session）的意思其实是：**使用**`**Redis**`**存储会话信息**，所有的服务器最终**统一地到 **`**Redis**`**中获取会话信息**即可。这样就**不需要负载均衡器 根据 **`**userID**`** 做会话保持**。<br />![image.png](D:\test\image-2.png)


<a name="A5b9g"></a>
#### （3）把`Redis`当消息队列
消息队列是一种在**应用程序之间传递消息的通信模式**。它允许发送者（生产者）将消息发送到一个队列中，然后接收者（消费者）可以从队列中获取并处理这些消息。消息队列实现了**异步通信机制，提供了解耦和缓冲的功能**，使得不同的应用程序或服务可以在时间和空间上解耦，以便更高效地进行通信。

1）消息队列通常由以下组件构成：

- 消息：要传递的数据或信息。
- 队列：存储消息的容器，通常是先进先出（FIFO）的数据结构。
- 生产者：将消息发送到队列的应用程序或服务。
- 消费者：从队列中获取消息并进行处理的应用程序或服务。
- 中间件：支持消息传递的中间软件，负责管理和维护队列，确保消息的可靠传递和处理。

2）消息队列的优点包括：

- 异步处理：生产者和消费者之间解耦，不需要即时响应，可以自由选择处理消息的时间和顺序。
- 削峰填谷：通过缓冲消息，消费者可以按照自身处理能力来消化消息，避免了系统因突发高峰而崩溃。
- 可伸缩性：可以根据需求增加或减少生产者和消费者的数量，以适应系统的负载变化。
- 系统解耦：不同的应用程序或服务之间通过消息队列进行通信，解耦了彼此之间的依赖关系，提高了系统的灵活性和可维护性。
- 可靠性：消息队列通常会提供持久化机制来确保消息的可靠传递，防止消息丢失。

消息队列在分布式系统、微服务架构、任务调度等场景下具有广泛的应用。一些常见的消息队列实现包括 RabbitMQ、Apache Kafka、ActiveMQ 等。但是，如果不想使用消息队列的功能，又不想引入过多的中间件，可以考虑只用`Redis`做消息队列。



<a name="VbOyk"></a>
## 基本指令
```bash
redis-server /etc/redis/redis.conf # 启动Redis
redis-cli # 进入客户端
set key1 value1
get key1
get key2 # 不存在返回nil
```
![image.png](D:\test\image-3.png)

```bash
# 匹配模式
# ？ 一个字符
# * 多个字符
# [xxxx] 出现什么匹配什么
# [^xxx] 出现什么排除什么
# [a-z] 出现范围（一位字符）
```
注意在生产环境上不要随便使用 `keys *` ，原因很简单：我们能够清晰地感受到这个操作的时间复杂度是 `O(n)`，而`Redis`是一个单线程服务器，且生产环境上的`key`是很多的，那么这个操作执行的时间就很长，整`Redis`服务器就会被阻塞住，无法给其他客户端提供服务！此时，其余的`Redis`操作就会超时。<br />![image.png](D:\test\image-4.png)


```bash
exists key1 key2 ...
# 时间复杂度是 O(N) ，N 是 key 的个数
```
![image.png](D:\test\image-5.png)

```bash
keys * # 先查看一下 keys 有多少
del hllo 
del hallo hello hzllo # 返回的是个数
```
![image.png](D:\test\image-6.png)


```bash
set k v
get k
expire k 5 # 单位是秒，pexpire 单位是毫秒
ttl k # 当被删除之后返回的结果就是 -2；如果没有关联过期时间返回结果就是 -1；pttl 单位是毫秒
```
这个功能还是非常有用的，常见的场景比如：验证码，优惠券过期 等。<br />那么这个过期删除策略是怎么样的呢？主要有两个：**定期删除 + 惰性删除。**<br />所谓惰性删除就是：如果`key`已经过期，但是并没有请求来查询，那么这个`key`就暂时不会被删除，直到被请求访问的时候`Redis`才会删除`key`，同时返回一个`nil`。但是这样并不靠谱的。**因此还需要结合“定时删除”。**不过，当有很多`key`时，全部过滤一遍成本可能会很高，占用的时间可能会边长，占用时间一旦过长，就很可能出现跟上方`keys *`一样的情况！因此，**定期删除只是抽取一部分**`**key**`**已经检查**。<br />注意：`Redis`并没有使用定时删除的策略。就算采用也不必未每一个`key`设置一个定时器，未引入的原因很可能是因为`Redis`最初是单线程的设计模式，如果使用定时器，势必要引入多线程，这与设计的初衷相悖。<br />但是上方两种策略依旧不能让人满意，因此`**Redis**`**还有内存淘汰机制**。<br />![image.png](D:\test\image-7.png)


```bash
# 常见数据类型有以下几种：
# none、string、list、set、zset、hash、stream
keys *
type key1

lpush key2 111 222 333
type key2

sadd key3 111 222 333
type key3

hset key4 k1 v1 k2 v2
type key4
```
![image.png](D:\test\image-8.png)


<a name="ez81y"></a>
# 数据类型 与 编码方式
<a name="tYSX4"></a>
## 数据类型
![image.png](D:\test\image-9.png)

上方只是常用的`Redis`数据类型，`Redis`还有其他的若干种数据类型，但适用的场景可能没这么广。这里还需要说明的是：`**Redis**`**底层在实现上述数据结构时，会在源码层面针对上述实现进行特定优化，以 节省时间、节省空间。**也就是说：`Redis`会承诺相关的操作时间复杂度不变，但背后实现的细节并不一定是所见的标准数据类型，**在特定场景下，会采用其他数据类型实现。**

<a name="s89vv"></a>
## 内部编码
![image.png](D:\test\image-10.png)

为什么`hash`要压缩为`ziplist`？<br />因为在`Redis`中的`key`很多，对于的`value`为`hash`的也很多，但是每个`hash`不是很大的情况下压缩成`ziplist`就会比较节省空间，整个的空间占用率就会下降 且 查询速度并没有降下来。

跳表是什么？<br />跳表也是链表，只不过每个节点上有多个指针域。根据设置的特殊规则，使用跳表查询元素的时间复杂度是`O(logN)`

```bash
keys *
type key1
object encoding key1 # error 注意'object' 需要大写
OBJECT encoding key1
```
![image.png](D:\test\image-11.png)


<a name="LbcMf"></a>
# 各类型指令介绍
```bash
keys *
FLUSHALL # 清空数据库中所有的 k-v，不可在生产环境上使用！
keys * 
```
![image.png](D:\test\image-12.png)
<a name="ZaUiv"></a>
## String类型
<a name="R1P0t"></a>
### 基础指令
```bash
# NX 不存在才设置
# XX 存在才设置
set k1 v1
get k1

set k2 v2 ex 5 # 设置过期时间
ttl k2

keys * # k2 已经过期
set k2 v2 NX # 不存在才设置
get k2

set k2 v2-2 NX # nil - 不存在才设置，当前已存在

get k2
set k2 v2-2 XX # 存在才设置
get k2

set k3 v3 XX # nil - 存在才设置，当前不存在
get k3 # nil
```
![image.png](D:\test\image-13.png)


```bash
# 一次设置多个 k-v，可以减少网络请求的消耗
# 但是也别设置太多，免得把 Redis 阻塞住了
mset k3 v3 k4 v4 k5 v5
mget k3 k4 k5
```
![image.png](D:\test\image-14.png)


```bash
keys *
setnx k6 v6 # 不存在才设置
setnx k6 v6-6 # 不存在才设置
get k6

setex k7 5 v7
ttl k7
get k7

psetex k7 5000 v7
pttl k7
get k7
```
![image.png](D:\test\image-15.png)


```bash
set k1 9
get k1
incr k1 # ++i；若key不存在会把value当0，返回1

set k2 hello
get k2
incr k2 # value 不能是字符串

set k2 1.5
get k2
incr k2 # value 不能是小数

set k2 2222222222222222222222222222222
get k2
incr k2 # value 不能是特别大的数

decr k1 # --i；若key不存在会把value当0，返回-1
get k1

# incrby key 10 # i += 10
# incrby key -1 # i += (-1)

# decrby key 10 # i -= 10
# decrby key -1 # i -= (-1)

# incrbyfloat key 0.5 # i += 0.5
```
![image.png](D:\test\image-16.png)


```bash
set key hello
append key world # append 返回的是长度的字节，具体长度需要结合具体的字符编码
get key

append k hi~ # 会直接为不存在的key设置value
get k

set key 你好
append key 世界
get key # 返回的结果是不认识的二进制

redis-cli --raw # 使用 --raw 让Redis尝试翻译二进制数据
get key
```
![image.png](D:\test\image-17.png)


```bash
# 在Redis中指定的区间是 [ , ]
set key HelloWorld
get key
getrange key 0 -1 # -1 是最后一个字符
getrange key 1 -2

set key 你好
get key
getrange key 0 -1
getrange key 1 -2 # 奇奇怪怪的符号，这个 getrange 的强行切字节


set key HelloWorld
setrange key 5 world
get key
setrange -1 D # 无法使用负数来达成 倒数 的操作
setrange k1 1 hello # 不存在的key，旧版本的Redis会使用 0x00 来填充
```
![image.png](D:\test\image-18.png)


```bash
# strlen 获取到的字符串的长度，单位是字节
keys *
get key
strlen key

set key 你好
get key
strlen key # utf8字符集中汉字是3byte

strlen kk # key不存在返回0
```
![image.png](D:\test\image-19.png)

<a name="ofRkZ"></a>
### 内部编码
```bash
set k1 123 # 当 long 来存储
OBJECT encoding k1 # int

set k2 hello # 若干存储小数，那么就当作字符串来存储
OBJECT encoding k2 # embstr

set k3 aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
OBJECT encoding k3 # raw
```
![image.png](D:\test\image-20.png)


<a name="uxmpg"></a>
## Hash类型
<a name="PRSdU"></a>
### 基础指令
```bash
hset k1 f1 v1
hget k1 f1

hset k2 f2 v2 f3 v3
hget k2 f2 f3 # error，一次只能返回一个
hget k2 f2
hget k2 f3
hget k3 f2 # key不存在会返回 nil
hget k2 f22 # field不存在也会返回 nil
```
![image.png](D:\test\image-21.png)

```bash
hexists k1 f1
hexists k1 f11 # field不存在
hexists k3 f3  # key不存在
```
![image.png](D:\test\image-22.png)


```bash
# del 删除的是整个 hash
# 删除的是field，返回的是删除的个数
hdel k2 f2 
hdel k2 f3 

hset k2 f22 v22 f33 v33
hdel k2 f22 f33
```
![image.png](D:\test\image-23.png)


```bash
hset k2 f222 v222 f333 v333 f444 v444
keys *

# 注意下方两个指令要慎重使用，因为 field-value 可能有很多
hkeys k1
hkeys k2 # 查看这个key中的所有field

hvals k1
hvals k2 # 查看这个key中的所有field对于的value

hgetall k1 # 查看这个key所有的 field-value 
hgetall k2 

hmget k2 f222 f333 # 查看这个key中指定field的value
hmget k2 f222 f333 f444
```
![image.png](D:\test\image-24.png)


```bash
# hscan 渐进式遍历，连续执行多次就能遍历整个hash，
# 避免了遍历过程中执行时间过程导致的Redis被阻塞
```

```bash
hlen k1 # 获取 hash 元素的个数，时间复杂度是O(1)，直接存在起来，不需要遍历
hlen k2 


hsetnx k2 f5 v5 # 存在则失败
hget k2 f5
hsetnx k2 f5 v55 # 因为第一次使用hsetnx时不存在，那是正常，存在则失败


hset k3 f1 1 f2 2.2
hgetall k3
hincrby k3 f1 10
hincrby k3 f1 -1
hincrbe k3 f2 0.2
hincrbe k3 f2 -0.2
```
![image.png](D:\test\image-25.png)

<a name="gdCB6"></a>
### 内部编码
```bash
hset key f1 111
OBJECT encoding key

hset key f2 222222222222...2222222222
OBJECT encoding key
```
![image.png](D:\test\image-26.png)


<a name="OSfaQ"></a>
## List类型
<a name="j2bCq"></a>
### 基本指令
```bash
 lpush k1 1 2 3 4
 lpush k1 5 6 7 8

 # 两边都是闭区间 且 下标支持负数
 lrange k1 0 -1
 lrange k1 0 7
 lrange k1 0 100 # 下标越界的话将尽可能获取
```
![image.png](D:\test\image-27.png)

```bash
keys *
lpushx key 1 2 3 4 5 # 会检查 key 是否 exists

lpush key 1 2 3
rpush key 0 -1 -2
lrange key 0 -1

rpushx k2 1 2 3
```
![image.png](D:\test\image-28.png)


```bash
lrange key 0 -1
lpop key
lpop key

lrange key 0 -1

rpop key
rpop key

lrange key 0 -1
```
![image.png](D:\test\image-29.png)



```bash
rpush key 0 1 2 3 4 5 6 7
lrange key 0 -1

lindex key 3
lindex key -1
lindex key 100
```
![image.png](D:\test\image-30.png)

```bash
# 找的是第一个出现的基准值
lrange key 0 -1
linsert key before 2 100 # 插入到下标是2的元素之前
lrange key 0 -1

linsert key after 2 100 # 插入到下标是2的元素之后
lrange key 0 -1

# 获取列表长度
llen k1
llen key
```
![image.png](D:\test\image-31.png)


```bash
# lrem 指定元素删除
# count > 0 从左向右找删除 count 个 element
# count = 0 删除所有的 element
# count < 0 从右向左找删除 count 个 element
lrange key 0 -1
lrem key 2 1 # 删除 前2个 1
lrem key -2 1 # 删除 后2个 1
lrem key 0 2 # 删除所有 2
lrange key 0 -1


# ltrim 保留[start, stop]
rpush key 0 1 2 3 4 5 6 7 8 9 10
ltrim key 2 8 # 保留下标 [2, 8] 的数值
```
![image.png](D:\test\image-32.png)


```bash
rpush key 0 1 2 3 4 5 6 7 8 9 10
lrange key 0 -1
lset key 5 50 # lindex访问下标越界会返回nil，lset则直接报错
lrange key 0 -1
```
![image.png](D:\test\image-33.png)


```bash
# blpop brpop阻塞版本的命令
# （1）如果队列为空，尝试出队列，就会产生阻塞。直到队列不空，阻塞解除 √
# 如果队列为满，尝试入队列，也会产生阻塞。直到队列不满，阻塞解除 ×
# （2）但此处 Redis 在阻塞期间（也就说有超时时间）也可以正常处理其他请求
# （3）若有多个key，则会从左向右遍历，哪个列表先弹出就返回哪个
# （4）多个客户端对同一个key执行pop，最先执行命令的客户端会先弹出元素
rpush key 1 2 3 4
blpop key 0
lrange key 0 -1


# A窗口
blpop key 100
# B窗口
rpush key 1 2 3 4
# A窗口立即返回结果


# A窗口
blpop k1 k2 k3 k4 500
# B窗口
lrange k1 0 -1
lrange k2 0 -1
lrange k3 0 -1
lrange k4 0 -1
lpush k1 1 2 3
# A窗口立即返回结果
```
![image.png](D:\test\image-34.png)


<a name="yGhiq"></a>
### 内部编码
```bash
OBJECT encoding key
```
![image.png](D:\test\image-35.png)


<a name="yIfuy"></a>
## Set类型
<a name="AnhTO"></a>
### 基本指令
```bash
sadd key 1 2 3 4
type key

sadd k 1 1 2 3 4 # 添加元素，有重复会自动删除

smembers key
smembers k 

sismember key 1
sismember key 5 # 删除不存在的元素，找不到该元素则删除0个
```
![image.png](D:\test\image-36.png)


```bash
smembers key
# 随机删除 -- 返回删除成功的元素
spop key 
spop key
spop key
spop key

# 指定删除 -- 返回删除成功的元素个数
smembers k
srem k 11
srem k 11 # 删除不存在的元素
smembers k
srem k 8 9 10
smembers k
```
![image.png](D:\test\image-37.png)

```bash
sadd key 1 2 3 4 5 6 7 8
smembers key

srandmember key # 获取随机数
```
![image.png](D:\test\image-38.png)

```bash
smembers key
smembers k

# 若元素重复则不会返回错误，只不过最终不会重复
# 若key中不存在该元素，则最终返回0
smove key k 5 6 7 8 # error
smove key k 5 
smove key k 6
smove key k 7
smove key k 8

smembers key
smembers k
```
![image.png](D:\test\image-39.png)


```bash
# inter 交集；union 并集；
# diff 差集：在 k1 中存在 在 k2 中不存在
smembers key
smembers k
sinter key k
sinterstore kk key k
smembers kk

smembers k1
smembers k2
sunion k1 k2
sunionstore key k1 k2
smemberss key

smembers k1
smembers k2
sdiff k1 k2 # 在 k1 中存在，而在 k2 中不存在
sdiff k2 k1 # 在 k2 中存在，而在 k1 中不存在
sdiffstore kk k1 k2
sdiffstore kk k2 k1
```
![image.png](D:\test\image-40.png)


<a name="r6YhW"></a>
### 内部编码
```bash
OBJECR encoding key # intset

sadd kkk hello world
smembers kkk
OBJECT encoding kkk # hashtable
```
![image.png](D:\test\image-41.png)


<a name="URzkP"></a>
## Zset类型
`**Zset**`** 有序集合，即：按照升序 或 降序排列**`**Set**`**。**但是我们应该怎么给`Set`排序呢？此处的`Redis`引入了`**score**`**这个浮点类型的属性**，我们可以根据这个`score`排序。我们可以**把**`**score**`**与**`**member**`**的关系当作一对**`**Pair**`，注意并不是键值对，因为它们可以根据一方找到另一方，而键值对只能通过“键 找到 值”。

<a name="DBcaa"></a>
### 基本指令
```bash
# NX 不存在则添加；XX 存在则更新分数
# LT 小于 ； GT 大于 才成功
# CH -- changed 描述返回值（修改和添加的元素个数）
# INCR 根据现有分数进行运算

zadd k1 99 吕布 98 赵云 97 典韦 96 关羽 # 默认升序
zrange k1 0 -1
zrange k1 0 -1 withscores # 把分数也展示出来

zadd k1 97 赵云 # 修改已有元素，默认返回0
zadd k1 10 吕布
zrange k1 0 -1 withscores

zadd k1 95 张飞 # 添加不存在的元素
zrange k1 0 -1 withscores
zadd k1 NX 92 张飞 # 不存在则创建
zrange k1 0 -1 withscores
zadd k1 XX 92 张飞 # 存在则修改
zrange k1 0 -1

zadd k1 XX 90 马超 # 存在则修改，此时不存在
zrange k1 0 -1 withscores

zadd k1 CH 90 张飞 # 返回 修改+添加 的个数
zrange k1 0 -1 withscores

zadd k1 incr 8 张飞 # 返回分数
zrange k1 0 -1 withscores
```
![image.png](D:\test\image-42.png)


```bash
zrange k1 0 -1 withscores
zcard k1

# 这里的 min 与 max 可以写成浮点数
# -inf 负无穷大 ； inf 无穷大
zcount k1 96 97
zcount k1 (96 97 # (96, 97]
zcount k1 (96 97) # error
zcount k1 96 97) # error
zcount k1 96 (97 # [96, 97)
zcount k1 (96 (97 # (96, 97)
```
![image.png](D:\test\image-43.png)


```bash
zrange k1 0 -1 withscores # 正序：从小到大
zrevrange k1 0 -1 withscores # 逆序：从大到小

zrange k1 0 -1 withscores
zrangebyscore k1 94 97
zrangebyscore k1 94 97 withscores
```
![image.png](D:\test\image-44.png)


```bash
zrange k1 0 -1 withscores
zpopmax k1 2 # 删除2个，时间复杂度是O(logN * M) N-元素个数；M-要删除的个数
zrange k1 0 -1 withscores
```
![image.png](D:\test\image-45.png)


```bash
# A窗口
bzpopmax key 600 # 时间复杂度 O(logN)
# B窗口
zadd key 10 张三 20 李四 30 王五、
# A窗口立刻弹出结果
zrange key 0 -1
```
![image.png](D:\test\image-46.png)


```bash
zadd key 10 张三 20 李四 30 王五 40 赵六 50 钱七
zrange key 0 -1
zpopmin key 2
zrange key 0 -1

bzpopmin k1 600
```
![image.png](D:\test\image-47.png)


```bash
zrange key 0 -1
zrank key 王五 # 返回排名（默认 数字越小，分数越小）
zrevrank key 王五 # 返回排名 （数字越大，分数越小）
```
![image.png](D:\test\image-48.png)


```bash
zrange key 0 -1 withscores
zscore key 王五 # 时间复杂度是 O(1)
```
![image.png](D:\test\image-49.png)


```bash
zadd key 10 张三 20 李四 30 王五 40 赵六 50 钱七
zrange key 0 -1

zrem key 王五
zrange key 0 -1

zrem key 李四 赵六
zrange key 0 -1


# -------------------------------------------------
zrange key 0 -1
zremrangebyrank key 1 -1 # 根据下标删除
zrange key 0 -1


# -------------------------------------------------
zrange key 0 -1
zremrangebyscore key 20 70 # 根据分数删除
zrange key 0 -1
```
![image.png](D:\test\image-50.png)


```bash
zrange k1 0 -1 withscores
zincrby key 50 张三
zrange k1 0 -1 withscores
zincrby key -50 张三
zrange k1 0 -1 withscores
```
![image.png](D:\test\image-51.png)


```bash
zadd k1 10 zhangsan 20 lisi 30 wangwu
zadd k2 15 zhangsan 25 lisi 35 wangwu
zinterstorce k3 2 k1 k2
zrange k3 0 -1 withscores # 默认使用 SUM 模式

# 添加权重，权重也可以使用小数
zinterstore k4 2 k1 k2 weights 2 3 # 加上权重 k1的 score * 2；k2的 score * 3
zrange k4 0 -1 withscores

zinterstore k5 2 k1 k2 aggregate max
zrange k5 0 -1 withscores
zinterstore k5 2 k1 k2 aggregate min
zrange k5 0 -1 withscores
```
![image.png](D:\test\image-52.png)


```bash
zadd k1 10 zhangsan 20 lisi 30 wangwu
zadd k2 15 zhangsan 25 lisi 35 wangwu
zunionstore k3 2 k1 k2
zrange k3 0 -1 withscores # 默认使用 SUM 模式

zunionstore k4 2 k1 k2 weights 0.5 0.5
zrange k4 0 -1 withscores

zunionstore k5 2 k1 k2 aggregate max
zrange k5 0 -1 withscores
zunionstore k5 2 k1 k2 aggregate min
zrange k5 0 -1 withscores
```
![image.png](D:\test\image-53.png)



<a name="odJs6"></a>
### 内部编码
```bash
OBJECT encoding key
zadd key 40 aaaaaaaaaaa...aaaaaaaaaaaa
OBJECT encoding key
```
![image.png](D:\test\image-54.png)


<a name="VReeJ"></a>
## 其他类型
<a name="AhWI0"></a>
### Steam类型


<a name="Ix86Y"></a>
### geospatial类型


<a name="BQDU4"></a>
### hyperloglog类型


<a name="DTcJY"></a>
### bigmap类型


<a name="HUCot"></a>
### bigfield类型



<a name="AeAbb"></a>
# 项目相关
<a name="l2JmI"></a>
# ![image.png](D:\test\image-55.png)
![image.png](D:\test\image-56.png)

![image.png](D:\test\image-57.png)

<a name="yV7r7"></a>
## 短信登录 与 Session共享
```java
public Result sendCode(String phone, HttpSession session) {
    // 1.校验手机号
    	// 2.如果不符合，返回错误信息

    // 3.符合，生成验证码
    String code = RandomUtil.randomNumbers(6);

    // 4.保存验证码到 session
    stringRedisTemplate.opsForValue().set(LOGIN_CODE_KEY + phone, code, LOGIN_CODE_TTL, TimeUnit.MINUTES);

    // 5.发送验证码 -- 用日志模拟验证码发送
    log.debug("发送短信验证码成功，验证码：{}", code);
}


public Result login(LoginFormDTO loginForm, HttpSession session) {
    // 1.校验手机号
		// 2.如果不符合，返回错误信息

    // 3.从redis获取验证码并校验
    String cacheCode = stringRedisTemplate.opsForValue().get(LOGIN_CODE_KEY + phone);
    String code = loginForm.getCode();
    if (cacheCode == null || !cacheCode.equals(code)) {
        // 不一致，报错
        return Result.fail("验证码错误");
    }

    // 4.一致，根据手机号查询用户 select * from tb_user where phone = ?
    
    // 5.判断用户是否存在
        // 6.不存在，创建新用户并保存


    
    // token登录 -- session共享
    // 7.保存用户信息到 redis中
    // 7.1.随机生成token，作为登录令牌
    String token = UUID.randomUUID().toString(true);
    // 7.2.将User对象转为HashMap存储
    UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
    Map<String, Object> userMap = BeanUtil.beanToMap(userDTO, new HashMap<>(),
            CopyOptions.create()
                    .setIgnoreNullValue(true)
                    .setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString()));
    // 7.3.存储
    String tokenKey = LOGIN_USER_KEY + token;
    stringRedisTemplate.opsForHash().putAll(tokenKey, userMap);
    // 7.4.设置token有效期
    stringRedisTemplate.expire(tokenKey, LOGIN_USER_TTL, TimeUnit.MINUTES);

    // 8.返回token
    return Result.ok(token);
}
```


<a name="bIHhS"></a>
## 签到功能
```java
// 签到
public Result sign() {
    // 1.获取当前登录用户
    // 2.获取日期
    LocalDateTime now = LocalDateTime.now();
    // 3.拼接key
    String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 4.获取今天是本月的第几天
    int dayOfMonth = now.getDayOfMonth();
    
    // 5.写入Redis SETBIT 命令用于设置或更新字符串类型的指定偏移量上的位值
    // SETBIT mykey 2 1   # 将 mykey 字符串的第一个字节的第三个位设置为 1
    stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);
    return Result.ok();
}


// 签到计数
public Result signCount() {
    // 1.获取当前登录用户
    // 2.获取日期
    LocalDateTime now = LocalDateTime.now();
    // 3.拼接key
    String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 4.获取今天是本月的第几天
    int dayOfMonth = now.getDayOfMonth();
    // 5.获取本月截止今天为止的所有的签到记录，返回的是一个十进制的数字 BITFIELD sign:5:202203 GET u14 0
    // todo 此处的 bigfield 是什么？
    List<Long> result = stringRedisTemplate.opsForValue().bitField(
            key,
            BitFieldSubCommands.create()
                    .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth)).valueAt(0)
    );
    if (result == null || result.isEmpty()) {
        // 没有任何签到结果
        return Result.ok(0);
    }
    // todo 此处的0是什么？
    Long num = result.get(0);
    if (num == null || num == 0) {
        return Result.ok(0);
    }
    // 6.循环遍历
    int count = 0;
    while (true) {
        // 6.1.判断这个bit位是否为0
        if ((num & 1) == 0) {
            // 如果为0，说明未签到，结束
            break;
        } else {
            // 如果不为0，说明已签到，计数器+1
            count++;
        }
        // 丢弃右边的n位，用0填充左边
        num >>>= 1;
    }
    return Result.ok(count);
}
```
[Redis Bitmap 学习和使用](https://zhuanlan.zhihu.com/p/401726844)



<a name="Zw7lv"></a>
## 点赞功能
```java
// 是否点赞
private void isBlogLiked(Blog blog) {
    // 1.获取登录用户
    UserDTO user = UserHolder.getUser();
    if (user == null) {
        // 用户未登录，无需查询是否点赞
        return;
    }
    Long userId = user.getId();

    // 2.判断当前登录用户是否已经点赞
    String key = "blog:liked:" + blog.getId();
    // ZSCORE 命令用于获取有序集合（sorted set）中指定成员的分值
    Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());
    blog.setIsLike(score != null);
}


// 点赞 与 取消点赞
public Result likeBlog(Long id) {
    // 1.获取登录用户
    Long userId = UserHolder.getUser().getId();
    // 2.判断当前登录用户是否已经点赞
    String key = BLOG_LIKED_KEY + id;
    Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());
    if (score == null) {
        // 3.如果未点赞，可以点赞
        // 3.1.数据库点赞数 + 1
        boolean isSuccess = update().setSql("liked = liked + 1").eq("id", id).update();
        // 3.2.保存用户到Redis的set集合  zadd key value score
        if (isSuccess) {
            stringRedisTemplate.opsForZSet().add(key, userId.toString(), System.currentTimeMillis());
        }
    } else {
        // 4.如果已点赞，取消点赞
        // 4.1.数据库点赞数 -1
        boolean isSuccess = update().setSql("liked = liked - 1").eq("id", id).update();
        // 4.2.把用户从Redis的set集合移除
        if (isSuccess) {
            stringRedisTemplate.opsForZSet().remove(key, userId.toString());
        }
    }
    return Result.ok();
}


// 点赞排行榜
public Result queryBlogLikes(Long id) {
    String key = BLOG_LIKED_KEY + id;
    // 1.查询top5的点赞用户 zrange key 0 4
    Set<String> top5 = stringRedisTemplate.opsForZSet().range(key, 0, 4);
    if (top5 == null || top5.isEmpty()) {
        return Result.ok(Collections.emptyList());
    }
    // 2.解析出其中的用户id
    List<Long> ids = top5.stream().map(Long::valueOf).collect(Collectors.toList());
    String idStr = StrUtil.join(",", ids); // 拼接用户ID 为 IN( ) 中的内容
    // 3.根据用户id查询用户 WHERE id IN ( 5 , 1 ) ORDER BY FIELD(id, 5, 1)
    List<UserDTO> userDTOS = userService.query()
            .in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list()
            .stream()
            .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
            .collect(Collectors.toList());
    // 4.返回
    return Result.ok(userDTOS);
}
```


<a name="mM3Bw"></a>
## 关注功能
```java
// 是否关注
public Result isFollow(Long followUserId) {
    // 1.获取登录用户
    Long userId = UserHolder.getUser().getId();
    // 2.查询是否关注 select count(*) from tb_follow where user_id = ? and follow_user_id = ?
    Integer count = query().eq("user_id", userId).eq("follow_user_id", followUserId).count();
    // 3.判断
    return Result.ok(count > 0);
}


// 关注 与 取消关注
public Result follow(Long followUserId, Boolean isFollow) {
    // 1.获取登录用户
    Long userId = UserHolder.getUser().getId();
    String key = "follows:" + userId;
    // 1.判断到底是关注还是取关
    if (isFollow) {
        // 2.关注，新增数据
        Follow follow = new Follow();
        follow.setUserId(userId);
        follow.setFollowUserId(followUserId);
        boolean isSuccess = save(follow);
        if (isSuccess) {
            // 把关注用户的id，放入redis的set集合 sadd userId followerUserId
            stringRedisTemplate.opsForSet().add(key, followUserId.toString());
        }
    } else {
        // 3.取关，删除 delete from tb_follow where user_id = ? and follow_user_id = ?
        boolean isSuccess = remove(new QueryWrapper<Follow>()
                .eq("user_id", userId).eq("follow_user_id", followUserId));
        if (isSuccess) {
            // 把关注用户的id从Redis集合中移除
            stringRedisTemplate.opsForSet().remove(key, followUserId.toString());
        }
    }
    return Result.ok();
}


// 共同关注
public Result followCommons(Long id) {
    // 1.获取当前用户
    Long userId = UserHolder.getUser().getId();
    String key1 = "follows:" + userId;
    // 2.求交集
    String key2 = "follows:" + id;
    Set<String> intersect = stringRedisTemplate.opsForSet().intersect(key1, key2);
    if (intersect == null || intersect.isEmpty()) {
        // 无交集
        return Result.ok(Collections.emptyList());
    }
    // 3.解析id集合
    List<Long> ids = intersect.stream().map(Long::valueOf).collect(Collectors.toList());
    // 4.查询用户
    List<UserDTO> users = userService.listByIds(ids)
            .stream()
            .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
            .collect(Collectors.toList());
    return Result.ok(users);
}
```



<a name="LJbUL"></a>
## 缓存优化 及 GEO应用
GEO（Geographic Information System，地理信息系统）是一种用于存储、管理、分析和展示地理数据的计算机系统。它将空间数据与属性数据相结合，可以在地图上显示各种地理信息、分析地理现象，支持决策制定和规划设计等应用。**GEO 数据主要包括两部分：空间数据 和 属性数据**。其中，空间数据包括点、线、面等地理要素，通常以矢量形式存储；属性数据则包含这些地理要素的各种属性信息，例如名称、坐标、面积、人口等等。

在计算机科学中，GEO 还可以指代一些相关的技术或工具，例如：

1.  **GEO 编码**：一种将地址或地理位置编码成字符串的技术，可以用于快速查询和识别地点信息。 
2.  **GEO 数据库**：一种专门用于存储和查询地理数据的数据库系统，可以支持空间查询和分析等功能。 
3.  **GEO 定位**：一种利用 GPS、Wi-Fi、蓝牙等技术进行定位的方法，可以实现基于地理位置的服务，例如导航、打车、附近优惠等。 

总之，GEO 在地理信息领域中非常重要，它为我们提供了一种便捷的方式来处理和利用地理数据，可以广泛应用于城市规划、资源管理、气象预测、区域分析等领域。
```java
// 使用缓存优化修改
@Transactional // 数据库的修改 和 缓存的修改必须是一个事务内的！
public Result update(Shop shop) {
    Long id = shop.getId();
    if (id == null) {
        return Result.fail("店铺id不能为空");
    }
    // 1.更新数据库
    updateById(shop);
    // 2.删除缓存
    stringRedisTemplate.delete(CACHE_SHOP_KEY + id);
    return Result.ok();
}


// GEO实现附件店铺功能
public Result queryShopByType(Integer typeId, Integer current, Double x, Double y) {
    // 1.判断是否需要根据坐标查询
    if (x == null || y == null) {
        // 不需要坐标查询，按数据库查询
        Page<Shop> page = query()
                .eq("type_id", typeId)
                .page(new Page<>(current, SystemConstants.DEFAULT_PAGE_SIZE));
        // 返回数据
        return Result.ok(page.getRecords());
    }

    // 2.计算分页参数
    int from = (current - 1) * SystemConstants.DEFAULT_PAGE_SIZE;
    int size = current * SystemConstants.DEFAULT_PAGE_SIZE;

    // 3.查询redis、按照距离排序、分页。结果：shopId、distance
    String key = SHOP_GEO_KEY + typeId;
    // GEOSEARCH key BYLONLAT x y BYRADIUS 10 WITHDISTANCE
    // BYLONLAT 经纬度定位中心点：x - 经度；y - 纬度
    // BYRADIUS 搜索半径
    // WITHDISTANCE 返回结果的同时会显示【每个匹配项与中心点的距离】
    GeoResults<RedisGeoCommands.GeoLocation<String>> results = stringRedisTemplate.opsForGeo()
            .search(
                    key,
                    GeoReference.fromCoordinate(x, y),
                    new Distance(5000), // 单位是米，此处为5KM
                    // newGeoSearchArgs()：创建一个新的地理位置搜索参数对象
                    // includeDistance()：返回结果的同时会【显示每个匹配项与中心点的距离】
                    // limit(end)：指定【返回结果的数量上限】为 end
                    RedisGeoCommands.GeoSearchCommandArgs.newGeoSearchArgs().includeDistance().limit(size)
            );

    // 4.解析出id
    if (results == null) {
        return Result.ok(Collections.emptyList());
    }

    List<GeoResult<RedisGeoCommands.GeoLocation<String>>> list = results.getContent();
    if (list.size() <= from) {
        // 没有下一页了，结束
        return Result.ok(Collections.emptyList());
    }

    // 4.1.截取 from ~ end的部分
    List<Long> ids = new ArrayList<>(list.size());
    Map<String, Distance> distanceMap = new HashMap<>(list.size());
    list.stream().skip(from).forEach(result -> {
        // 4.2.获取店铺id
        String shopIdStr = result.getContent().getName();
        ids.add(Long.valueOf(shopIdStr));
        // 4.3.获取距离
        Distance distance = result.getDistance();
        distanceMap.put(shopIdStr, distance);
    });

    // 5.根据id查询Shop
    String idStr = StrUtil.join(",", ids);
    List<Shop> shops = query().in("id", ids).last("ORDER BY FIELD(id," + idStr + ")").list();
    for (Shop shop : shops) {
        shop.setDistance(distanceMap.get(shop.getId().toString()).getValue());
    }

    // 6.返回
    return Result.ok(shops);
}
```


<a name="XJSWx"></a>
## UV 统计
UV（Unique Visitor）是网站或应用程序的独立访客数量，通常用来衡量网站或应用程序的流量和用户数量。在网站或应用程序中，每个访问者只被计算为一个 UV。因此，**UV 统计是指记录和统计一个网站或应用程序中独立访客的数量。**

UV 统计可以帮助网站或应用程序的管理员了解其受众的规模和特征，**包括访客的地理位置、访问时间、浏览器信息等**。通过对 UV 的分析，管理员可以更好地了解访客的兴趣和行为习惯，从而优化网站或应用程序的内容和功能，提高用户满意度和留存率。

为了实现 UV 统计，通常会使用各种技术手段，例如 Cookie、IP 地址、用户代理等。其中，Cookie 是最常用的一种技术，通过在用户浏览器中设置 Cookie，可以跟踪用户的访问行为并计算 UV。但是，由于 Cookie 可能被禁用或删除，因此为了更准确地统计 UV，通常还需要结合其他技术进行分析。
