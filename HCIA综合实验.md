### 综合实验



**先配VLAN**

​	这是底层的“物理和链路层”准备。只有先把 VLAN 标签剥掉（dot1q），给子接口配好 IP 地址，路由器之间以及路由器和交换机之间才能建立物理连通性。这是后续所有协议跑起来的前提。

```
[L1]vlan 2

[L1]interface GigabitEthernet 0/0/2


[L1-GigabitEthernet0/0/2]port link-type access 
接口配置属于什么类型

[L1-GigabitEthernet0/0/2]port default vlan 2
```



```
[L1]vlan 3
[L1]vlan 2

[L1]interface GigabitEthernet 0/0/1

[L1-GigabitEthernet0/0/1]port link-type trunk 

[L1-GigabitEthernet0/0/1]port trunk allow-pass vlan 2 3

```



```
[r1\]interface GigabitEthernet 0/0/1.1

[r1\-GigabitEthernet0/0/1.1]ip address 192.168.1.60 25

[r1\-GigabitEthernet0/0/1.1]dot1q termination vid 2

[r1\-GigabitEthernet0/0/1.1]arp broadcast enable

举一反三去吧
```







**再配OSPF**

```
[r1]ospf 1 router-id 1.1.1.1

[r1-ospf-1]area 0

[r1\-ospf-1-area-0.0.0.0]network 192.168.1.128 0.0.0.63
[r1\-ospf-1-area-0.0.0.0]network 192.168.1.60 0.0.0.63
[r1\-ospf-1-area-0.0.0.0]network 192.168.1.120 0.0.0.63

只要是路由器的接口并且在这个范围内，要加进来
```



```
[r2]ospf 1 router-id 2.2.2.2

[r2-ospf-1]area 0

[r2-ospf-1-area-0.0.0.0]network 192.168.1.128 0.0.0.63
[r2-ospf-1-area-0.0.0.0]network 192.168.1.59 0.0.0.63
[r2-ospf-1-area-0.0.0.0]network 192.168.1.199 0.0.0.63

如果 R2 把 12.0.0.1 宣告进 OSPF 区域 0，它会一直试图和 AR3 建立邻居，但由于 AR3 根本不回应 OSPF 报文，R2 会疯狂报错，且邻居永远起不来
```









**之后配DHCP**

```
[r1\]dhcp enable

[r1\]ip pool l2

[r1\-ip-pool-l2]network 192.168.1.0 mask 27

[r1\-ip-pool-l2]gateway-list 192.168.1.1 

[r1\-ip-pool-l2]dns-list 114.114.114.114

[r1\]interface GigabitEthernet 0/0/0.1

[r1\-GigabitEthernet0/0/0.1]dhcp select global


```



```
[r1\]ip pool l3

[r1\]ip pool l3

[r1\-ip-pool-l3]network 192.168.1.32 mask 27

[r1\-ip-pool-l3]gateway-list 192.168.1.33

[r1\-ip-pool-l3]dns-list 114.114.114.114

[r1\-GigabitEthernet0/0/1.2]dhcp select global 

```

这是left路由器的，right自己写吧









![image-20260723000315497](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260723000315497.png)

到第五步了





网关搅一块了

结果发现ping都ping不通😑😣😫😢😨😰😭😵



### 作者这里搅在一起了，全删了重写



**VLAN**

```

[L1]vlan 2

[L1]interface GigabitEthernet 0/0/2

[L1-GigabitEthernet0/0/2]port link-type access 

[L1-GigabitEthernet0/0/2]port default vlan 2

```



```
骨干路线
[L1]interface GigabitEthernet 0/0/1

[L1-GigabitEthernet0/0/1]port link-type trunk 

[L1-GigabitEthernet0/0/1]port trunk allow-pass vlan 2 3

```

把L1和L2都配了









```
[r1]interface GigabitEthernet 0/0/1.2

[r1-GigabitEthernet0/0/1.2]ip address 192.168.1.1 26

[r1-GigabitEthernet0/0/1.2]dot1q termination vid 2

[r1-GigabitEthernet0/0/1.2]arp broadcast enable 
然后给R1配
```

```
[r1]interface GigabitEthernet 0/0/1.3

[r1-GigabitEthernet0/0/1.3]ip address 192.168.1.65 26

[r1-GigabitEthernet0/0/1.3]dot1q termination vid 3

[r1-GigabitEthernet0/0/1.3]arp broadcast enable 

```



记得把R2也配下









**DHCP**



 ```
 [r1]dhcp enable 
 
 [r1]ip pool l2
 
 [r1-ip-pool-l2]network 192.168.1.0 mask 26
 
 [r1-ip-pool-l2]gateway-list 192.168.1.1
 
 [r1-ip-pool-l2]excluded-ip-address 192.168.1.32 192.168.1.62
 禁止这个池子用192.168.1.32到192.168.1.62的地址
 
 [r1-ip-pool-l2]dns-list 114.114.114.114
 
 [r1-ip-pool-l2]qu
 
 [r1]interface GigabitEthernet 0/0/1.2
 
 [r1-GigabitEthernet0/0/1.2]dhcp select global 
 
 ```



```

[r2]ip pool r2

[r2-ip-pool-r2]network 192.168.1.0 mask 26

[r2-ip-pool-r2]gateway-list 192.168.1.2

[r2-ip-pool-r2]excluded-ip-address 192.168.1.3 192.168.1.31

[r2-ip-pool-r2]dns-list 114.114.114.114

[r2-ip-pool-r2]qu

[r2]interface GigabitEthernet 0/0/2.2

[r2-GigabitEthernet0/0/2.2]dhcp select global 
```



把192.168.1.64/26也配下













