---
title: Mac下hadoop部署与操作
typora-root-url: ../../source
date: 2021-03-30 18:41:13
tags:
categories:
sticky:
---

部署：

http://codingxiaxw.cn/2016/12/06/59-mac-hadoop/

其他问题：

1） hadoop下载速度太慢，切换国内镜像无效。
采用手动下载方式 https://zhuanlan.zhihu.com/p/107469378

2） 启动hadoop权限问题：

```shell
(base) MacBook-Pro:sbin joy$ ./start-dfs.sh 
Starting namenodes on [localhost]
localhost: joy@localhost: Permission denied (publickey,password,keyboard-interactive).
Starting datanodes
localhost: joy@localhost: Permission denied (publickey,password,keyboard-interactive).
Starting secondary namenodes [account.jetbrains.com]
account.jetbrains.com: Warning: Permanently added 'account.jetbrains.com,0.0.0.0' (ECDSA) to the list of known hosts.
account.jetbrains.com: joy@account.jetbrains.com: Permission denied (publickey,password,keyboard-interactive).
2021-03-30 18:38:31,682 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

https://blog.csdn.net/zhangvalue/article/details/103748438

3）按照http://codingxiaxw.cn/2016/12/07/60-mac-spark/内容进行spark安装时，由于下载速度问题，手动下载安装包进行安装，发现scala使用正常，spark命令无法正常运行：

```shell
MacBook-Pro:conf joy$ scala
Welcome to Scala 2.13.5 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_271).
Type in expressions for evaluation. Or try :help.

scala> 
```

```shell
(base) MacBook-Pro:conf joy$ spark-shell
-bash: spark-shell: command not found
```

**问题排查：主要原因在于scala下载后用homebrew安装，spark没有经过brew安装，建立软连接**

homebrew进行Scala安装时，手动下载安装包之后，按照2）解决方案进行后续安装操作，brew将scala自动安装在/usr/local/Cellar下，然后自动建立软连接到/usr/local/bin/目录下（安装时使用-v命令可查看详细过程）

```shell
(base) MacBook-Pro:~ joy$ brew reinstall scala -v
rm /usr/local/bin/fsc
rm /usr/local/bin/scala
rm /usr/local/bin/scalac
rm /usr/local/bin/scaladoc
rm /usr/local/bin/scalap
rm /usr/local/share/doc/scala
rm /usr/local/share/man/man1/fsc.1
rm /usr/local/share/man/man1/scala.1
rm /usr/local/share/man/man1/scalac.1
rm /usr/local/share/man/man1/scaladoc.1
rm /usr/local/share/man/man1/scalap.1
==> Downloading https://downloads.lightbend.com/scala/2.13.5/scala-2.13.5.tgz
/usr/bin/curl --disable --globoff --show-error --user-agent Homebrew/3.0.10-42-ge5bda4e\ \(Macintosh\;\ Intel\ Mac\ OS\ X\ 10.15.7\)\ curl/7.64.1 --header Accept-Language:\ en --retry 3 --location --silent --head --request GET https://downloads.lightbend.com/scala/2.13.5/scala-2.13.5.tgz
Already downloaded: /Users/joy/Library/Caches/Homebrew/downloads/f0d485510e9c8d56e9ed6b020888e122c1ef09fff596c0c93ca946952b4624c2--scala-2.13.5.tgz
==> Verifying checksum for 'f0d485510e9c8d56e9ed6b020888e122c1ef09fff596c0c93ca946952b4624c2--scala-2.13.5.tgz'
==> Reinstalling scala 
/usr/bin/sandbox-exec -f /private/tmp/homebrew20210331-48919-1kaua9k.sb nice ruby -W1 -- /usr/local/Homebrew/Library/Homebrew/build.rb /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/scala.rb --verbose
tar xof /Users/joy/Library/Caches/Homebrew/downloads/f0d485510e9c8d56e9ed6b020888e122c1ef09fff596c0c93ca946952b4624c2--scala-2.13.5.tgz -C /private/tmp/d20210331-48920-5yc016
cp -pR /private/tmp/d20210331-48920-5yc016/scala-2.13.5/. /private/tmp/scala-20210331-48920-3r6y9a/scala-2.13.5
chmod -Rf +w /private/tmp/d20210331-48920-5yc016
==> Cleaning
==> Finishing up
ln -s ../Cellar/scala/2.13.5/bin/fsc fsc
ln -s ../Cellar/scala/2.13.5/bin/scala scala
ln -s ../Cellar/scala/2.13.5/bin/scalac scalac
ln -s ../Cellar/scala/2.13.5/bin/scaladoc scaladoc
ln -s ../Cellar/scala/2.13.5/bin/scalap scalap
ln -s ../../Cellar/scala/2.13.5/share/doc/scala scala
ln -s ../../../Cellar/scala/2.13.5/share/man/man1/fsc.1 fsc.1
ln -s ../../../Cellar/scala/2.13.5/share/man/man1/scala.1 scala.1
ln -s ../../../Cellar/scala/2.13.5/share/man/man1/scalac.1 scalac.1
ln -s ../../../Cellar/scala/2.13.5/share/man/man1/scaladoc.1 scaladoc.1
ln -s ../../../Cellar/scala/2.13.5/share/man/man1/scalap.1 scalap.1
/usr/bin/sandbox-exec -f /private/tmp/homebrew20210331-48976-3t2vk6.sb nice ruby -W1 -I $LOAD_PATH -- /usr/local/Homebrew/Library/Homebrew/postinstall.rb /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/scala.rb
==> Caveats
To use with IntelliJ, set the Scala home to:
  /usr/local/opt/scala/idea
==> Summary
🍺  /usr/local/Cellar/scala/2.13.5: 43 files, 23.4MB, built in 14 seconds
```

而homebrew无法下载正确版本的spark，因此手动下载后，安装博客步骤进行安装，手动解压安装在/usr/loacal，配置时无脑copy

<img src="/images/Mac%E4%B8%8Bhadoop%E9%83%A8%E7%BD%B2%E4%B8%8E%E6%93%8D%E4%BD%9C/image-20210331150851400.png" alt="image-20210331150851400" style="zoom:80%;" />

其中scala_home路径应该为实际路径：brew自动安装后建立软连接的 `/usr/local/scala/bin` 而不是 `/usr/local/scala`。

**解决一：**保持sparka安装在/usr/local目录下，修改conf中的scala home路径

上图修改为如下后，重新运行spark-shell命令成功。

```
export SCALA_HOME=/usr/local/bin/scala
export SPARK_MASTER_IP=localhost
export SPARK_WORKER_MEMORY=4g
```

```shell
MacBook-Pro:conf joy$ spark-shell
21/03/31 15:00:37 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://192.168.36.116:4040
Spark context available as 'sc' (master = local[*], app id = local-1617174045126).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.1.1
      /_/
         
Using Scala version 2.12.10 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_271)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
```

**解决二（理论上可行）：**想要和brew安装方式对齐，将spark移动到/usr/local/Cellar下，按brew安装的方式重新建立软连接至/usr/local/bin

首先，将解压后的spark包移动到Cellar目录下，修改文件格式，和其他安装包保持一致，即Cellar/spark/3.1.1/

然后`brew link spark`

```shell
Error: Could not symlink sbin/start-all.sh
Target /usr/local/sbin/start-all.sh
is a symlink belonging to hadoop. You can unlink it:
  brew unlink hadoop
```

提示/usr/local/bin里已经有一个名为 ’start-all.sh‘ 的同名脚本了。也就是hadoop和spark里有脚本名重复了，所以方便起见，采用了第一种解决方案，但是为了/usr/local目录的整洁性，如果可以，还是优先用brew安装，或者靠齐安装目录格式比较好。

