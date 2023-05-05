[**新手入门一篇就够：从零开发移动端IM**](http://www.52im.net/thread-464-1-1.html)

### IM介绍

IM其本质是一套**消息发送与投递系统**，或者说是一套网络通信系统，归根结底就是两个词：**存储与转发**（消息发送后存储消息---怎么存储，多人与单人的区别---为了以后的转发，转发就可以实现离线和在线的消息转发）。但一个成熟的移动端IM系统要想正常运转，涉及的内容则远不止这些，而最考验技术功底的就是服务端架构的设计与实现。

没有过IM系统开发经验的人，可能对以上观点嗤之以鼻，在此借用TeamTalk的设计者的一段话：“IM服务器开发，从功能抽象的角度看可能非常简单，可以认为是管理大量的客户端连接和在不同的客户端之间传递消息，但具体到实现细节就比较复杂了。打个不恰当的比喻，OS的功能抽象也非常简单，无非是进程间的调度和硬件资源的管理，但要是自己去实现一个，一般人也就只能呵呵了。”

### 功能

1. 单聊---***
2. 群聊---***

1. 离线消息（单聊+群聊，支持消息提醒（MQ）），重上接收
2. 聊天记录（单聊、群聊）---***

## 没有路由

这个部分因为没有路由所以我得提前存入redis获得uid以及token,这里假设我能连接到双方（这样是不是不能设计心跳保活）（记录两个的fd位置，朝FD口发送），两个客户端，连接到路由，A向B发送,A找到B的路由，A登录时记住gate（ip:port）位置,

[^所以我这里得仿照login，连接即为登录]:正常应该短链接验证，然后长连接,并且记录gate（token,ip:port），然后缓存gate，然后双方由gate进行查询并且进行交互,但是会有问题，我们每一次都去查redis吗（gate得有水平扩展）

**IM的架构可以抽象成这么几层：**

- 1）客户端：例如pc微信，手机qq
- 2）服务端：
       2.1）入口层gate集群：能够水平扩展，保持与客户端的连接；
       2.2）逻辑层logic、路由层router集群：高可用可扩展，实现业务逻辑，进行消息的路由；
       2.3）cache：高可用cache集群，用来存储用户的在线状态，与接入节点（用户具体连接在哪个gate节点）；
       2.4）db：固化存储消息，群信息，好友关系链等信息。

[^如果我是redis中存储]:那我怎么发送呢，还有，安全送达的收发模型怎么建立

## 需要解决的问题

#### 关系链

##### **群聊是IM中架构设计的技术难点，以下是群聊方面的专门文章：**

- 《[IM单聊和群聊中的在线状态同步应该用“推”还是“拉”？](http://www.52im.net/thread-715-1-1.html)》
- 《[IM群聊消息如此复杂，如何保证不丢不重？](http://www.52im.net/thread-753-1-1.html)》
- 《[移动端IM中大规模群消息的推送如何保证效率、实时性？](http://www.52im.net/thread-1221-1-1.html)》
- 《[关于IM即时通讯群聊消息的乱序问题讨论](http://www.52im.net/thread-1436-1-1.html)》
- 《[IM群聊消息的已读回执功能该怎么实现？](http://www.52im.net/thread-1611-1-1.html)》
- 《[IM群聊消息究竟是存1份(即扩散读)还是存多份(即扩散写)？](http://www.52im.net/thread-1616-1-1.html)》
- 《[一套高可用、易伸缩、高并发的IM群聊、单聊架构方案设计实践](http://www.52im.net/thread-2015-1-1.html)》
- 《[网易云信技术分享：IM中的万人群聊技术方案实践总结](http://www.52im.net/thread-2707-1-1.html)》
- 《[IM群聊消息的已读未读功能在存储空间方面的实现思路探讨](http://www.52im.net/thread-3054-1-1.html)》
- 《[直播系统聊天技术(一)：百万在线的美拍直播弹幕系统的实时推送技术实践之路](http://www.52im.net/thread-1236-1-1.html)》
- 《[直播系统聊天技术(二)：阿里电商IM消息平台，在群聊、直播场景下的技术实践](http://www.52im.net/thread-3252-1-1.html)》
- 《[直播系统聊天技术(三)：微信直播聊天室单房间1500万在线的消息架构演进之路](http://www.52im.net/thread-3376-1-1.html)》
- 《[直播系统聊天技术(四)：百度直播的海量用户实时消息系统架构演进实践](http://www.52im.net/thread-3515-1-1.html)》
- 《[融云IM技术分享：万人群聊消息投递方案的思考和实践](http://www.52im.net/thread-3687-1-1.html)》

### 消息协议参考

PS注意需要自定的部分，比如消息序号[IM消息ID技术专题(六)：深度解密滴滴的高性能ID生成器(Tinyid)](http://www.52im.net/thread-3129-1-1.html)，用于去重逻辑

[瓜子](http://www.52im.net/thread-812-1-1.html)协议参考，c2c,c2g设计流程

[一个通用即时通讯（IM）系统的设计](https://zhuanlan.zhihu.com/p/409927117))

[轨迹](https://gitee.com/xchao) / [J-IM](https://gitee.com/xchao/j-im)

#### 数据协议实现

认证

```c
enum MSG
{
	Login_Request		=	10001;
	Login_Response		=	10002;
	Logout_Request		=	10003;
	Logout_Response		=	10004;
	Keepalive_Request	=	10005;
	Keepalive_Response	=	10006;

	Get_Friends_Request	=	10007;
	Get_Friends_Response =	10008;
	Send_Message_Request	= 10009;
	Send_Message_Response	= 10010;

	Friend_Notification	= 20001;
	Message_Notification	= 20002;
	Welcome_Notification	= 20003;
}

message AuthRequest{
​	string token=1;
​	string uid=2;
​	int64 timestamp =3;//发包时间戳 
}

message AuthResponse{
//状态枚举
enum Status 
{
​	OK = 0;
​	ERR = -1;
}
	int32 status = 1;//状态，使用枚举
	int32 err_code = 2;//错误码，统一定义的错误码
	string err_msg = 3;//错误描述
}
```

心跳

```
消息包为空
```

单对单聊天(c2c)

发送者使用UUID，msgid使用消息ID生成策略[**IM消息ID技术专题(三)：解密融云IM产品的聊天消息ID生成策略**](http://www.52im.net/thread-2747-1-1.html)

```c
message C2CSendRequest{
	string from =1;//发送者
    string to =2;//目的群
    string content=3;//消息内容
}
message C2CSendResponse{
	int64 msgid=1//落地存储的消息ID，可以参考其他的消息ID设计理念，因为可能会重复
    int64 timestamp=2//收到单聊消息的时间戳    
}
message C2CPushRequest{
    string from=1;//发送者
    string content=2;//消息内容
    int64 msgid=3;//消息服务器对消息的编号
    int64 timestamp =4;//收到单聊消息的时间戳
}
message C2CPushResponse{
    int64 msgid=1;//消息服务器对消息的编号
}

```

群聊(c2g)

发送者使用UUID，[IM消息ID技术专题(六)：深度解密滴滴的高性能ID生成器(Tinyid)](http://www.52im.net/thread-3129-1-1.html)

msgid使用消息ID生成策略[**IM消息ID技术专题(三)：解密融云IM产品的聊天消息ID生成策略**](http://www.52im.net/thread-2747-1-1.html)

```c
message C2GSendRequest{
	string from=1;//发送者
    string group=2；//目的群
    string content=3；//消息内容
}
message C2GSendRespones{
    int64 msgid=1;//落地存储的消息ID(针对此条消息)
    int64 timestamp=2;//收到群聊消息的时间戳
}
message C2GPushRequest{
	string from=1;//发送者
    string group=2；//目的群
    string content=3；//消息内容
    int64 msgid=4;//消息服务器对消息的编号(针对每一个用户独立的一份消息)
    int64 timestamp=5;//收到群聊消息的时间戳
}
message C2GPushRespones{
    int64 msgid=1;//消息服务器对消息的编号(针对每一个用户独立的一份消息)?????
}
```

### 持久存储

**发送消息表**

<img src="C:\Users\刘磊\AppData\Roaming\Typora\typora-user-images\image-20230504140523762.png" alt="image-20230504140523762" style="zoom:50%;" />

cmd_id有啥用，msg_seq怎么设计

**推送消息表**

保存某个用户收到了哪些消息。这里如果是群接收者的填什么

<img src="C:\Users\刘磊\AppData\Roaming\Typora\typora-user-images\image-20230504140749909.png" alt="image-20230504140749909" style="zoom:50%;" />

**群基本信息表**

<img src="C:\Users\刘磊\AppData\Roaming\Typora\typora-user-images\image-20230504140943266.png" alt="image-20230504140943266" style="zoom:50%;" />

**群用户关系表**

这个有啥用

<img src="C:\Users\刘磊\AppData\Roaming\Typora\typora-user-images\image-20230504141005770.png" alt="image-20230504141005770" style="zoom:50%;" />

### Redis缓存

**用户状态及路由信息：**
Redis缓存以uid为key，检索channel(socketid)，last_packet_time等。
Gate层，session以channel(socketed)为key，检索uid，及其他信息。
交互接口：gate->logic，通过将channel转换为uid作为key。
logic->gate，将uid转换为channel作为key。

### 客户端与服务端消息交互整体原理

[**融云技术分享：全面揭秘亿级IM消息的可靠投递机制**](http://www.52im.net/thread-3638-1-1.html)

##### 概述

**一个完整的IM消息交互逻辑，通常会为两段：**

- 1）消息上行段：即由消息发送者通过IM实时通道发送给服务端；

- 2）消息下行段：**由服务端按照一定的策略送达给最终的消息接收人。**

  ### 消息下行段


  经过总结，消息下行段主要有三种行为。

  **1）客户端主动拉取消息，主动拉取有两个触发方式：\***

  - ① 拉取离线消息：与 IM 服务新建立连接成功，用于获取不在线的这段时间未收到的消息；
  - ② 定时拉取消息：在客户端最后收到消息后启动定时器，比如 3-5 分钟执行一次。主要有两个目的，一个是用于防止因网络、中间设备等不确定因素引起的通知送达失败，服务端客户端状态不一致，一个是可通过本次请求，对业务层做状态机保活。


  ***2）服务端主动-发送消息（直发消息）：\***

  这是在线消息发送机制之一，简单理解为服务端将消息内容直接发送给客户端，适用于消息频率较低，并且持续交互，比如二人或者群内的正常交流讨论。

  ***3）服务端主动-发送通知（通知拉取）：\***

  这是在线消息发送机制之一，简单理解为服务端给客户端发送一个通知，通知包含时间戳等可作为排序索引的内容，客户端收到通知后，依据自身数据，对比通知内时间戳，发起拉取消息的流程。

  这种场景适用于较多消息传递：比如某人有很多大规模的群，每个群内都有很多成员正在激烈讨论。通过通知拉取机制，可以有效的减少客户端服务端网络交互次数，并且对多条消息进行打包，提升有效数据载荷。既能保证时效，又能保证性能。

[^通知拉取的场景]:可以这样思考场景，因为我们获得一个消息是有一定时延的，如果一条条的获得肯定会时间过长，所以当开始出现堆积消息时，就第一条进行直发后续打包发送,PS----那么我们怎么判断什么时候进行通知拉取
[^由直发转变为通知拉取]:在消息发送过程中，如果上一条消息发送流程未结束，下一条消息则不用直发(s_msg)，而是用通知(s_ntf)。下行消息拉取放于的消息缓存中吗，消息缓存有时效性吗，写入数据库的消息从哪儿来，我是不是可以，单条的发送，都直发，后续检测状态，然后从redis读，DB中的后续数据，也可以从缓存获取。

  <img src="http://www.52im.net/data/attachment/forum/202107/25/235438on77meekp9je79fq.png" alt="融云技术分享：全面揭秘亿级IM消息的可靠投递机制_8.png" style="zoom: 50%;" />

- 
  -----------------------------------------------------------------------

  

1. 消息可达

[M开发干货分享：浅谈IM系统中离线消息、历史消息的最佳实践](http://www.52im.net/thread-3887-1-1.html)

[**融云技术分享：全面揭秘亿级IM消息的可靠投递机制**](http://www.52im.net/thread-3638-1-1.html)PS如果在分布式下怎么实现

**（1）应用层确认+im消息可靠投递的六个报文**

[^--一定的吗，这样怎么解决离线消息,因为这样会往回给一个ack:N--]: 如果client-B不在线，im-server保存了离线消息后，要伪造ack:N发送给client-A
[^如此复杂的消息传递，很多处于重传不就会服务器宕机]:
[^,**还有，为什么不回复一个ACK:N就行了**]:不行，因为B要给server确认收到了R,然后服务器给B确认服务器能确认B一定收到，然后发给A确认B收到



**~~(2) 离线消息的拉取，**为了保证消息的可靠性，也需要有ack机制，但由于拉取离线消息不存在N报文，故实际情况要简单的多，即先发送offline:R报文拉取消息，收到offline:A后，再发送offlineack:R删除离线消息。~~

~~**（3）消息的超时与重传**~~

~~[**IM消息送达保证机制实现(二)：保证离线消息的可靠投递**](http://www.52im.net/thread-594-1-1.html)~~

~~client-A发出了msg:R，收到了msg:A之后，在一个期待的时间内，如果没有收到ack:N，client-A会尝试将msg:R重发。可能client-A同时发出了很多消息，故client-A需要在本地维护一个等待ack队列，并配合timer超时机制，来记录哪些消息没有收到ack:N，以定时重发。~~

[^~~为什么是A来重发，既然可以确保server拿到了消息，应该由server进行重发呀~~]:

##### ~~消息接收方不在线时的典型消息发送流程~~



![IM消息送达保证机制实现(二)：保证离线消息的可靠投递_1.png](http://www.52im.net/data/attachment/forum/201611/13/215119i6746o666z54hlhe.png)



~~如上图所述，通常此类情况下消息的发送流程如下：~~

- ~~Step 1：用户A发送一条消息给用户B；~~
- ~~Step 2：服务器查看用户B的状态，发现B的状态为“offline”（即B当前不在线）；~~
- ~~Step 3：服务器将此条消息以离线消息的形式持久化存储到DB中（当然，具体的持久化方案可由您IM的具体技术实现为准）；~~
- ~~Step 4：服务器返回用户A“发送成功”ACK确认包（**注：**对于消息发送方而言，消息一旦落地存储至DB就认为是发送成功了）。~~

## 推送系统

**推送系统的核心任务：**是接收到给用户发送下行消息的请求以后，去信令服务查询用户是否在线，如果在线走信令推送，如果不在线走离线推送（如[iOS的APNS](http://www.52im.net/thread-345-1-1.html)、华为推送、小米推送等）。