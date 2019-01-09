---
layout:     post   				    # 使用的布局（不需要改）
title:      LLDP FloodLight LinkDiscovery 研究 				# 标题 
subtitle:   链路层发现协议 floodlight 链路发现模块 #副标题
date:       2018-12-9 				# 时间
author:     BY myran 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 毕业设计
    - floodlight
---

## Hey

* 学习链路层发现协议（LLDP）

  * LLDP帧格式
  * LLDPDU
  * TLV
  * 工作机制
    * LLDP是一个用于信息通告和获取的协议，但是需要注意的一点是，LLDP发送的信息通告不需要确认，不能发送一个请求来请求获取某些信息，也就是说LLDP是一个单向的协议，只有主动通告一种工作方式，无需确认，不能查询、请求（比如像ARP协议那样请求某个IP的MAC地址）。
    * LLDPDU发送机制
    * 发送状态机
    * 发送定时器状态机
    * LLDPDU 接收机制
    * 接收状态机
    * LLDP工作模式
    * LLDP可以工作在多种模式下：
      * TxRx：既发送也接收LLDP 帧。
      * Tx：只发送不接收LLDP 帧。
      * Rx：只接收不发送LLDP 帧。
      * Disable：既不发送也不接收LLDP 帧（准确的说，这并不是一个LLDP的状态，这可能是LLDP功能被关闭了，也可能是设备就不支持）。

* floodlight链路发现模块(LinkDiscovery)

  * LLDP和BDDP发现链路的原理:

    * ![1544348846310](img/1544348846310.png)
    * ![1544348885237](img/1544348885237.png)
    * 第一种情况，控制器发出LLDP后，并通过PACK_IN消息收到了相同的LLDP，则可以确定链路为直连。若LLDP发出后，没有收到相同的LLDP，则发送BDDP，如果收到了相同的BDDP，则确定链路中间有非OF设备。如果连BDDP也没收到，则认为端口处于网络边缘.

  * ![1544348927537](img/1544348927537.png)

  * 最后封装成PACK_OUT消息

  * LLDP和BDDP区别在于以太网帧中目的地址的不同，LLDP使用的组播地址，BDDP使用广播地址。

  * 四个队列的作用
    quarantineQueue:放入这个队列中的端口，会被遍历并发送BDDP消息。 
    maintenanceQueue:控制器在向每个端口发送了LLDP后，会把端口放入这个队列，之后会从这个队列中取出来发送BDDP。

    toRemoveFromQuarantineQueue:从quarantineQueue移出的端口放入这个队列. toRemoveFromMaintenanceQueue:从maintenanceQueue移出的端口放入这个队列. 

  * LLDP的发送分为两个方面，第一个方面是链路发现模块启动后，通过线程周期的轮训所有的交换机的所有Enable端口进行发送。第二个方面是在新来的交换机亦或者交换机端口改变时进行发送.(后两个通过注册IOFSwitchListener实现) 

  * BDDP的发送控制器是通过quarantineQueue和maintenanceQueue队列中取出端口，并查看这些端口是否存在于toRemoveFromMaintenanceQueue和toRemoveFromQuarantineQueue当中，如果存在则不发送BDDP，不存在则发送BDDP。

