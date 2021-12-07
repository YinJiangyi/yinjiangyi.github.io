---
title: DDos攻击与保护
typora-root-url: ../../source
date: 2021-07-12 14:21:43
tags: DDos
categories: 安全
sticky:
---

参考：

https://www.cloudflare.com/learning/

## 1. 什么是DDos攻击

### 工作原理

Distribution Denial-of-Service 分布式拒绝服务。

#### 如何识别

明显迹象：

- 来自单个 IP 地址或 IP 范围的可疑流量
- 来自共享单个行为特征（例如设备类型、地理位置或 Web 浏览器版本）的用户的大量流量
- 对单个页面或端点的请求数量出现不明原因的激增
- 奇怪的流量模式，例如一天中非常规时间段的激增或看似不自然的模式（例如，每 10 分钟出现一次激增）
- DDoS 攻击还有其他更具体的迹象，具体取决于攻击的类型。

### 常见攻击类别（简要介绍）

<img src="/images/DDos%E6%94%BB%E5%87%BB%E4%B8%8E%E4%BF%9D%E6%8A%A4/image-20210712180840976.png" alt="image-20210712180840976" style="zoom: 50%;" />

#### 应用层攻击

此类攻击有时称为[第 7 层](https://www.cloudflare.com/learning/ddos/what-is-layer-7/) DDoS 攻击（指 OSI 模型第 7 层），其目标是耗尽目标资源。

攻击目标是生成网页并传输网页响应HTTP请求的服务器应用层，在客户端执行HTTP请求的计算成本很低，但是目标服务器做出响应的成本很高。防御难度较大，因为难以区分恶意流量和合法流量。

<img src="/images/DDos%E6%94%BB%E5%87%BB%E4%B8%8E%E4%BF%9D%E6%8A%A4/http-flood-ddos-attack.png" alt="http-flood-ddos-attack" style="zoom:25%;" />

##### HTTP flood

HTTP 洪水攻击类似于同时在大量不同计算机的 Web 浏览器中一次又一次地按下刷新 ——大量 HTTP 请求涌向服务器，导致拒绝服务。

这种类型的攻击有简单的，也有复杂的。较简单的实现可以使用相同范围的攻击 IP 地址、referrer 和用户代理访问一个 URL。复杂版本可能使用大量 IP 地址，并使用随机 referrer 和用户代理来绕过检测。

#### 协议攻击

协议攻击也称为状态耗尽攻击，这类攻击会过度消耗服务器资源和/或[防火墙](https://www.cloudflare.com/learning/security/what-is-a-firewall/)和负载平衡器之类的网络设备资源，从而导致服务中断。

协议攻击利用协议堆栈第 3 层和第 4 层的弱点致使目标无法访问。

<img src="/images/DDos%E6%94%BB%E5%87%BB%E4%B8%8E%E4%BF%9D%E6%8A%A4/syn-flood-ddos-attack.png" alt="syn-flood-ddos-attack" style="zoom:15%;" />

##### TCP SYN攻击

 [SYN 洪水](https://www.cloudflare.com/learning/ddos/syn-flood-ddos-attack/)就好比补给室中的工作人员从商店的柜台接收请求。

工作人员收到请求，前去取包裹，再等待确认，然后将包裹送到柜台。工作人员收到太多包裹请求，但得不到确认，直到无法处理更多包裹，实在不堪重负，致使无人能对请求做出回应。

此类攻击利用 [TCP 握手](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/)（两台计算机发起网络连接时要经过的一系列通信），通过向目标发送大量带有[伪造](https://www.cloudflare.com/learning/ddos/glossary/ip-spoofing/)源 IP 地址的 TCP“初始连接请求”SYN 数据包来实现。

目标计算机响应每个连接请求，然后等待握手中的最后一步，但这一步确永远不会发生，因此在此过程中耗尽目标的资源。

#### 容量耗尽攻击

Volumetric attacks，此类攻击试图通过消耗目标与较大的互联网之间的所有可用带宽来造成拥塞。攻击运用某种放大攻击或其他生成大量流量的手段（如僵尸网络请求），向目标发送大量数据。

<img src="https://cloudflare.com/img/learning/ddos/what-is-a-ddos-attack/ntp-amplification-botnet-ddos-attack.png" alt="ntp-amplification-botnet-ddos-attack" style="zoom:20%;" />

##### DNS 放大

[DNS 放大](https://www.cloudflare.com/learning/ddos/dns-amplification-ddos-attack/)就好比有人打电话给餐馆说“每道菜都订一份，请给我回电话复述整个订单”，而提供的回电号码实际上属于受害者。几乎不费吹灰之力，就能产生很长的响应并发送给受害者。

利用伪造的 IP 地址（受害者的 IP 地址）向开放式 [DNS 服务器](https://www.cloudflare.com/learning/ddos/glossary/domain-name-system-dns/)发出请求后，目标 IP 地址将收到服务器发回的响应。

### 常见防御手段

难点在于区分真实客户流量与攻击流量。

#### 黑洞路由

有一种解决方案几乎适用于所有网络管理员：创建[黑洞](https://www.cloudflare.com/learning/ddos/glossary/ddos-blackhole-routing)路由，并将流量汇入该路由。在最简单的形式下，当在没有特定限制条件的情况下实施黑洞过滤时，合法网络流量和恶意网络流量都将路由到空路由或黑洞，并从网络中丢弃。

如果互联网设备遭受 DDoS 攻击，则该设备的互联网服务提供商（ISP）可能会将站点的所有流量发送到黑洞中作为防御。这不是理想的解决方案，因为它相当于让攻击者达成预期的目标：使网络无法访问。

#### 速率限制

限制服务器在某个时间段接收的请求数量也是防护拒绝服务攻击的一种方法。虽然速率限制对于减缓 Web 爬虫窃取内容及防护[暴力破解](https://www.cloudflare.com/learning/bots/brute-force-attack/)攻击很有帮助，但仅靠速率限制可能不足以有效应对复杂的 DDoS 攻击。然而，在高效 DDoS 防护策略中，速率限制不失为一种有效手段。

#### Web 应用程序防火墙

[Web 应用程序防火墙（WAF）](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/) 是一种有效工具，有助于缓解第 7 层 DDoS 攻击。在互联网和源站之间部署 WAF 后，WAF 可以充当[反向代理](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/)，保护目标服务器，防止其遭受特定类型的恶意流量入侵。

通过基于一系列用于识别 DDoS 工具的规则筛选请求，可以阻止第 7 层攻击。有效的 WAF 的一个关键价值是能够快速实施自定义规则以应对攻击。

#### Anycast 网络扩散

此类缓解方法使用 Anycast 网络，将攻击流量分散至分布式服务器网络，直到网络吸收流量为止。

这种方法就好比将湍急的河流引入若干独立的小水渠，将分布式攻击流量的影响分散到可以管理的程度，从而分散破坏力。

[Anycast 网络](https://www.cloudflare.com/learning/cdn/glossary/anycast-network/)在缓解 DDoS 攻击方面的可靠性取决于攻击规模及网络规模和效率。采用 Anycast 分布式网络是 Cloudflare 实施 DDoS防护策略的一个重要组成部分。

Cloudflare 拥有 67 Tbps 的网络，比有记录的最大 DDoS 攻击大一个数量级。

## 2. 常见攻击类别

### 2.1 内存缓存 DDoS 攻击

memcached DDoS attack

内存缓存[分布式拒绝服务 (DDoS) ](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/)攻击是一种网络攻击，攻击者试图使目标受害者的网络流量超载。攻击者将欺骗性的请求发送到易受攻击的UDP内存缓存服务器，该服务器随后向目标受害者发送 Internet 流量，从而可能使受害者的资源不堪重负。当目标的 Internet 基础设施过载时，就无法处理新请求，而常规流量也无法访问Internet 资源，从而导致[拒绝服务](https://www.cloudflare.com/learning/ddos/glossary/denial-of-service/)。

内存缓存是用于加速网站和网络的数据库缓存系统。

#### 攻击原理

内存缓存攻击的工作方式类似于所有 DDoS 放大攻击，例如[ NTP 放大](https://www.cloudflare.com/learning/ddos/ntp-amplification-ddos-attack/)和[ DNS 放大](https://www.cloudflare.com/learning/ddos/dns-amplification-ddos-attack/)。这种攻击欺骗性请求发送到易受攻击的服务器，该服务器随后会发出比初始请求大的数据量作为响应，因此放大了流量。

内存缓存放大攻击就好比是一个心怀恶意的青少年打电话给一家餐厅说“我要菜单上的东西每样来一份，请给我回电话并告诉我整个订单的信息”。当餐厅询问回叫号码时，他却给出目标受害者的电话号码。然后，目标会收到来自餐厅的呼叫，接到他们未请求的大量信息。

**这种放大攻击的方法之所以成为可能，因为内存缓存服务器可以选择使用 UDP 协议进行操作。UDP允许在不首先获得所谓握手的情况下发送数据 - 握手是指双方都同意通信的网络过程。之所以使用 UDP，是因为不用咨询目标主机是否愿意接收数据，无需事先征得它们的同意，就可以将大量数据发送给目标主机。**

**内存缓存攻击分为 4 个步骤：**

1. 攻击者将大量数据有效载荷*植入暴露的内存缓存服务器上。
2. 接下来，攻击者使用目标受害者的IP 地址伪造HTTP GET请求。
3. 带有漏洞的内存缓存服务器接收到请求，试图通过响应来提供帮助，因此将大量响应发送到目标。
4. 目标服务器或其周围的基础设施无法处理从内存缓存服务器发送的大量数据，因此导致过载和对正常请求拒绝服务。

**内存缓存放大攻击的规模可以达到多大？**

这种攻击的放大倍数十分惊人；在实践中，我们见过高达 51,200 倍的放大倍数！这意味着对于 15 字节的请求，可以发送 750 kB 的响应。这是一个巨大的放大倍数，无法承受如此大量攻击流量的 Web 资产则面临巨大的安全风险。巨大的放大倍数加上带有漏洞的服务器使内存缓存放大攻击成为攻击者针对各种目标发起 DDoS 攻击的主要用例。

#### 如何防护

1. **禁用 UDP** - 对于内存缓存服务器，请确保在不需要时禁用 UDP 支持。默认情况下，内存缓存启用了 UDP 支持，这可能会使服务器容易受到攻击。
2. **对内存缓存服务器进行防火墙保护** - 通过在内存缓存服务器和 Internet 之间添加[防火墙](https://www.cloudflare.com/learning/security/what-is-a-firewall/)保护，系统管理员可以根据需要使用 UDP，而不必暴露于风险中。
3. **防止 IP 欺骗** - 只要可以伪造 IP 地址，DDoS 攻击就可以利用此漏洞将流量定向到受害者的网络。防止 IP 欺骗是一个规模较大的解决方案，无法由特定的系统管理员实施，它要求传输提供商禁止源 IP 地址源自网络外部的任何数据包离开其网络。换句话说，Internet 服务提供商 (ISP) 之类的公司必须筛选流量，以使离开其网络的数据包不得假装成来自其他地方的其他网络。如果所有主要的传输提供商都实施了这种筛选，基于欺骗的攻击将在一夜之间消失。
4. **开发具有减少 UDP 响应的软件** - 消除放大攻击的另一种方法是**去除任何传入请求的放大因素**；如果由于 UDP 请求而发送的响应数据小于或等于初始请求，则放大就不复可能。

Cloudflare 在网络边缘筛选 UDP 流量，消除了诸如此类的放大攻击带来的风险。探索 Cloudflare 的[高级 DDoS ](https://www.cloudflare.com/ddos/)防护。

要更深入地了解 Cloudflare 遇到内存缓存攻击的情况以及用于防护的特定命令和过程，请浏览博客文章[内存缓存 - 来自 UDP 端口 11211 ](https://blog.cloudflare.com/memcrashed-major-amplification-attacks-from-port-11211/)的主要放大攻击。

### 2.2 SYN Flood攻击

#### 攻击原理

SYN 洪水攻击利用 [TCP](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/) 连接的握手过程发动攻击。正常情况下，TCP 连接将完成三次握手以建立连接。

1. 首先，客户端向服务器发送 SYN 数据包以发起连接。
2. 接着，服务器通过 SYN/ACK 数据包对该初始数据包做出响应，以便确认通信。
3. 最后，客户端返回 ACK 数据包以确认接到服务器发出的数据包。完成这一系列数据包发送和接收操作后，TCP 连接将处于打开状态并且能够发送和接收数据。

<img src="/images/DDos%E6%94%BB%E5%87%BB%E4%B8%8E%E4%BF%9D%E6%8A%A4/syn-flood-attack-ddos-attack-diagram-1.png" alt="syn-flood-attack-ddos-attack-diagram-1" style="zoom: 33%;" />

为发起[拒绝服务](https://www.cloudflare.com/learning/ddos/glossary/denial-of-service/)攻击，攻击者需利用这样一项事实：收到初始 SYN 数据包后，服务器将通过一个或多个 SYN/ACK 数据包做出回响，等待完成握手过程的最后一步。工作方式如下：

1. 攻击者通常使用[伪造的](https://www.cloudflare.com/learning/ddos/glossary/ip-spoofing/) IP 地址向目标服务器发送大量 SYN 数据包。
2. 然后，服务器分别对每一项连接请求做出响应，并确保打开的端口做好接收响应的准备。
3. 在服务器等待最后一个 ACK 数据包（永远不会到达）的过程中，攻击者将继续发送更多 SYN 数据包。每当有新的 SYN 数据包到达，服务器都会临时打开一个新的端口并在一段特定时间内保持连接；用遍所有可用端口后，服务器将无法正常运行。

<img src="/images/DDos%E6%94%BB%E5%87%BB%E4%B8%8E%E4%BF%9D%E6%8A%A4/syn-flood-attack-ddos-attack-diagram-2.png" alt="syn-flood-attack-ddos-attack-diagram-2" style="zoom:25%;" />

在网络中，如果服务器连接处于打开状态但另一端的机器连接未打开，则视为半开连接。在此类 DDoS 攻击中，目标服务器将使连接一直处于打开状态，静待各个连接超时，避免再次开放端口。因此，此类攻击可视为**“半开连接攻击”**。



**恶意用户可通过三种不同方式发起 SYN 洪水攻击：**

1. **直接攻击：**不伪造 [IP 地址](https://www.cloudflare.com/learning/dns/glossary/what-is-my-ip-address/)的 SYN 洪水攻击称为直接攻击。在此类攻击中，攻击者完全不屏蔽其 IP 地址。由于攻击者使用具有真实 IP 地址的单一源设备发起攻击，因此很容易发现并清理攻击者。为使目标机器呈现半开状态，黑客将阻止个人机器对服务器的 SYN-ACK 数据包做出响应。为此，通常采用以下两种方式实现：部署[防火墙](https://www.cloudflare.com/learning/security/what-is-a-firewall/)规则，阻止除 SYN 数据包以外的各类传出数据包；或者，对传入的所有 SYN-ACK 数据包进行过滤，防止其到达恶意用户机器。这种方法防护起来难度较小，所以并不常见
2. **欺骗攻击：**恶意用户还可以伪造其发送的各个 SYN 数据包的 IP 地址，以增加防护难度。虽然数据包可能经过伪装，但还是可以通过这些数据包追根溯源。
3. **分布式攻击（DDoS）：**如果使用僵尸网络发起攻击，则追溯攻击源头的可能性很低。攻击者可能还会命令每台分布式设备伪造其发送数据包的 IP 地址。

恶意用户可以通过 SYN 洪水攻击尝试在目标设备或服务中创建拒绝服务，**其流量远低于其他 DDoS 攻击**。SYN 攻击不属于容量耗尽攻击，其目的并非使目标周围的网络基础设施达到饱和，只需保证大于目标操作系统的可用积压工作即可。如果攻击者能够确定积压工作规模以及每个连接保持打开状态的时间长度（超出时间将进入超时状态），攻击者将可以找出禁用系统所需的确切参数，从而将创建拒绝服务所需的总流量降至最低。

#### 防护手段

##### 扩展积压工作队列

目标设备安装的每个操作系统都允许具有一定数量的半开连接。若要响应大量 SYN 数据包，一种方法是增加操作系统允许的最大半开连接数目。为成功扩展最大积压工作，系统必须额外预留内存资源以处理各类新请求。如果系统没有足够的内存，无法应对增加的积压工作队列规模，将对系统性能产生负面影响，但仍然好过拒绝服务。

##### 回收最先创建的 TCP 半开连接

另一种缓解策略是在填充积压工作后覆盖最先创建的半开连接。这项策略要求完全建立合法连接的时间低于恶意 SYN 数据包填充积压工作的时间。当攻击量增加或积压工作规模小于实际需求时，这项特定的防御措施将不奏效。

##### SYN Cookie

此策略要求服务器创建 Cookie。为避免在填充积压工作时断开连接，服务器使用 SYN-ACK 数据包响应每一项连接请求，而后从积压工作中删除 SYN 请求，同时从内存中删除请求，保证端口保持打开状态并做好重新建立连接的准备。如果连接是合法请求并且已将最后一个 ACK 数据包从客户端机器发回服务器，服务器将重建（存在一些限制）SYN 积压工作队列条目。虽然这项缓解措施势必会丢失一些 TCP 连接信息，但好过因此导致对合法用户发起拒绝服务攻击。

##### Cloudflare 如何缓解 SYN 洪水攻击？

Cloudflare 通过隔离目标服务器与 SYN 洪水攻击来缓解此类攻击。当发出初始 SYN 请求时，Cloudflare 将在云中完成握手过程，拒绝与目标服务器建立连接，直到完成 TCP 握手过程。

<img src="/images/DDos%E6%94%BB%E5%87%BB%E4%B8%8E%E4%BF%9D%E6%8A%A4/syn-flood-attack-ddos-attack-diagram-3.png" alt="Cloudflare 阻止 SYN Flood 攻击图" style="zoom:25%;" />

### 2.3 ICMP Flood攻击

也叫Ping Flood攻击

##### 攻击原理

ICMP Flood（即ICMP 洪水攻击）：当 ICMP ping 产生的大量回应请求超出了系统的最大限度，以至于系统耗费所有资源来进行响应直至再也无法处理有效 的网络信息流，这就是 ICMP 洪水攻击。简单说攻击者向一个子网的广播地址发送多个ICMP Echo请求数据包。并将源地址伪装成想要攻击的目标主机的地址。然后该子网上的所有主机均会对此ICMP Echo请求包作出答复，向被攻击的目标主机发送数据包，使此主机受到攻击，导致网络阻塞。ICMP Flood攻击主要的目的使网络瘫痪。

Ping Flood 的破坏性影响与向目标服务器发出的请求数量成正比。与[NTP放大](https://www.cloudflare.com/learning/ddos/ntp-amplification-ddos-attack/)、[DNS放大](https://www.cloudflare.com/learning/ddos/dns-amplification-ddos-attack/)等基于反射的DDoS攻击不同，Ping Flood攻击流量是对称的；目标设备接收的带宽量只是从每个机器人发送的总流量的总和。