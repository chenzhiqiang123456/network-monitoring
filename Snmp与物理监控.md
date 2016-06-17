#监控服务器硬件状态IPMI
##一、IPMI介绍9
###**1.1 IPMI介绍**
>  IPMI是智能型平台管理接口（Intelligent Platform Management Interface）的缩写，是管理基于 Intel结构的企业系统中所使用的外围设备采用的一种工业标准，该标准由英特尔、惠普、NEC、美国戴尔电脑和SuperMicro等公司制定。用户可以利用IPMI监视服务器的物理健康特征，如温度、电压、风扇工作状态、电源状态等。而且更为重要的是IPMI是一个开放的免费标准，用户无需为使用该标准而支付额外的费用。 硬件管理接口规格
> 
> 
###**1.2 IPMI工作原理** 


>IPMI的核心是一个专业芯片/控制器（叫做服务器处理器或者基板管理控制器(BMC)），其并不依赖服务器的处理器、BIOS或操作系统来工作，可谓是非常地独立，是一个独立在系统内允许的无代理管理子系统，而BMC通常是一个案子在服务器板上的独立板卡，

>在工作时，所以的IPMI功能都是向BMC发送命令来完成的，命令使用IPMI规范中规定的指令，BMC接受并在系统事件日志中记录事件消息，维护描述系统中传感器情况的传感器数据记录。

>当需要对系统文本控制台进行远程访问时，Serial Over LAN (SOL) 功能将非常有用。SOL 通过 IPMI 会话重定向本地串行接口，允许远程访问Windows 的紧急事件管理控制台 (EMS) 特殊管理控制台 (SAC)，或访问 LINUX 串行控制台。这个过程的步骤是 IPMI固件截取数据，然后通过局域网重新发送定向到串行端口的信息。 这就提供了远程查看BOOT、OS 加载器或紧急事件管理控制台以诊断并修复服务器相关问题的标准方法，而无需考虑供应商。它允许在引导阶段配置各种组件。


##二、IPMI安装
##**2.1 yum安装IPMI**
<pre>
yum install OpenIPMI ipmitool -y 对物理机进行控制和监控
systemctl start ipmi   启动
</pre>

##三、SNMP监控
###**3.1 Snmp 协议的介绍**
> ####SNMP的基本思想：为不同种类的设备、不同厂家生产的设备、不同型号的设备，定义为一个统一的接口和协议，使得管理员可以是使用统一的外观面对这些需要管理的网络设备进行管理。通过网络，管理员可以管理位于不同物理空间的设备，从而大大提高网络管理的效率，简化网络管理员的工作。###

>####SNMP的工作方式：管理员需要向设备获取数据，所以SNMP提供了【读】操作；管理员需要向设备执行设置操作，所以SNMP提供了【写】操作；设备需要在重要状况改变的时候，向管理员通报事件的发生，所以SNMP提供了【Trap】操作。####

>**1. 简单网络管理协议（SNMP－Simple Network Management Protocol)是一个与网络设备交互的简单方法。**
>
>**2. 一个网络设备以守护进程的方式运行SNMP代理，该守护进程能够响应来自网络的各种请求信息**

>**3. 该SNMP代理提供大量的对象标识符（OID－Object Identifiers）。一个OID是一个唯一的键值对， 每个管理对象都有自己的OID(Object Identifier)，**

>**4. SNMP的OID是可读或可写的。尽管向一个SNMP设备写入信息的情况非常少，但它是各种管理应用程序用来控制设备的方法（例如针对交换机的可管理GUI**

>**5. SNMP采用UDP协议在管理端和agent之间传输信息。 SNMP采用UDP 161端口接收和发送请求，162端口接收trap，执行SNMP的设备缺省都必须采用这些端口。SNMP消息全部通过UDP端口161接收，只有Trap信息采用UDP端口162。。**

>**6. 人们就设计了一种将数字OID翻译为人们可读的格式。这种翻译映射被保存在一个被称为 “管理信息基础"（Management Infomation Base) 或MIB的、可传递的无格式文本文件里**  MIB管理信息库

>**7. 使用SNMP或者向SNMP设备查询，你不需要使用MIB，但是，如果没有MIB，你就得猜测你正在查看的数据是什么。**

>**8. 某些情况下，不使用MIB也非常简单，例如查看主机名、磁盘使用率数字，或者端口状态信息。**

>**9. 对于准备编写的应用程序来说，为了让用户避免妥当安装MIB带来的麻烦，而严格使用数字OID很常见。安装一个MIB的动作，只是将他放置到你的SNMP客户端应用软件能够搜索到并进行上述翻译映射工作的某个位置而已。**

>**10. 　管理信息(MIB)库可以理解成为agent维护的管理对象数据库，MIB中定义的大部分管理对象的状态和统计信息都可以被NMS访问。MIB是一个按照层次结构组织的树状结构，每个被管对象对应树形结构的一个叶子节点，称为一个object，拥有唯一的数字标识符**

>**11. SNMP可以按照两种方式来使用：轮询和陷阱。轮询就是说你编写一个应用程序能够设置一个发送给一个SNMP代理查看某些值的SNMP GET请求。这种方法非常有用，因为如果该设备响应了请求，你就得到了你需要的信息，如果该设备没有响应请求，你就能够知道存在某些问题。轮询是网络监控的一种主动形式。**

>**12. 另一方面，SNMP陷阱能够被用来进行被动形式的网络监控。SNMP陷阱是通过配置SNMP设备的代理，让他在某些动作发生时联系另一个SNMP代理来实现的。**

###3.2 SNMP的操作命令
>SNMP协议之所以易于使用，这是因为它对外提供了三种用于控制MIB对象的基本操作命令。它们是：Get、Set 和 Trap。
>
>Get：管理站读取代理者处对象的值。它是SNMP协议中使用率最高的一个命令，因为该命令是从网络设备中获得管理信息的基本方式。
>
>Set：管理站设置代理者处对象的值。它是一个特权命令，因为可以通过它来改动设备的配置或控制设备的运转状态。它可以设置设备的名称，关掉一个端口或清除一个地址解析表中的项等。
>
>Trap： 代理者主动向管理站通报重要事件。它的功能就是在网络管理系统没有明确要求的前提下，由管理代理通知网络管理系统有一些特别的情况或问题 发生了。如果发生意外情况，客户会向服务器的162端口发送一个消息，告知服务器指定的变量值发生了变化。通常由服务器请求而获得的数据由服务器的161 端口接收。Trap 消息可以用来通知管理站线路的故障、连接的终端和恢复、认证失败等消息。管理站可相应的作出处理。

###3.3 SNMP的使用
<pre>
(1)snmp的安装
yum install net-snmp net-snmp-utuls -y
(2)安装后的路径
[root@zabbix-server snmp]# pwd
/etc/snmp
(3)配置snmp
[root@zabbix-server snmp]# cat snmpd.conf
rocommunity oldboy 192.168.56.12
(4)启动snmp
[root@zabbix-server snmp]# systemctl start snmpd
SNMP默认监控UTP 161端口
udp        0      0 0.0.0.0:161             0.0.0.0:*  
SNMP代理 net-snmp 启动
SNMP server   net-snmp-utils

每个MIB的对象是OID
[root@zabbix-server snmp]# snmpget -v2c -c oldboy 192.168.56.12 1.3.6.1.2.1.1.3.0
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (75041) 0:12:30.41 

[root@zabbix-server snmp]# snmpget -v2c -c oldboy 192.168.56.12 1.3.6.1.4.1.2021.10.1.3.1
UCD-SNMP-MIB::laLoad.1 = STRING: 0.01

负载
UCD-SNMP-MIB::laLoad.1 = STRING: 0.00
[root@zabbix-server snmp]# snmpwalk -v2c -c oldboy 192.168.56.12 1.3.6.1.4.1.2021.10.1.3.3
UCD-SNMP-MIB::laLoad.3 = STRING: 0.12
[root@zabbix-server snmp]# snmpwalk -v2c -c oldboy 192.168.56.12 1.3.6.1.4.1.2021.10.1.3.1
UCD-SNMP-MIB::laLoad.1 = STRING: 0.00
[root@zabbix-server snmp]# snmpwalk -v2c -c oldboy 192.168.56.12 1.3.6.1.4.1.2021.10.1.3.2
UCD-SNMP-MIB::laLoad.2 = STRING: 0.04
</pre>


##四、OID总结##
<table class="table table-bordered table-striped table-condensed">
<tr>
    <td colspan=4>系统参数(1.3.6.1.2.1.1)</td>
</tr>
<tr>
    <td>OID</td>
    <td>描述</td>
    <td>备注</td>
	<td>请求方式</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.1.1.0</td>
    <td>获取系统基本信息</td>
    <td>SysDesc</td>
	<td>GET</td>
</tr>

<tr>
    <td>.1.3.6.1.2.1.1.3.0</td>
    <td>监控时间</td>
    <td>sysUptime</td>
	<td>GET</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.1.4.0</td>
    <td>系统联系人</td>
    <td>sysContact</td>
	<td>GET</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.1.5.0</td>
    <td>获取机器名</td>
    <td>SysName</td>
	<td>GET</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.1.6.0</td>
    <td>机器坐在位置</td>
    <td>SysLocation</td>
	<td>GET</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.1.7.0</td>
    <td>机器提供的服务</td>
    <td>SysService</td>
	<td>GET</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.25.4.2.1.2</td>
    <td>系统运行的进程列表</td>
    <td>hrSWRunName</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.25.6.3.1.2</td>
    <td>系统安装的软件列表</td>
    <td>hrSWInstalledName</td>
	<td>WALK</td>
</tr>

</table>

<table class="table table-bordered table-striped table-condensed">
<tr>
    <td colspan=4>系统参数(1.3.6.1.2.1.2)</td>
</tr>
<tr>
    <td>OID</td>
    <td>描述</td>
    <td>备注</td>
	<td>请求方式</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.1.0</td>
    <td>网络接口的数目</td>
    <td>IfNumber</td>
	<td>GET</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.2</td>
    <td>网络接口信息描述</td>
    <td>IfDescr</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.3</td>
    <td>网络接口类型</td>
    <td>IfType</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.4</td>
    <td>接口发送和接收的最大IP数据报[BYTE]</td>
    <td>IfMTU</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.5</td>
    <td>接口当前带宽[bps]</td>
    <td>IfSpeede</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.6</td>
    <td>接口的物理地址</td>
    <td>IfPhysAddress</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.8</td>
    <td>接口当前操作状态[up|down]</td>
    <td>IfOperStatuse</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.10</td>
    <td>接口收到的字节数</td>
    <td>IfInOctet</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.16</td>
    <td>接口发送的字节数</td>
    <td>IfOutOctet</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.11</td>
    <td>接口收到的数据包个数</td>
    <td>IfInUcastPkts</td>
	<td>WALK</td>
</tr>
<tr>
    <td>.1.3.6.1.2.1.2.2.1.17</td>
    <td>接口发送的数据包个数</td>
    <td>IfOutUcastPkts</td>
	<td>WALK</td>
</tr>
</table>


<pre>
System Group
sysDescr 1.3.6.1.2.1.1.1
sysObjectID 1.3.6.1.2.1.1.2
sysUpTime 1.3.6.1.2.1.1.3

sysContact 1.3.6.1.2.1.1.4

sysName 1.3.6.1.2.1.1.5

sysLocation 1.3.6.1.2.1.1.6
sysServices 1.3.6.1.2.1.1.7
Interfaces Group
ifNumber 1.3.6.1.2.1.2.1
ifTable 1.3.6.1.2.1.2.2
ifEntry 1.3.6.1.2.1.2.2.1
ifIndex 1.3.6.1.2.1.2.2.1.1
ifDescr 1.3.6.1.2.1.2.2.1.2
ifType 1.3.6.1.2.1.2.2.1.3
ifMtu 1.3.6.1.2.1.2.2.1.4
ifSpeed 1.3.6.1.2.1.2.2.1.5
ifPhysAddress 1.3.6.1.2.1.2.2.1.6
ifAdminStatus 1.3.6.1.2.1.2.2.1.7
ifOperStatus 1.3.6.1.2.1.2.2.1.8
ifLastChange 1.3.6.1.2.1.2.2.1.9
ifInOctets 1.3.6.1.2.1.2.2.1.10
ifInUcastPkts 1.3.6.1.2.1.2.2.1.11
ifInNUcastPkts 1.3.6.1.2.1.2.2.1.12
ifInDiscards 1.3.6.1.2.1.2.2.1.13
ifInErrors 1.3.6.1.2.1.2.2.1.14
ifInUnknownProtos 1.3.6.1.2.1.2.2.1.15
ifOutOctets 1.3.6.1.2.1.2.2.1.16
ifOutUcastPkts 1.3.6.1.2.1.2.2.1.17
ifOutNUcastPkts 1.3.6.1.2.1.2.2.1.18
ifOutDiscards 1.3.6.1.2.1.2.2.1.19
ifOutErrors 1.3.6.1.2.1.2.2.1.20
ifOutQLen 1.3.6.1.2.1.2.2.1.21
ifSpecific 1.3.6.1.2.1.2.2.1.22
IP Group
ipForwarding 1.3.6.1.2.1.4.1
ipDefaultTTL 1.3.6.1.2.1.4.2
ipInReceives 1.3.6.1.2.1.4.3
ipInHdrErrors 1.3.6.1.2.1.4.4
ipInAddrErrors 1.3.6.1.2.1.4.5
ipForwDatagrams 1.3.6.1.2.1.4.6
ipInUnknownProtos 1.3.6.1.2.1.4.7
ipInDiscards 1.3.6.1.2.1.4.8
ipInDelivers 1.3.6.1.2.1.4.9
ipOutRequests 1.3.6.1.2.1.4.10
ipOutDiscards 1.3.6.1.2.1.4.11
ipOutNoRoutes 1.3.6.1.2.1.4.12
ipReasmTimeout 1.3.6.1.2.1.4.13
ipReasmReqds 1.3.6.1.2.1.4.14
ipReasmOKs 1.3.6.1.2.1.4.15
ipReasmFails 1.3.6.1.2.1.4.16
ipFragsOKs 1.3.6.1.2.1.4.17
ipFragsFails 1.3.6.1.2.1.4.18
ipFragCreates 1.3.6.1.2.1.4.19
ipAddrTable 1.3.6.1.2.1.4.20
ipAddrEntry 1.3.6.1.2.1.4.20.1
ipAdEntAddr 1.3.6.1.2.1.4.20.1.1
ipAdEntIfIndex 1.3.6.1.2.1.4.20.1.2
ipAdEntNetMask 1.3.6.1.2.1.4.20.1.3
ipAdEntBcastAddr 1.3.6.1.2.1.4.20.1.4
ipAdEntReasmMaxSize 1.3.6.1.2.1.4.20.1.5
ICMP Group
icmpInMsgs 1.3.6.1.2.1.5.1
icmpInErrors 1.3.6.1.2.1.5.2
icmpInDestUnreachs 1.3.6.1.2.1.5.3
icmpInTimeExcds 1.3.6.1.2.1.5.4
icmpInParmProbs 1.3.6.1.2.1.5.5
icmpInSrcQuenchs 1.3.6.1.2.1.5.6
icmpInRedirects 1.3.6.1.2.1.5.7
icmpInEchos 1.3.6.1.2.1.5.8
icmpInEchoReps 1.3.6.1.2.1.5.9
icmpInTimestamps 1.3.6.1.2.1.5.10
icmpInTimestampReps 1.3.6.1.2.1.5.11
icmpInAddrMasks 1.3.6.1.2.1.5.12
icmpInAddrMaskReps 1.3.6.1.2.1.5.13
icmpOutMsgs 1.3.6.1.2.1.5.14
icmpOutErrors 1.3.6.1.2.1.5.15
icmpOutDestUnreachs 1.3.6.1.2.1.5.16
icmpOutTimeExcds 1.3.6.1.2.1.5.17
icmpOutParmProbs 1.3.6.1.2.1.5.18
icmpOutSrcQuenchs 1.3.6.1.2.1.5.19
icmpOutRedirects 1.3.6.1.2.1.5.20
icmpOutEchos 1.3.6.1.2.1.5.21
icmpOutEchoReps 1.3.6.1.2.1.5.22
icmpOutTimestamps 1.3.6.1.2.1.5.23
icmpOutTimestampReps 1.3.6.1.2.1.5.24
icmpOutAddrMasks 1.3.6.1.2.1.5.25
icmpOutAddrMaskReps 1.3.6.1.2.1.5.26
TCP Group
tcpRtoAlgorithm 1.3.6.1.2.1.6.1
tcpRtoMin 1.3.6.1.2.1.6.2
tcpRtoMax 1.3.6.1.2.1.6.3
tcpMaxConn 1.3.6.1.2.1.6.4
tcpActiveOpens 1.3.6.1.2.1.6.5
tcpPassiveOpens 1.3.6.1.2.1.6.6
tcpAttemptFails 1.3.6.1.2.1.6.7
tcpEstabResets 1.3.6.1.2.1.6.8
tcpCurrEstab 1.3.6.1.2.1.6.9
tcpInSegs 1.3.6.1.2.1.6.10
tcpOutSegs 1.3.6.1.2.1.6.11
tcpRetransSegs 1.3.6.1.2.1.6.12
tcpConnTable 1.3.6.1.2.1.6.13
tcpConnEntry 1.3.6.1.2.1.6.13.1
tcpConnState 1.3.6.1.2.1.6.13.1.1
tcpConnLocalAddress 1.3.6.1.2.1.6.13.1.2

tcpConnLocalPort 1.3.6.1.2.1.6.13.1.3
tcpConnRemAddress 1.3.6.1.2.1.6.13.1.4
tcpConnRemPort 1.3.6.1.2.1.6.13.1.5
tcpInErrs 1.3.6.1.2.1.6.14
tcpOutRsts 1.3.6.1.2.1.6.15
UDP Group
udpInDatagrams 1.3.6.1.2.1.7.1
udpNoPorts 1.3.6.1.2.1.7.2
udpInErrors 1.3.6.1.2.1.7.3
udpOutDatagrams 1.3.6.1.2.1.7.4
udpTable 1.3.6.1.2.1.7.5
udpEntry 1.3.6.1.2.1.7.5.1
udpLocalAddress 1.3.6.1.2.1.7.5.1.1
udpLocalPort 1.3.6.1.2.1.7.5.1.2
SNMP Group
snmpInPkts 1.3.6.1.2.1.11.1
snmpOutPkts 1.3.6.1.2.1.11.2
snmpInBadVersions 1.3.6.1.2.1.11.3
snmpInBadCommunityNames 1.3.6.1.2.1.11.4
snmpInBadCommunityUses 1.3.6.1.2.1.11.5
snmpInASNParseErrs 1.3.6.1.2.1.11.6
NOT USED 1.3.6.1.2.1.11.7
snmpInTooBigs 1.3.6.1.2.1.11.8
snmpInNoSuchNames 1.3.6.1.2.1.11.9
snmpInBadValues 1.3.6.1.2.1.11.10
snmpInReadOnlys 1.3.6.1.2.1.11.11
snmpInGenErrs 1.3.6.1.2.1.11.12
snmpInTotalReqVars 1.3.6.1.2.1.11.13
snmpInTotalSetVars 1.3.6.1.2.1.11.14
snmpInGetRequests 1.3.6.1.2.1.11.15
snmpInGetNexts 1.3.6.1.2.1.11.16
snmpInSetRequests 1.3.6.1.2.1.11.17
snmpInGetResponses 1.3.6.1.2.1.11.18
snmpInTraps 1.3.6.1.2.1.11.19
snmpOutTooBigs 1.3.6.1.2.1.11.20
snmpOutNoSuchNames 1.3.6.1.2.1.11.21
snmpOutBadValues 1.3.6.1.2.1.11.22
NOT USED 1.3.6.1.2.1.11.23
snmpOutGenErrs 1.3.6.1.2.1.11.24
snmpOutGetRequests 1.3.6.1.2.1.11.25
snmpOutGetNexts 1.3.6.1.2.1.11.26
snmpOutSetRequests 1.3.6.1.2.1.11.27
snmpOutGetResponses 1.3.6.1.2.1.11.28
snmpOutTraps 1.3.6.1.2.1.11.29
snmpEnableAuthenTraps 1.3.6.1.2.1.11.30
</pre>


##四、监控
###**PS :监控分为收集 存储 展示 报警**

<pre>

</pre>
