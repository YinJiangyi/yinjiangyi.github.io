### 写在前面的背景扫盲

#### 制定SIP的IETF

IETF: Internet Engineering Task Force，互联网工程任务组

> 说到SIP是怎么出来的就要提H.323，而提到这个标准由不得不提到ITU-T，我们就先说说指定SIP的IETF（Internet Engineering Task Force）和制定H.323的ITU-T（International Telecommunications Union–Telecommunications Standard Sector）之间一些小趣事吧。ITU-T和IETF想事儿总是不一样的，它们俩往往从两个不同的角度来进行。ITU-T作为一个隶属于美国的国际标准化组织，传承了美国政策的一致性和试图完成维护世界和平的角色，所以它制定出来的标准一定是经过无数轮的反复和草案，花了N年制定出一个最终的，广为接受的结果。而恰恰相反，IETF则更趋近于实用主义，信奉“rough consensus and running code”，也就是说能用进行，边用边补充呗。因此在大多数时候，IETF标准制定的周期要比ITU-T短一些，这可能也归因于IETF每年举办三次开放性论坛。这些春季、夏季、秋季会议会安排在世界各地，并且向各个对此感性趣的组织开放（去瞅一瞅吧：http://www.ietf.org/meetings/meetings.html ）多个IETF工作组在这些会议上相聚，针对他们现在的系统开发敲定一些技术细节，对于要是到了会议结束时还是没有解决的问题那就几个月后的下次开会继续研究呗。简而言之，IETF以一种实际实用的角度去完善体系架构和协议设计。
>
> 说了这么多可能你还没有意识到我在扯这些是为了什么，但是请记住这些正是**SIP（Session Initiation Protocol）架构所体现的核心思想——先用着，再扩展。**SIP的结构是建立于两个常用协议之上的：在RFC 2821中的**SMTP 协议**(Simple Mail Transfer Protocol )——它定义了电子邮件的消息格式，以及定义在RFC 2616的HTTP协议 (Hypertext Transfer Protocol )——它定义了基于Web的多媒体通信消息。另外，SIP又使用了定义在RFC 3550中的RTP/RTCP协议(Real Time Transport Protocol/Real Time Control Protocol )——它定义了在IP网上的多媒体包格式，还使用了定义在RFC 2327的SDP协议(Session Description Protocol )——它定义了一个多媒体会话的参数和特征。因此，**SIP是建立在其他IETF提出的协议之上的**，这有点像H.323建立在诸如H.225.0和H.245等ITU-T制定的协议之上，它的两个比较基础的RFC分别是版本1.0 RFC2543和版本2.0 RFC3261。当然，SIP还运行于其他IETF定义的传输协议之上，比如TCP(Transport Control Protocol ), UDP(User Datagram Protocol ) 和IP (Internet Protocol )等。这样，**这么多著名的，并且被广泛应用的协议为SIP提供了超于H.323的简单明了的特性。**





### Reference
[【SIP协议】学习初学笔记](https://www.cnblogs.com/gnuhpc/archive/2012/01/16/2323637.html)

