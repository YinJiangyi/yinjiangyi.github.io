---
title: Superset 上手记录
typora-root-url: ../../source
date: 2021-04-21 10:56:56
tags: superset
categories: 环境部署
sticky:
---



**含泪在卷首血书：**

**有新不用旧啊！版本兼容问题会被搞到死！**



# 1. 关于安装

## Mac系统踩坑

安装方式：

- 基于docker安装：快速方便，但是不适合二次开发，只适合对superset直接进行使用
- 基于pip安装：快速方便，但是源码需要手动下载，并且版本切换坑比较多
- （推荐）基于源码安装：git方便进行版本切换，可以根据网上二次开发的资料完整度checkout到合适的版本进行开发

#### 基于docker安装

**安装docker**

已安装，顺便更新了一下

>Once you have Docker for Mac installed, open up the preferences pane for Docker, go to the "Resources" section and increase the allocated memory to 6GB. With only the 2GB of RAM allocated by default, Superset will fail to start.

修改docker内存配置至6GB

**拉取superset 仓库**

`git clone https://github.com/apache/superset.git` 巨慢

**docker中启动superset**

```
$ cd superset
$ git checkout latest
$ docker-compose up
```

等待容器下载。下载完毕后找不到文件，未解决……转至install from scratch

```shell
superset_tests_worker exited with code 127
superset_init exited with code 127
superset_node exited with code 1
superset_worker exited with code 127
superset_worker          | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_worker          | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_app exited with code 127
superset_app             | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_app             | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_worker exited with code 127
superset_worker          | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_worker          | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_worker          | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_app exited with code 127
superset_app             | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_app             | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_app             | /usr/bin/docker-entrypoint.sh: line 21: /app/docker/docker-bootstrap.sh: No such file or directory
superset_worker exited with code 127
```

错误原因：官网步骤不详细，需要下载superset镜像，创建superset容器，启动容器

参考https://blog.csdn.net/nikeylee/article/details/115264818解决，主要步骤：

```
docker search superset
# 可以自选版本
docker pull amancevice/superset
# 查看镜像是否下载成功：
docker images
# 创建Superset容器
# 创建挂载的目录：
mkdir /opt/superset
# 创建容器
docker run --name my_superset -d -p 8088:8088 -v /opt/superset:/home/superset amancevice/superset
# 进入docker镜像
docker exec -it my_superset /bin/bash
# superset 配置
superset db upgrade
superset init
export FLASK_APP=superset
flask fab create-admin
superset load_examples
# 启动superset命令
superset run -p 8088
```



#### 基于pip安装

参考https://superset.apache.org/docs/installation/installing-superset-from-scratch

遇到问题：

1） 安装client_driver时报错 OSError: mysql_config not found

解决：https://stackoverflow.com/questions/25459386/mac-os-x-environmenterror-mysql-config-not-found

> Ok, well, first of all, let me check if I am on the same page as you:
>
> - You installed python
> - You did `brew install mysql`
> - You did `export PATH=$PATH:/usr/local/mysql/bin`
> - And finally, you did `pip install MySQL-Python` (or `pip3 install mysqlclient` if using python 3)



#### 源码安装

参考：https://blog.eric7.site/2020/02/23/superset/

**Anaconda 创建python环境**

```
conda create -n superset python=3.7
```

激活环境

```
conda activate superset
```

**github 下载源码包**

```
git clone https://github.com/apache/incubator-superset.git

git tag　列出所有版本号

#切换到自己想要的版本 
git checkout　+某版本号
```

这里我是用资料较多的0.28版本

**安装扩展依赖**

```fallback
#进入到目录
cd incubator-superset
pip install -r requirements.txt -i https://pypi.douban.com/simple/

安装开发依赖
pip install -r requirements-dev.txt -i https://pypi.douban.com/simple/
```

这一步安装报错：

>版本问题报错
>
>pandas, cryptography
>
>对下面配置superset造成影响

**安装superset**

```
#incubator-superset 目录执行 pip install -e . 
```

**配置superset**

```fallback
创建管理员账户和密码
这个命令已经不推荐使用了。
fabmanager create-admin --app superset 
推荐使用
$export FLASK_APP=superset
flask fab create-admin
```

> 报错：
>
> 1）
>
> ```
> ImportError: cannot import name '_maybe_box_datetimelike' from 'pandas.core.common' (/Users/joy/opt/anaconda3/envs/superset/lib/python3.7/site-packages/pandas/core/common.py)
> ```
>
> 要求pandas版本是0.23.1，安装的版本为1.2.4
>
> ```
> (superset) ➜  incubator-superset git:(0.28) pip show pandas
> Name: pandas
> Version: 1.2.4
> Summary: Powerful data structures for data analysis, time series, and statistics
> Home-page: https://pandas.pydata.org
> Author: None
> Author-email: None
> License: BSD
> Location: /Users/joy/opt/anaconda3/envs/superset/lib/python3.7/site-packages
> Requires: python-dateutil, pytz, numpy
> Required-by: superset
> ```
>
> 尝试安装低版本pandas 要求的0.23.1在python3.7上build失败，
>
> 安装pandas==0.23.4  
>
> 安装后，superset upgrade 和init命令执行报错
>
> 2）
>
> 安装MarkupSafe==1.0时报错
>
> 解决：
>
> pip install --upgrade pip setuptools==45.2.0
>
> 后重新运行
>
> **其他版本问题基本可以解决，pypi可以下载到python36的安装包，然后自行pip install 安装包进行安装**



_____

修改配置环境：

重新创建conda虚拟环境，superset版本从0.28为0.36

```
conda deactive
# 删除原有环境
conda remove -n superset --all
# 新建
conda create -n superset python=3.7
conda activate superset

git checkout 0.36
……（后续安装步骤类似）
```

>注意：创建用户时有时可能因为用户邮箱等信息重复而创建失败

还是不行，初始化superset时报错

> ERROR [root] Error: Can't locate revision identified by 'c878781977c6'



---

尝试直接安装最新版1.0.1（直接pip install -e .）

总算正常运行



**配置superset**

```
#初始化数据库
python superset db upgrade

#初始化角色和权限
python superset init

#加载示例数据
python superset load_examples
```

**安装编译前端文件，并进入前端开发者模式**

- 安装nodejs：解压，建立软连接

  然后node -v 查看版本，查看正常即可

- 设置npm为淘宝镜像（原始地址：https://registry.npmjs.org）

  ```
  npm config set registry https://registry.npm.taobao.org
  验证 `npm config get registry`
  ```

- 进入superset/assets目录下（新版目录incubator-superset/superset-frontend/）,安装编译所需模块

  ```
  npm install -d
  ```

- 编译前端文件，并进入前端开发者模式

  ```
  npm run build npm run dev 
  ```

  注：如果这两部编译失败，可能是因为nodejs版本过低。可尝试升级nodejs再试。

  - 运行完npm run dev，命令窗口会停止，不要误以为是执行不下去，保持窗口开启状态即可，一有文件变动时，它会重新编译

#### 其他问题

- pip 方式安装好之后，clickhouse连接总是出问题：

  - 按照官网要求安装sqlalchemy-clickhouse后，test连接是出现 Error: Unexpected error occured
  - 然后发现安装clickhouse-sqlalchemy之后可以解决连接问题，但是查询出来的数据没有表头

  终于在https://github.com/apache/superset/issues/9719找到解决办法：

  这位朋友和我的情况也一样

  <img src="../images/Superset%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/image-20210415104252896.png" alt="image-20210415104252896" style="zoom:80%;" />

  解决：

  ```
  pip uninstall infi.clickhouse_orm
  pip install infi.clickhouse_orm==1.0.4
  ```

## Window 系统下安装完全解决

mac本出了点问题，转战工作电脑window系统

#### pip方式快速安装

- 新建conda虚拟环境（主要为了降低试错成本，不影响其他环境）

  ```
  conda create -n superset(环境名) python=3.7
  conda activate superset
  ```

- 虚拟环境中，安装superset

  ```
  pip install apache-superset -i https://pypi.mirrors.ustc.edu.cn/simple/
  ```

  安装过程中可能出现的问题：

  - 提示需要Vitual c++ 14.0以上版本，个别依赖库编译失败，无法生成wheel：

    解决：直接下载wheel，pip install wheel文件安装成功后，重新执行

- 安装成功后配置

  这一步之后很多博客要求进入superset/bin目录下执行python superset ……命令，可能由于版本问题，并不适用。之后步骤以此为准：

  ```
  # 初始化数据库
  superset db upgrade
  # 创建管理员用户
  superset fab create-admin
  (fabmanager create-admin --app superset 命令会出现nontype "au_then"?（记不清了）报错)
  # 初始化角色和权限
  superset init
  ```

#### 二次开发环境安装

pip安装仅适合使用，不适合二次开发。二次开发需要源码下载

- 另外新建一份虚拟环境

- 从github上下载源码 

  `git clone https://github.com/apache/incubator-superset.git`

- 安装 nodejs（这一步不需要在虚拟环境中），下载地址为 https://nodejs.org/dist/v12.4.0/node-v12.4.0-x64.msi

  测试安装是否成功：`node -v`  `npm -v`

- （这一步不确定什么用）制作软连接（因为下载下来的源代码是superset\static\assets这个软连接可以在linux或者Mac上正常工作，但是在windows下不能正常工作）

  删除G:\pyProduct\incubator-superset\superset\static\assets文件
  输入代码(根据你下载代码的路径而定) 

  ```
  mklink /D "D:\incubator-superset\superset\static\assets" "D:\incubator-superset\superset
  需要管理员模式运行
  ```

- 安装。进入下载好的源码目录 `incubator-superset` 中

  `pip install -e .`

  (新版的superset最外层目录下已经没有requirement.txt了，直接安装就可以)

  这一步出现的无法生成wheel文件问题解决同上

- 创建用户、初始化数据库、用户权限步骤类似

- 编译前端文件，进入superset-fronted 目录，`npm install -d`

  报错：

  ```
  ERROR in ./src/visualizations/TimeTable/TimeTable.jsx 101:10
  Module parse failed: Unexpected token (101:10)
  You may need an appropriate loader to handle this file type, currently no loader
  ……
  ```

  解决：https://github.com/apache/superset/issues/10997

  修改目录下 webpack.config.js 文件：

  <img src="../images/Superset%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/106452635-2493d300-64c3-11eb-9e7c-69c6d03bf426.png" alt="img" style="zoom:67%;" />

  成功！

- 进入前端开发者模式 `npm run dev`

- 目录导入pycharm，配置好编译器就可以进行开发了


## 最终版解决方案

### 环境

python：3.7

superset：源码安装最新版1.1.0

### 安装步骤

windows和mac系统亲测有效

#### pip方式快速安装

- 新建conda虚拟环境（主要为了降低试错成本，不影响其他环境）

  ```
   conda create -n superset(环境名) python=3.7
   conda activate superset
  ```

- 虚拟环境中，安装superset

  ```
   pip install apache-superset -i https://pypi.mirrors.ustc.edu.cn/simple/
  ```

  安装过程中可能出现的问题：

  - 提示需要Vitual c++ 14.0以上版本，个别依赖库编译失败，无法生成wheel：

    解决：直接下载wheel，pip install wheel文件安装成功后，重新执行

- 安装成功后配置

  这一步之后很多博客要求进入superset/bin目录下执行python superset ……命令，可能由于版本问题，并不适用。之后步骤以此为准：

  ```
   # 初始化数据库
   superset db upgrade
   # 创建管理员用户
   superset fab create-admin
   (fabmanager create-admin --app superset 命令会出现nontype "au_then"?（记不清了）报错)
   # 初始化角色和权限
   superset init
  ```

#### 二次开发环境安装

pip安装仅适合使用，不适合二次开发。二次开发需要源码下载

- 另外新建一份虚拟环境

- 从github上下载源码 

  `git clone https://github.com/apache/incubator-superset.git`

- 安装 nodejs（这一步不需要在虚拟环境中），下载地址为 https://nodejs.org/dist/v12.4.0/node-v12.4.0-x64.msi

  测试安装是否成功：`node -v`  `npm -v`

- （这一步不确定什么用，mac 不需要）制作软连接（因为下载下来的源代码是superset\static\assets这个软连接可以在linux或者Mac上正常工作，但是在windows下不能正常工作）

  删除G:\pyProduct\incubator-superset\superset\static\assets文件 输入代码(根据你下载代码的路径而定) 

  ```
   mklink /D "D:\incubator-superset\superset\static\assets" "D:\incubator-superset\superset
   需要管理员模式运行
  ```

- 安装。进入下载好的前端目录 `incubator-superset` 中

  `pip install -e .`

  (新版的superset最外层目录下已经没有requirement.txt了，直接安装就可以)

  这一步出现的无法生成wheel文件问题解决同上

- 创建用户、初始化数据库、用户权限步骤类似

- 编译前端文件，进入superset-fronted 目录，`npm install -d`

  报错：

  ```
   ERROR in ./src/visualizations/TimeTable/TimeTable.jsx 101:10
   Module parse failed: Unexpected token (101:10)
   You may need an appropriate loader to handle this file type, currently no loader
   ……
  ```

  解决：https://github.com/apache/superset/issues/10997

  修改目录下 webpack.config.js 文件：

  ![img](file:///Users/joy/MYBLOG/yinjiangyi.github.io/hexo/source/images/Superset%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/106452635-2493d300-64c3-11eb-9e7c-69c6d03bf426.png?lastModify=1626061918)

  成功！

- 进入前端开发者模式 `npm run dev`

- 目录导入pycharm，配置好编译器就可以进行开发了



#### 无网络环境下基于docker安装和Dashboard迁移

xj环境superset安装

**1）提前下载源码和docker镜像**

源码直接在github下载

需要提前下载的镜像：apache/superset, redis, node, post和postgress

```
 # 在有网络的机器上进行镜像拉取
 docker pull xxx(镜像名)
 docker save -o xxx.tar(要保存的压缩包名) xxx(镜像名)
```

**2）在目标机器上安装**

```
 # 将压缩包拷贝到目标机器上
 docker load -i xxx.tar(压缩包名)
 # 在目标机器上的源码目录下运行，等待容器创建和运行
 docker-compose up
 或 docker-compose -f docker-compose.yml up
```





### 遇到的问题

- 安装client_driver时报错 OSError: mysql_config not found

  解决：https://stackoverflow.com/questions/25459386/mac-os-x-environmenterror-mysql-config-not-found

  > Ok, well, first of all, let me check if I am on the same page as you:
  >
  > - You installed python
  > - You did `brew install mysql`
  > - You did `export PATH=$PATH:/usr/local/mysql/bin`
  > - And finally, you did `pip install MySQL-Python` (or `pip3 install mysqlclient` if using python 3)

- pip 方式安装好之后，clickhouse连接总是出问题：

  - 按照官网要求安装sqlalchemy-clickhouse后，test连接是出现 Error: Unexpected error occured
  - 然后发现安装clickhouse-sqlalchemy之后可以解决连接问题，但是查询出来的数据没有表头

  终于在https://github.com/apache/superset/issues/9719找到解决办法：

  这位朋友和我的情况也一样

  ![image-20210415104252896](file:///Users/joy/MYBLOG/yinjiangyi.github.io/hexo/source/images/Superset%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/image-20210415104252896.png?lastModify=1626061918)

  解决：

  ```
   pip uninstall infi.clickhouse_orm
   pip install infi.clickhouse_orm==1.0.4
  ```

- 如果出现pip install 安装过程中build wheel失败，直接下载对应库的wheel包进行单独pip install 即可。





# 2. 性能优化

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