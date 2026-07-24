ACL————访问控制列表

作用：

​	1，访问控制——在路由器流量流入或者流出的接口上，匹配流量。之后执行相应的动作--允许permit，拒绝deny（当然，不止路由器可以做）
​	2，抓取感兴趣流——ACL可以和其他服务结合使用，ACL只负责抓取流量，而其他服务负责执行相应动作



相当于是个看门的保安，做出允许流量通过和拒绝流量通过的行为，那么他肯定会有一套规则，一张表，那么，他会怎么看表呢

​	匹配规则---自上而下，逐一匹配，如果匹配到了，则执行对应的动作，不再向下匹配。

思科设备---ACL列表末尾隐含了一条拒绝所有的规则。

华为设备--- ACL列表末尾没有隐含规则（表上没有的话相当于此处不存在ACL，其实就是不管）

​	我们这次讲华为的：

ACL的分类：
	基础ACL——主要关注数据包中的源IP地址
	高级ACL——可以关注数据包中的源IP地址，也可以关注数据包中的目标IP地址，协议以及端口号。

​	二层ACL
​	用户自定义ACL



------------------------

好，我们开始配置吧

![image-20260719002436322](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719002436322.png)

**基础ACL**

### 需求一：PC1可以访问3.0网段，但是PC2不行

​	基础ACL的位置原则：因为基础ACL仅关注源IP地址，可能造成误伤，所以，部署时，应该尽可能的靠近目标，减少误伤。

```
[r2]acl 2000 
[r2-acl-basic-2000]

具体序号分类自己敲问号

创建ACL——————————————————————————————
```



```
[r2-acl-basic-2000]rule deny source 192.168.1.3 0.0.0.0
[r2-acl-basic-2000]
这里指的是拒绝192.168.1.3

配置ACL的规则————————————————————————

0.0.0.0是通配符，作用和反掩码一样，0代表不可变，1代表可变，注意，通配符可以让0和1穿插使用，比如0.255.0.255


[r2-acl-basic-2000]rule permit source any 
放通所有
```

![image-20260719003432123](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719003432123.png)







这里是查看视图

------------------------------

------------------------

```
[r2-acl-basic-2000]display acl 2000
Basic ACL 2000, 1 rule
Acl's step is 5
 rule 5 deny source 192.168.1.3 0 

[r2-acl-basic-2000]

查看规则列表————————————————————————————
```

注意：ACL列表会默认以5为步调添加序号，方便添加，删除规则

```
[r2-acl-basic-2000]rule 7 permit source 192.168.1.1 0.0.0.0

自定义序号配置规则（序号为7）——————————————
```

```
[r2-acl-basic-2000]undo rule 7

删除序号7————————————————————————————
```

------------------------

------------------









创建和配置都好了，现在该调用了

```
[r2-GigabitEthernet0/0/1]traffic-filter outbound  acl 2000
[r2-GigabitEthernet0/0/1]

作为流出接口应用acl2000的规则表
```

![image-20260719005055939](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719005055939.png)

怎么看是流出还是流入，很简单，看从源IP到这个网段经过你这个接口的时候对于本路由器是流出还是流入就行了



注意：一个接口的一个方向上，只能调用一张ACL列表







**高级ACL**

![image-20260719005640419](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719005640419.png)

### 需求二：PC1只能访问PC3但是不能访问PC4

​	高级ACL的位置原则：因为高级ACL可以实现精确的匹配，所以，越靠近源越好，这样可以节省链路资源



```
[r1]acl 3000
[r1-acl-adv-3000]

创建高级列表————————————————————————————
```



```
[r1-acl-adv-3000]rule deny ip source 192.168.1.2 0.0.0.0 destination 192.168.3.3
 0.0.0.0
[r1-acl-adv-3000]

配置高级规则————————————————————————————————

拒绝192.168.1.2到192.168.3.3的IP协议
```

![image-20260719010849539](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719010849539.png)



```
[r1-GigabitEthernet0/0/1]traffic-filter inbound acl 3000
[r1-GigabitEthernet0/0/1]

应用————————————————————————————————————
```

完成啦！



### 需求三：PC10不能telnetR1，只能ping通R1

​	高级ACL的位置原则：因为高级ACL可以实现精确的匹配，所以，越靠近源越好，这样可以节省链路资源



![image-20260719013220111](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719013220111.png)




____

现在我要PC10来模拟电脑使用telnet

怎么解决PC10因为没有网关而ping不到其他网段的问题

给他加个缺省路由就行了

[PC]ip route-static 0.0.0.0 0 192.168.3.1

顺嘴一提罢了

------------



```
[r2]acl 3300

创建acl
```



```
[r2-acl-adv-3300]rule deny tcp source 192.168.3.10 0.0.0.0 destination 192.168.2.1 0.0.0.0 destination-port eq 23
[r2-acl-adv-3300]

拒绝tcp协议和端口等于23的
```

![image-20260719015023578](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719015023578.png)

分别是等于，大于，小于，两端口之间





```
[r2-GigabitEthernet0/0/1]traffic-filter inbound acl 3300

应用
```



结果：

```
<PC>telnet 192.168.2.1
  Press CTRL_] to quit telnet mode
  Trying 192.168.2.1 ...
  Error: Can't connect to the remote host
<PC>
<PC>ping 192.168.2.1
  PING 192.168.2.1: 56  data bytes, press CTRL_C to break
    Reply from 192.168.2.1: bytes=56 Sequence=1 ttl=254 time=60 ms
    Reply from 192.168.2.1: bytes=56 Sequence=2 ttl=254 time=50 ms
    Reply from 192.168.2.1: bytes=56 Sequence=3 ttl=254 time=60 ms
    Reply from 192.168.2.1: bytes=56 Sequence=4 ttl=254 time=30 ms
    Reply from 192.168.2.1: bytes=56 Sequence=5 ttl=254 time=60 ms

  --- 192.168.2.1 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 30/52/60 ms

<PC>

成功！

```

不过，想要防止telnet的话需要把本路由器的所有接口都设置规则，都得封锁，不然PC10仍然可以telnet192.168.1.1来控制R1











注：

```
[r1]acl name xgz 3001
[r1-acl-adv-xgz]

通过重命名的方式配置创建的ACL列表
```





![image-20260719020133909](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260719020133909.png)

作业