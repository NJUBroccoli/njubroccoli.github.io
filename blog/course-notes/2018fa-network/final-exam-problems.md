# 2018计算机网络期末考题回忆

## 题型设计

共有五个大题，每个大题约有5到6小问，全部是简答题。重点考察概念，有些大题的题干跟没有一样，概念记得牢的话可以直接默写。

## 提醒

因为记忆会产生偏差，题目具体的表述不太记得清，我统一用“阐述、简述”来表示对一个概念的澄清和解释。

## 考察知识点

- 第一大题，重点考察第一章内容：<u>因特网协议层次以及主机、路由器、链路层交换机的协议层次</u>（2个小问）；<u>应用层报文、运输层报文段和网络层数据报的封装和转发过程</u>（1个小问）；<u>分组交换和电路交换的利与弊</u>（1个小问）；<u>路由器和链路层交换机的区别</u>（1个小问）

- 第二大题，重点考察第五章（链路层）内容：主要考察了多路访问链路协议，题目背景是围绕CSMA/CD, 令牌环网和TDM讨论它们的原理，并进行比较。<u>有一个二进制指数后退算法的计算，已经n求某个k的概率</u>（1个小问）；<u>CSMA/CD, 令牌环网和TDM的效率及灵活性比较</u>（1个小问）；<u>阐述CSMA/CD的工作原理</u>（1个小问）；<u>已经因特网最小传播速率为50微秒，求20个时隙的时延</u>（1个小问）；<u>给出两个结点A和B同时向对方传输的情景。第20微秒两者检测到了碰撞，它们之间的传播需要200微秒，A选择立刻重传，B选择经过1个时隙再重传。描述它们传播的时序情况。</u>（1个小问）

- 第三大题，综合考察了二、三、四章，这道题出得比较好（赞赏.jpg）。题目背景是一个挺大的网络拓扑结构，有两个主机A和B，主机A在一台NAT后面，这两台主机之间有很多复杂的交换机进行连接。题目还给了一个从主机B发往A的一帧的首部信息（只给出了源MAC地址、目的MAC地址、源IP地址、目的IP地址、源端口号、目的端口号）。题目如下：<u>写出图中那个NAT使能路由器的NAT转换表</u>（1个小问）；<u>写出图中标出的三个链路的首部信息</u>（模仿题目给的那个首部，1个小问）；<u>简述IPv4隧道的工作原理和作用</u>（1个小问）；<u>写出封装着IPv6数据报的IPv4数据报的大致结构</u>（1个小问）；<u>问在什么情况下，一个数据报的目的IP地址或源IP地址才会改变</u>（1个小问）；还有一个小问记不起来了。

- 第四大题，重点考察了TCP拥塞控制原理。这个题目发生了命题事故，是这样的：题干定义了一个叫做TCP toy的拥塞控制方法，它不含有快速恢复阶段，但在慢启动阶段，每收到一个ACK，都有`CWND += 3 * CWND`；在拥塞避免阶段，每收到一个ACK，都有`CWND += 3`。后来发现助教（命题人）的意思是“每过一个RTT”而不是“每收到一个ACK“，导致后者计算出来的CWND巨大无比。题目如下：<u>简述TCP拥塞控制原理</u>（1个小问）；<u>TCP为什么需要拥塞控制？</u>（1个小问）；根据TCP toy的定义，假设在t0=0RTT时，cwnd为1MSS且刚进入慢启动阶段，初始ssthresh为64，<u>在图中画出[0RTT, 14RTT]的曲线图，其中在7RTT发生了丢包事件。</u>（1个小问）；<u>假设有多个报文段在这个链路同时进行传输，能否实现链路带宽的平均分配？</u>（1个小问）；<u>题目给出一个MSS的字节数，计算前5个RTT的吞吐量，单位为Mbps</u>（1个小问）；<u>若将该TCP toy运用在10G带宽的校园网中，是否合适？否则如何改进？</u>（1个小问）

- 第五大题，重点考察了第八章的RSA加密、散列函数以及电子签名、报文鉴别。题干给出了一个单向散列函数$$RSAH(B_1, B_2)=RSA(RSA(B_1)\oplus B_2)$$其中$B_1$和$B_2$是两个数据的片段。题干认为这个散列函数不安全。题目如下：<u>简述RSA的加密原理</u>（1个小问）；<u>简述数字签名的过程</u>（1个小问）；<u>若任意指定一个$C_1$，试给出$C_2$的推导，满足$RSAH(C_1, C_2)=RSAH(B_1, B_2)$。</u>（1个小问）；<u>表示$RSAH(B_1, B_2, B_3)$，并简述$RSAH(B_1, B_2, \dots, B_n)$为什么不够安全？</u>（1个小问）；<u>提出对RSAH的改进方法，或者提供一个较好的报文鉴别方法。</u>（1个小问）