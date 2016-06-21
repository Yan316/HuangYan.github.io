---
layout: post
title: AWS相关的网络知识
description: AWS相关的网络知识
tag: AWS
category: blog
---

Amazon VPC(Amazon Virtual Private Cloud)是在Amazon Web Services(AWS)云中预置一个逻辑隔离分区，这样就可以在自己定义的虚拟网络中启动AWS资源。在这个资源中可以完全掌握虚拟联网环境，包括选择自有的IP地址范围、创建子网、以及配置路由表和网关，安全配置等等。

######首先我们通过以下网络图来看看安全组，子网，Network ACL， 路由表直接的关系

![网络访问控制列表](/images/githubpages/vpc_example.png)

####（1）弹性IP地址
弹性IP地址（Elastic IPs）是公有IP地址，可以通过Internet访问。如果实例没有公有IP地址，可以用弹性IP地址与实例关联以启动与Internet通信。它是与AWS账号绑定的。特定区域的弹性IP地址只能在一个特定区域中使用。

####（2）路由表
路由表包含一系列被称为路由的规制，可用于判断网络流量的导向目的地。

具有以下几个特点：

* VPC的每个子网必须与一个路由表关联，并且只能关联一个路由表。但一个路由表可以被多个子网关联。
* VPC会自动生成主路由表（即默认路由表），不可被删除。如果子网未与特定路由表间建立显示关联，则该子网将于主路由表建立隐式关联。
* 每个路由都指定了一个目的CIDR和目标。

路由表的优先级（从高到低）：

1. VPC本地路由
2. 目标为Internet网关、虚拟专用网关、网络接口、实例ID、VPC对等连接或VPC终端节点的静态路由
3. 来自VPN连接或AWS Direct Connect连接的任意传播路由

VPN连接内存在重叠路由，则优先级从高到低如下所示：

1. 来自AWS Direct Connect连接的BGP传播路由
2. 为VPN连接手动添加的静态路由
3. 来自VPN连接的BGP传播路由

####（3）网路地址转换
私有子网可用于您不希望能直接从 Internet 寻址的实例。私有子网中的实例无需暴露其私有 IP 地址即可访问 Internet，具体方法是在公有子网中通过网络地址转换 (NAT) 网关路由其流量。

NAT设备用于支持私有子网中的实例连接到Internet（例如进行软件更新）或其他AWS服务，但组织Internet发起与实例的连接。

当流量流向Internet时，源IP地址将替换成NAT设备的地址，同样，当响应流量流向实例时，NAT设备将地址转换成这些实例的私有IP地址。

NAT设备分为NAT网关和NAT实例。

1. NAT网关支持一下协议：TCP、UDP和ICMP
2. 每个NAT网关只能管理一个弹性IP地址
3. 不能通过与VPC管理的ClassicLink连接来访问NAT网关
4. 必须指定NAT网关处于哪个公有子网中。

注意：在账户中创建和使用 NAT 网关会产生费用

####（4）安全配置
Amazon VPC 提供了安全组（security groups）和网络访问控制列表（network access control lists）等高级安全功能，实现在实例级别和子网级别启动入站和出站筛选功能。

###### * 网络访问控制列表Network Access Control Lists
网络访问控制列表(Network Access Control Lists)作为VPC的一个可选安全层，可作为防火墙来控制进出一个或多个子网的流量。

1. VPC会自动带有一个可以修改的默认网络ACL。这个网络ACL会允许所有入站和出站的数据流。
2. 自定义的ACL，默认情况是拒绝所有入站和出站流量，直至添加规制。
3. VPC中的子网必须和一个也只能和一个网络ACL关联，若未指定，则与默认网络ACL关联。
4. 一个网络ACL可以和多个子网关联。
5. 网络ACL包含规则的标号列表，从小编号的规制开始进行。只要有一条规制与流量匹配，则应用该规则。

对网络ACL规则包含的信息如下：规制标号、Type（HTTP/HTTPS/SSH/RDP/自定义TCP）、协议（TCP等）、端口范围、源、允许/拒绝 

如下列表所示：

![网络访问控制列表](/images/githubpages/nacl.png)

###### * 安全组
安全组是充当实例的虚拟防火墙以控制入站和出站的流量。

1. 它是在实例级别运行，而不是在子网级别。所以每个子网的实例可归属不同的安全组集合。
2. 最多为实例指定五个VPC的安全组，如果不指定，实例会自动归属到VPC的默认安全组。默认情况下，出站规制允许所有的出站流量。
3. 可指定允许规制，不可指定拒绝规制。
4. 安全组与网络接口关联。

规制内容包含：数据流源（入站）/数据流目的地（出站），协议，端口范围，注释。
如下列表所示：

![安全组规则](/images/githubpages/security-group.png)

下面我们来对比两组安全保障措施

| 安全组 | 网络ACL |
| ------|:------:|
| 在实例级别操作 | 在子网级别操作 |
| 仅支持允许规则 | 支持允许规则 |
| 有状态：返回数据流会被自动允许，不受任何规制的影响 |无状态：返回数据流必须被规则明确允许|
| 我们会在决定是否允许数据流前评估所有规则 | 我们会在哎决定是否允许数据流时按照数字顺序处理所有规则 |
| 只有在启动实例的同事指定安全组、或稍后将安全组与实例关联的情况下，操作才会被应用到实例 | 自动应用到关联子网内的所有实例（备份防御层，因此您变不需要比人为您指定安全组）|

###### （5）子网掩码
子网掩码是一种用来指明一个IP地址的那些位标识的是主机所在子网，以及那些位标识的是主机的位掩码。
必须和IP地址结合使用，将某个IP地址划分为网络地址和主机地址两个部分。
子网掩码位数也是32位，1的数目等于网络位的长度，0的数目是主机位的长度。一般情况下IP地址后加上“/”符号以及数字

* 对于A类地址，默认的子网掩码是255.0.0.0；
* 对于B类地址子网掩码是255.255.0.0；
* 对于C类地址，子网掩码是255.255.255.0

（主机号全为1是标识该网络的广播地址，全为0时标识该网络的网络号。）

子网掩码的两种标识方法：

1. 通过与IP地址格式相同的点分十进制表示
2. 在IP地址后加上“/”符号以及1-32的数字，数字标识子网掩码中网络标识位的长度
如: 192.168.1.1/24

计算两台计算机是否属于统一网段的方法: 将计算机十进制的IP地址和子网掩码转换为二进制的形式，然后进行”与”运算（全1得1，不全1得0,如果出来的结果相同，则两台计算机就属于同一网段。

注：同一个子网的主机，交换信息不通过路由器，如果不属于同一网络区间，两个地址的信息交换就要通过路由器。

####网络间的互连
VPC可以连接到Internet, 数据中心或其他的VPC。下面我们来看看VPC是怎么跟它们交互的。

（1）VPC的公有子网可直接直接连接到Internet
 
（2）私有子网通过网络地址转换连接到Internet
 
（3）同一个VPC下的子网络之间的互连
  同一个VPC下的子网络本身默认就是可以互相访问的，用户只需要对ACL和Security Group进行访问控制即可；

（4）同一个Region下的VPC之间的互连

![网络访问控制列表](/images/githubpages/vpc_peering.png)

同一个Region下VPC之间的VPC之间的互连主要通过VPC Peering（VPC对等）来实现的。
VPC对等连接是连个VPC之间的网络连接，通过此连接，两个VPC的实例可以彼此通信，就像在同一网络中一样。

* 在同一账户中创建VPC的对等连接
* 与其他AWS账号中的VPC对等连接

在AWS中的VPC->Peering Connnections下可以设置，具体流程如下：

创建对等连接（发出等等连接请求）  ->  接受对等连接  -> 为VPC对等连接更新路由表 -> 更新安全组以引用对等的VPC安全组

（5）不同Region下VPC之间的互连
不同Region下VPC的连接主要是通过各自VPC公共网络中配置VPN来实现IPSec连接。

![VPC与本地数据中心的互连](/images/githubpages/different_region_vpc_connect.png)

（4）AWS VPC与本地数据中心网络之间的互连
VPC与数据中心的连接也是通过VPN的IPSEC联系起来，在VPC的VPN Connection中配置Virtual Private Gateway。公司的数据中心配置Customer Getway。

如图所示：

![VPC与本地数据中心的互连](/images/githubpages/vpc-to-datacenter.png)

（6）VPC与其他AWS服务之间的互连，目前只支持S3
	
想要知道VPC与其他AWS服务之间是怎么互连的就不得不提到VPC终端节点。

VPC终端节点是虚拟设备，让账户中的VPC的实例通过私有IP地址与其他AWS服务直接创建私有连接，而无需通过Internet、NAT设备、VPN连接或AWS Direct Connect进行访问。

目前，仅支持带有Amazon S3的连接终端节点。

####VPC四种拓扑图
在我们通过VPC服务创建VPC的时候，有四个可选的拓扑图供我们选择。具体情况如下：

1. 只拥有一个单独公共子网的VPC （A /16 network with /24 subnet)

![打开页面](/images/githubpages/vpc-single-subnet.png =250x)

* 通过Elastic IPs or Public IP来访问Internet

* 通过网络安全组和网络访问控制列表来控制进出网络。

2. VPC with Public and Private Subnets （A /16 network with two /24 subnet）
私有子网通过外部子网利用NAT进行访问（outbound connections）。

![打开页面](/images/githubpages/vpc_with_public_and_private_subnets.png =250x)


3. VPC with Public and Private Subnets and Hard VPN Access
通过VPN（IPSec Virtual Private Network）连接 VPC 和数据中心，一个公共的子网直接连接Internet

![打开页面](/images/githubpages/vpc_with_public_and_private_subnet_vpn.png =250x)

4.  VPC with a Private Subnet Only and Hard VPN Access

通过VPN（IPSec Virtual Private Network）连接 VPC 和数据中心

![打开页面](/images/githubpages/vpc_with_private_subnet.png =250x)



