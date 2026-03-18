# Overview

## data link layer

- 以太网帧格式。MAC地址
- CSMA/CD（已启用，学习仅做了解历史） CSMA/CA 无线网络情况



##  network layer



ip address allocation

A...E type -> 最长前置匹配 。 划分子网

IP protocol format v4 v6

|version |IHL| ECN|SCS  ...

IP slicing ，<=> MMU 最小链路单元

​	Q: avoid slicing . count slicing(offset)

- task: routing and forwarding

- routing : routing table 

​	generate: RIP , OSPF ，BGP （difference , the bases: vetor-based link-based ,  spining tree(CSMA/CD)

- key protocol:

​	ARP IP->MAC

​	mechieum ,  broadcast, not matched will drop ; matched siglecast

​	protocol format ;

### Transport

TCP:

​	protocol format 

​	3way shaking

​	recieve: sliding window ( all mechiem from tcp , all are based on sliding windows )

​	retran: go-back-N ; selective repeat

​		 超时重传：2x RTT( round trip time)

​	flow control

​	congation control	

UDP：

​	protocol format (source port , desti port)

​	QUIC		

## Application layer

http (TCP)

https ( TLS )

DHCP (UDP )

DNS (UDP )



