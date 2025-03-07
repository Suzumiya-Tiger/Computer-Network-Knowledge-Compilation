# TCP的三次握手

![image-20250224142433482](D:\HeinrichHu\resource\computer network knowledge\02_TCP的三次握手和四次挥手.assets\image-20250224142433482.png)

## 流程概述

客户端和服务端第一次建立连接时，会通过 **SYN**建立连接，并且会生成一个 **seq**序列号。

服务端获取到客户端的seq序列号后，会生成 **ack序列号**，**它是根据客户端的seq+1生成的**。

注意，**服务端也会生成一个自己的seq的序列号**，然后再拼接 **SYN**发送回给客户端。

客户端收到服务端的seq，它会判断该服务端的seq是否正确。

紧接着客户端会再次生成一个seq，**这个seq是客户端第一次生成的一个seq+1生成的**，**而ack则是获取到的服务端的seq+1生成的**。

**如果正确就打一个ACK标记，使得ACK=1**，此时三次握手已经完成。

![image-20250224143601549](D:\HeinrichHu\resource\computer network knowledge\02_TCP的三次握手和四次挥手.assets\image-20250224143601549.png)

## 总结

### 第一次握手 (客户端 -> 服务端)

**客户端发送 SYN 包**

SYN=1

seq = x (x为随机生成的初始序列号)

**客户端进入 SYN_SENT 状态**

### 第二次握手 (服务端 -> 客户端)

**服务端发送 SYN+ACK 包**

**SYN=1, ACK=1**

seq = y (y为服务端随机生成的初始序列号)

ack = x + 1 (客户端的seq+1)

**服务端进入 SYN_RECV 状态**

### 第三次握手 (客户端 -> 服务端)

**客户端发送 ACK 包**

**ACK=1**

seq = x + 1 (客户端第一次的seq+1)

ack = y + 1 (服务端的seq+1)

**连接建立成功，双方进入 ESTABLISHED 状态**





**需要补充的要点：**

每次握手都有特定的**标志位状态（SYN, ACK）**

需要说明双方在握手过程中的**状态变化（SYN_SENT, SYN_RECV, ESTABLISHED）**

三次握手的主要目的是：

- 同步双方的序列号
- 确认双方的接收和发送能力
- 防止历史连接的建立，避免产生错误的连接
- **任何一次握手失败或超时，都会导致连接建立失败**
- **整个过程是为了保证双方都能确认对方的收发能力是正常的**



# TCP的四次挥手

![image-20250224144801173](D:\HeinrichHu\resource\computer network knowledge\02_TCP的三次握手和四次挥手.assets\image-20250224144801173.png)

### 第一次挥手（客户端 -> 服务端）

**客户端发送 FIN 包，表示客户端想要关闭连接**

客户端生成序列号 seq = u（u为当前序列号）

**FIN=1**

**客户端进入 FIN_WAIT_1 状态**

### 第二次挥手（服务端 -> 客户端）

**服务端收到FIN后，发送 ACK 确认包**

**ACK=1**

ack = u + 1（客户端的seq+1）

**服务端进入 CLOSE_WAIT 状态**

**客户端收到ACK后，进入 FIN_WAIT_2 状态**

### 第三次挥手（服务端 -> 客户端）

**服务端处理完剩余数据后，发送 FIN+ACK 包**

**FIN=1, ACK=1**

服务端开始生成序列号 seq = w（w为服务端的序列号）

ack = u + 1

服务端进入 LAST_ACK 状态

### 第四次挥手（客户端 -> 服务端）

**客户端发送最后的 ACK 确认包**

ACK=1

seq = u + 1

**ack = w + 1（服务端的seq+1）**

客户端进入 TIME_WAIT 状态，等待2MSL时间后关闭

服务端收到ACK后直接进入 CLOSED 状态

### 重要补充说明：

为什么需要四次挥手？

**因为TCP是全双工通信，两个方向的连接需要分别关闭**

**服务端收到关闭请求时可能还有数据需要发送，所以ACK和FIN分开发送**

关于TIME_WAIT状态：

**持续2MSL（Maximum Segment Lifetime，最大报文生存时间）**

**目的是确保最后一个ACK能够到达服务端**

**防止历史连接中的数据包影响新连接**

通常是2-4分钟

任何一次挥手失败：

**都会重传相应的控制报文**

**超过重传次数后，强制关闭连接**

**这个过程保证了TCP连接的可靠关闭，确保双方都能够安全地断开连接，不会造成资源的浪费或数据的丢失。**



# seq和ack的关键角色



### 序列号(seq)和确认号(ack)的作用

seq号用于标识数据包的顺序

ack号用于确认已经收到的数据包

seq+1表示期望收到的下一个数据包的序列号

### 为什么是+1而不是其他数字？

+1表示当前数据包已经被正确接收

这个数字增量与TCP的字节流特性有关：

如果发送的是数据包，ack = seq + 传输的字节数

如果是控制包（如SYN、FIN），则ack = seq + 1（因为控制包被视为占用一个序列号）

### 验证流程示例

```tcl
发送方                      接收方
seq=100 ----数据包A---→   
                         检查：包A的seq是否是期望的序号？
                         处理：存储数据
                         响应：ack=101 (100+1)
        ←----ACK包-----   

seq=101 ----数据包B---→   
                         检查：包B的seq是否等于上次的ack？
                         处理：存储数据
                         响应：ack=102 (101+1)
```

### 验证机制的具体作用

1.顺序控制

- 确保数据按照正确的顺序到达
- 如果收到seq=103但期望是seq=101，会要求重传101

2.丢包检测

- 如果发送方没收到对应的ack，会触发重传
- 接收方如果发现seq不连续，说明中间有包丢失

3.重复包过滤

- 如果收到已确认过的seq，可以直接丢弃
- 防止重复数据的处理

4.流量控制

- 通过确认机制，可以控制发送速率
- 配合滑动窗口机制使用

### 5. 实际应用中的保护措施

1.超时重传

- 如果在规定时间内没收到ack，会重发数据包
- 重传超过一定次数后才会断开连接

2.快速重传

- 如果收到3个相同的ack，说明有包丢失
- 不等超时就立即重传

3.选择性确认(SACK)

- 可以确认非连续的数据块
- 提高网络效率

所以，seq+1的验证机制不是简单的加法，而是一个完整的可靠传输保证机制的组成部分，它配合其他TCP机制（如超时重传、滑动窗口等）一起工作，确保数据传输的可靠性和顺序性。



## 如何进行理解和记忆？

### 三次握手 (建立连接)

想象成两个人见面打招呼:

第一次: A说"你好!"

SYN=1

seq=x (随机数)

第二次: B回答"你好,我听到你了!"

SYN=1

ACK=1

seq=y (随机数)

ack=x+1

第三次: A说"好的,我也听到你了!"

ACK=1

seq=x+1

ack=y+1

记忆要点:

**SYN是同步标志(打招呼)**

**ACK是确认标志(听到了)**

**seq是序列号(我的编号)**

**ack是确认号(你的编号+1)**

### 四次挥手 (断开连接)

想象成两个人道别:

第一次: A说"我要走了"

FIN=1

seq=u

第二次: B说"好的,我知道了"

ACK=1

seq=v

ack=u+1

第三次: B说"我也要走了"

FIN=1

ACK=1

seq=w

ack=u+1

第四次: A说"好的,再见"

ACK=1

seq=u+1

ack=w+1

记忆要点:

FIN是结束标志(要走了)

**中间有个等待时间(2MSL)是为了确保B收到最后的ACK**

### 理解技巧

把它想象成人与人之间的对话

每个标志都有实际含义:

SYN = 打招呼

ACK = 听到了

FIN = 说再见

3. 序列号(seq)就是自己的编号

确认号(ack)就是对方编号+1

这样理解的话,就不用死记硬背了,而是通过生活中的场景来理解这个过程。每个参数的变化都是有逻辑的,而不是随意的。