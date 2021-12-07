---
title: 读书笔记-《图解HTTP》
date: 2021-01-01 13:52:53
tags:
- HTTP
- HTTPS
- 学习笔记
categories:
- 计算机网络
typora-root-url: ../../source
---

《图解http》的阅读过程主要内容笔记。<!--more-->

# 第一章 了解web及网络基础

* HTTP的诞生背景和演进过程
* TCP/IP协议族基础
* 与HTTP密切相关的IP，TCP和DNS协议
  * 负责传输的ip协议
  * 确保可靠性的tcp协议
  * 负责域名解析的DNS服务
* URL和URI

# 第二章 简单的HTTP协议

* HTTP协议用于**客户端和服务端**之间的通信

* 通过请求和响应交换达成通信

* HTTP是**不保存状态/无状态**的协议

  * 所谓**不保存状态**是指协议对发送过的请求或响应都不做持久化处理，有新的请求就有新的响应

  * **原因**是为了更快的处理大量事务，但同时导致用户使用问题，比如用户跳转后无法保持登录状态

  * 引入cookie技术，可以管理状态。

    服务器端发送的响应报文内有一个叫Set-Cookie的首部字段信息，客户端会保存下来，当下次客户端在往该服务器发送请求时，请求报文中会加入Cookie值。

* HTTP方法

  * **GET**获取资源
  * **POST**传输实体主体
  * **PUT**传输文件（不带验证机制，存在安全性问题，一般web不采用，当**配合 Web 应用程序的验证机制**或**遵守 REST 标准**时还是有可能会开放使用的。）
  * **HEAD**获得报文首部（不返回报文主体，用于确认URI有效性和资源更新时间等）
  * **DELETE**删除文件（同PUT存在安全性问题）
  * **OPTION**询问支持的方法
  * **TRACE**追踪路径，将之前的请求通信回环给客户端。发送请求时，在 Max-Forwards首部字段中填入数值，每经过一个服务器端就将该数字减 1，当数值刚好减到 0 时，就停止继续传输，最后接收到请求的服务器端则返回状态码 200OK 的响应。（容易引发跨站追踪Cross-site Tracking，不常用）		

  * **CONNECT**要求**隧道协议连接代理**服务器进行TCP通信，主要使用SSL和TLS协议把通信内容加密后经网络隧道传输

* 持久连接节省通信量

  * HTTP协议的初始版本中，每进行一次HTTP通信就要断开一次TCP连接。

    持久连接：建立一次连接后可以进行多次请求和响应的交互（HTTP/1.1版本默认）

  * Piplining：持久连接使得多数请求以管线化方式发送成为可能。

# 第三章 HTTP报文内信息

### 3.1 报文结构

* HTTP报文由**报文首部、CR+LF、报文主体**构成

  * 请求报文：报文首部包括--**请求行、请求首部字段、通用首部字段和实体首部字段**

    * 请求行：方法、URI、HTTP版本

    * 请求首部字段：

      <img src="/images/《图解HTTP》笔记/image-20210101140905350.png" alt="image-20210101140905350" style="zoom:75%;" />

  * 响应报文：报文首部包括--**状态行、响应首部字段、通用首部字段和实体首部字段**

    * 状态行：状态码、原因短语、http版本

    * 响应首部字段

      <img src="/images/《图解HTTP》笔记/image-20210101140915475.png" alt="image-20210101140915475" style="zoom:75%;" />

      ​				

  * 通用首部字段

    <img src="/images/《图解HTTP》笔记/image-20210101140934991.png" alt="image-20210101140934991" style="zoom:75%"/>

  * 实体首部字段

    <img src="/images/《图解HTTP》笔记/image-20210101140934991.png" alt="image-20210101140934991" style="zoom:75%" />

### 3.2 报文编码

* ***实体主体***和***报文主体***（报文结构的一部分）的区别：

  原文：

  >报文(**message**)
  >
  >是 HTTP 通信中的基本单位，由 8 位组字节流(octet sequence， 其中 octet 为 8 个比特)组成，通过 HTTP 通信传输。
  >
  >实体(**entity**) 作为请求或响应的有效载荷数据(补充项)被传输，其内容由实
  >
  >体首部和实体主体组成。
  >HTTP 报文的主体用于传输请求或响应的实体主体。
  >
  >通常，报文主体等于实体主体。只有当传输中进行编码操作时，实体 主体的内容发生变化，才导致它和报文主体产生差异。
  >
  >报文和实体这两个术语在之后会经常出现，请事先理解两者的差异。

  理解：

  未编码的原始数据是实体（网页中的文本、图片等数据载体）主体，当不进行编码传输时，报文主体和实体主体等价，而原始数据经过编码再传输时，报文主体就不再是实体主体了，而是实体首部+实体主体的编码数据

* 内容编码

  为了提高传输效率，服务器对实体进行编码，客户端进行解码复原。

  常用的编码格式：gzip（GNU zip）、compress（UNIX系统的标准压缩）、deflate（zlib）、identity（不编码）

* 分割发送 Chuncked transferred coding

  实体的最后一块会用“0（CR+LF）”来标记

### 3.3 发送多种数据的多部分对象集合

发送邮件时可以添加多种类型的附件，是因为采用的MIME扩展机制。
HTTP中也采纳了多部分对象集合，发送的一份报文主体可以含有多类型实体。通常是在图片或文本的上传中使用。`RFC2046`

* 多部分对象集合包含的对象如下：

  * multipart/form-data Web表单文件上传时使用
  * multipart/byteranges 状态码为206的响应报文中包含多个范围的内容时使用

```
Content-Type: multipart/form-data; boundary=AaB03x

--AaB03x
Content-Disposition: form-data; name="field1"

Joe Blow
--AaB03x
Content-Disposition: form-data; name="pics"; filename="file1.txt" Content-Type: text/plain

...(file1.txt的数据)... 
--AaB03x--
```

  这里boundary是划分各类实体的边界标识符，结束时在标识符后加`--`

### 3.4 获取部分内容的范围请求（中断恢复下载）

实现该功能需要指定下载的实体范围。用首部字段Range指定资源的byte范围。指定形式如下

```
Range: bytes=5001-10000
----
Range: bytes=-3000, 5000-7000
```

对于多范围要求的请求报文，响应返回状态码为206的报文，响应报文中标明multipart/byteranges。

### 3.5 内容协商

内容协商机制是指客户端和服务端就响应的资源内容进行交涉，返回最合适的资源。判断基准为请求报文中部分首部字段：Accept, Accept-Charset, Accept-Encoding, Accept-Language, Content-Language

内容协商技术有三钟类型：服务器驱动、客户端驱动和透明协商

# 第七章 确保Web安全的HTTPS

### 7.1 HTTP安全性问题：

* 通信使用明文可能会被窃听
  * TCP/IP是可能被窃听的网络
  * 加密处理防止被窃听
    * 通信的加密：SSL/TLS建立安全通信线路
    * 内容的加密：不同于 SSL 或 TLS 将整个通信线路加密 处理，所以内容仍有被篡改的风险。
* 不验证通信方身份就可能遭遇伪装 -> 证书
* 无法证明报文完整性
  * 中间人攻击（MITM, man-in-the-middle）

### 7.2 HTTPS = HTTP+加密+认证+完整性保护

<font color=#CD6155 >**HTTPS不是应用层的新协议，只是HTTP通信接口部分用SSL和TLS协议代替**</font>。通常情况下HTTP直接和TCP通信，使用SSL是，则先和SSL通信，在由SSL和TCP通信。其他应用层协议都可以配合SSL使用。

- 对称加密：只要拿到密钥，任何人都能破解密码，无法保证安全的转交密钥。

  非对称加密：发送者用对方的公钥加密，接受方用自己的私钥解密。公钥可以公开，处理速度慢。

* HTTPS采用<font color=#CD6155>**对称和非对称加密算法并用**</font>的加密机制。交换秘钥环节使用公开密钥加密方式，安全交换共享密钥后，通信采用共享密钥加密（高效）
* 如何证明服务器发送来的公钥就是靠谱的，没有被替换的 -> 数字证书认证机构（CA）
  1. 服务器把自己的公开密钥登录至数字证书认证机构CA
  2. CA用自己的私钥向服务器的公开密码署数字签名并颁发公钥证书
  3. 客户端拿到服务器的公钥证书后，使用数字证书认证机构的公开密钥，向数字证书认证机构验证公钥证书上的数字签名，以确认服务器的公开密钥的真实性
  4. 使用服务器的公开密钥对报文加密后发送

#### HTTPS通信机制

- 详细过程

  >步骤 **1**: 客户端通过发送 Client Hello 报文开始 SSL 通信。报文中包 含客户端支持的 SSL 的指定版本、加密组件(Cipher Suite)列表(所 使用的加密算法及密钥长度等)。
  >
  >步骤 **2**: 服务器可进行 SSL 通信时，会以 Server Hello 报文作为应答。和客户端一样，在报文中包含 SSL 版本以及加密组件。服务器的 加密组件内容是从接收到的客户端加密组件内筛选出来的。
  >
  >步骤  **3**: 之后服务器发送 Certificate 报文。报文中包含公开密钥证 书。
  >
  >步骤  **4**: 最后服务器发送 Server Hello Done 报文通知客户端，最初阶段的 SSL 握手协商部分结束。
  >
  >步骤 **5**: SSL 第一次握手结束之后，客户端以 Client Key Exchange 报文作为回应。报文中包含通信加密中使用的一种被称为 Pre-master secret 的随机密码串。该报文已用步骤 3 中的公开密钥进行加密。
  >
  >步骤 **6**: 接着客户端继续发送 Change Cipher Spec 报文。该报文会提 示服务器，在此报文之后的通信会采用 Pre-master secret 密钥加密。
  >
  >步骤 **7**: 客户端发送 Finished 报文。该报文包含连接至今全部报文的 整体校验值。这次握手协商是否能够成功，要以服务器是否能够正确 解密该报文作为判定标准。
  >
  >步骤 **8**: 服务器同样发送 Change Cipher Spec 报文。 步骤 **9**: 服务器同样发送 Finished 报文。
  >
  >步骤 **10**: 服务器和客户端的 Finished 报文交换完毕之后，SSL 连接就算建立完成。当然，通信会受到 SSL 的保护。从此处开始进行应用 层协议的通信，即发送 HTTP 请求。
  >
  >步骤 **11**: 应用层协议通信，即发送 HTTP 响应。
  >
  >步骤 **12**: 最后由客户端断开连接。断开连接时，发送 close_notify 报文。上图做了一些省略，这步之后再发送 TCP FIN 报文来关闭与 TCP 的通信。

![image-20210105104233304](/images/《图解HTTP》笔记/image-20210105104233304.png)

- SSL和TLS

  当前主流版本为SSL 3.0 和TLS1.0



# 第八章 确认访问用户身份的认证

### 8.1 BASIC认证

服务器收到客户端请求后，返回401告知用户需要进行认证；客户端将ID和密码发送给服务器，具体内容为：ID和psd由：连接，经过base64编码，写入首部字段authorization。

**HTTP/1.0，非加密模式，任何人获取到编码字符串不需要附加信息即可解码，安全性极差。**

### 8.2 DIGEST认证

<img src="/images/《图解HTTP》笔记/image-20210106095159217.png" alt="image-20210106095159217" style="zoom: 67%;" />

<center style="color:#C0C0C0;text-decoration:underline">DIGEST认证步骤</center>

**basic和digest的区别：**

- basic将”ID:psd“打包并采用base64编码（可直解码）

- digest**不以明文发送密码**，服务器响应返回随机字符串nonce，客户端发送响应摘要HA1=MD5(username:realm:password),HA2=MD5(method:digestURI)
  使用MD5加密达成不可逆。

参数细节参考 https://www.jianshu.com/p/18fb07f2f65e

>username: 用户名（网站定义）
> password: 密码
> realm:　服务器返回的realm,一般是域名
> method: 请求的方法
> nonce: 服务器发给客户端的随机的字符串
> nc(nonceCount):请求的次数，用于标记，计数，防止重放攻击
> cnonce(clinetNonce): 客户端发送给服务器的随机字符串
> qop: 保护质量参数,一般是auth,或auth-int,这会影响摘要的算法
> uri: 请求的uri(只是ｐａｔｈ)
> response:　客户端根据算法算出的摘要值
>
>
>**digest的算法：**
>
>A1 = username:realm:password
>A2 = mthod:uri
>
>HA1 = MD5(A1)
>如果 qop 值为“auth”或未指定，那么 HA2 为
>HA2 = MD5(A2)=MD5(method:uri)
>如果 qop 值为“auth-int”，那么 HA2 为
>HA2 = MD5(A2)=MD5(method:uri:MD5(entityBody))
>
>如果 qop 值为“auth”或“auth-int”，那么如下计算 response：
>response = MD5(HA1:nonce:nc:cnonce:qop:HA2)
>
>如果 qop 未指定，那么如下计算 response：
>response = MD5(HA1:nonce:HA2)
>上面的算法，是不是把你绕晕了，下面，用实例介绍一下，便于你的理解



python实现：

```python
from requests.auth import HTTPDigestAuth
url = 'http://httpbin.org/digest-auth/auth/user/pass'
response = requests.get(url, auth=HTTPDigestAuth('user', 'pass'))```
# 上面获取请求的内容，可以使用response.content，response.text等等
```

### 8.3 SSL客户端认证

用户ID和密码被盗的情况下，basic和digest失效，需要通过客户端认证阻止冒用行为。

认证方式：双因素认证：SSL客户端证书 +表单认证

为达到SSL客户端认证的目的，需要事先将客户端证书分发给客户端，且客户端必须安装此证书。

步骤一：接收到需要认证资源的请求，服务器会发送Certificate Request 报文，要求客户端提供客户端证书。

步骤二：用户选择将发送的客户端证书后，客户端会把客户端证书信息已Client Certificate报文方式发送个服务器。

步骤三：服务器验证客户端证书验证通过后方可领取证书内容客户端的公开密钥，然后开始HTTPS加密通信。

### 8.4 基于表单认证

**不是HTTP协议中定义的**，但因为方便灵活使用较为广泛。用户根据web应用程序的实际要求的用户界面及认证方式提供认证信息。

* **Session管理及Cookie应用**

  因为HTTP的无状态特性，基于表单认证过的状态无法保留。所以使用Cookie来管理会话（Session）。Cookie中包含用于识别用户的SessionID，和用户的认证状态绑定后被记录在服务器端。SessionID一般是难以推测的字符串

> <font color=#CD6155>**Q**</font>**一个疑问**
>
> 原文中提到：
>
> > 如果 Session ID 被第三方盗走，对方就可以伪装成你的身份进 行恶意操作了。因此必须防止 Session ID 被盗，或被猜出。为了做到 这点，Session ID 应使用难以推测的字符串，且服务器端也需要进行 有效期的管理，保证其安全性。
>
> 但是TLS中client hello报文中session id字段不是可以直接从流量中获取吗？
>
> 详细内容待了解

# 第九章 基于HTTP的功能追加协议

### 9.2 消除HTTP瓶颈的SPDY

SPDY（发音speedy），google，2010， 解决性能瓶颈，缩短页面加载时间 http://www.chromium.org/spdy/

- 一条连接上只可发送一个请求
- 请求只能从客户端开始
- 首部未经压缩就发送，首部信息越多延迟越大
- 互相发送相同的首部浪费资源
- 可任意选择数据压缩格式。非强制压缩发送。

解决：

- Ajax（Asynchronous javascript and xml，异步js和xml技术）
  - 有效利用js和DOM（document object model）达到局部页面替换加载的异步通信手段。
  - 核心技术是名为XMLHttpRequest的API，通过js脚本调用能和服务器进行http通信，从已加载完毕的web页面上发起请求，只更新局部页面。
  - 缺点：会产生大量重复请求，未解决http本身问题
- Comet
  - 收到更新请求，先将响应至于挂起状态，一旦sever由内容更新，直接对客户端返回响应。这是一种通过**延时应答**，模拟服务器推送的功能。
  - 做到了实时更新，但连接持续时间边长，消耗资源，且未解决http问题

**SPDY**

- 在TCP/IP的应用层和传输层之间加会话层。同时规定使用SSL

  <img src="/images/《图解HTTP》笔记/image-20210106115743469.png" alt="image-20210106115743469" style="zoom: 67%;" />

- 功能：

  - 多路复用流
  - 赋予请求优先级
  - 压缩首部
  - 推送功能，服务器主动向用户推送
  - 服务器提示功能：主动提示客户端所需的资源

### 9.3 浏览器-服务器全双工通信协议WebSocket

WebSocket是Web浏览器和Web服务器之间全双工通信标准。由于是建立在HTTP基础上的协议，发起方仍是客户端。一旦连接诶连接建立，通信过程中可互相发送任意格式的数据。

**主要特点：**

- 推送功能

- 减少通信量

  - 为了实现WebSocket通信，**在HTTP连接建立之后**，需要完成一次握手。

    - 握手-请求

      HTTP的upgrade首部字段，用户告知协议发生变化

      ![image-20210106140420372](/images/《图解HTTP》笔记/image-20210106140420372.png)

    - 握手-响应

      对于之前的请求，返回状态吗101 switching protocols响应

      ![image-20210106140602140](/images/《图解HTTP》笔记/image-20210106140602140.png)、

    成功握手后，确立websocket连接，通信时不再使用HTTP的数据帧，采用WebSocket独立的数据帧。

**WebSocket API**

> JavaScript 可调用“The WebSocket API”(http://www.w3.org/TR/websockets/，由 W3C 标准制定)内 提供的 WebSocket 程序接口，以实现 WebSocket 协议下全双工通 信。
>
> 以下为调用 WebSocket API，每 50ms 发送一次数据的实例。
>
> ```javascript
> var socket = new WebSocket('ws://game.example.com:12010')
> socket.onopen = function () {
>   setInterval(function() {
>     if (socket.bufferedAmount == 0)
>       socket.send(getUpdateData()); }, 50);
> };
> ```

### 9.4 HTTP2

单独整理

# 第十章 构建Web内容的技术

- DOM

- Web应用

- CGI

- XML

- Servlet

- HTML & CSS & JavaScript

  **HTML** 定义了网页的内容；**CSS** 描述了网页的布局；**JavaScript** 网页的行为

  HTML定义了一套语法规则，来告诉浏览器如何把一个丰富多彩的页面显示出来。网页中不但包含文字，还有图片、视频、Flash小游戏，有复杂的排版、动画效果。HTML文档就是一系列的Tag组成，最外层的Tag是`<html>`。规范的HTML也包含`<head>...</head>`和`<body>...</body>`（注意不要和HTTP的Header、Body搞混了），由于HTML是富文档模型，所以，还有一系列的Tag用来表示链接、图片、表格、表单等等。

  ```html
  <html>
  <head>
    <title>Hello</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
  </body>
  </html>
  ```

  CSS是Cascading Style Sheets（层叠样式表）的简称，CSS用来控制HTML里的所有元素如何展现，比如，给标题元素`<h1>`加一个样式，变成48号字体，灰色，带阴影：

  ```html
  <html>
  <head>
    <title>Hello</title>
    <style>
      h1 {
        color: #333333;
        font-size: 48px;
        text-shadow: 3px 3px 3px #666666;
      }
    </style>
  </head>
  <body>
    <h1>Hello, world!</h1>
  </body>
  </html>
  ```

  JavaScript虽然名称有个Java，但它和Java没有关系。JavaScript是为了让HTML具有交互性而作为脚本语言添加的，JavaScript既可以内嵌到HTML中，也可以从外部链接到HTML中。如果我们希望当用户点击标题时把标题变成红色，就必须通过JavaScript来实现。

  ```html
  <html>
  <head>
    <title>Hello</title>
    <style>
      h1 {
        color: #333333;
        font-size: 48px;
        text-shadow: 3px 3px 3px #666666;
      }
    </style>
    <script>
      function change() {
        document.getElementsByTagName('h1')[0].style.color = '#ff0000';
      }
    </script>
  </head>
  <body>
    <h1 onclick="change()">Hello, world!</h1>
  </body>
  </html>
  ```

# 第十章 Web的攻击技术

### 10.1 针对Web的攻击技术

- 在客户端篡改请求。

  在http请求报文内加载攻击代码，就能发起对web应用的攻击。通过URL查询字段或表单、HTTP首部、Cookie等途径

- 针对web应用的攻击模式

  - 主动攻击：攻击者通过直接访问web应用，把攻击代码传入，针对服务器上的资源。如<font color=CD6155>**SQL注入和OS命令注入**</font>

  - 被动攻击：

    利用圈套策略执行攻击代码。攻击者不直接对目标web应用访问发起攻击。

    <img src="/images/《图解HTTP》笔记/image-20210113104843747.png" alt="image-20210113104843747" style="zoom:80%;" />

    <img src="/images/《图解HTTP》笔记/image-20210113104905000.png" alt="image-20210113104905000" style="zoom: 67%;" />

### 10.2 因输出值转义不完全引发的安全漏洞

- 跨站脚本攻击XSS

  web网站注册用户的浏览器内，运行非法的HTML标签或JavaScript进行的一种攻击。

  

- SQL注入攻击

  SQL 注入是攻击者将 SQL 语句改变成开发者意想不到的形式以 达到破坏结构的攻击。

  <img src="/images/《图解HTTP》笔记/image-20210113111432351.png" alt="image-20210113111432351" style="zoom:67%;" />

- OS命令注入

  类似Sql注入，通过web应用，实行非法的操作系统命令。

- HTTP首部注入

  通过在某些响应首部字段需要处理输出 值的地方，插入换行发动攻击。

- 邮件首部注入

- 目录遍历攻击

- 远程文件包含漏洞

### 10.3 因设置或设计上的缺陷引发的安全漏洞

- 强制浏览
- 不正确的错误消息处理
- 开放重定向

### 10.4 因会话管理疏忽引发的安全漏洞

- 会话劫持。攻击者获得会话id，伪装用户进行会话
- 会话固定攻击
- 跨站点请求伪造

 ### 10.5 其他安全漏洞

- 加密漏洞
- 点击劫持
- DoS攻击/DDos攻击
- 后门程序

# TODO

* RESTFUL标准
* Cross-site Tracking攻击
* 代理&网关&隧道
* http会话管理

