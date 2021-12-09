---
title: 内网实现maltego-TDS-transform
date: 2021-01-11 15:49:39
tags:
- HTTP服务
- OSNIT
categories: 计算机网络
typora-root-url: ../../source
---

将本地数据库中实体关系通过maltego产品界面进行画布探索展示。maltego产品支持自定义local transform（基于给定节点进行关联节点的探索过程），但是local方式的实现需要用户根据设备的具体情况，对python编译器等进行配置，不方便功能共享和展示。TDS-transform结合内网穿透可以解决分布式操作需求。

<!-- more -->

## 背景

工作任务：

将本地数据库中实体关系通过maltego产品界面进行画布探索展示。maltego产品支持自定义local transform（基于给定节点进行关联节点的探索过程），但是local方式的实现需要用户根据设备的具体情况，对python编译器等进行配置，不方便功能共享和展示。

maltego的另一种transform形式为Transform Distribution Server（TDS），这种方式就是为了解决多人协作情况下的transform开发问题。用户通过简单配置在maltego客户端集成TDS transform组。当执行其中的TDS transform时，客户端发送必要信息（seed, transform name, input entity）请求TDS，TDS执行代理角色，查询对应的transform配置信息，定位到transform代码真正部署的web服务器进行查询。

Maltego的公共TDS服务器https://cetas.paterva.com/TDS/，注册登录即可使用。客户端配置时仅需要添加seed信息。在自定义transform时需要填写transform代码所在的server地址。

<img src="/images/内网实现maltego-TDS-transform/image-20210111171505571.png" alt="image-20210111171505571" style="zoom:50%;" />



**需要解决的问题**：maltego提供公共TDS的使用，python第三方库maltego-trx提供了transform开发工具包和运行transform server的模板。因此唯一需要解决的就是TDS和transform server间通信。需要一台公网可访问的web服务器运行transform server。

**方法**：基于ngrok实现内网穿透。[Ngrok在mac下实现内网穿透](https://yinjiangyi.github.io/2021/01/09/nginx+Ngrok%E5%9C%A8mac%E4%B8%8B%E5%AE%9E%E7%8E%B0%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/)

## 具体步骤

主要步骤：

- 环境：python，maltego-trx，clickhouse

- 开发transform代码

  - 需要在maltigo-trx代码中添加新增实体

    ![image-20210111165020200](/images/内网实现maltego-TDS-transform/image-20210111165020200.png)

  - `python project.py list` 测试代码

  - `python project.py runserver`

- 内网穿透

  - 安装ngrok，修改端口，运行映射

- 配置TDS

### 1. 开发transform代码

- 安装第三方python库 `pip install  maltego-trx`

- 在自定路径中新建工程目录 `maltego-trx start your_project_name`，生成目录中包括project.py文件和transform文件夹。
- 在transform文件夹中新建py文件开发transform代码。
  - 过程中如果需要定义其他类别实体，如客户端ip，需要在修改maltego-trx中entites.py，添加实体名和ip。添加后即可正常运行。
  - 在客户端中创建实体集合set（可选，方便查找），在集合中添加实体，定义实体logo等。

### 2. 配置TDS

- public TDS[地址]，注册、登录。

- 添加transform

  <img src="/images/内网实现maltego-TDS-transform/image-20210111164025867.png" alt="image-20210111164025867"  />

  - 自定义name和UI Display

  - Transform URL

    cd进项目目录，运行 `python project.py runserver`

    ![image-20210111164604629](/images/内网实现maltego-TDS-transform/image-20210111164604629.png)

    这里列出的就是已经开发好的transforms。

    在transform配置的图中，URL为 http://08112e20c814.ngrok.io/run/mailtomail，其中http://08112e20c814.ngrok.io 是ngrok分配的域名，如果是未经过穿透直接可访问的公网server，这里为http://server_ip:port，`/run/mailtomail` 按transform server urls列表中添加。

  - 选择input entity。如果是自定义，按照代码中添加的实体ID进行添加（cybernarrator.ClientIP）：

    ![image-20210111165020200](/images/内网实现maltego-TDS-transform/image-20210111165020200.png)

  - transform添加成功。

  - seeds选择默认即可。

- 客户端配置

  - 自定义代码中添加的实体类别。

    <img src="/images/内网实现maltego-TDS-transform/image-20210111165329645.png" alt="image-20210111165329645" style="zoom:67%;" />

    这里的unique type name必须和代码中的id一致。可以添加一个合适icon，便于区分。或者直接进行导入。

  - TDS seed添加到客户端

    TDS web interface中，点击seeds，复制url

    <img src="/images/内网实现maltego-TDS-transform/image-20210111170025189.png" alt="image-20210111165941925" style="zoom:50%;" />

    <img src="/images/内网实现maltego-TDS-transform/image-20210111170050676.png" alt="image-20210111170050676" style="zoom:50%;" />

    进入maltego客户端，点击transform->transform hub，添加tranform。自定义ID和name，将复制的seed url粘贴过来，补充其他描述内容，OK。

    <img src="/images/内网实现maltego-TDS-transform/image-20210111170455362.png" alt="image-20210111170455362" style="zoom:67%;" />

    完成添加后，install安装。

    <img src="/images/内网实现maltego-TDS-transform/image-20210111170553250.png" alt="image-20210111170553250" style="zoom: 67%;" />

    安装完成后，查看transform->transform manager -> transform Server -> default，开发的transform已经添加完成

    <img src="/images/内网实现maltego-TDS-transform/image-20210111170907163.png" alt="image-20210111170907163" style="zoom:67%;" />

### 3 使用

客户端点击创建graph图标，即可进行实体添加和transform运行。

![image-20210111171002971](/images/内网实现maltego-TDS-transform/image-20210111171002971.png)

<img src="/images/内网实现maltego-TDS-transform/image-20210111171140927.png" alt="image-20210111171140927" style="zoom:67%;" />

<img src="/images/内网实现maltego-TDS-transform/image-20210111171224959.png" alt="image-20210111171224959" style="zoom:67%;" />

<img src="/images/内网实现maltego-TDS-transform/image-20210111171316270.png" alt="image-20210111171316270" style="zoom:67%;" />