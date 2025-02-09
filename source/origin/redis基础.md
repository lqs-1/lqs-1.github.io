---
title: redis基础
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-19 15:32:03
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: redis基础
tags: redis
categories: db
---
## Redis
### NoSQL一种新出现的数据库(not only sql)
```text
1、不认识sql语句
2、存储结构和关系型数据库中的关系表完全不同，nosql存储的数据都是kv（key-value）形式
3、nosql的世界没有一种通用的语言，每一种nosql数据库都有自己的api和语法，以及应用场景
4、nosql的产品种类多

### Redis数据库
1、支持数据的持久化
2、支持简单的key-vlaue，还有list,set,zset,hash等类型
3、支持备份，master-slave模式的数据备份
```
### Redis数据类型
```text
1、字符串string
2、哈希hash
3、列表list
4、集合set
5、有序集合zset
```


### Redis操作行为
```
1、保存
2、修改
3、获取
4、删除
```

#### string类型

> redis中最基础的数据类型，是二进制安全的，可以接受任何格式的数据，照片、json等

-	定义字符串:set key value,  setex key sconds value 这种方式设置过期时间，  mset key value key value设置多个
-	查看字符串:get key，mget key1 key2,获取多个
-	字符串的追加:append key value
-	查看所有的键:keys 通配符
-	查看键是否存在:exists key
-	查看键值的类型:type key
-	设置过期时间:expire key sconds
-	查看键的有效时间:ttl key
-	删除:del key1 key2……

#### hash类型
> hash本来就有key，hash的value就是一个字符串string，所以还有一个string的key
-	定义哈希类型:hset key string-key value，  hmset key string-key1 value1 string-key2 value2一次设置多个，一个hash中可以有多个value
-	查看指定键的值：hkey key
-	获取hash的值:hget key string-key,   hmget key string-key1 string-key2……多个值，  hvals key获取所有的值
-	删除:del key，删除整个hash，hdel key string-key删除对应的hash值

#### 列表类型（list）
-	定义：lpush key value vlaue2……从左侧插入，rpush key value value2…… 从右侧插入，linsert key before或者after 某一个已经存在的value value value2……
-	获取:lrange key start stop
-	修改:lset key index new_value
-	删除:lrem key count value,count是一个数字，表示删除多少个（相同元素），整数表示前往后，负数从后往前，为0的时候表示删除所有给定的value


#### 集合类型(set)
> 不能重复，无序
-	定义:sadd key value value2……
-	获取:smenbers key获取所有元素
-	删除:srem key value value2……
	

#### 有序集合(zset)
> 有序权重从小到大排序
-	定义:zadd key 权重 value 权重2 value2……,权重就是数字
-	获取:zrange key start stop
-	获取给定的权值范围元素:zrangebyscore key start stop
-	获取权值:zscore key value
-	删除指定的元素:zrem key value1 value2……
-	删除给定权值范围元素:zremrangebyscore key start stop


#### Redis和python的交互:

##### python基本使用
```python
from redis import *
xx = StrictRedis(host = 'xxx', port = 6379, db = num)
```
> 对xx进行操作

##### django中使用Redis:多数用来 存储session
```python
pip install django-redis-sessions

# setting.py中设置：
SESSION_ENGINE = 'redis_sessions.session'
SESSION_REDIS_HOST = 'localhost'
SESSION_REDIS_PORT = 6379
SESSION_REDIS_DB = 2
SESSION_REDIS_PASSWORD = ''
SESSION_REDIS_PREFIX = 'session'
# 这样就可以将原本存储在django的数据库中的session存储到redis中，存储成hash

# 配置成caches:
CACHES = {
    "default": {
    "BACKEND": "django_redis.cache.RedisCache",
    "LOCATION": "redis://127.0.0.1:6379/2",
    "OPTIONS": {
        "CLIENT_CLASS": "django_redis.client.DefaultClient",
    }
    }
}

# 使用
from dajango_redis import get_redis_connection
xxx = get_redis_connection('default')
# 可以使用redis中的所有方法
```
	
	
	
### redis的问题
>	缓存穿透：用户查询一个数据库中一定不会有的数据，比如id=-1,因为数据库中没有这个数据，所以不会保存到缓存中，下次再来访问的时候，缓存中没有，又会去数据库中查询，这样就增加了数据库的压力，如果被黑客发现了，就会利用这个漏洞来攻击数据库，可能会让数据库崩溃，解决办法就是就算没有查到，也放到缓存中，只不过值为null，然后把过期时间设置的短一点。
	
>	缓存雪崩：意思就是大量的缓存数据同时失效（过期），这个时候，又有大并发的请求的话，就会对数据库加大访问压力，可能让数据库崩溃，解决办法就是，可以设置不同的过期时间，如果是双十一这种可以设置永不过期。
	
>	缓存击穿：意思就是用户频繁会访问的key，比如，双十一有一个活动见做一分钱领取iphone13 pro max，在非常多的人访问的时候，在这种大并发的情况下，如果缓存中的这个key过期了，那么对数据的压力就太大了，解决办法就是设置这个key用不过期。
