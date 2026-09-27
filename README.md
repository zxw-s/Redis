# 一、Redis 是什么

**Redis（Remote Dictionary Server）**：开源**内存型键值（ Key-Value ）数据结构服务器**，属于 NoSQL，C语言开发。

> 单线程模型（网络IO），value支持多种数据结构，读写性能极高，单机QPS可达十万级，不仅是缓存，还可做数据库、消息队列、分布式锁、排行榜等。

> 默认端口：**6379**

✅ 核心特性

1. 高性能：**内存存储**，纯内存操作，无磁盘IO瓶颈，速度快；
2. 持久化：支持持久化（RDB快照/AOF日志），防止断电内存数据丢失
3. 高可用：主从复制、哨兵、集群部署
4. 多数据结构：`String、Hash、List、Set、ZSet`等原生结构
5. 单线程模型（命令执行单线程，网络IO多线程），避免锁竞争
6. 扩展功能：支持过期时间、发布订阅、位图、地理位置、限流、Lua脚本
7. 原子操作：所有命令天然原子，支持事务、Lua脚本
8. 过期时间、发布订阅等

✅ 主流应用场景

1. 缓存（最常用）：商品详情、用户会话、接口结果缓存，减轻数据库压力

2. 计数器：阅读量、点赞数、秒杀库存计数

3. 分布式锁：秒杀、并发资源控制、限流

4. 排行榜（ZSet）：ZSet有序集合实现榜单排序

5. 消息队列（List）：List/Stream 做轻量消息中间件

6. 签到统计：Bitmap 位存储，极小内存实现用户签到记录

7.会话Session存储、验证码

# 二、安装（推荐Docker，跨平台，推荐）

## Docker（Windows / Mac / Linux 通用）

```bash
# 拉取redis镜像
docker pull redis

# 启动容器，后台运行，端口6379映射
docker run -d --name redis -p 6379:6379 redis:7

# 进入redis-cli客户端
docker exec -it redis redis-cli
```

## Linux Ubuntu原生安装

```bash
# 添加官方源
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt update && sudo apt install redis-serversudo apt update
sudo apt install redis-server -y
# 启动
sudo systemctl start redis
# 开机自启
sudo systemctl enable redis
# 连接客户端
redis-cli
```

## Mac Homebrew

```bash
brew install redis
brew services start redis
redis-cli
```

## Windows（仅测试）

下载基于官方 Redis 源码编译的 Windows 版本Redis压缩包

[https://github.com/redis-windows/redis-windows/releases]()

解压后：

1. 双击  redis-server.exe  启动服务

2. 双击  redis-cli.exe  打开命令行，输入  ping  返回  PONG  即连接成功

3.安装为 Windows 服务

> Windows官方没有原生版，学习可用社区版或WSL，**不建议生产环境使用**

### 验证是否成功

双击  redis-cli.exe  打开命令行

```redis
127.0.0.1:6379> PING
PONG
```

返回`PONG`说明服务正常。

> 默认端口：6379，进入客户端： `redis-cli` ；带密码连接： `redis-cli -a 密码`

### 使用方式：RedisService.exe（推荐）

自动处理路径转换，支持 Windows 原生路径。

```cmd
# 前台运行
RedisService.exe run --foreground --port 6379 --dir C:\redis

# 安装为 Windows 服务
RedisService.exe install -c C:\config\redis.conf --dir D:\data\redis --port 6379
net start Redis

# 卸载服务
RedisService.exe uninstall方式一：RedisService.exe（推荐）
```

## 图形化工具

[Redis Insight - Free GUI & CLI Tool for Redis](https://redis.io/insight/)

# 三、通用Key命令（所有数据结构通用）

> 官网命令文档：https://redis.io/commands

```redis
# 测试连接
ping

# 获取key的值
GET key

# 设置key，不存在新增，存在覆盖
SET key value

# 设置键过期时间（单位秒）
expire key seconds

# 查看剩余过期时间，-1永不过期，-2key不存在
ttl key

# 移除过期时间，永久保存
PERSIST key

# 删除key
del key

# 判断key是否存在，返回1存在，0不存在
EXISTS key

# 查看所有key【生产禁止使用！阻塞redis】
KEYS *

# 渐进式遍历key（生产推荐）
SCAN 0

# 重命名key
RENAME oldkey newkey

# 获取key的数据类型
TYPE key
```

# 四、Redis for Windows

[](https://github.com/redis-windows/redis-windows/actions)
[](https://github.com/redis-windows/redis-windows/releases)

基于官方 Redis 源码编译的 Windows 版本。

## 快速开始

```cmd
# 下载解压后，直接运行
redis-server.exe redis.conf

# 或使用 RedisService（推荐）
RedisService.exe run --foreground
```

## 使用方式

### 方式一：RedisService.exe（推荐）

自动处理路径转换，支持 Windows 原生路径。

```cmd
# 前台运行
RedisService.exe run --foreground --port 6379 --dir C:\redis-data

# 安装为 Windows 服务
RedisService.exe install -c C:\config\redis.conf --dir D:\data\redis --port 6379
net start Redis

# 卸载服务
RedisService.exe uninstall
```

### 方式二：redis-server.exe（直接运行）

**重要：** 此版本使用 Cygwin 运行时，命令行路径必须使用 Cygwin 格式。

```cmd
# ✅ 正确 - Cygwin 路径格式
redis-server.exe /cygdrive/c/config/redis.conf --dir /cygdrive/d/data --port 6379

# ❌ 错误 - Windows 路径不支持
redis-server.exe C:\config\redis.conf --dir D:\data
```

**路径转换规则：**

| Windows   | Cygwin             |
| --------- | ------------------ |
| `C:\path` | `/cygdrive/c/path` |
| `D:\path` | `/cygdrive/d/path` |
| `.\data`  | `./data`（相对路径直接使用） |

**配置文件中：** 可直接使用 Windows 风格的正斜杠路径。

```conf
# redis.conf 中推荐写法
dir C:/redis/data
logfile C:/redis/logs/redis.log
```

## RedisService 命令参考

```cmd
RedisService.exe [command] [options]

Commands:
  install       安装为 Windows 服务
  uninstall     卸载 Windows 服务
  run           运行 Redis（默认）

Options:
  -c, --config <FILE>      配置文件路径
  --port <PORT>            端口号
  --dir <DIRECTORY>        数据目录
  --loglevel <LEVEL>       日志级别 (debug/verbose/notice/warning)
  -f, --foreground         前台运行
  --service-name <NAME>    服务名称（默认: Redis）
  --start-mode <MODE>      启动类型 (auto/manual)
  -h, --help               显示帮助
  -v, --version            显示版本
```

## 跨分区/跨目录配置

配置文件、数据目录、程序可位于任意位置：

```cmd
# 程序: C:\redis\RedisService.exe
# 配置: D:\config\redis.conf
# 数据: E:\data\redis

RedisService.exe run -c D:\config\redis.conf --dir E:\data\redis --foreground
```

## 数据持久化

关闭 Redis 时数据自动保存。`RedisService.exe` 会正确传递 `--dir` 参数确保数据保存到指定目录。

```cmd
# 启动
RedisService.exe run --foreground --dir C:\redis-data

# 写入数据
redis-cli SET mykey myvalue

# 优雅关闭
redis-cli SHUTDOWN

# 重启后数据仍存在
redis-cli GET mykey   # 返回 "myvalue"
```

## 常见问题

### redis-server.exe 找不到配置文件？

使用 Cygwin 路径格式：

```cmd
redis-server.exe /cygdrive/c/config/redis.conf
```

或使用 `RedisService.exe`，它自动处理路径转换。

### 数据丢失？

1. 确保指定了 `--dir` 参数
2. 使用 `redis-cli SHUTDOWN` 或 `Ctrl+C` 优雅关闭，不要强制杀进程
3. 重启时使用相同的 `--dir` 参数

## 技术细节

- 编译工具：MSYS2 / Cygwin
- 服务包装：.NET 10.0
- 路径转换：RedisService 自动处理 Windows ↔ Cygwin

---

[English](README.md) | 简体中文

## 免责声明

本项目与 Redis Ltd. 无关。提供的许可证仅适用于本仓库内容，不适用于官方 Redis 项目。

建议仅在本地开发环境使用。生产环境请按照 Redis 官方指导在 Linux 上部署。本项目不对使用造成的任何损失承担责任。

# 五、Redis 5大基础数据结构 + 常用命令

> 所有命令不区分大小写，习惯大写；key建议用冒号分隔命名：`user:100:name`

## 1. String 字符串（最简单，最常用）

最大存储512MB，底层字节数组，可以存字符串、数字、JSON。

```redis
# 设置值 key=name value=zhangsan
SET name "zhangsan"
# 获取值
GET name
# 设置值+过期60秒（场景：缓存、验证码、计数器、Token存储）
SETEX code 60 "123456"
# key不存在才设置（分布式锁核心）
SETNX lock "1"
# 数字自增+1（原子计数器）
INCR num
# 批量设置
MSET a 1 b 2
# 批量获取
MGET a b
# 删除key
DEL name
# 判断key是否存在
EXISTS name
# 查看剩余过期时间
TTL code
```

## 2. Hash 哈希（适合存储对象，类似map）

（对象存储 key -> field-value）

适合存储对象，比如用户信息：`user:100`

key-field-value 三层结构，不用整体更新对象，只改单个字段

> 场景：用户信息、商品信息存储、部分字段更新

```redis
# 设置用户字段
HSET user:100 name "lisi" age 20
# 获取单个字段
HGET user:100 name
# 批量查询命令，可减少网络IO开销
HMGET user:100 name age
# 获取所有字段+值
HGETALL user:100
# 原子整数增减命令
HINCRBY user:100 age 1 # age +1
# 哈希删除命令
HDEL user:100 age
```

## 3. List 列表（双向链表）

有序，可重复；按插入顺序排序，两端进出，可做消息队列。左边/右边插入。

> 场景：简单消息队列、最新动态列表、最新消息。

```redis
LPUSH list1 a b c # 左边推入 c b a
RPUSH list1 d # 右边推入 c b a d
LRANGE list1 0 -1 # 遍历列表 查看全部，0开始，-1最后一个
LPOP list1 # 左边弹出
RPOP list1 # 右边弹出
LLEN list1 # 获取长度
```

## 4. Set 集合（无序、元素唯一）

**无序、元素唯一不可重复**，自动去重，支持交集、并集、差集（标签、共同好友）

> 场景：标签、好友去重、点赞去重、共同好友计算

```redis
SADD tags redis java python # 添加元素
SMEMBERS tags # 查看所有元素
SISMEMBER tags redis # 判断是否存在
SCARD tags # 集合元素数量
SINTER set1 set2 # 交集
SUNION set1 set2 # 并集
SDIFF set1 set2 # 差集
```

## 5. ZSet 有序集合（排序+去重 Sorted Set）

**元素不可重复，每个元素带分数score，按score排序**，排行榜首选。

> 场景：热搜榜、游戏排行榜、积分排名

```redis
ZADD rank 90 "zhangsan" 85 "lisi" # 添加：分数 成员
ZRANGE rank 0 -1 WITHSCORES # 正序查看排行榜 从小到大展示+分数
ZREVRANGE rank 0 -1 WITHSCORES # 倒序查看 从大到小（排行榜）
ZSCORE rank zhangsan # 获取分数
ZINCRBY rank 5 zhangsan # 分数+5
ZCARD rank # 元素总数
```

# 六、Redis 持久化（防止重启丢失数据）

Redis数据在内存，宕机数据会丢失，提供两种持久化方案：

1. **RDB（快照）**：定时把内存全量写入磁盘`.rdb`文件。优点：文件小、恢复快；缺点：可能丢失两次快照之间数据。

2. **AOF（Append Only File追加日志）**：记录每一条写命令，重启回放命令恢复数据。优点：增量持久化，数据安全性更高，丢失数据极少；缺点：文件体积大。
   
   > 生产一般**RDB+AOF同时开启**。

# 七、高可用简单介绍

- **主从复制**：1主多从，主写从读，数据同步，读写分离。

- **Sentinel哨兵**：监控主从，主节点宕机自动选新主，故障转移。

- **Cluster集群**：分片存储，数据分散在多个节点，支持水平扩容。

# 八、Java/Spring Boot 快速集成实战

## 1.引入依赖（Spring Boot 2.x）

```xml
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
```

<dependency>

</dependency>

## 2.配置文件 application.yml

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: # 无密码留空
    database: 0
```

## 3.简单使用

```java
@Autowired
private StringRedisTemplate redisTemplate;

// 缓存数据
redisTemplate.opsForValue().set("name", "redis", Duration.ofMinutes(10));

// 获取数据
String name = redisTemplate.opsForValue().get("name");
```

# 九、入门学习计划（3天路线）

## Day1：基础

1.安装redis，学会redis-cli连接

2.掌握5种数据结构，敲一遍上面所有命令

3.理解key过期、DEL/EXISTS/TTL

✅ 习题：

用String存验证码，设置60秒过期

Hash存储一个用户id=101，姓名、年龄、邮箱

ZSet做一个学生分数排行榜，实现加分、从高到低查询

## Day2：原理 + 项目简单调用

1. 持久化RDB/AOF概念

2. 过期策略

3. 淘汰机制

4. 单线程模型，为什么快

5. 用你熟悉语言（Python/Java）写简单客户端连接Redis，读写数据
   ✅ 习题：Python示例
   
   ```python
   # pip install redis
   import redis
   r = redis.Redis(host='127.0.0.1', port=6379, db=0, decode_responses=True)
   r.set("name","test")
   print(r.get("name"))
   ```

## Day3：Spring Boot整合Redis

实现缓存、分布式锁、计数器

## Day4：场景 + 常见坑

1. 缓存雪崩、缓存击穿、缓存穿透解决方案（面试高频）

2. 主从复制、哨兵、集群基础概念

3. 图形工具：RedisInsight / Another Redis Desktop Manager 可视化查看redis

# 十、常见坑（入门必看）

1. Redis内存满了会触发淘汰策略（默认`noeviction`，写报错）

2. 设置合理的key过期时间，避免内存溢出。key过期是惰性删除+定期删除，不是到点立刻删。

3. 命令是单线程，**不要执行慢命令（KEYS）**，生产用SCAN代替KEYS。耗时命令（复杂计算）不要执行，会拖慢整体性能。

4. 分布式锁要考虑过期、原子性问题，不要简单SETNX直接上生产

5. 生产环境禁止使用  keys *  遍历所有key，会阻塞Redis 

6. 大key拆分，不要存储超大字符串/集合，避免阻塞服务

# 十一、练习题

1. String、Hash、List、Set、ZSet各自适合什么业务场景？

2. `KEYS *` 为什么生产环境禁止使用？

3. RDB 和 AOF 的区别？

4. 写一段 Java 客户端连接 Redis 的入门示例代码

## 课后习题参考答案

1. String：简单字符串、数字、验证码、JSON缓存；
   Hash：存储对象（用户、商品），部分字段更新；
   List：消息队列、时间线、最新消息；
   Set：标签、共同好友、去重；
   ZSet：排行榜、延时任务。
2. `KEYS *` 会遍历redis中所有key，是阻塞命令，数据量大时会阻塞整个Redis服务，所以生产禁止。
3. RDB：快照，全量持久化，体积小，恢复快；故障会丢失最近一段时间数据。
   AOF：记录每一条写命令，丢失数据极少；文件体积更大，恢复速度慢。生产一般两者搭配开启。

# 十二、Redis 常用命令速查表（Markdown）

> 说明：key命名推荐 `业务:id:字段`，例如 `user:100:name`；命令大小写不敏感，习惯大写。

## 通用 Key 命令

| 命令                        | 作用                  |
| ------------------------- | ------------------- |
| `SET k v`                 | 设置键值                |
| `GET k`                   | 获取值                 |
| `DEL k1 k2`               | 删除key               |
| `EXISTS k`                | 判断key是否存在，返回1/0     |
| `EXPIRE k 秒`              | 设置过期时间              |
| `TTL k`                   | 查询剩余过期时间，-1永久，-2不存在 |
| `PERSIST key`             | 移除过期时间，永久保存         |
| `KEYS pattern`            | 匹配所有key，**生产禁用**    |
| `SCAN 0 MATCH * COUNT 10` | 迭代遍历key，替代KEYS      |
| `TYPE k`                  | 查看key的数据类型          |
| `RENAME oldkey newkey`    | 重命名key              |

## String 字符串

| 命令                 | 作用               |
| ------------------ | ---------------- |
| `SETEX k 秒 v`      | 设置+过期时间          |
| `SETNX k v`        | key不存在才设置，分布式锁基础 |
| `INCR k`           | 数字自增+1，原子操作      |
| `INCRBY k n`       | 自增指定步长，增加n       |
| `DECR k`           | 数字自减1            |
| `MSET k1 v1 k2 v2` | 批量设置             |
| `MGET k1 k2`       | 批量获取             |
| `APPEND k str`     | 追加字符串            |
| `STRLEN k`         | 获取字符串长度          |

## Hash 哈希（对象）

| 命令              | 作用        |
| --------------- | --------- |
| `HSET h f v`    | 设置hash字段  |
| `HGET h f`      | 获取单个字段    |
| `HMGET h f1 f2` | 获取多个字段    |
| `HGETALL h`     | 获取全部字段和值  |
| `HKEYS h`       | 获取所有字段名   |
| `HVALS h`       | 获取所有value |
| `HINCRBY h f n` | hash内数字自增 |
| `HDEL h f1 f2`  | 删除hash字段  |
| `HEXISTS h f`   | 判断字段是否存在  |
| `HLEN h`        | hash字段总数  |

## List 列表（双向链表）

| 命令                      | 作用                          |
| ----------------------- | --------------------------- |
| `LPUSH list v1 v2`      | 左侧插入                        |
| `RPUSH list v1 v2`      | 右侧插入                        |
| `LPOP list`             | 左侧弹出                        |
| `RPOP list`             | 右侧弹出                        |
| `LRANGE list start end` | 区间查看，0第一个元素，`-1`代表末尾        |
| `LLEN list`             | 获取列表长度                      |
| `LINDEX list idx`       | 索引取值，通过下标取值                 |
| `LSET list idx val`     | 指定下标修改值                     |
| `LREM list count val`   | 删除指定元素                      |
| `BLPOP list 秒`          | 阻塞弹出，没有数据就等待timeout秒，简易消息队列 |

## Set 集合（无序，不重复）

| 命令                | 作用             |
| ----------------- | -------------- |
| `SADD set v1 v2`  | 添加元素           |
| `SMEMBERS set`    | 获取集合全部元素       |
| `SISMEMBER set v` | 判断元素是否存在       |
| `SCARD set`       | 获取集合元素个数       |
| `SREM set v1 v2`  | 删除元素           |
| `SPOP set`        | 随机弹出一个元素       |
| `SINTER s1 s2`    | 交集：两个集合共同元素    |
| `SUNION s1 s2`    | 并集：合并两个集合去重    |
| `SDIFF s1 s2`     | 差集：在s1存在，s2不存在 |

## ZSet 有序集合（score排序，不重复member）

| 命令                                       | 作用                                |
| ---------------------------------------- | --------------------------------- |
| `ZADD z score member`                    | 添加成员与分数                           |
| `ZRANGE z start end [WITHSCORES]`        | 正序查询，从小到大查询，0 -1全部，withscores带出分数 |
| `ZREVRANGE z start end [WITHSCORES]`     | 倒序，从大到小查询（排行榜）                    |
| `ZSCORE z member`                        | 获取成员分数                            |
| `ZINCRBY z num member`                   | 增加成员分数                            |
| `ZCARD z`                                | 元素总数                              |
| `ZREM z m1 m2`                           | 删除成员                              |
| `ZRANK z member`                         | 升序排名                              |
| `ZREVRANK z member`                      | 降序排名                              |
| `ZRANGEBYSCORE z start end [WITHSCORES]` | 按分数范围查询                           |

## 服务相关

| 命令            | 作用                           |
| ------------- | ---------------------------- |
| `PING`        | 测试连通性                        |
| `SELECT dbid` | 切换数据库（默认db0，redis默认16个库0~15） |
| `FLUSHDB`     | 清空当前库【慎用！】                   |
| `FLUSHALL`    | 清空所有库【高危！】                   |
| `INFO`        | 查看redis服务信息（内存、持久化、客户端）      |
| `SLOWLOG GET` | 查看慢查询日志                      |
| `CLIENT LIST` | 查看连接客户端                      |

## 发布订阅 Pub/Sub

```redis
# 订阅频道
SUBSCRIBE news

# 向频道发消息
PUBLISH news "hello redis"
```

## Stream（redis5+ 专业消息队列）

```redis
# 添加消息，*自动生成id
XADD mystream * name tom age 20

# 读取消息
XREAD COUNT 2 STREAMS mystream 0
```

⚠️ 重要避坑提示

1.  KEYS * 、 HGETALL  大数据量会阻塞Redis，生产慎用

2. 所有命令是单线程执行，避免执行耗时指令

3. key命名规范： 业务:类型:id  例： user:info:100 

4. 合理设置过期时间，防止内存持续增长触发淘汰策略

## 补充：Spring RedisTemplate 对应方法简记

- opsForValue → String

- opsForHash → Hash

- opsForList → List

- opsForSet → Set

- opsForZSet → ZSet

---

# 十三、Redis 脚本&配置项目

> Redis 仓库用于存放 Redis 相关脚本、配置、数据初始化脚本。
> Redis 无表结构，仓库只维护 `.conf` 配置文件、`.lua` 脚本、redis-cli 批量执行脚本。
> **不提交持久化数据文件、dump.rdb、appendonly.aof，不存放真实业务密钥。**

## 📖 项目简介

本仓库管理 Redis 相关资源：Redis 服务配置、Lua 原子脚本、批量初始化命令、示例数据脚本。
脚本可通过 `redis-cli` 执行，适用于单机/主从/哨兵/集群环境。

## 🧰 环境依赖

- Redis：6.2+ / 7.x
- 客户端：redis-cli、RedisInsight、Another Redis Desktop Manager

## 📂 目录结构

```
.
├── conf/ -- redis 配置文件（redis.conf 模板）
├── lua/ -- Lua 原子脚本
├── scripts/ -- redis-cli 批量执行命令脚本
├── migrations/ -- 版本变更脚本（新增key、修改过期策略、Lua脚本更新）
├── .gitignore
└── README.md
```

## 🚀 脚本执行顺序

1. 部署 Redis 实例，加载对应环境配置文件
2. 按版本号升序执行 `migrations` 目录下所有 `V*.redis` 增量脚本（一次性执行）
3. 全部V脚本执行完成后，执行 `migrations` 目录下 `R*.redis` 可重复脚本（加载Lua脚本、重置基础字典）
4. 【测试环境可选】执行 `scripts/init_test_data.redis` 导入测试数据，生产环境禁止执行

> 规则：
> 
> - V开头脚本：提交后不可修改，出错新建更高版本脚本修复
> - R开头脚本：修改后会重新执行，适合加载Lua脚本、基础公共数据

## 📌 迁移脚本命名规范（沿用Flyway风格，保持和SQL项目统一）

### 命名格式

V<主版本>.<次版本>.<修订号>__<描述文本>.redis

- `V`：Version，一次性执行的版本迁移脚本，只会执行一次
- 版本号：语义化版本 `x.y.z`
- `__`：两个下划线，分隔版本号与描述
- 描述：英文小写，单词下划线连接，禁止中文、空格

示例：

V1.0.0__init_base_keys.redis
V1.0.1__add_user_token_key.redis
V1.0.2__update_session_ttl.redis
V1.1.0__load_lua_script.redis

### R可重复脚本

R__<描述>.redis

每次修改脚本，都会重新执行。适合：加载Lua脚本、公共基础字典。
示例：`R__reload_common_lua_script.redis`

### 脚本头部注释模板

```redis
/*
 * Version: V1.0.0
 * Date: 2026-09-26
 * Description: 初始化基础业务Key，设置默认过期时间
 */
```

## 📌 开发规范

1. 仓库只维护配置模板、lua脚本、redis命令脚本；**禁止提交 dump.rdb、appendonly.aof 持久化文件**
2. 配置文件内禁止写真实密码、生产环境IP
3. 配置区分模板文件，真实密码单独在服务器环境变量配置，不要提交到仓库
4. 迁移脚本放入 `migrations` 目录，变更新增脚本，不修改已经提交的历史脚本
5. Lua脚本尽量放在 `lua/` 目录，migrations脚本只负责加载 `SCRIPT LOAD`

## ❗ 重要提醒

- 上线执行脚本前，务必在测试Redis实例完整验证
- 生产环境执行前，做好数据备份（`BGSAVE`）
- Redis没有事务DDL，批量操作需要评估性能，大key操作禁止在业务高峰执行
- 生产环境不要直接执行 `FLUSHDB` / `FLUSHALL`

## 📜 License

MIT

## ✅ R脚本示例（Redis，migrations/R__reload_common_lua_script.redis）

```
/*
 * Repeatable Script: R__reload_common_lua_script.redis
 * Description: 加载公共Lua脚本
 */
SCRIPT LOAD "return redis.call('GET',KEYS[1])"
```
