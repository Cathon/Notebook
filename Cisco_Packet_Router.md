	Theme: EasyIn Lab 
	Author: Cathon Z.H.D
	Time: 2014.9.30

**Content**

+ <a href="#mmhy">密码还原</a>
+ <a href="#pzip">配置路由接口ip</a>
+ <a href="#static">静态路由</a>
+ <a href="#dhcp">DHCP配置</a>
+ <a href="#dns">DNS配置</a>
+ <a href="#vlan">交换机配置</a>
+ <a href="#singlerouter">单臂路由</a>
+ <a href="#acl">ACL访问控制</a>

	
How to use Cisco Packet Tracer
==============================

#<center>基础知识</center>

###OSI模型层面:

    应用层：提供了操作系统平台
    表示层：进行编码方式的转换
    会话层：进行资源的分配
    传输层：根据端口号进行识别
    网络层：提供三层地址（ip地址）
    链路层：提供二层标识节点,硬件地址(mac地址)
    物理层：规范化硬件标准


###ip地址
	类	前8位		默认掩码		特殊地址
	A	1-126		/8  
	B	128-191		/16  
	C	192-223		/24			单播地址  
	D	224-239					组播地址  
	E	240-255					科研地址  

	127		环回地址簇，作环回测试

###Mac地址（48bit):

    0~24bit		厂商id(由IEEE分配)
    25~48bit	网卡唯一id(厂家分配)

###子网掩码

- 用于区分网络位和主机位
- 从左起，二进制ip地址中，1必须连续
- 可以没有1，即可以是0.0.0.0  -->  空地址  

**注意**

- 同一网段，ip地址网络位一致  
- 网络位不可变化，主机位可以变化 
- 主机位不能全是0或者1
- 本网段的广播地址是 -->  主机位全是1

**练习：**

	1. 10.1.1.1/25  -->		网段：10.1.1.1-126
	2. 0.0.0.0  	-->		空地址  
	3. 255.255.255.255  --> 全网广播地址

###主类号
+ 即用默认掩码匹配得到的网络号  
+ 100.1.1.1 的主类号是 100.0.0.0  
+ 172.123.12.24 的主类号是 172.123.0.0  

###网线
+ 颜色：白橙 橙 白绿 蓝 白蓝 绿 白棕 棕  
+ [如何制作网线](http://jingyan.baidu.com/article/7e440953f107532fc0e2ef18.html)  
+ 同构设备相连用交叉线  
+ 异构设备相连用直通线


#<center>设备配置</center>

**_注意：跨网段通讯一定要设置网关!!!!!!_**


**四种模式：**  
`用户模式：Router> en (enable进入特权模式)`  
`特权模式：Router# conf t（configure terminal进入全局配置模式）`  
`全局配置模式：Router(config)#`  
`子接口模式`

**基本命令：**  
`查询: ?`  
`返回到上一级模式: ex (exit)`  
`返回到特权模式: end`  
`查看配置文件: #show running-config`  
`------------#show startup-config`  
`保存当前配置: #w (write)` 


**Router:(in CLI)**  
`修改名字: (config)#hostname xxx`

`配置密码: (config)#enable password xxx`  
`删除密码: (config)#no enable password`  
`---------(config)#enable secret xxx`  
`---------(config)#no enable secret`

+ 设置远程登录
+ 串口路由互连

**<a name="mmhy">密码还原</a>**  
关于寄存器:  
+ 0x2142 不加载配置文件  
+ 0x2102 加载配置文件  

1. 重启路由器,自检结束前按下ctrl+break  
2. 进入rommon模式修改寄存器值  
`----rommon >confreg 0x2142`(不加载配置文件)  
3. `--rommon >boot`  
4. 进入路由器特权模式下进行startup-config的复制  
`----#copy startup-config running-config`  
5. 进入全局配置模式(configure terminal)修改密码  
`----(config)#no enable secret`  
6. 修改寄存器值  
`----(config)#config-register 0x2102`(加载配置文件)  
7. 查看寄存器是否被修改  
`----#show version`  
8. `--#write`  
9. `--#reload`

**<a name="pzip">配置路由器接口ip</a>**  
+ 路由器上的不同接口要配置不同的网段  
+ 接口不够时,可以添加`WIC-1ENET`模块

1. 进入接口配置模式  
`(config)#in f0/0`即  
`(config)#interface fastEthernet 0/0`
2. 添加ip及子网掩码  
`(config-if)#ip address 1.1.1.1 255.255.255.0`  
3. 打开接口 ----------(关闭接口是: `shutdown`)  
`(config-if)#no shutdown`
4. 查看接口信息  
`#show ip interface brief`





**<a name="static">静态路由</a>**  
+ 不同网段通讯时要配置网关！  
+ `#ping ip` //测试直连是否通  
+ `#(config)#ip route [ip] [subnet mask] [next hop]`  
_其中ip为要目的ip（通常是网络号,表示一个网段），目标子网掩码,net hop指下一跳ip，即转发的路由器的接收端口_  
+ `#ip route 0.0.0.0 0.0.0.0 [next hop]`//_转发所有包_  
+ `#ip route [ip] 255.255.255.255 [next hop]`//指定ip通信  
+ `#show ip route` //查看本地路由器路由表


**<a name="dhcp">DHCP</a>**  
分配过程：  
client------>discovery找服务器  
server------>offer响应，给地址  
client------>request  
server------>response  


1. `(config)#ip dhcp pool [name]`  
	//_可以设置多个DHCP池_  
2. `(dhcp-config)#network [网段] [掩码]`  
3. `(dhcp-config)#default-router [路由接口ip地址]`
4. `dhcp(config)#interface FastEthernet 0/0`  
5. `dhcp(config-if)#ip add 192.168.1.254 255.255.255.0`  
	//_不能设置为...255 -->这是广播地址_
6. `dhcp(config-if)#no shutdown`  
7. `dhcp(config)#ip dhcp excluded-address 192.168.1.254`  
	//_去除接口ip_


**<a name="dns">DNS服务器</a>**  
+ 路由器为子网主机设置默认DNS服务器  
+ DNS服务器设置A record,并打开(on)DNS开关

**<a name="vlan">vlan虚拟局域网/交换机配置/trunck</a>**  
+ 生成树协议：若两个交换机之间有多个接口会堵掉多余的口，只剩一个口可通  
+ vlan的实质是，数据包是否能通过交换机的两个接口，同vlan可通，不同vlan不可通  
+ 交换机分access(普通)模式和trunck(特殊)模式  
+ 普通模式只通过vlan 1, 而特殊模式可以通过所有的vlan  

	+ Switch(config)#int f0/1  
	+ Switch(config-if)#switchport mode trunk   

1. `Switch#show vlan`  
2. `Switch(config)#do show vlan`  
	//_查看vlan：默认初始全部在vlan1_  
3. `Switch(config)#vlan 10`  
	//_创建vlan_  
4. `Switch(config-vlan)#name vvv`  
5. `Switch(config)#int fastEthernet 0/1`  
6. `Switch(config-if)#switchport mode access`  
7. `Switch(config-if)#switchport access vlan 10`

或者  
1. `Switch(config)#vlan 10`  
2. `Switch(config-vlan)#int f0/0`  
3. `Switch(config-if)#switchport access vlan 10`  
4. `switchport access vlan 10`  



**lo本地环回**
	
	添加环回口
	(config)#interface loopback [number]
	(config-if)#ip address 192.168.100.1 255.255.255.0

**<a name="singlerouter">单臂路由/子接口</a>**  
作用：实现不同vlan之间的通讯  
子接口的概念  

	交换机上：
	设置接口为trunck模式
	
	路由器下：
	Router(config)#int f0/0.10  
	Router(config-subif)#encapsulation dot1Q 10	//封装，用来读trunck,可以通过valn 10
	Router(config-subif)#ip add 192.168.1.1 255.255.255.0
	
	Router(config)#int f0/0.20
	Router(config-subif)#encapsulation dot1Q 20
	Router(config-subif)#ip add 192.168.2.1 255.255.255.0
	Router(config-subif)#ex
	
	Router(config)#int f0/0	//进入主接口打开接口
	Router(config-if)#no shutsown

**<a name="acl">ACL--访问控制列表</a>**

	两个动作：
		1. permit
		2. deny
	
	(config)#access-list [number 1-99] permit/deny [ip]
	(config)#int f0/0  		//接口挂接
	(config-if)#ip access-group [number 1-99] in
	注意：
		1.最后匹配默认的隐含拒绝所有--access-list 1 deny any
		2.每个接口只可设置一个ACL访问控制列表  
		3.一个ACL访问控制列表可以设置多个条件，按优先级判断
	
	删除：
	(config)#no access-list [number]
	(config)#int f0/0		// 删除借口挂接
	(config-if)#no ip access-group [number] in

**如何排错**  

`#show running-config----------查看当前配置文件`  
`#show ip route----------------查看本地路由器的路由表`  
`#show vlan--------------------查看vlan（包括接口，命名）`  
`#show access-lists------------查看ACL语句`  
`#show ip interface brief------查看接口配置和up down状态`


 
**后续：CCNA-CCNP-CCIE:**
	
	CCNA内容：

	报文协议  
	路由协议--静态路由
			-动态路由（rip,ospf,eigrp）
	交换--vlan--单臂路由
		--vtp
	ACL,NAT
	等等...

<hr>


