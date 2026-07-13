VLAN虚拟局域网-虚拟广播域

V虚拟
LAN局域网

VLAN---交换机和路由器协同工作后，将原来的一个广播域逻辑上分割成多个虚拟的广播域。



### 第一步：创建VLAN

```
<Huawei>display vlan

查看vlan（这是在交换机敲的命令）
```

![image-20260713135032812](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260713135032812.png)

注意：交换机默认存在VLAN1，并且所有的接口默认属于VLAN1

VID---VLAN ID----12位二进制构成---1-4094





```
[Huawei]vlan 2

创建VLAN（1-4094）
```

```
[Huawei]vlan batch 4 to 100

批量创建
```



### 第二步：将接口划入VLAN



![image-20260713151221362](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260713151221362.png)



那么问题来了，PC1和PC3想给另一个路由器的同一个VALN发送消息时，LSW2该怎么区分来的数据是交给VLAN2还是VLAN3的呢

![image-20260708103112945](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260708103112945.png)

这是数据链路层的以太网二型帧



而802.1Q中，在以太网Ⅱ型帧源MAC地址和类型字段之间（作者不会英语，是Source Address和Type之间），增加了4个字节的tag（这里面肯定包含了12位的VID），这个叫802.1Q帧

802.1Q帧---tagged帧

普通的数据帧--untagged帧



根据以上特性，我们将**交换机和终端设备之间的连线称为access链路**，交换机侧的接口称为access接口，这样的链路只能通过untagged帧，且这些数据帧只能属于某一种vlan；**交换机和交换机之间的链路称为trunk干道**，交换机侧的接口称为trunk接口，这样的链路可以通过tagged帧，且这些数据帧可以属于多种不同的VLAN。



### 第三步：配置access链路

```
[L2-GigabitEthernet0/0/1]port link-type access（可修改assess）
翻译：端口接口类型
在0/0/1接口配置属于什么类型
```

```
[L1-GigabitEthernet0/0/1]port default vlan 2
翻译：端口默认值
配置此条路段可以给哪种VLAN通过
```

------------

**小知识：**

```
[L1]port-group group-member GigabitEthernet 0/0/3 GigabitEthernet 0/0/4

进入一个端口组，这样可以同时配置0/0/3和0/0/4接口
```

------------

### 第四步：配置trunk干道

```
[L1-GigabitEthernet0/0/5]port link-type trunk 

在0/0/5接口配置属于什么类型
```

```
[L1-GigabitEthernet0/0/5]port trunk allow-pass vlan 2 3

表示这条干路允许VLAN2和VLAN3通信
```

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260713164655130.png" alt="image-20260713164655130" style="zoom:80%;" />

配置好之后，PC1，PC2，PC5是一个广播域，PC3，PC4，PC6是一个广播域

但是，PC1无法和PC3之间通信，因为在不同广播域，这时候我们需要交换机

### 第五步：VLAN间路由

其实就是添个路由器，两个广播域，一个广播域对应一条线

多臂路由

![image-20260713170435443](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260713170435443.png)



就实现了VLAN技术，广播域的个数不再被交换机个数所限制

！？强强？！

------------------

那么我想更节省成本，路由器上一个物理接口对应一个广播域还是太贵了，有没有什么更好的方法

有的兄弟有的



可以用子接口（为什么叫子接口的，其实是原本一个真实的物理接口下衍生出来的虚拟接口）

```
[xgz]interface GigabitEthernet 0/0/0.1

开通一个在0/0/0下的子接口
```

```
[xgz-GigabitEthernet0/0/0.1]ip address 192.168.1.254 24
设置子接口网关

[xgz-GigabitEthernet0/0/0.1]dot1q termination vid 2
让子接口管理VLAN2的流量

[xgz-GigabitEthernet0/0/0.1]arp broadcast enable 
开启ARP广播应答（因为子接口默认无法应答）
```

![image-20260713235022855](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260713235022855.png)



那么现在就完全实现单臂路由了

------------

小知识：如何修改本接口的类型（access，trunk）

```
当时是无法直接修改的，得恢复默认，那么怎么恢复默认呢，默认VLAN是1呀
所以直接
port default vlan 1
就可以了
```

------------

![image-20260714000955185](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260714000955185.png)

OK，写作业吧