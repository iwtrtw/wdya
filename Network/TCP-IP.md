### TCP/IP模型

+ 应用层(Message)：处理应用程序细节。（Telnet远程登录-TCP-21、FTP文件传输协议-TCP-23、SMTP简单邮件传输协议-TCP、DNS域名系统-UDP-53、TFTP简单文件传输协议-UDP-69、SNMP简单网络管理协议-UDP）
+ 传输层(Segment)：为两台主机上的应用程序提供端到端的通信。
  + TCP(传输控制协议)会把数据分成合适的小块再交给网络层，并确认接收到的分组，设置发送确认超时时间等，提供的是一种高可靠性的端到端的通信，应用层可以不用管这些
  + UDP(用户数据报协议)只是简单的把数据报从一台主机发送到另一台主机，并不不能保证数据报能安全无误地到达最终目的，任何必须的可靠性需要应用层来提供。(一个数据报是指从发送方传输到接收方的一个信息单元)
+ 网络层(Packet)：处理分组在网络中的活动，包括路由选择
  + IP(网际协议)：IP是网络层上的主要协议，同时被TCP和UDP使用。TCP和UDP的每组数据都通过端系统和每个中间路由器中的IP层在互联网中进行传输。
  + ICMP(Internet互联网控制报文协议)：CMP是IP协议的附属协议，IP层用它来与其他主机或路由器交换错误报文和其他重要信息。尽管ICMP主要被IP使用，但应用程序也有可能访问它，Ping和Traceroute，它们都使用了ICMP。
  + IGMP(Internet组管理协议) ：用来把一个UDP数据报多播到多个主机。
  + ARP(地址解析协议)：ARP协议提供了网络层地址(IP地址)到物理地址(MAC地址)之间的动态映射
  + RARP：ARP协议提供了物理地址(MAC地址)到网络层地址(IP地址)之间的动态映射
+ 网络接口层(Frame+Bit)：通常包括操作系统中的设备驱动程序和计算机口中对应的网络接口卡，与物理传输介质交互

应用层通常是在用户空间进行，处理的是应用程序细节；而传输层、网络层、链路层通常才内核空间进行，它们处理的是通信细节。TCP/IP协议族是一组不同的协议组合在一起构成的协议族。在TCP/IP协议族中，网络层IP提供的是一种不可靠的服务。也就是说，它只是尽可能快地把分组从源结点送到目的结点，但是并不提供任何可靠性保证。而另一方面，TCP在不可靠的IP层上提供了一个可靠的运输层。为了提供这种可靠的服务，TCP采用了超时重传、发送和接收端到端的确认分组等机制。网络层和运输层之间的区别是最为关键的：网络层（IP）提供点到点的服务，而传输层（TCP和UDP）提供端到端的服务。

#### IP地址

|      | 网络号                     | 主机号 | 范围                      |
| ---- | -------------------------- | ------ | ------------------------- |
| A类  | 0+7位（2^8-2=126）         | 24位   | 0.0.0.0~127.255.255.255   |
| B类  | 10+14位（2^14-1=16383）    | 16位   | 128.0.0.0~191.255.255.255 |
| C类  | 110+21位（2^21-1=2097151） | 8位    | 192.0.0.0~223.255.255.255 |
| D类  | 1110+28位                  |        | 224.0.0.0~239.255.255.255 |
| E类  | 11110+27位                 |        | 240.0.0.0~247.255.255.255 |

互联网络信息中心(Internet Network Information Centre)，称作InterNIC，InterNIC只分配网络号，主机号的分配由系统管理员来负责。计算A、B、C类的网络号时，去除网络号全为0的地址以及127.0.0.1

#### DNS

域名系统(DNS)是一个分布的数据库，由它来提供IP地址和主机名之间的映射信息。

#### 数据封装

TCP传给IP的数据单元称作 TCP报文段或简称为TCP段（TCP segment）；IP和网络接口层之间传送的数据单元是分组(packet)，分组既可以是一个IP数据报(IP datagram)，也可以是IP数据报的一个片(fragment);通过以太网传输的比特流称作帧(Frame)。以太网数据帧的物理特性是其长度必须在 46～1500字节之间。UDP数据与TCP数据基本一致。唯一的不同是UDP传给IP的信息单元称作UDP数据报（UDP datagram），而且UDP的首部长为8字节。

传输层中的TCP和UDP都用一个16bit的端口号来标识不同的应用程序，TCP和UDP把源端口号和目的端口号分别存入报文首部中。网络层中IP首部用8bit的协议域标识数据属于哪一层(1代表ICMP、2代表IGMP、6代表TCP、17代表UDP)。网络接口层中以太网帧首部用16bit的帧类型域指明生成数据的网络层协议

![TCP-IP数据封装](..\image\TCP-IP数据封装.png)

#### 分用(数据解封)

当目的主机收到一个以太网数据帧时，数据就开始从协议栈中由底向上升，同时去掉各
层协议加上的报文首部。每层协议盒都要去检查报文首部中的协议标识，以确定接收数据的
上层协议。这个过程称作分用（Demultiplexing）

![TCP-IP分用](..\image\TCP-IP分用.png)



### 链路层

链路层主要有三个目的：为IP模块发送和接收IP数据报、为ARP模块发送ARP请求和接收ARP应答、为RARP发送RARP请求和接收RARP应答。

#### 以太网与IEEE 802

​		以太网是一个标准，它是当今TCP/IP采用的主要的局域网技术。它采用一种称作CSMA/CD的媒体接入方法，其意思是带冲突检测的载波侦听多路接入(Carrier Sense, Multiple Access with Collision Detection)，它的速率为10Mb/s，地址为48bit。后来，IEEE802委员发布了稍有不同的标准集。其中802.3针对整个CSMA/CD网络，802.4针对令牌总线网络，802.5针对令牌环网络。这三者的共同特性由802.2标准来定义，那就是802网络共有的逻辑链路控制（LLC）。不幸的是，802.2和802.3定义了一个与以太网不同的帧格式。

​		TCP/IP世界中，以太网的IP数据报的封装是在RFC 894中定义的，IEEE 802网络的IP数据报封装是在RFC 1042中定义的。最常使用的封装格式是以太网定义的格式，两种帧首部(14字节)格式均采用：目的硬件地址(48bit)+源硬件地址(48bit地址)+2字节。最后2个字节在两种帧格式所表示的信息不同，可以用于区分两种不同的帧格式。在802标准中，这2个字节表示的是长度字段，指的的是它后续数据的字节长度，但不包括CRC校验码，类型字段则由后续的子网接入协议(Sub-network Access Protocol,SNAP)的首部给出；在以太网中，2字节表示的后续数据的类型。

​		在以太网帧格式中，类型字段之后就是数据；而在802帧格式中，跟随在后面的是3字节的802.2 LLC(逻辑链路控制)和5字节的802.2  SNAP(子网接入协议)；目的服务访问点(Destination Service Access Point, DSAP)和源服务访问点(Source Service Access Point, SSAP)的值都设为0xaa，Ctrl字段的值设为3，随后的3个字节org code都置为0，再接下来的2个字节类型字段和以太网帧格式一样。

​		802.3标准定义的帧和以太网的帧都有最小长度要求。802.3规定数据部分必须至少为38字节，而对于以太网，则要求最少要有46字节。为了保证这一点，必须在不足的空间插入填充（pad）字节。CRC字段用于帧内后续字节差错的循环冗余码检验（检验和）

ARP和RARP协议就是负责对32bit的IP地址和48bit的硬件地址进行映射。

![TCP-IP以太网封装](..\image\TCP-IP以太网封装.png)

+ SLIP(Serial Line IP)是一种在串行线路上对IP数据报进行封装的简单形式

+ PPP(Point-to-Point Protocol)点对点协议：为在同等单元之间传输数据包这样的简单链路设计的链路层协议。这种链路提供全双工操作，并按照顺序传递数据包。

#### 最大传输单元MTU

​		以太网和802.3对数据帧的长度都有一个限制，其最大值分别是1500和1492字节。如果IP层有一个数据报要传，而且数据的长度比链路层的MTU还大，那么IP层就需要进行分片（fragmentation），把数据报分成若干片，这样每一片都小于MTU。

​		当在同一个网络上的两台主机互相进行通信时，该网络的MTU是非常重要的。但是如果两台主机之间的通信要通过多个网络，那么每个网络的链路层就可能有不同的MTU。重要的不是两台主机所在网络的MTU的值，重要的是两台通信主机路径中的最小MTU。它被称作路径MTU。两台主机之间的路径MTU不一定是个常数。它取决于当时所选择的路由。而选路不一定是对称的（从A到B的路由可能与从B到A的路由不同），因此路径MTU在两个方向上不一定是一致的。

#### ？？？串行线路吞吐量计算？？？

### 网络层

IP协议提供的数据报传送服务是不可靠、无连接的；不可靠是指它不能保证IP数据报能成功地到达目的地，如果发生某种错误时，如某个路由器暂时用完了缓冲区， IP有一个简单的错误处理算法：丢弃该数据报，然后发送ICMP消息报给信源端。任何要求的可靠性必须由上层来提供(如TCP)；无连接是指IP并不维护任何关于后续数据报的状态信息；每个数据报的处理是相互独立的，这意味着IP数据报的接收顺序可能与发送时的顺序不一样(比如发送方先发送 A，然后B，但它们选择的路由线路可能不同，而导致B可能在A到达之前先到达接收方)







1. 在OSI模型中ARP/RARP协议属于链路层，而在TCP/IP模型中，ARP/RARP协议属于网络层
2. 一般来说， TCP服务器是并发的(服务端每次生成新的服务处理客户端的请求，如果操作系统允许多任务，那么就可以同时为多个客户服务)，而UDP服务器是重复的(服务端在处理客户请求的时候不能为其他客户机提供服务)
3. 知名端口号介于1～255之间。256～1023之间的端口号通常都是由Unix系统占用，以提供一些特定的Unix服务。大多数TCP/IP实现给临时端口分配1024～5000之间的端口号。服务器使用知名端口号，而客户使用临时设定的端口号。

