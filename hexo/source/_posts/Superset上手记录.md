---
title: Superset 上手记录
typora-root-url: ../../source
date: 2021-04-21 10:56:56
tags: superset
catagories: 
sticky:
---



**含泪在卷首血书：**

**有新不用旧啊！版本兼容问题会被搞到死！**



## 1. 关于安装

### Mac系统踩坑

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

### Window 系统下安装完全解决

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

  

  