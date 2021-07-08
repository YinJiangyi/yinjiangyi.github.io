---
title: WebSKT项目技能积累总结
typora-root-url: ../../source
date: 2021-03-05 14:00:28
tags:
categories:
sticky:
---

- docker&宿主机交互

  - 宿主机上非root用户无法操作docker：

    ```shell
    [webskt_client@Geedge-CN-API-001 db_sync]$ docker exec mariadb ls
    Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/mariadb/json: dial unix /var/run/docker.sock: connect: permission denied
    ```

    - 解决方法尝试：

      - 创建docker组，将当前用户加入docker组：

        ```shell
        [root@Geedge-CN-API-001 db_csv]# groupadd docker
        [root@Geedge-CN-API-001 db_csv]# gpasswd -a webskt_client docker
        正在将用户“webskt_client”加入到“docker”组中
        ```

      - 重启docker服务，重启mariadb容器

        ```sh
        [root@Geedge-CN-API-001 db_csv]# service docker restart
        Redirecting to /bin/systemctl restart docker.service
        [root@Geedge-CN-API-001 db_csv]# docker ps -a
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
        28653f5ad633        mariadb:10.2.26     "docker-entrypoint.s…"   2 weeks ago         Up 7 seconds        0.0.0.0:3306->3306/tcp   mariadb
        [root@Geedge-CN-API-001 db_csv]# docker restart 28653f5ad633
        28653f5ad633
        ```

      - 重新登录用户后成功

        ```
        [WebSKT@Geedge-CN-API-001 db_sync]$ su root 
        [WebSKT@Geedge-CN-API-001 db_sync]$ su webskt_client
        [webskt_client@Geedge-CN-API-001 db_sync]$ docker exec mariadb ls
        app
        bin
        boot
        database_bak
        ```

        

- maraidb/mysql操作

- java 
  - jdbc
  - timer定时器
  - Java后台程序自动关闭原因排查：
    - 内存空间不足：java.lang.OutOfMemoryError: Java heap space
  
- linux 
  - crontab定时任务
  
  - shell - 基本语法、传参
  
    - 关于单双引号的反复嵌套：
  
      shell脚本编辑：
  
      ```sh
      #!/bin/bash
      # csv export
      docker exec ${DockerName} bash -c "mysql -u${User} -p${Passwd} -e 'select * from ${Database}.${Table} where create_time >= \"$StartTime\" and create_time <= \"$EndTime\" into outfile \"${CsvTmp}/${CsvFile}\" character set gbk fields terminated by \",\" optionally enclosed by \"\\\"\" '"
      ```
  
      实际执行命令：
  
      ```shell
      docker exec DockerName bash -c "mysql -uUser -pPasswd -e 'select * from Database.Table where create_time >= "StartTime" and create_time <= "EndTime" into outfile "CsvTmp/CsvFile" character set gbk fields terminated by "," optionally enclosed by "\"" '"
      ```
  
  - 文件权限设置、修改
  
  - vim 操作