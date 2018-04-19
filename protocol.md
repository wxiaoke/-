## OSPF
    链路状态协议：给邻居发送的是链路状态的信息（LSA），方便邻居获得全网的拓扑结构
    OSPF：层次的网络结构
* 链路状态路由协议和距离矢量路由协议的区别？
    - LS：通告的是链路状态信息，构建一个全网的拓扑结构，运行路由算法（SPF）
	- DV：通告的是路由信息，可能产生环路（直接信任从邻居学到的信息），EIGRP的DUAL算法可以防止环路
- 链路状态协议的优势？
    - 运行链路状态路由协议的路由器比运行距离矢量路由器协议的路由器识别更多的网络信息
	- 每一台路由器都拥有整个拓扑结构
	- 能够根据准确的网络拓扑信息独立地做出决策

      |   协议  |  协议号  |  组播地址  |
      | ------- | :-----: | ----------:  |
      |  OSPF   |  89     |  224.0.0.5  224.0.0.6 |
      | EIGRP   |  88     |  224.0.0.9 |
     OSPF协议号89    组播地址 224.0.0.5  224.0.0.6（DR、BDR）
     EIGRP协议号88   组播地址 224.0.0.9
     OSPF和EIGRP是传输层协议，RIP是应用层协议（udp  520)
- OSPF包的类型

        1、HELLO包：用来建立和维持邻居关系；
        2、DATEBASE DESCRIPTION (DBD)：用来检验路由器之间的数据库并进行同步的；
        3、Link state request （LSR）：链路状态请求； 
        4、Link state update（LSU）：“特定”链路之间的请求记录； 
        5、Link state acknowledgement （LSAck）：确认包
        
        1、hello包是周期发送（直连网段），该包有router ID，保持时间40s，router优先级，邻居信息，area ID，DR、BDR、password，stub area标记；（就是发送看对方挂了没有）
        2、DBD 数据库描述包，该包其实是对LSA的摘要，是用来进行比较的（就像一本书的目录）
        3、LSR 链路状态请求，进行DBD比较后发现自己DBD中少lsa，会发送该包（两本书的目录不一样，少的向多的要）
        4、LSU 链路状态更新，收到LSR后把自己lsa发送给少的路由器，（把自己多的目录给他，让他和我的一样，少的向多的要）
        5、LSACK 确认包，收到LSA后发送确认，（我收到了！谢了！）
- 七个状态：down、init、two-way、exstart、exchange、loading、full；

        1、down 双方接口down状态
        2、init 初始化状态即单向通信，A收到B的hello（或B收到A的hello）；
        3、two-way 双方互相通信状态，彼此收到对方的hello，并且从hello包中读取信息，建立邻居关系；
        4、exstart 就是将要开始发送DBD，在发送之前确认谁先发，谁后发，他们自动协商，依靠router id，越大越优先；
        5、exchange 上边确认主从关系后，开始交换DBD即摘要，会有确认，
        6、loading 收到BDB后进行比较是否相同（比较依据查看序列号），然后进行LSR、LSU的请求和更新
        7、full  彼此的数据相同即LSA一样，此状态为邻接关系

 **1类LSA，router LSAs**
1. 每一台路由器都能产生1类LSA，并且用自己的router-id去标识
2. 包含了本路由器直连链路的信息
3. 在本区域内泛洪

       查看1类LSA
       show ip ospf database router
       ps异常状态：EXstart状态：MTU不同（最大传输单元）
	
**2类LSA，network LSAs**
    产生条件：broadcast或者NBMA（需要选举DR和BDR的网络）
1. 由DR产生，用DR的router-id去标识
2. 本区域的路由器有哪些，标识链路的掩码
3. 在本区域内泛洪

        show ip ospf data network
        默认情况下把loopback口当成主机接口，默认通告的是32位
        想让loopback口以实际的掩码通告（int lo0  ip ospf net point-to-point）
	
**3类LSA，summary LSAs：**
    用于将本区域产生的LSA信息泛洪到其他区域
1. 起始区域的ABR产生通告
2. 本区域的内的链路信息（网络前缀和掩码）
	    本区域的路由器看到的是其他区域
3. 被泛洪到整个AS
	
	    show ip ospf database summary
	
**4类LSA，summary LSAs：**
    用于通告ASBR路由器信息
1. 起始区域的ABR产生
2. 向其他区域通告ASBR的信息
3. 泛洪到整个AS
	
**5类LSA，external LSAs**
    通告如何去往外自治系统
1. ASBR产生
2. 包含外部自治系统的路由信息
3. 泛洪到整个AS

#### OSPF特殊区域：

     1）stub：
	  过滤掉外部路由（5类LSA），4类LSA
	  允许存在1类LSA、2类LSA、3类LSA  ABR自动产生一条默认路由（3类LSA），替代去往外部路由条目
     2）totally stub：
      过滤掉外部路由和区域间路由，过滤3,4,5类LSA
	  允许1,2类LSA、默认路由（3类LSA）
	  在配置stub的基础上，在ABR上配置totally stub 
	  3）NSSA
	  外部路由传入NSSA区域，以7类LSA存在
	  NSSA区域传入到其他区域（骨干区域），由ABR将7类LSA转为5类LSA
	  允许外部路由存在，以7类LSA形式存在，并且是本区域的ASBR产生的
	  过滤掉其他ASBR产生的5类LSA和4类LSA
	  允许存在1类LSA、2类LSA、3类LSA、7类LSA
	  
	  默认情况下，在NSSA区域内学到的外部路由是以7类LSA形式存在，显示是ON2
	  可以在ABR上向NSSA区域内下发一条默认路由（7类LSA），目的是替代被过滤的4类和5类（其他ASBR产生的）LSA
	  
	  4）NSSA Totally Stubby
	  完全NSSA，允许存在1类、2类、7类LSA
	  自动产生一条3类LSA的默认路由，替代过滤掉的3类4类5类LSA


***在多路访问网络中，区域内的路由信息是通过1类LSA和2类LSA产生的	
在点对点网络中，区域内的路由信息是通过1类 lsa产生的
其他区域的路由信息是通过3类LSA
外部自治系统路由信息是通过5类LSA产生***
        
**路由条目的类型：**

     O 由1类LSA和2类LSA产生的
     OIA 通过3类LSA学习
     O E1和O E2表示外部自治系统的路由条目，通过5类LSA学习，默认是O E2，cost值是20
	
**O E1和O E2的区别？**

        OE2：只计算外部的cost值
        OE1：计算外部的cost值+OSPF自治系统的cost值
	
#### OSPF路由汇总
     1）区域汇总   3类LSA
	 在ABR上
	（config-router）#area 0 range 172.16.0.0 255.255.0.0
	 advertise，通告这条汇总路由，默认通告
	 not-advertise，不通告这条汇总路由
	 cost，设置汇总路由的起始cost值
	 
	 2）外部路由汇总 5类LSA 
	 在ASBR上
    （config-router）#summary-address ip-address mask[not-advertise]


	  





        
