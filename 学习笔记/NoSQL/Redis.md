## 1.工具类

```java

import com.snbc.smcp.stock.constant.Constants;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

import java.util.List;
import java.util.Map;

/**
 * @desc: redis使用工具类
 * @author: liuzicheng
 * @date: 2019/12/23 19:40
 */
@Component
public class JedisUtil {

    @Autowired
    private JedisPool jedisPool;

    /**
     * 获取指定key的值,如果key不存在返回null
     * @param key
     * @return
     */
    public String get(String key) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.get(key);
        }
    }

    /**
     * 设置key的值为value
     * @param key
     * @param value
     * @return
     */
    public String set(String key, String value) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.set(key, value);
        }
    }

    /**
     * 删除指定的key,也可以传入一个包含key的数组
     * @param keys
     * @return
     */
    public Long del(String... keys) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.del(keys);
        }
    }

    /**
     * 判断key是否存在
     * @param key
     * @return
     */
    public Boolean exists(String key) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.exists(key);
        }
    }

    /**
     * 通过key同时设置 hash的多个field
     * @param key
     * @param hash
     * @return
     */
    public String hmset(String key, Map<String, String> hash) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.hmset(key, hash);
        }
    }

    /**
     * 通过key 和 field 获取指定的 value
     * @param key
     * @param failed
     * @return
     */
    public String hget(String key, String failed) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.hget(key, failed);
        }
    }

    /**
     * 设置key的超时时间为seconds
     * @param key
     * @param seconds
     * @return
     */
    public Long expire(String key, int seconds) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.expire(key, seconds);
        }
    }

    /**
     * 通过key和field判断是否有指定的value存在
     * @param key
     * @param field
     * @return
     */
    public Boolean hexists(String key, String field) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.hexists(key, field);
        }
    }

    /**
     * 通过key 删除指定的 field
     * @param key
     * @param fields 可以是 一个 field 也可以是 一个数组
     * @return
     */
    public Long hdel(String key, String... fields) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.hdel(key, fields);
        }
    }

    /**
     * 通过key获取所有的field和value
     * @param key
     * @return
     */
    public Map<String, String> hgetall(String key) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.hgetAll(key);
        }
    }

    /**
     * 通过key 和 fields 获取指定的value 如果没有对应的value则返回null
     * @param key
     * @param fields 可以是 一个String 也可以是 String数组
     * @return
     */
    public List<String> hmget(String key, String... fields) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.hmget(key, fields);
        }
    }

    /**
     * 通过key向指定的set中添加value
     * @param key
     * @param members 可以是一个String 也可以是一个String数组
     * @return 添加成功的个数
     */
    public Long sadd(String key, String... members) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.select(Constants.DB_INDEX);
            return jedis.sadd(key, members);
        }
    }

    public JedisPool getJedisPool(){
        return this.jedisPool;
    }

}

```

## 2.配置类

```java
package com.snbc.smcp.stock.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Scope;
import org.springframework.util.StringUtils;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

/**
 * redis配置
 * @desc: redis配置
 * @author: liuzicheng
 * @date: 2019/12/31 16:07
 * @return:
 */
@Configuration
public class Config {

    @Value("${platform.redis.host}")
    private String redisHost;
    @Value("${platform.redis.port}")
    private int redisPort;
    @Value("${platform.redis.database}")
    private int database;
    @Value("${platform.redis.pool.maxIdle}")
    private int maxIdle;
    @Value("${platform.redis.pool.minIdle}")
    private int minIdle;
    @Value("${platform.redis.pool.maxActive}")
    private int maxActive;
    @Value("${platform.redis.pool.maxWait}")
    private int maxWait;
    @Value("${platform.redis.password}")
    private String password;
    @Value("${platform.redis.timeout}")
    private int timeout;

    /**
     * redis连接
     * @desc: redis连接
     * @author: liuzicheng
     * @date: 2019/12/31 16:08
     * @return: redis.clients.jedis.JedisPool
     */
    @Bean
    @Scope
    @Primary
    public JedisPool jedisPool() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(maxActive);
        jedisPoolConfig.setMaxIdle(maxIdle);
        jedisPoolConfig.setMaxWaitMillis(maxWait);
        jedisPoolConfig.setMinIdle(minIdle);
        if (StringUtils.isEmpty(password)) {
            return new JedisPool(jedisPoolConfig, redisHost, redisPort);
        } else {
            return new JedisPool(jedisPoolConfig, redisHost, redisPort, timeout, password, database);
        }
    }

}
```

## 3.配置文件	

```properties
platform.redis.database=0
platform.redis.host=127.0.0.1
platform.redis.port=6379
platform.redis.password=
platform.redis.pool.maxActive=200
platform.redis.pool.maxWait=-1
platform.redis.pool.maxIdle=10
platform.redis.pool.minIdle=0
platform.redis.timeout=1000
```

## 4.数据类型

```properties
字符串	     -  String    ---适合存储key-value,可以存一些数量啥的
字典    	  -  Hash	   ---适合存储对象
列表    	  -  List
集合	      -  Set
有序集合     -  SortedSet
```

