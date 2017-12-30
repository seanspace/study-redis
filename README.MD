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
- `Incr/decr/incrby/decrby`只有数字才能加减
- `getrange/setrange`
- `setex(set with expire)`键秒值/`setnx(set if not exist)`
- `mset/mget/msetnx`
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
- `len`
```
- LLEN list01 ->5
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
- Set
- 无序集合
#### 4. Sorted sets
- Zset
- 有序集合。
- 每个元素关联一个double类型的分数。分数可以重复。
#### 5. Hashes
- Hash
- String类型的field和value的映射表





