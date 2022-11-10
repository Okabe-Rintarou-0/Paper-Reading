# An advanced decision model enabling two-way initiative offloading in edge computing

## 笔记

### 层级
   
![](imgs/hierarchy.png)

随着物联网的发展，很多设备从数据产生者变为了数据消费者，对于很多时延敏感的请求，可以考虑在本地执行（如果具有充足计算能力），也可以
将任务卸载到尽可能近的节点（通过若干跳）。

当一个请求被发送时，它实际上会经过多个服务器，并形成从原始服务器到中央服务器的路径。
一个请求将生成一条路径；大量请求将生成大量不同的路径。由于层次结构，这些路径可能相交、重叠或收敛。
然后，位于连接点的边缘服务器可能会过载，形成任务队列，甚至因为请求的涌入而拥塞。
例如，如图1所示，如果下面的所有设备都决定将其请求卸载到购物中心的边缘服务器，则该服务器将变得拥塞。
因为更高层的服务器更可能是更多路径的连接点，所以拥塞也可能发生在更高层，而不管这些服务器通常更强大。
一些传统的解决方案假设边缘服务器是黑匣子，这样它们可以按预期处理所有请求（看起来像我们的原始决策模型）。
不幸的是，拥堵在现实中可能会出乎意料地发生。
其他解决方案应用一些全局放置策略来调度这些请求。但是，结果是，调度变成了一项离线工作，预期的建模变得不可避免。
    
### 系统模型

+ 请求和服务器模型

    ![](imgs/request_and_server_model.png)

+ 计算损耗模型
    
    ![](imgs/communicate_model_1.png)

+ 本地开销模型

    ![](imgs/local_overhead_1.png)
    
    ![](imgs/local_overhead_2.png)

+ 卸载开销模型

    包含任务卸载消耗 + 任务执行消耗。
    
    ![](imgs/offload_overhead.png)
 
+ 忽略了传回结果的时间消耗，因为对比于输入的数据规模，结果的数据规模通常可以忽略不记。
    
     > Similar to many studies such as [9,6,15], we also neglect the
      time overhead for sending back the outcomes, because of the fact
      that the size of outcomes is generally very small compared with
      the input data.

+ 决策过程

    根据原始决策模型的假设，一个服务器只为一个到达的请求提供服务，我们可以根据两个选择的开销之间的比较直接做出决策。
       
### 更进一步的决策模型

+ 原始决策模型不会产生任何拥塞，但是实际上我们必须考虑拥塞的情况。通过如下修改，将卸载任务的目标节点的任务等待队列纳入考量，避免将任务卸载到已经过载的节点。

    ![](imgs/congestion_1.png)

    但是如上的改进仍然不够，这是因为设备获取的状态信息可能是过时的，并且拥塞可能在做出决定后立即发生。

+ RED
    
    随机早期检测（RED），也称为随机早期丢弃或随机早期丢弃，是适用于拥塞避免的网络调度器的著名排队规则。
    只有排队的数据报将被处理。`RED` 通过加权移动平均模型监测队列大小，并根据统计概率丢弃数据图。
    当缓冲区为空时，它将接受所有传入的数据报。随着队列的增长，它将根据增长的概率丢弃传入的数据报。
    当队列最终满时，丢弃的概率将达到1，所有传入的数据报都将被丢弃。
    
    > 当平均队列长度达到 red-min-threshold 时，RED 随机选择该丢弃哪个包。当平均队列长度变长时，堆砌多少包数的可能性会增加。如果平均队列长度达到 red-max-threshold，则丢弃该包。
    尽管如此，也存在真实队列长度（非平均的）远大于 red-max-threshold 时，丢弃所有超过 red-limit 的数据报的情况。
    
    ![](imgs/RED.png)
        
    > 在卸载上下文中，“丢弃请求”表示将其卸载到下一跳服务器，而“处理请求”表示对其做出决定。
    当当前服务器已经过载时删除请求可以帮助请求直接跳转队列。这也使得作为接收者的服务器能够根据其能力选择性地接受请求。
    确实，这种主动丢弃可能会自私地将负担推给下一跳服务器。然而，高层服务器通常更强大。
    当当前服务器和下一跳服务器都有负担时，后者应该承担更多的责任。它并不违背边缘计算的初衷。
    
    这一点很有意思，**将 RED 的 drop 的语义定义为任务卸载到上一级**。
    
    论文对 RED 进行了一系列修改：
    
    + 首先，将“队列大小”的计算更改为计算完成处理所有排队请求的预期时间。
        
      > the evaluation of "the queue size" is altered into calculating the expected time to finish processing all enqueued requests.