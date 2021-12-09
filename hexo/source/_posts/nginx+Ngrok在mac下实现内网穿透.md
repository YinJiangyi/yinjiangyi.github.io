---
title: nginx+Ngrok在mac下实现内网穿透
date: 2021-01-09 12:51:07
tags:
- web服务
categories:
- 计算机网络
typora-root-url: ../../source
---

使用maltego产品，用公司数据做关系探索演示。服务部署在内网环境中，和产品云服务平台交互需要实现内网穿透。简单了解下相关流程。

<!-- more -->

# Nginx介绍

## 产生

Nginx和Apache一样，都是一种**web服务器软件**。基于REST架构风格，以统一URI或URL作为沟通依据，通过HTTP协议提供各种网络服务。但**Apche**发展较早，虽然作为世界第一大服务器软件，具有**稳定、开源、跨平台**的优点，但是他被设计为一个**重量级的不支持高并发的服务器**。数以万计的并发访问会导致服务器消耗大量内存。操作系统对其进行进程或线程间的切换也消耗了大量的CPU资源，导致HTTP请求的平均响应速度降低。

**因此轻量级高并发的服务器Nginx应运而生。**

Nginx由俄罗斯工程师采用C语言开发，具有以下**特点**：

1. Nginx使用基于**事件驱动架构**，使得其可以支持数以百万级别的TCP连接
   （[事件驱动架构](https://developer.ibm.com/zh/technologies/messaging/articles/advantages-of-an-event-driven-architecture/)：是指**松散耦合**的**微服务系统**）
2. 高度的模块化和自由软件许可证使得第三方模块层出不穷
3. 跨平台服务器，可以运行在Linux，Windows，FreeBSD，Solaris，AIX，Mac OS等操作系统上
4. 极大的稳定性

# Nginx 安装与配置

- 安装

  ``` yum install nginx```

- 配置文件解读 https://zhuanlan.zhihu.com/p/74256946

  - 文件结构：

  ```
  ...
  events {
          ...
  }
  http {
      ...
      server {
          ....
          location {
              root html;
              ...
          }
      }
  }
  ```

  nginx配置指令分为**指令块**和**单个指令**（如root html）

  - 全局配置

  ```
  user nginx;
  worker_processes auto;
  error_log /var/log/nginx/error.log;
  pid /run/nginx.pid;
  
  # Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
  include /usr/share/nginx/modules/*.conf;
  
  events {
      worker_connections 1024;
  }
  ```

  `user`：指定nginx的工作进程的用户及用户组，默认是nobody用户
  `worker_processes`:指定工作进程的个数，默认是1个。具体可以根据服务器cpu数量进行设置，比如cpu有4个，可以设置为4。如果不知道cpu的数量，可以设置为auto。nginx会自动判断服务器的cpu个数，并设置相应的进程数。
  `error_log`:设置nginx的错误日志路径，并设置相应的输出级别。如果编译时没有指定编译调试模块，那么 info就是最详细的输出模式了。如果有编译debug模块，那么debug时最为详细的输出模式。这里设置为默认就好了。
  `pid`: 指定nginx进程pid的文件路径。
  `events ` :这个指令块用来设置工作进程的工作模式以及每个进程的连接上限。
  -`use` :用来指定nginx的工作模式，通常选择epoll，除了epoll，还有select,poll。
  -`worker_connections`:定义每个工作进程的最大连接数，默认是1024。客户端最大连接数，就要考虑有几个工作进程了，两者相乘就是的。当nginx作反向代理时，需要除以2.

- 修改端口号和html目录

  ```
      server {
          listen       128 default_server;
          listen       [::]:128 default_server;
          server_name  _;
          # root         /usr/share/nginx/html;
          root         /root/CyberNarrator/ShowCyberNarrator;
  
          # Load configuration files for the default server block.
          include /etc/nginx/default.d/*.conf;
  
          location / {
              autoindex on; # 显示目录
              autoindex_exact_size on;
              autoindex_localtime on;
          }
  
          error_page 404 /404.html;
          location = /404.html {
          }
  
          error_page 500 502 503 504 /50x.html;
          location = /50x.html {
          }
      }
  ```

  重启nginx服务`nginx service start` 后，访问192.168.40.154:128出现403 forbedden

  原因：Web目录/root/CyberNarrator/ShowCyberNarrator权限问题，修改为777也没有解决。将web目录改为home下目录/home/cyber_narrator/show_maltego成功。

  

  ## 基于Nginx在mac上搭建本地文件服务器

  在服务器上尝试成功后，实现在mac上搭建文件服务器

- ```brew install nginx``` 安装nginx，发现已经安装过，不知道啥时候安装的。

- 运行nginx ```nginx``` 提示端口被占用

- 查看端口 占用情况 ```lsof -i: 8080```

  ```
  COMMAND   PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
  nginx   40125  joy    6u  IPv4 0xd9490f7db5cbf449      0t0  TCP *:http-alt (LISTEN)
  nginx   40126  joy    6u  IPv4 0xd9490f7db5cbf449      0t0  TCP *:http-alt (LISTEN)
  ```

- 如果需要杀掉进程，输入 ```kill -9 pid ``` 或 `kill-9 name`

- 修改nginx.conf

  ```
     server {
          listen       8080;
          server_name  localhost;
          root /Users/joy/work/iie/project/cyber_narrator/MALTEGO/html;
          location / {
              autoindex on; 
              autoindex_exact_size on;
              autoindex_localtime on;
          }
  ```

- 保存配置文件，`nginx` 运行，`nginx -s reload` 重新加载配置文件

- 访问 http://localhost:8080/transforms/ 即出现web root目录下transform文件夹的文件。



## Ngrok实现内网穿透

[Ngrok](https://learnku.com/articles/36129)是一个**反向代理**，内网穿透工具

- 注册账号 

- 下载安装包

- [配置authtoken](https://dashboard.ngrok.com/get-started/setup)

  `./ngrok authtoken 1muCRyfXzexS8Ui5RWIi4pvUNvR_2rGsMdvDwQpKcwS1iTpAo`

- 运行命令

  ngrok http -host-header=shop.test -region us 8080

  ```
  Session Status                online                                                                                  
  Account                       yinjiangyi (Plan: Free)                                                                 
  Version                       2.3.35                                                                                  
  Region                        United States (us)                                                                      
  Web Interface                 http://127.0.0.1:4040                                                                   
  Forwarding                    http://08112e20c814.ngrok.io -> http://localhost:8080                                   
  Forwarding                    https://08112e20c814.ngrok.io -> http://localhost:8080                                  
                                                                                                                        
  Connections                   ttl     opn     rt1     rt5     p50     p90                                             
                                3       0       0.00    0.00    0.00    0.01  
  ```

  这里forwarding表示分配的域名，**免费账号每次分配的域名无法固定，关闭后再次运行代理服务后域名改变**。配置完成后，就可以实现外网访问服务。



------

以下记录我在服务器上做了什么，以防后续产生问题有迹可循：：

- 安装nginx，搭建文件服务器，web目录为/home/cyber_narrator/show_maltego
- 添加python3环境，
  - home目录下新建yjy目录
  - 安装目录/usr/local/python3  **没有动现有的python2环境**



参考：
https://www.cnblogs.com/wcwnina/p/8728391.html
