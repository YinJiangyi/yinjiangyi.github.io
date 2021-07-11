---
title: Linux服务器安装Python3
typora-root-url: ../../source
date: 2021-04-20 09:35:02
tags:
categories:
sticky:
---

参考：https://blog.csdn.net/a2099948768/article/details/81531774

### 步骤

由于centos7原本就安装了Python2，而且这个Python2不能被删除，因为有很多系统命令，比如yum都要用到。

```
[root@iZuf6ititjgl7x9tgf1cyiZ ~]# python
Python 2.6.6 (r266:84292, Aug 18 2016, 15:13:37) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-17)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
```

输入Python命令，查看可以得知是Python2.6.6版本

输入

```
which python
```

可以查看位置，一般是位于/usr/bin/python目录下。

下面介绍安装Python3的方法

首先安装依赖包

```
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

然后根据自己需求下载不同版本的Python3，我下载的是Python3.7.4

```
https://www.python.org/ftp/python/3.7,4/Python-3.7.4.tar.xz
```

如果速度不够快，可以直接去官网下载，利用WinSCP等软件传到服务器上指定位置，我的存放目录是/usr/local/python3，使用命令：

```
mkdir /usr/local/python3 
```

建立一个空文件夹

然后解压压缩包，进入该目录，安装Python3

```
tar -xvJf  Python-3.7.4.tar.xz
cd Python-3.7.4
./configure --prefix=/usr/local/python3
make && make install
```

最后创建软链接

```
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```

在命令行中输入python3测试

### 问题

- 依赖包查找失败

  编译make的时候会出现：

  ```shell
  Python build finished successfully!
  The necessary bits to build these optional modules were not found:
  _curses               _curses_panel         _lzma
  _tkinter              _uuid                 readline
  To find the necessary bits, look in setup.py in detect_modules() for the module's name.
  
  
  The following modules found by detect_modules() in setup.py, have been
  built by the Makefile instead, as configured by the Setup files:
  _abc                  atexit                pwd
  time
  ```

  这种情况下安装python命令可以正常执行，但是有些任务会import 报错，安装superset时就除了问题

  **解决：**

  - _sqlite: 

    https://stackoverflow.com/questions/1210664/no-module-named-sqlite3

    `yum install sqlite-devel`

    然后重新编译安装python3

  - _lzma:

    https://github.com/ultralytics/yolov5/issues/1298