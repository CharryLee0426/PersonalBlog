## How to install Redis on Mac running on Apple silicon

### 1. Make sure that you have installed homebrew

You can install redis without homebrew surely, but you'd better install it with homebrew because it makes you easier.

For installing homebrew on Mac running on Apple silicon, you can see this website:

[https://www.igeeksblog.com/how-to-install-homebrew-on-mac/](https://www.igeeksblog.com/how-to-install-homebrew-on-mac/)

### 2. Install redis

Use the following simple command to install redis:

```shell
$ brew install redis
```

If you really don't want to install it with homebrew, you can see the official website of redis to install.

[https://redis.io](https://redis.io)

### 3. How to start

After install, you can start the service backgroundly or just in a terminal.

#### 3.1 backgroundly

Use the following simple command to start service backgoundly:

```shell
$ brew services start redis
```

If success, you will see that:

```shell
==> Successfully started `redis` (label: homebrew.mxcl.redis)
```

Then you can do other things in this terminal window.

#### 3.2 Redis-server

You can use `redis-server` to start the redis.

However, if you want use redis, you can't close this terminal because redis-server still runs on it.

### 4. How to operate the Redis

After starting the service, you can use `redis-cli`, a simple command line tool to CRUD. You can also use some visual IDE such as Navicat or Data Grip.

The default port is `6379`. (What a strange number!)

### 5. How to stop the Redis

You can test if your redis service is running by using `redis-li ping`. If the response is `Pong`, it just tells you that your service is running now.

Use the following command to stop service backgroundly:

```shell
$ brew services stop redis
```

If success, you will see that:

```bash
==> Successfully stopped `redis` (label: homebrew.mxcl.redis)
```

After that, when you test with `redis-cli ping` again, you will see that warning `Could not connect to Redis at 127.0.0.1:6379: Connection refused`.



### In the end

I hope that this article can help you to install and use Redis. And it's also my first English technical article.

