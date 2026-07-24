NAT————网络地址转换



在IP地址空间中，A，B，C三类地址中各有一部分地址，他们被称为私网地址（私有地址），其余的地址为公网地址（公有地址）。

A：10.0.0.0-10.255.255.255 ---1个A类网段

B：172.16.0.0-172.31.255.255---16条B类网段

C：192.168.0.0-192.168.255.255 -- 256条c类网段



NAT——网络地址转换——实现公网地址和私网地址之间的转换



就比如说，我们公司内算私网，你在本私网内的传输用的是你的私网IP，一旦要访问公网，就会自动转化为公网IP，非常节省IP地址







![image-20260719210853819](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719210853819.png)

华为设备所有和NAT相关的配置都是在**边界设备的出接口**上配置的，图上就是AR2



### **静态NAT**

一对一的NAT —— 静态NAT就是在边界设备上维护一张静态地址映射表，映射表中记录的是私网地址和公网地址一一对应的关系。（你可能会说这根本没法增加IP地址的利用率）

不急，我们先配下

```
[r2-GigabitEthernet0/0/2]nat static global 12.0.0.3 inside 192.168.1.2
[r2-GigabitEthernet0/0/2]

配置了一个静态NAT,内网129.168.1.2对应公网12.0.0.3
```

![image-20260719231817893](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719231817893.png)



```
[r2-GigabitEthernet0/0/2]display nat static 
  Static Nat Information:
  Interface  : GigabitEthernet0/0/2
    Global IP/Port     : 12.0.0.3/---- 
    Inside IP/Port     : 192.168.1.2/----
    Protocol : ----     
    VPN instance-name  : ----                            
    Acl number         : ----
    Netmask  : 255.255.255.255 
    Description : ----

  Total :    1
[r2-GigabitEthernet0/0/2]


查看——————————————————————————
```

在静态NAT，这个12.0.0.3是漂浮地址，有点绑定内网那个地址的意思





你知道的，这根本没法增加IP地址的利用率





### **动态NAT**

多对多（想象成一个终端用完地址之后，可以把地址给其他人用），不过同一时间还是一对一NAT

​	1，创建一个公网地址池

```
[r2]nat address-group 0 12.0.0.3 12.0.0.5

创建序号为0的公网地址池，12.0.0.3，12.0.0.4，12.0.0.5
```



​	2，设置一个通过ACL抓取私网流量（抓来给NAT用）

```
[r2]acl 2000
[r2-acl-basic-2000]rule permit source 192.168.0.0 0.0.255.255

允许192.168.0.0/16网段通过
```



​	3，做动态NAT

```
[r2-GigabitEthernet0/0/2]nat outbound 2000 address-group 0 no-pat

把ACL 2000和序号为0的公网地址池绑定起来
```

配好啦！！



然后从体验效果来看并不好，虽然都能通

### **NAPT**——PAT——网络地址端口转换

那该怎么解决这个问题：

​	其实理想状态下，应该是一个内网用一个公网IP对吧，但是回来的数据包边界设备压根分辨不出来是给内网里谁的，所以需要有东西进行标记，就用端口号吧

​	比如：Sport：5555      Dport：80

​		  SIP：192.168.1.2   DIP：1.1.1.1



​		  Sport：6666      Dport：80

​		  SIP：192.168.1.3   DIP：1.1.1.1

端口号在传输时不会变，记端口就行了，边界设备（也就是AR2）里写上端口和IP地址的对应关系就行了（你可能会问，端口号是随机的，重复了怎么办？AR2都能改IP地址了，端口号自然也能改啊，如下）

表内：192.168.1.2//5555<=>12.0.0.1//5555

​	  192.168.1.3//5555<=>12.0.0.1//6666（甚至端口重复了都没关系）





#### 有两种：

​	**Easy ip     一对多**

1，设置一个通过ACL抓取私网流量（抓来给NAT用）

```
[r2]acl 2000
[r2-acl-basic-2000]rule permit source 192.168.0.0 0.0.255.255

允许192.168.0.0/16网段通过
```



2,配置easy ip（为什么不要地址池了，因为一对多，一就是12.0.0.1）

```
[r2-GigabitEthernet0/0/2]nat outbound 2000

很简单
```









​	**NAPT        多对多**（跟动态NAT差不多）

​	1，创建一个公网地址池

```
[r2]nat address-group 0 12.0.0.3 12.0.0.5

创建序号为0的公网地址池，12.0.0.3，12.0.0.4，12.0.0.5
```



​	2，设置一个通过ACL抓取私网流量（抓来给NAT用）

```
[r2]acl 2000
[r2-acl-basic-2000]rule permit source 192.168.0.0 0.0.255.255

允许192.168.0.0/16网段通过
```



​	3，做动态NAT

```
[r2-GigabitEthernet0/0/2]nat outbound 2000 address-group 0
注意，这里没有no-nat
把ACL 2000和序号为0的公网地址池绑定起来
```

也配好啦！！





### **端口映射**







以下如图：

![image-20260720005750466](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260720005750466.png)

那么，刚才是私网对公网的通信，那么，公网设备怎么访问私网里面的东西呢，只要在边界设备（AR2）上加一条：

​	192.168.1.10//80<=>12.0.0.1//80

```
[r2-GigabitEthernet0/0/2]nat server protocol tcp global 12.0.0.1 80 inside 192.
168.1.10 80


[r2-GigabitEthernet0/0/2]nat server protocol tcp global current-interface 80 ins
ide 192.168.1.10 80


二者都可，current-interface指的是本端口，有些时候直接写本端口不允许
```









问题来了，如果我这个内网里有两个服务器呢，都用的80端口啊

![image-20260720011344678](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260720011344678.png)

其实也很好想到：

​	192.168.1.20//80<=>12.0.0.1//8080

让外面访问8080端口就行了

外面访问时会这么访问：http://12.0.0.1:8080

```
[r2-GigabitEthernet0/0/2]nat server protocol tcp global current-interface 8080 ins
ide 192.168.1.20 80
```





