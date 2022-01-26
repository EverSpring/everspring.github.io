---
title: Redis使用规范
date: 2022-01-07 13:25:00
author: EverSpring
top: false
toc: false
mathjax: false
categories: 数据库
tags:
  - Redis
  - 规范
---
根据日常在使用redis时形成的，欢迎大家补充
# Redis使用规范
#### 一、key、value规范
1. 【强制】必须以业务名(或数据库名)为前缀，用冒号:分隔，比如 业务名:表名:id
2. 【强制】key设计必须简洁
	* 保证语义的前提下，控制key的长度，当key较多时，内存占用也不容忽视。
	* 不要包含特殊字符：空格、换行、单双引号以及其他转义字符
3. 【强制】string类型控制在10KB以内
说明：防止网卡流量、慢查询
4. 【推荐】hash、list、set、zset元素个数不要超过1万。如果数量过大，可以分成多个key
5. 【推荐】不要使用del删除非字符串的bigkey, 使用hscan、sscan、zscan方式渐进式删除
   说明：因为时间复杂度是 O(N), hscan、sscan、zscan方式渐进式删除
	* 如果是Hash类型的大Key，推荐使用hscan + hdel
	* 如果是List类型的大Key，推荐使用ltrim
	* 如果是Set类型的大Key，推荐使用sscan + srem
	* 如果是SortedSet类型的大Key，推荐使用zscan + zrem
6. 【推荐】尽量设置过期时间
说明：条件允许可以打散过期时间，防止集中过期，不过期的数据重点关注idletime。遇到大bigkey 过期，会触发del，造成主线程阻塞

#### 二、命令规范
1. 【强制】禁用keys
说明：Keys 命令效率极低，属于 O(N)操作，会阻塞其他正常命令，在 cluster 上，会是灾难性的操作
2. 【强制】禁用flushall、flushdb
说明：命令会清空所有数据，属于高危操作
3. 【强制】严禁对 zset 的不设范围操作
4. 【强制】ZRANGE、 ZRANGEBYSCORE等多个操作 ZSET 的函数，严禁使用 ZRANGE myzset 0 -1 等这种不设置范围的操作。如要使用请指定范围，如 ZRANGE myzset 0 100。如不确定长度，可使用 ZCARD 判断长度
5. 【强制】严禁对大数据量 Key 使用 HGETALL
6. 【推荐】谨慎使用 sunion, sinter, sdiff等一些聚合操作，如果要用，需要提前报备

#### 三、其他使用要求
1. 【推荐】必要情况下可以使用monitor命令，但在生产上要注意不要长时间使用
2. 【推荐】redis实例默认支持16个数据库，可以把不同的业务分到不同的库。但尽量不是切换库(即select库)，如果需要跨库的通过key名来处理
3. 【推荐】 如果有批量操作，优先使用mget、mset或pipeline，提高效率，但要注意控制一次批量操作的元素个数
4. 【强制】不要在lua脚本中使用比较耗时的代码，比如长时间的sleep、大循环等， lua脚本的执行超时时间为5秒钟
5. 【推荐】只用来保存热数据，不准当作落地数据库使用，即redis中的数据需要在其他数据库中查询到

#### 四、集群使用
Redis集群版本在使用Lua上有特殊要求
1. 【强制】所有key都应该由 KEYS 数组来传递，redis.call/pcall 里面调用的redis命令，key的位置，必须是KEYS array, 否则直接返回error，"-ERR bad lua script for redis cluster, all the keys that the script uses should be passed using the KEYS array"
2. 【强制】所有key，必须在1个slot上，否则直接返回error, "-ERR eval/evalsha command keys must in same slot"
说明：redis在使用hash算法将键映射到slot时，只会计算{}里面的内容，若{}内的内容相同，则将键映射到同一个slot

#### 五、代码使用
1. 【推荐】value为string时，序列化尽量少采用JDK自带的序列化方式（JdkSerializationRedisSerializer），因为序列化速度慢
2. 【强制】目前已经提供puzzle-cache公共组件，开发过程中尽量使用此组件，统一序列化方式
3. 【强制】同一工程中，由统一的类保管key，不准在业务类中私自定义key
