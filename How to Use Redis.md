# How to Use Redis

## What is Redis?

TL;DR. Redis is an in-memory key-value pairs database. Because it stores data in RAM but not ROM, users can get data from Redis much much quicker than from in-disk databases such as MySQL. Therefore, Redis is used to store cache in many projects.

## How to install Redis?

If you use linux, windows linux sub system, or macOS, it is quite easy. Just use your package manager to install redis. For example, you can use homebrew to install redis if you use macOS.

```
brew install redis
```

Then you can also install redis client by using this command:

```
brew install redis-cli
```

After installing these two softwares, you can start or stop the service in background by using this command:

```
brew services start/stop redis
```

Then you can use redis-cli to handle your redis database. The default port is `6379`.

## How to use Redis? (Basic Commands)

### 1. get and set

```
set key value

get key
```

All values will be stored as strings by default.

### 2. delete key-value pair

```
del key
```

The delete operation was executed successfully when it returns integer 1. Only nil will be returned when getting key's value which deleted bafore again.

### 3. if the key-value pair exists in the database

```
exists key
```

It returns integer 1 if the key-value pair exists.

### 4. get all keys

```
keys *
```

In fact, this command can be used to match all keys in the database which follow some pattern.

### 5. refresh the database quickly

```
flushall
```

### 6. get some key's time to leave

```
ttl key
```

The return result -1 means this key doesn't have expiration time. This key can keep in the database 

### 7. set expire time for some key

If the key has already been in the database,

```
expire key seconds
```

If you want to create a new key-value pair which has an expiration time,

```
setex key seconds value
```

### 8. store lists in redis

You can imagine lists in redis as a double-direction stack. You can push/pop elements at both the beginning and the end. You can't use 'get' to get values in a list.

```
lpush/rpush key value
lpop/rpop key value
```

Lpush/rpush will return the length of the list. Lpop/rpop will return the value.

```
lrange key start stop
```

This can show many elements in the list. If the stop is -1, all elements in this list will be returned.

### 9. store sets in redis

Set is a data structure which can make sure every element is distinct from each other.

If you want to create a set,

```
sadd name value
```

If you want to check members in the set,

```
smembers name
```

If you want to remove a member,

```
srem name value
```

### 10. hash in redis

If you imagine data in redis is a json object, the process that you create a hash object in redis is creating nested json. So the commands you used to handle hash in redis are very similar to those used for normal data. You can just add an "h" to these commands.

```
hset name field value
hget name field
hgetall name
hdel name field
hexists name field
```

You have to notice that you can't set expiration time for hash objects in redis because the creator of redis wants to keep redis simple.



## How to integrate Redis into Spring Boot project?

I tried to integrate Redis into Spring Boot project. It's not easy to do it. There are many compatiable problems when I tried. Finally, however, I did it. Thus I wanna record what problems I met and how I solved them.

### 1. Dependencies

For compatible reasons, you'd better use Spring Boot version 2 and JDK 8 or 11 or 17. Lombok is very good to eliminate unnecessary code in entities class.

Here are all dependencies you need:

```xml
 				<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.9.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
```

Until June 2023, there are still lot of bugs in Jedis version 4. So you should use version 3.9.0, the latest version in version 3. The new Jedis will cause an error "JedisShardInfo Class not found".

In addition, spring devtools must be removed if you wanna find just one specific record in redis. That is because the restartclassloader can't cast the target class from Object.

### 2. RedisConfig

You will use two beans to do configuration. One is for connecting your redis, the other is for defining the redis template.

The first configuration is normal. It is a JedisConnectionFactory.

```java
    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {
        RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration();
        configuration.setHostName("127.0.0.1");
        configuration.setPort(6379);

        return new JedisConnectionFactory(configuration);
    }
```

The second configuration is more complicated because the spring boot can't know which bean should be autowired. That is RedisTemplate bean, which is used for defining how to store records in redis. A given name is needed to prevent such problem.

Here is the bean code:

```java
    @Bean(name="template")
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(jedisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new JdkSerializationRedisSerializer());
        redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.afterPropertiesSet();

        return redisTemplate;
    }
```

The given name of this bean is "template" right now. When you wanna injected this bean to other class, you should use `@Qualifier` annotation to declare which bean you really wanna inject.

Here is the code:

```java
    @Autowired
    @Qualifier("template")
    private RedisTemplate redisTemplate;
```

### 3. Spring Redis CRUD

Imagining that you have an entity called User. It is a POJO and its key is long integer property called Id. You should define four methods to create, read, update, and delete your entities. Here is code.

```java
    @Autowired
    @Qualifier("template")
    private RedisTemplate redisTemplate;

    private static final String KEY = "USER";

    @Override
    public boolean saveUser(User user) {
        try {
            redisTemplate.opsForHash().put(KEY, user.getId(), user);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    @Override
    public List<User> fetchAllUser() {
        List<User> users;
        users = redisTemplate.opsForHash().values(KEY);
        return  users;
    }

    @Override
    public User fetchUserById(Long id) {
        User user;
        user = (User) redisTemplate.opsForHash().get(KEY, id);
        return user;
    }

    @Override
    public boolean deleteUser(Long id) {
        try {
            redisTemplate.opsForHash().delete(KEY,id);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    @Override
    public boolean updateUser(Long id, User user) {
        try {
            redisTemplate.opsForHash().put(KEY, id, user);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
```

## In the end

I believe that you have a detailed figure about how to use redit as cache in your spring boot project. By using in memory nosql database, you can accelerate your request speed a lot. It can also improve your customers' experience.