# 目录
[TOC]
# Redis学习

## 文档
### SpringData Redis 官方文档
- http://docs.spring.io/spring-data/redis/docs/1.7.5.RELEASE/reference/html/
- https://redis.io/
### Redis文档
- 菜鸟教程 http://www.runoob.com/redis/redis-java.html
- 易百教程 http://www.yiibai.com/redis/redis_strings.html
- Redisdoc http://redisdoc.com/

## 基础知识
### redis-beachmark 性能测试
- `` 命令：redis-benchmark``
- 1.7秒10w个set请求
### 其他
- redis单进程
- port 6379 = merz
- redis默认配置了16个database,相当于分片
```
1. redis.conf 配置:database 16 。下标从0~15
2. 切换database命令:select 7
```
- 命令
```
- dbsize 查看key的数量
- keys * 列出所有的key
- flushall 清空所有库 
- flushdb 清空当前库
```
- Redis索引都是从0开始

## 数据类型
### Key 关键字
- 校验关键字是否存在 ``exists k1``
- 移到库2 ``move k3 2``
- 给key设置过期时间，秒钟 `expire key`
- 查看还有多少秒过期，-1永不过期，-2表示已过期 `ttl k1`
- 查看key是什么类型 `type k1`
- 删除key `DEL k1`

### 五大数据类型
#### 1. Binary-safe strings
##### String
- 可以包含任何数据，比如jpg或者序列化对象
- 一个redis中字符串value最多可以是512M
##### 命令
- `set/get/del/append/strlen`
```
- set k1 v1
- get k1 -> v1
- strlen k1 -> 2
- append k1 123 ->v1123
```
- `Incr/decr/incrby/decrby`只有数字才能加减
```
- set k2 2
- incr k2 ->3
- decr k2 ->2
- incrby k2 3 ->5
- decrby k2 3->2
```
- `getrange/setrange`
```
- set k3 123456
- getrange k3 0 -1 ->123456
- getrange k3 0 3 ->1234
- setrange k3 0 xxx ->xxx456
```
- `setex(set with expire)`键秒值/`setnx(set if not exist)`
```
- setex k4 10 v4 ->10s有效
- setnx k4 v11 ->v4 生效1，不生效0
- setnx k5 v11 ->v11
```
- `mset/mget/msetnx` merge，一次设置多个值
```
- mset k1 v1 k2 v2 k3 v3 ->v1,v2,v3
- mget k1 k2 k3 ->v1,v2,v3
- msetnx k3 v3 k4 v4 ->0  只要一个存在，全部都失败
```
- `getset` 先get再set

#### 2. Lists
##### List
- 底层是双向链表
##### 命令
- `lpush/rpush/lrange`
```
- LPUSH list01 1 2 3 4 5 -> 54321
- RPUSH list02 1 2 3 4 5 -> 12345
- LRANG list02 0 -1 ->12345
```
- `lpop/rpop` 出栈
```
- lpop list01 -> 5
- lpop list02 -> 1
- rpop list01 -> 1
- rpop list02 -> 5
```
- `lindex`按照索引下标获取元素
```
- LINDEX list01 2 ->3
- LINDEX list01 5 ->nil
```
- `llen`
```
- llen list01 ->5
```
- `lrem key` 删N个value
```
- RPUSH list03 11122233344455567
- LREM list03 2 3 ->111222344455567删除2个3

```
- `ltrim key index index`截取指定范围的值后在赋值给key
```
- lpush list04 12345678 ->87654321
- ltrim list04 0 4 ->87654
```
- `rpoplpush 源列表 目的列表`
```
- rpoplpush list01 list02 ->list01:5432 list02:112345

```
- `lset key index value`指定索引设置值
```
- lset list01 2 x ->12x45
```
- `linsert key before/after val1 val2`
```
- linsert list01 before x java ->12javax45
```
##### 性能总结
- 字符串双向链表，双向操作
- 键不存在创建新的链表
- 键存在新增内容
- 如果值全部移除对应的键也就消失了
- 链表的操作无论是头还是尾效率都很高，从中间操作效率低

#### 3. Sets
##### Set
- 不能重复
- 无序集合
##### 
- `sadd/smembers/sismember`
```
- sadd set01 1 1 2 2 3 3 ->123
- smembers set01 ->123 列出所有成员
- sismember set01 1 ->1 是否存在，1：存在，0：不存在
```
- `scard` 获取集合里面的元素个数
```
- scard set01
```
- `srem` 删除集合中元素
```
- srem set01 1 ->23
```
- `srandmember key`某个整数（随机出现几个数）
```
- sadd set02 01 1 2 3 4 5 6 7
- srandmember set02 3 ->随机出现3个set02的成员
```
- `spop key` 随机出栈
```
- sadd set01 1 2 3 4 5 6 7 8
- spop set01 ->返回出栈的值
```
- `smove key1 key2 在key1里某个值` 将key1里的某个值，赋给key2
```
- smove set01 set02 4 ->4被移到set02
```
- 数学集合 `sdiff/sinter/sunion` 差集/交集/并集
```
- sadd set03 1 2 3 4 5
- sadd set04 1 2 3 a b
- sdiff set03 set04 -> 45
- sinter set03 set04 ->1234
- sunion set03 set04 ->12345ab
```

#### 4. Sorted sets
##### Zset
- 有序集合。
- 每个元素关联一个double类型的分数score。分数可以重复。
##### 命令
- `zadd/zrange`
```
- zadd zset01 60 v1 70 v2 80 v3 90 v4 100 v5
- zrange zset01 0 -1 ->v1v2v3v4v5
- zrange zset01 0 -1 withscores ->v1 60 v2 70 .....
```
- `zrangebyscore 开始score 结束score` 
```
- zrangebyscore 60 90 ->大于等于60小于等于90
- zrangebyscore 60 (90 ->大于等于60小于90
- zrangebyscore (60 (90 ->大于60小于90
- zrangebyscore 60 90 limit 1 2 ->从下标1开始截取2个
```
- `zrem`
```
- zrem zset01 v5
```
- `zcard/zcount key score的区间/zrank key values值/zscore key 对应值，获得分数` 
```
- zadd zset01 100 v5
- zcard zset01 -> 5
- zcount zset01 60 80 ->3
- zrank zset01 v4 ->3 获取值对应的下标
- zscore zset01 v4 ->90
```
- `zrevrank key values `逆序获得下标值
```
zrevrank zset01 v4 ->1
```
- `zrevrange` 逆序列出指定返回的值
```
- zrevrange zset01 0 -1  ->v5v4v3v2v1
- zrange zset01 0 -1 ->v1v2v3v4v5
```
- `zrevrangebyscore key`
```
- zrevrangbyscore zset01 60 90 ->v4v3v2v1
```

#### 5. Hashes
##### Hash
- String类型的field和value的映射表
##### 命令
- `hset/hget/hmset/hmget/hgetall/hdel`
```
- hset user id 11
- hget user id ->11
- hset user name z3
- hget user name ->z3
- hmset customer id 11 name 1i4 age 26
- hmget customer id name age
- hgetall customer ->id 11 name li4 age 26
- hdel user name ->删除user下的name键值对
```
- `hlen`
```
- hlen user ->1
- hlen customer ->3 
```
- `hexists key key1`
```
- hexists customer email ->0
```
- `hkeys/hvals`
```
- hkeys customer ->id name age 
- hvals customer -> 11 li4 26
```
- `hincrby/hincrbyfloat`
```
- hincrby customer age 2 ->28
- hset customer score 91.5
- hincrbyfloat customer score 0.5 ->92
```
- hsetnx
```
- hsetnx customer age 25 ->0 失败，1成功
```

### 配置文件


