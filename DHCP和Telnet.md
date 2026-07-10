

带内管理（控制端和被控制端必须可以ping到）---telnet，SSH，web，snmp(简单网络管理协议)

带外管理---console线（串线，只能建立一个点到点的网络），Mini USB







Telenet--远程登录协议

传输层用的TCP协议

登录设备和被登录设备之间需要保证网络的联通
c/s架构（客户端/服务器，客户端受益）

```
[xgz]aaa

[xgz-aaa]local-user xgz privilege level 15 password cipher 123456//用户名叫xgz，访问权限为15（0-15），密码是123456，cipher可以让密码在本地存储中变成密文，防止别人偷看
Info: Add a new user.

[xgz-aaa]local-user xgz service-type telnet//创建一个名为xgz的本地用户并启用Telnet服务

1，创建登录时所使用的用户名和密码
```



```
[xgz]user-interface vty 0 4//开启0-4这五个端口，vty是虚拟登录端口（限制登录人数）

[xgz-ui-vty0-4]authentication-mode aaa//要认证的时候就按照aaa里的用户名和密码进行认证

2，设备开启登录的端口
```



当然，这里如果你是在eNSP里面模拟，用的是路由器代替PC的话，就得给连接被控端的网口配个IP地址

```
<h>telnet 192.168.1.1
  Press CTRL_] to quit telnet mode
  Trying 192.168.1.1 ...
  Connected to 192.168.1.1 ...

Login authentication


Username:xgz
Password:
<xgz>

//这样就从h这个设备登录到xgz里了
```









------------------------

DHCP---动态主机配置协议（自动配置IP地址）
c/s架构
用的UDP协议，端口67，68

67---DHCP服务器

68---DHCP客户端





1.首次获取IP地址
	1.客户端->服务器---广播包---DHCP-DiscoVer

​	应用层：DHCP-Discover

​	传输层：UDP---SPORT:68 DPORT：67

​	网络层：IP--SIP：0.0.0.0 DIP:255.255.255.255

​	数据链路层：以太网---SMAC：自己 DMAC：全F

​	

​	2.服务器->客户端，单播或者广播发（同一广播域下只要知道MAC地址就可实现单播）————DHCP-Offer

​	

​	3.客户端->服务器（以为可能不止一个DHCP服务器给你发Offer，所以你还得回应），广播（得告诉所有服务器我选了哪个，剩下的我没	选）————DHCP-request

​	

​	4.服务器---客户端---DHCP-ACK

2.再次获取IP地址

​	1.客户端->服务器（电脑是有记忆的，会想沿用上次的IP）（以为可能不止一个DHCP服务器给你发Offer，所以你还得回应），广播（得告诉所有服务器我选了哪个，剩下的我没选）————DHCP-request

​	

​	2.服务器---客户端---DHCP-ACK或者DHCP-NCK（IP被人用了，不给）

​	如果不给的话，就把上面四次重新走一遍



------------------------------

租期---1天

T1时间---50%---12小时
	1，客户端---服务器---单播---DHCP-request
	2，服务器---客户端---单播/广播---DHCP-ACK
T2时间---87.5%---21小时
	1，客户端---服务器---广播--DHCP-request
	2，服务器---客户端---单播/广播---DHCP-ACK





------------------

配置DHCP服务器（一般在路由器上）



```
[xgz]dhcp enable 

开启DHCP服务
```





```
[xgz]ip pool myn
创建一个名叫myn的IP池


Info: It's successful to create an IP address pool.
[xgz-ip-pool-myn]
```



```
[xgz-ip-pool-myn]network 192.168.1.0 mask 24
将192.168.1.0的网段倒到池子里，配置地址池

[xgz-ip-pool-myn]gateway-list 192.168.1.1
配置网关

[xgz-ip-pool-myn]dns-list 114.114.114.114
配置DNS（114是没问题的）
```

```
[xgz]interface GigabitEthernet 0/0/0
[xgz-GigabitEthernet0/0/0]dhcp select global 
真正干活的是网关，所以要把全局配置给到0/0/0这个接口
```

可以同时将多个IP池配到同一个网关，会根据网关的IP地址自动分配，不用担心混乱

























