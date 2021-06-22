---
title: 不学废之——DNS篇
typora-root-url: ../../source
date: 2021-06-21 16:46:26
tags:
catagories:
sticky:

---

### 参考

阮一峰 http://www.ruanyifeng.com/blog/2016/06/dns.html

## DNS 

DNS （Domain Name System 的缩写）的作用非常简单，就是根据域名查出IP地址。你可以把它想象成一本巨大的电话本。

举例来说，如果你要访问域名`math.stackexchange.com`，首先要通过DNS查出它的IP地址是`151.101.129.69`。

### 解析过程

#### 递归查询和迭代查询

困惑已久的递归查询和迭代查询，看一次忘一次。

- 字面理解：递归和迭代的区别。递归被用来描述以自相似方法重复事务的过程，在方法上显示为A调用A；迭代 重复反馈过程，每次迭代的结果为下一次迭代的初始值，在方法上显示为A调用B。
- 在DNS查询过程中
  - 递归查询是只发出一次请求，请求方逐级查询，查询结果逐级返回。如本地主机向本地域名服务器查询，有则返回，没有则请求设置的转发器或根域名服务器，有则返回至本地域名服务器，没有则查询下一级……
  - 迭代查询是发出多次请求，每次请求结束后由请求方进行下一轮请求。如本地主机请求本地域名服务器，如果未能解析，由本地主机向根域名服务器发起请求。
  - **实际网络中，一般采用两段式查询。由本机到本地服务器采用递归查询，由本地服务器到最终结果采用迭代查询。**

#### 具体解析流程

场景：用户在浏览器输入网址：clondant.blog.51cto.com，其解析过程如下：

**第1步：**浏览器将会检查缓存中有没有这个域名对应的解析过的IP地址，如果有该解析过程将会结束。

**第2步：**如果用户的浏览器中缓存中没有，操作系统会先检查自己本地的hosts文件是否有这个网址映射关系，如果有，就先调用这个IP地址映射关系，完成域名解析。

**第3步：**如果hosts里没有这个域名的映射，则查找本地DNS解析器缓存，是否有这个网址映射关系或缓存信息，如果有，直接返回给浏览器，完成域名解析。

**第4步：**如果hosts与本地DNS解析器缓存都没有相应的网址映射关系，则会首先找本地DNS服务器，一般是公司内部的DNS服务器，此服务器收到查询，如果此本地DNS服务器查询到相对应的IP地址映射或者缓存信息，则返回解析结果给客户机，完成域名解析，此解析具有权威性。

**第5步：**如果本地DNS服务器无法查询到，则根据本地DNS服务器设置的转发器进行查询；

**未用转发模式**：**本地DNS就把请求发至根DNS进行**（迭代）查询**，根DNS服务器收到请求后会判断这个域名(.com)是谁来授权管理，并会返回一个负责该顶级域名服务器的一个IP。本地DNS服务器收 到IP信息后，将会联系负责.com域的这台服务器。这台负责.com域的服务器收到请求后，如果自己无法解析，它就会找一个管理.com域的下一级 DNS服务器地址给本地DNS服务器。当本地DNS服务器收到这个地址后，就会找域名域服务器，重复上面的动作，进行查询，直至找到域名对应的主机。

**使用转发模式：**此DNS服务器就会把请求转发至上一级DNS服务器，由上一级服务器进行解析，上一级服务器如果不能解析，或找根DNS或把转请求转至 上上级，以此循环。不管是本地DNS服务器用是是转发，还是根提示，最后都是把结果返回给本地DNS服务器，由此DNS服务器再返回给客户机。

#### dig命令查看解析流程

工具软件`dig`可以显示整个查询过程。

> ```bash
> $ dig math.stackexchange.com
> ```

上面的命令会输出六段信息。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061501.png)

第一段是查询参数和统计。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061502.png)

第二段是查询内容。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061503.png)

上面结果表示，查询域名`math.stackexchange.com`的`A`记录，`A`是address的缩写。

第三段是DNS服务器的答复。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061504.png)

上面结果显示，`math.stackexchange.com`有四个`A`记录，即四个IP地址。`600`是TTL值（Time to live 的缩写），表示缓存时间，即600秒之内不用重新查询，IN代表IP协议，及Internet。

第四段显示`stackexchange.com`的NS记录（Name Server的缩写），即哪些服务器负责管理`stackexchange.com`的DNS记录。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061505.png)

上面结果显示`stackexchange.com`共有四条NS记录，即四个域名服务器，向其中任一台查询就能知道`math.stackexchange.com`的IP地址是什么。

第五段是上面四个域名服务器的IP地址，这是随着前一段一起返回的。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061506.png)

第六段是DNS服务器的一些传输信息。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061514.png)

上面结果显示，本机的DNS服务器是`192.168.1.253`，查询端口是53（DNS服务器的默认端口），以及回应长度是305字节。

如果不想看到这么多内容，可以使用`+short`参数。

> ```bash
> $ dig +short math.stackexchange.com
> 
> 151.101.129.69
> 151.101.65.69
> 151.101.193.69
> 151.101.1.69
> ```

上面命令只返回`math.stackexchange.com`对应的4个IP地址（即`A`记录）。

### DNS服务器

下面我们根据前面这个例子，一步步还原，本机到底怎么得到域名`math.stackexchange.com`的IP地址。

首先，本机一定要知道DNS服务器的IP地址，否则上不了网。通过DNS服务器，才能知道某个域名的IP地址到底是什么。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061507.jpg)

DNS服务器的IP地址，有可能是动态的，每次上网时由网关分配，这叫做DHCP机制；也有可能是事先指定的固定地址。Linux系统里面，DNS服务器的IP地址保存在`/etc/resolv.conf`文件。

上例的DNS服务器是`192.168.1.253`，这是一个内网地址。有一些公网的DNS服务器，也可以使用，其中最有名的就是Google的[`8.8.8.8`](https://developers.google.com/speed/public-dns/)和Level 3的[`4.2.2.2`](https://www.tummy.com/articles/famous-dns-server/)。

本机只向自己的DNS服务器查询，`dig`命令有一个`@`参数，显示向其他DNS服务器查询的结果。

> ```bash
> $ dig @4.2.2.2 math.stackexchange.com
> ```

上面命令指定向DNS服务器`4.2.2.2`查询。

### 域名的层级

DNS服务器怎么会知道每个域名的IP地址呢？答案是分级查询。

请仔细看前面的例子，每个域名的尾部都多了一个点。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061503.png)

比如，域名`math.stackexchange.com`显示为`math.stackexchange.com.`。**这不是疏忽，而是所有域名的尾部，实际上都有一个根域名。**

**举例来说，`www.example.com`真正的域名是`www.example.com.root`，简写为`www.example.com.`**。因为，根域名`.root`对于所有域名都是一样的，所以平时是省略的。

根域名的下一级，叫做"顶级域名"（top-level domain，缩写为TLD），比如`.com`、`.net`；再下一级叫做"次级域名"（second-level domain，缩写为SLD），比如`www.example.com`里面的`.example`，这一级域名是用户可以注册的；再下一级是主机名（host），比如`www.example.com`里面的`www`，又称为"三级域名"，这是用户在自己的域里面为服务器分配的名称，是用户可以任意分配的。

总结一下，域名的层级结构如下。

> ```bash
> 主机名.次级域名.顶级域名.根域名
> 
> # 即
> 
> host.sld.tld.root
> ```

### 根域名服务器

DNS服务器根据域名的层级，进行分级查询。

需要明确的是，每一级域名都有自己的NS记录，NS记录指向该级域名的域名服务器。这些服务器知道下一级域名的各种记录。

所谓"分级查询"，就是从根域名开始，依次查询每一级域名的NS记录，直到查到最终的IP地址，过程大致如下。

> 1. 从"根域名服务器"查到"顶级域名服务器"的NS记录和A记录（IP地址）
> 2. 从"顶级域名服务器"查到"次级域名服务器"的NS记录和A记录（IP地址）
> 3. 从"次级域名服务器"查出"主机名"的IP地址

仔细看上面的过程，你可能发现了，没有提到DNS服务器怎么知道"根域名服务器"的IP地址。回答是"根域名服务器"的NS记录和IP地址一般是不会变化的，所以内置在DNS服务器里面。

下面是内置的根域名服务器IP地址的一个[例子](http://www.cyberciti.biz/faq/unix-linux-update-root-hints-data-file/)。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061508.png)

上面列表中，列出了根域名（`.root`）的三条NS记录`A.ROOT-SERVERS.NET`、`B.ROOT-SERVERS.NET`和`C.ROOT-SERVERS.NET`，以及它们的IP地址（即`A`记录）`198.41.0.4`、`192.228.79.201`、`192.33.4.12`。

另外，可以看到所有记录的TTL值是3600000秒，相当于1000小时。也就是说，每1000小时才查询一次根域名服务器的列表。

目前，世界上一共有十三组根域名服务器，从`A.ROOT-SERVERS.NET`一直到`M.ROOT-SERVERS.NET`。

### 分级查询的实例

`dig`命令的`+trace`参数可以显示DNS的整个分级查询过程。

> ```bash
> $ dig +trace math.stackexchange.com
> ```

上面命令的第一段列出根域名`.`的所有NS记录，即所有根域名服务器。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061509.png)

根据内置的根域名服务器IP地址，DNS服务器向所有这些IP地址发出查询请求，询问`math.stackexchange.com`的顶级域名服务器`com.`的NS记录。最先回复的根域名服务器将被缓存，以后只向这台服务器发请求。

接着是第二段。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061510.png)

上面结果显示`.com`域名的13条NS记录，同时返回的还有每一条记录对应的IP地址。

然后，DNS服务器向这些顶级域名服务器发出查询请求，询问`math.stackexchange.com`的次级域名`stackexchange.com`的NS记录。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061511.png)

上面结果显示`stackexchange.com`有四条NS记录，同时返回的还有每一条NS记录对应的IP地址。

然后，DNS服务器向上面这四台NS服务器查询`math.stackexchange.com`的主机名。

![img](/images/%E4%B8%8D%E5%AD%A6%E5%BA%9F%E4%B9%8B%E2%80%94%E2%80%94DNS%E7%AF%87/bg2016061512.png)

上面结果显示，`math.stackexchange.com`有4条`A`记录，即这四个IP地址都可以访问到网站。并且还显示，最先返回结果的NS服务器是`ns-463.awsdns-57.com`，IP地址为`205.251.193.207`。

### NS 记录的查询

`dig`命令可以单独查看每一级域名的NS记录。

> ```bash
> $ dig ns com
> $ dig ns stackexchange.com
> ```

`+short`参数可以显示简化的结果。

> ```bash
> $ dig +short ns com
> $ dig +short ns stackexchange.com
> ```

### DNS的记录类型

域名与IP之间的对应关系，称为"记录"（record）。根据使用场景，"记录"可以分成不同的类型（type），前面已经看到了有`A`记录和`NS`记录。

常见的DNS记录类型如下。

> （1） `A`：地址记录（Address），返回域名指向的IP地址。
>
> （2） `NS`：域名服务器记录（Name Server），返回保存下一级域名信息的服务器地址。该记录只能设置为域名，不能设置为IP地址。
>
> （3）`MX`：邮件记录（Mail eXchange），返回接收电子邮件的服务器地址。
>
> （4）`CNAME`：规范名称记录（Canonical Name），返回另一个域名，即当前查询的域名是另一个域名的跳转，详见下文。
>
> （5）`PTR`：逆向查询记录（Pointer Record），只用于从IP地址查询域名，详见下文。

一般来说，为了服务的安全可靠，至少应该有两条`NS`记录，而`A`记录和`MX`记录也可以有多条，这样就提供了服务的冗余性，防止出现单点失败。

`CNAME`记录主要用于域名的内部跳转，为服务器配置提供灵活性，用户感知不到。举例来说，`facebook.github.io`这个域名就是一个`CNAME`记录。

> ```bash
> $ dig facebook.github.io
> 
> ...
> 
> ;; ANSWER SECTION:
> facebook.github.io. 3370    IN  CNAME   github.map.fastly.net.
> github.map.fastly.net.  600 IN  A   103.245.222.133
> ```

上面结果显示，`facebook.github.io`的CNAME记录指向`github.map.fastly.net`。也就是说，用户查询`facebook.github.io`的时候，实际上返回的是`github.map.fastly.net`的IP地址。这样的好处是，变更服务器IP地址的时候，只要修改`github.map.fastly.net`这个域名就可以了，用户的`facebook.github.io`域名不用修改。

由于`CNAME`记录就是一个替换，所以域名一旦设置`CNAME`记录以后，就不能再设置其他记录了（比如`A`记录和`MX`记录），这是为了防止产生冲突。举例来说，`foo.com`指向`bar.com`，而两个域名各有自己的`MX`记录，如果两者不一致，就会产生问题。由于顶级域名通常要设置`MX`记录，所以一般不允许用户对顶级域名设置`CNAME`记录。

`PTR`记录用于从IP地址反查域名。`dig`命令的`-x`参数用于查询`PTR`记录。

> ```bash
> $ dig -x 192.30.252.153
> 
> ...
> 
> ;; ANSWER SECTION:
> 153.252.30.192.in-addr.arpa. 3600 IN    PTR pages.github.com.
> ```

上面结果显示，`192.30.252.153`这台服务器的域名是`pages.github.com`。

逆向查询的一个应用，是可以防止垃圾邮件，即验证发送邮件的IP地址，是否真的有它所声称的域名。

`dig`命令可以查看指定的记录类型。

> ```bash
> $ dig a github.com
> $ dig ns github.com
> $ dig mx github.com
> ```

### 其他DNS工具

除了`dig`，还有一些其他小工具也可以使用。

**（1）host 命令**

`host`命令可以看作`dig`命令的简化版本，返回当前请求域名的各种记录。

> ```bash
> $ host github.com
> 
> github.com has address 192.30.252.121
> github.com mail is handled by 5 ALT2.ASPMX.L.GOOGLE.COM.
> github.com mail is handled by 10 ALT4.ASPMX.L.GOOGLE.COM.
> github.com mail is handled by 10 ALT3.ASPMX.L.GOOGLE.COM.
> github.com mail is handled by 5 ALT1.ASPMX.L.GOOGLE.COM.
> github.com mail is handled by 1 ASPMX.L.GOOGLE.COM.
> 
> $ host facebook.github.com
> 
> facebook.github.com is an alias for github.map.fastly.net.
> github.map.fastly.net has address 103.245.222.133
> ```

`host`命令也可以用于逆向查询，即从IP地址查询域名，等同于`dig -x <ip>`。

> ```bash
> $ host 192.30.252.153
> 
> 153.252.30.192.in-addr.arpa domain name pointer pages.github.com.
> ```

**（2）nslookup 命令**

`nslookup`命令用于互动式地查询域名记录。

> ```bash
> $ nslookup
> 
> > facebook.github.io
> Server:     192.168.1.253
> Address:    192.168.1.253#53
> 
> Non-authoritative answer:
> facebook.github.io  canonical name = github.map.fastly.net.
> Name:   github.map.fastly.net
> Address: 103.245.222.133
> 
> > 
> ```

**（3）whois 命令**

`whois`命令用来查看域名的注册情况。

> ```bash
> $ whois github.com
> ```

## DNSSEC

ICANN文档 https://www.icann.org/zh/system/files/files/octo-006-24jul20-zh.pdf

### 运作方式

注册人签名

“注册人”是指有权控制域名相关信息（即，名称转换为地址的映射以及其他数据）的个人或 组织。DNSSEC 准许**注册人对他们存放在 DNS 中的信息进行数字签名**，这样一来，**客户端（例 如，您的 Web 浏览器）就能够验证它们收到的 DNS 查询请求的应答结果与签名时的内容相比是 否发生了修改。** 2010 年，ICANN 针对 DNS 的顶层（即“根”）实施 DNSSEC 签名，从而极大地推动了 DNSSEC 在全球范围内的部署。然而，即使在十年后的今天，DNSSEC 的部署仍然滞后。 

## DoH

## DoT



