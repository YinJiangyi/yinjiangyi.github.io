---
title: Superset 性能优化
typora-root-url: ../../source
date: 2021-04-21 10:56:56
tags: superset
catagories:
sticky:
---

### Caching配置

官方文档：https://superset.apache.org/docs/installation/cache

参考：https://yorke.gitbooks.io/superset-note/content/chapter1.html

#### 使用 Redis 做 Superset 的缓存

配置 Superset 的缓存后端，只需在其配置文件中提供一个 `CACHE_CONFIG` 常量即可。

而对于 Redis，我们还需要一个运行一个 Redis 服务，以及安装一个 redis-py 库（Redis 的 Python 接口）。

##### 安装 Redis

当前 Redis 最新版为 3.2.8，在 `/opt/` 目录下安装并启动：

```
cd /opt/
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
tar xzvf redis-3.2.8.tar.gz
cd redis-3.2.8/
make
cd src/
./redis-server
```

make时报错：

```shell
……
server.c: 在函数‘hasActiveChildProcess’中:
server.c:1476:1: 警告：在有返回值的函数中，控制流程到达函数尾 [-Wreturn-type]
 }
 ^
server.c: 在函数‘allPersistenceDisabled’中:
server.c:1482:1: 警告：在有返回值的函数中，控制流程到达函数尾 [-Wreturn-type]
 }
 ^
server.c: 在函数‘writeCommandsDeniedByDiskError’中:
server.c:3747:1: 警告：在有返回值的函数中，控制流程到达函数尾 [-Wreturn-type]
 }
 ^
server.c: 在函数‘iAmMaster’中:
server.c:4914:1: 警告：在有返回值的函数中，控制流程到达函数尾 [-Wreturn-type]
 }
 ^
make[1]: *** [server.o] 错误 1
make[1]: 离开目录“/opt/redis/redis-6.0.0/src”
make: *** [all] 错误 2
```

 解决： https://blog.csdn.net/weixin_45627031/article/details/107166867?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-0&spm=1001.2101.3001.4242

升级gcc版本：

```
[root@hadoop01 redis-6.0.5]# gcc -v                             # 查看gcc版本
[root@hadoop01 redis-6.0.5]# yum -y install centos-release-scl  # 升级到9.1版本
[root@hadoop01 redis-6.0.5]# yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
[root@hadoop01 redis-6.0.5]# scl enable devtoolset-9 bash
以上为临时启用，如果要长期使用gcc 9.1的话：
[root@hadoop01 redis-6.0.5]# echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

##### 测试

```
(superset) [root@bigdata-12 redis-6.0.0]# redis-cli
127.0.0.1:6379>
```

启动成功，查看进程

```
(superset) [root@bigdata-12 redis-6.0.0]# ps -ef | grep redis
root     17215 11601  0 16:14 pts/5    00:00:00 ./src/redis-server *:6379 [cluster]
```

这里主机号应该是*而不是127.0.0.1，否则外部机器无法连接redis服务。

如果为127.0.0.1，修改redis.conf文件：

```
1>注释掉bind
#bind 127.0.0.1
2>默认不是守护进程方式运行，这里可以修改
daemonize no
3>禁用保护模式
protected-mode no
```

启动Redis时指明配置文件

```
redis-server ../redis.conf
```

##### 安装 redis-py

在 Superset 所在虚拟环境中安装 redis-py：

```
pip install redis
```

##### 配置使用 Redis 做缓存

在 Superset 的 config.py 中，做如下配置：

```python
import redis
...
CACHE_DEFAULT_TIMEOUT = 500 # 默认超时时间
CACHE_CONFIG = {
    'CACHE_TYPE': 'redis', # 使用 Redis
    'CACHE_REDIS_HOST': 'localhost', # 配置域名
    'CACHE_REDIS_PORT': 6379, # 配置端口号
    'CACHE_REDIS_URL': 'redis://localhost:6379' # 配置 URL
}
```

重新启动 Superset

打开任意 Dashboard，观察后台打印情况



配置成功！



##### 其他问题