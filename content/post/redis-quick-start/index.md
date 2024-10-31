---
title: "Redis 快速入门"
description: "Redis 对于 Java 程序员的快速上手使用。"
tags: ["Redis","Quick Start"]
categories: ["Redis"]
date: 2024-09-25T18:11:03+08:00
image: "redis-quick-start-cover.jpg"
hidden: false
draft: true
---

文档参考：[Redis 官方文档](https://redis.io/docs/latest/develop/get-started/)

## SpringBoot 项目使用 Redis

### 依赖引入

在 Springboot 项目中，添加如下依赖：

```java
<!-- Redis 依赖 -->
<dependency>
    groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

 <!-- Json 序列化依赖 -->
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.13.0</version>
</dependency>
```

### 配置 Redis

1. 新建一个 RedisConfig 类用于 Redis 的配置

~~~java
/**
 * Redis 配置类
 *
 * @author limincai
 */
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connection) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connection);

        // 设置键序列化器为 StringRedisSerializer，所有的键会被序列化为字符串
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        // 设置值序列化器为 GenericJackson2JsonRedisSerializer，所有的键会被序列化为 Json
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
~~~

2. 配置 application.yaml

~~~java
spring:
  redis:
    host: localhost # Redis服务器的主机名，默认 localhost
    port: 6379 # Redis服务器的端口号，默认 6379
    password: # Redis服务器的密码，如果没有密码则留空，默认为空
    timeout: 3000 # 连接超时时间（单位：毫秒）
  lettuce:
    pool:
      max-active: 8 # 最大连接数
      max-wait: -1 # 最大等待时间，-1表示不限制
      max-idle: 8 # 最大空闲连接数
      min-idle: 1 # 最小空闲连接数
~~~

## 基本数据结构

### String

String 是 Redis 中最简单常用的数据类型，可以存储任何类型的数据比如字符串、整形、浮点、序列化后的对象等。

#### 应用场景

- 缓存：可以作为缓存数据库使用，提高系统性能，减少对数据库访问的压力。
- 会话：可以存储 seesion、token，等会话数据，可以用在分布式系统中。
- 计数器：Redis 的 incr/decr 命令可以用于实现计数器功能。可以将计数器存储为 String 类型，每次更新时通过 incr/decr 命令进行自增或自减操作。
- 分布式锁：通过 SETNX 命令可以实现一个具有过期时间的分布式锁。

#### 基本操作

~~~cmd
# 设置指定 key 值。
SET key value

# 获取指定 key 值。
GET key

# 判断 key 是否存在。
EXSITS key

# 删除 key
DEL key

# 批量设置 key 值
MSET key1 value1 [key2 value2 ...]

# 批量删除 key 值
MGET key1 [ key2 ...]

# 将 key 中存储的数值加1。
INCR key

# 将 key 中存储的数值减1。
DECR key

# 给 key 设置过期时间。
EXPIRE key seconds

# 设置 key 值并设置过期时间，如果 key 存在不做任何操作并返回0，否则返回1。
SETNX key value [EX seconds | PX milliseconds]

# 查看剩余过期时间。
TTL key
~~~

#### Java 代码测试

~~~java
@Test
    void testString() {
        // 设置指定 key 值
        redisTemplate.opsForValue().set("string1", "value1");
        // 获取指定 key 值
        System.out.println(redisTemplate.opsForValue().get("string1"));
        // 判断 key 是否存在
        System.out.println(redisTemplate.opsForValue().getOperations().hasKey("string1"));
        // 得到值后删除 key
        System.out.println(redisTemplate.opsForValue().getAndDelete("string1"));
        // 批量设置 key 值
        HashMap<String, Object> map = new HashMap<>();
        map.put("string1", "value1");
        map.put("string2", "value2");
        redisTemplate.opsForValue().multiSet(map);
        // 批量删除 key 值
        HashSet<String> set = new HashSet<>();
        set.add("string1");
        set.add("string2");
        redisTemplate.opsForValue().getOperations().delete(set);
        // 将 key 中存储的数值加1并返回修改后的值
        redisTemplate.opsForValue().set("num1", 1);
        System.out.println(redisTemplate.opsForValue().increment("num1"));
        // 将 key 中存储的数值减1并返回修改后的值
        System.out.println(redisTemplate.opsForValue().decrement("value1"));
        // 给 key 设置过期时间
        redisTemplate.opsForValue().set("string1", "value1");
        redisTemplate.opsForValue().getAndExpire("string1", 1000, TimeUnit.SECONDS);
        // 设置 key 值并设置过期时间，如果 key 存在不做任何操作并返回 false，否则返回 true
        System.out.println(redisTemplate.opsForValue().setIfAbsent("string1", "value1"));
        System.out.println(redisTemplate.opsForValue().setIfAbsent("string2", "value2", 1000, TimeUnit.SECONDS));
        // 查看剩余过期时间
        System.out.println(redisTemplate.opsForValue().getOperations().getExpire("string2"));
    }
~~~

### List

List 是简单的字符串列表，按照插入顺序排序。

#### 应用场景

- 最消息排行榜：可以使用 Redis 列表来存储最新的消息。每次有新的消息到达时，将其插入到列表的头，当列表的长度超过一定限制时，可以使用`LTRIM`命令进行修剪，以保持列表的长度。
- 实时消息记录：可以使用 Redis 列表来保存实时产生的消息记录。每当有新的消息产生时，将其插入到列表头部，以便可以追溯最近的一定数量的消息记录。
- 排行榜/计分系统：可以使用 Redis 列表来实现排行榜或计分系统。每个元素可以包含一个分数，通过对元按照分数进行排序，可以获取到排行榜中的前几名或指定范围内的元素。

#### 基本操作

~~~cmd
# 将一个或多个值插入到列表的左侧，返回插入后列表的长度。
LPUSH key value [value ...]

# 将一个或多个值插入到列表的右侧，返回插入后列表的长度。
RPUSH key value [value ...]

#移除返回列表的左侧第一个元素。
LPOP key

# 移除并返回列表的右侧第一个元素。
RPOP key

# 返回列表的长度。
LLEN key

# 获取列表中指定范围内的元素。
LRANGE key start stop

# 返回列表中指定索引位置的元素。
LINDEX key index

# 设置列表中指定索引位置的元素的值。
LSET key index value

# 移除列表中的指定元素。
LREM key count value

# 修剪列表，只保留指定范围内的元素。
LTRIM key start stop
~~~

#### Java 代码测试

~~~java
@Test
    void testList() {
        // 将一个或多个值插入到列表的左侧，返回插入后列表的长度。
        redisTemplate.opsForList().leftPushAll("mylist", "value1", "value2", "value3");
        // 将一个或多个值插入到列表的右侧，返回插入后列表的长度。
        redisTemplate.opsForList().rightPushAll("mylist", "value4", "value5", "value6");
        // 移除返回列表的左侧第一个元素。
        System.out.println(redisTemplate.opsForList().leftPop("mylist"));
        // 移除并返回列表的右侧第一个元素。
        System.out.println(redisTemplate.opsForList().rightPop("mylist"));
        // 返回列表的长度。
        System.out.println(redisTemplate.opsForList().size("mylist"));
        // 获取列表中指定范围内的元素。
        System.out.println(redisTemplate.opsForList().range("mylist", 2, 3));
        // 返回列表中指定索引位置的元素。
        System.out.println(redisTemplate.opsForList().index("mylist", 3));
        // 设置列表中指定索引位置的元素的值。
        redisTemplate.opsForList().set("mylist", 2, "value3");
        // 移除列表中的指定元素。
        redisTemplate.opsForList().remove("mylist", 1, "value1");
        // 修剪列表，只保留指定范围内的元素。
        redisTemplate.opsForList().trim("mylist", 1, 3);
    }
~~~

### Hash

Hash 是一个 String 类型的键值对。

#### 应用场景

- 缓存对象：可以将对象的字段和属性存储在Redis的Hash中，以便快速地读取和更新。例如，用户信息存储在一个Hash中，每个用户的字段可以是用户ID、用户名、年龄等。这样，在需要读取或用户信息时，可以直接通过用户ID来获取或更新相应的字段。
- 计数器：可以使用Hash来实现计数器功能。通过将计数器的名称作为Hash的键名，将计数值作为键值存储在Hash中。然后可以使用Redis提供的原子操作对计数器进行增减操作，如HINCRBY命令。
- 实时排行榜：可以使用Hash来实现实时排行榜功能。将用户ID作为Hash的键名，将用户的分数作为键值存储在Hash中。通过更新用户的分数来实现排行榜实时更新，并使用Redis提供的ZSET数据结构进行排名的计算。

#### 基本操作

~~~cmd
# 设置 Hash 中指定字段的值。
HSET key field1 value1 [field2 value2...]

# 获取 Hash 中指定字段的值
HGET key field

# 设置 Hash 中多个字段的值。
HMSET key field1 value1 [field2 value2...]

# 获取 Hash 中多个字段的值。
HMGET key field1 [field2...]

# 获取 Hash 中所有字段和值。
HGETALL key

# 删除 Hash 中的指定字段。
HDEL key field

# 判断 Hash 中是否存在指定字段。
HEXISTS key field

# 获取 Hash 中所有值
HVALS key
~~~

#### Java 代码测试

~~~java
@Test
    void testHash() {
        // 设置 Hash 中指定字段的值。
        redisTemplate.opsForHash().put("myhash", "field1", "value1");
        // 获取 Hash 中指定字段的值
        System.out.println(redisTemplate.opsForHash().get("myhash", "field1"));
        // 设置 Hash 中多个字段的值。
        HashMap<String, Object> map = new HashMap<>();
        map.put("field2", "value2");
        map.put("field3", "value3");
        redisTemplate.opsForHash().putAll("myhash", map);
        // 获取 Hash 中多个字段的值。
        HashSet<Object> set = new HashSet<>();
        set.add("field1");
        set.add("field2");
        System.out.println(redisTemplate.opsForHash().multiGet("myhash", set));
        // 获取 Hash 中所有字段和值。
        System.out.println(redisTemplate.opsForHash().entries("myhash"));
        // 删除 Hash 中的指定字段。
        redisTemplate.opsForHash().delete("myhash", "field1");
        // 判断 Hash 中是否存在指定字段。
        System.out.println(redisTemplate.opsForHash().hasKey("myhash","field2"));
        // 获取 Hash 中所有值
        System.out.println(redisTemplate.opsForHash().values("myhash"));
    }
~~~

### Set

Set 是一种无序集合，集合中的元素唯一，类似于 Java 中的 HashSet

#### 应用场景

- 抽奖和排行榜：Set可以用于实现抽奖功能和排行榜功能。例如，抽奖活动的参与者可以存储在Set，每次抽奖时从Set中随机选择一个用户；另外，将用户的得分、点击量或其他指标存储在Set中，可以据权重排序来生成排行榜。
- 好友关系：Set可以用于存储用户的好友关系。例如，每个用户对应一个Set，保存了该用户的好友列表，可以方便地进行好友操作，如查找共同的好友、计算好友数等。
- 集成员查找：Redis的Set数据结构可以高效地进行成员的查找操作。例如，可以用于实现黑白名单的判断，快速判断某个元素是否在集合中。
- 点赞、收藏等功能：可以使用Set来存储用户对某个实体（如文章、评论、商品等）的点赞、收藏关系。个实体的点赞、收藏用户就是一个Set。通过Set提供的添加、删除、计数等操作，可以方便地管理点赞、收藏关系。

#### 基本操作

~~~cmd
# 向 Set 中添加一个或多个元素。
SADD key value1 [value2...]

# 从 Set 删除一个或多个元素。
SREM key value1 [value2...]

# 判断一个元素是否存在于 Set 中。
SISMEMBER key value

# 获取 Set 中的所有元素。
SMEMBERS myset

# 获取集合的成员数
SCARD key

# 移除并返回 Set 中的一个随机元素
SPOP key

# 返回所有 Set 的交集
SINTER key1 [key2...]

# 返回所有 Set 的并集
SUNION key1 [key2...]

# 返回所有 Set 的差集
SDIFF key1 [key2...]
~~~

#### Java 代码测试

~~~java
@Test
    void testSet() {
        //  向 Set 中添加一个或多个元素。
        redisTemplate.opsForSet().add("myset", "value1", "value2", "value3", "value4");
        redisTemplate.opsForSet().add("myset1", "value1", "value2", "value6", "value5");
        // 从 Set 删除一个或多个元素。
        redisTemplate.opsForSet().remove("myset", "value3", "value4");
        // 判断一个元素是否存在于 Set 中。
        System.out.println(redisTemplate.opsForSet().isMember("myset", "value1"));
        // 获取 Set 中的所有元素。
        System.out.println(redisTemplate.opsForSet().members("myset"));
        // 获取集合的成员数
        System.out.println(redisTemplate.opsForSet().size("myset"));
        // 移除并返回 Set 中的一个随机元素
        System.out.println(redisTemplate.opsForSet().pop("myset"));
        // 返回所有 Set 的交集
        System.out.println(redisTemplate.opsForSet().intersect("myset", "myset1"));
        // 返回所有 Set 的并集
        System.out.println(redisTemplate.opsForSet().union("myset", "myset1"));
        // 返回所有 Set 的差集
        System.out.println(redisTemplate.opsForSet().difference("myset", "myset1"));
    }
~~~

### Sorted Set

Sorted Set 类似于 Set，但和 Set 相比，维护了一个 double 类型的分数，使得集合中的元素能够按分数排列。

#### 应用场景

- 排行榜：可以使用Sorted Set来存储用户的得分或其他评分指标，并按照分数进行排序，从而实现排行榜功能。通过Sorted Set提供的操作，如添加成员、更新分数、根据分数范围获取成员等，可以方便地进行排行榜的维护和查询。
- 积分系统：可以使用Sorted Set来存储用户的积分，并按照积分进行排序。通过Sorted Set提供的操作，如添加成员、更新分数、根据分数范围获取成员等，可以方便地查询用户的排名、前几名用户等功能。
- 热门文章：可以使用Sorted Set来存储文章以及其阅读量或点赞数等指标，并按照指标进行排序。通过Sorted Set提供的操作，如添加成员、更新分数、根据分数范围获取成员等，可以方便地查询热门文章、热门标签等功能。

#### 基本操作

~~~cmd
# 向 Sorted Set 添加一个或多个原素，或者更新已存在元素的分数
ZADD key score1 value1 [score2 value2...]

# 获取 Sorted Set 中元素的个数
ZCARD key

# 移除 Sorted Set 中一个或多个原素
ZREM key value1 [value2...]

# 获取  Sorted Set 中指定元素的分数
ZSCORE key value

# 给指定元素添加分数
ZINCRBY key increment value

# 根据分数范围获取 Sorted Set 中的元素
ZRANGEBYSCORE key score1 score2

# 通过索引返回 Sorted Set 中指定区间的成员（分数从低到高）
ZRANGE key start stop

# 通过索引返回 Sorted Set 中指定区间的成员（分数从高到低）
ZREVRANGE key start stop

# 获取指定元素排名，返回 Sorted Set 中的索引
ZRANK key value

# 获取多个 Sorted Set 中的交集并存储在新的 Sorted Set 中
ZINTERSTORE destination numkeys key1 [key2...]

# 获取多个 Sorted Set 中的并集并存储在新的 Sorted Set 中
ZUNIONSTORE destination numkeys key1 [key2...]

# 获取多个 Sorted Set 中的差集并存储在新的 Sorted Set 中
ZDIFF destination numkeys key1 [key2...]
~~~

#### Java 代码测试

~~~java
    @Test
    void testZSet() {
        // 向 Sorted Set 添加一个或多个原素，或者更新已存在元素的分数
        redisTemplate.opsForZSet().add("myzset", "value1", 3.14);
        redisTemplate.opsForZSet().add("myzset", "value2", 3.141);
        redisTemplate.opsForZSet().add("myzset", "value3", 3.1415);
        // 获取 Sorted Set 中元素的个数
        System.out.println(redisTemplate.opsForZSet().size("myzset"));
        // 移除 Sorted Set 中一个或多个原素
        redisTemplate.opsForZSet().remove("myzset", "value3");
        // 获取  Sorted Set 中指定元素的分数
        System.out.println(redisTemplate.opsForZSet().score("myzset", "value1"));
        // 给指定元素添加分数
        redisTemplate.opsForZSet().incrementScore("myzset", "value2", 1.1);
        // 根据分数范围获取 Sorted Set 中的元素
        System.out.println(redisTemplate.opsForZSet().rangeByScore("myzset", 1, 6));
        // 通过索引返回 Sorted Set 中指定区间的成员（分数从低到高）
        System.out.println(redisTemplate.opsForZSet().range("myzset", 1, 2));
        // 通过索引返回 Sorted Set 中指定区间的成员（分数从高到低）
        System.out.println(redisTemplate.opsForZSet().reverseRange("myzset", 1, 2));
        // 获取指定元素排名，返回 Sorted Set 中的索引
        System.out.println(redisTemplate.opsForZSet().rank("myzset", "value1"));
        // 获取多个 Sorted Set 中的交集并存储在新的 Sorted Set 中
        redisTemplate.opsForZSet().add("myzset1", "value1", 3.14);
        redisTemplate.opsForZSet().intersectAndStore("newzset1", "myzset1", "myzset");
        // 获取多个 Sorted Set 中的并集并存储在新的 Sorted Set 中
        redisTemplate.opsForZSet().unionAndStore("newzset1", "myzset1", "myzset");
        // 获取多个 Sorted Set 中的差集并存储在新的 Sorted Set 中
        redisTemplate.opsForZSet().differenceAndStore("newzset1", Collections.singleton("myzset1"), "myzset");
    }
~~~

## 高级数据结构

### Geospatial 

#### 应用场景

- 附近搜索：地理位置服务应用（如打车软件、外卖平台）可以利用 Redis 的地理空间索引来快速找到用户附近的司机或餐馆。
- 路线规划与距离计算：对于需要进行路径规划或者距离计算的应用程序，如物流配送系统，可以利用`GEODIST`命令来获取两个地理位置之间的距离。
- 基于位置的广告推送：零售业可以利用地理信息向顾客推送附近店铺的促销信息。例如，在商场区域内发送优惠券给进入该区域的顾客。

#### 基本操作

~~~cmd
# 添加一个或多个元素对应的经纬度信息到 GEO 中
GEOADD key longitude1 latitude1 member1 [longitude2 latitude2 member2...]

# 返回给定元素的经纬度信息
GEOPOS key member1 [member2...]

# 计算两个位置之间的距离
GEODIST key member1 member2 [m|km|ft|mi]

# 返回一个多多个位置对象的 geohash 值
GEOHASH key member1 [member2...]

# 根据给定的经纬度坐标来获取指定范围内的地理位置集合
GEORADIUS key longitude latitude radius [m|km|ft|mi]

# 根据存储在位置集合里面的某个地点获取指定范围内的地理位置集合
GEORADIUSBYMEMBER key member radius [m|km|ft|mi]
~~~

#### Java 代码测试

~~~java
@Test
    void testGeo() {
        // 添加一个或多个元素对应的经纬度信息到 GEO 中
        redisTemplate.opsForGeo().add("cities", new Point(-73.935242, 40.73061), "New York");
        redisTemplate.opsForGeo().add("cities", new Point(-0.127758, 51.507351), "London");
        redisTemplate.opsForGeo().add("cities", new Point(116.407526, 39.90403), "Beijing");
        // 返回给定元素的经纬度信息
        List<Point> position = redisTemplate.opsForGeo().position("cities", "New York", "London");
        System.out.println(position);
        // 计算两个位置之间的距离
        Distance distance = redisTemplate.opsForGeo().distance("cities", "New York", "London", RedisGeoCommands.DistanceUnit.KILOMETERS);
        System.out.println(distance);
        System.out.println(distance.getValue());
        // 返回一个多多个位置对象的 geohash 值
        List<String> hash = redisTemplate.opsForGeo().hash("cities", "New York", "London");
        System.out.println(hash);
        // 根据给定的经纬度坐标来获取指定范围内的地理位置集合
        Circle circle = new Circle(-73.935242, 40.73061, Metrics.KILOMETERS.getMultiplier());
        RedisGeoCommands.GeoRadiusCommandArgs args = RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs()
                .includeDistance() //包含距离
                .includeCoordinates() //包含坐标
                .sortAscending() //升序
                .limit(50);
        GeoResults<RedisGeoCommands.GeoLocation<Object>> byxy = redisTemplate.opsForGeo().radius("cities", circle, args);
        byxy.forEach(r -> System.out.println(r));
    }
~~~

