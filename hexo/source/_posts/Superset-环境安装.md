---
title: Superset-环境安装
typora-root-url: ../../source
date: 2021-04-18 10:14:01
tags:
categories:
sticky:
---

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

  删除G:\pyProduct\incubator-superset\superset\static\assets文件
  输入代码(根据你下载代码的路径而定) 

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

  <img src="/images/Superset%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/106452635-2493d300-64c3-11eb-9e7c-69c6d03bf426.png" alt="img" style="zoom:67%;" />

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

```\
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

  <img src="/images/Superset%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/image-20210415104252896.png" alt="image-20210415104252896" style="zoom:80%;" />

  解决：

  ```
  pip uninstall infi.clickhouse_orm
  pip install infi.clickhouse_orm==1.0.4
  ```

- 如果出现pip install 安装过程中build wheel失败，直接下载对应库的wheel包进行单独pip install 即可。



