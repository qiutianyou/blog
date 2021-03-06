---
layout: post
title: 网络通信——TCP连接/断开
category: 网络通信
tags: [TCP]
---

对TCP三次握手四次分手还不清楚，超简单解析

关于TCP三次握手四次分手，之前看资料解释的都很笼统，很多地方都不是很明白，所以很难记，前几天看的一个博客豁然开朗，可惜现在找不到了。现在把之前的疑惑总结起来，方便一下大家。

## 先上个TCP三次握手和四次分手的图
![TCP_SYN.JPEG](/assets/images/TCP_SYN.JPEG)

![TCP_FIN.JPEG](/assets/images/TCP_FIN.JPEG)

网上好多都是错的，只能自己画了，一个正确的图的确可以方便理解。




## 疑问一，上图传递过程中出现的几个字符（SYN,ACK,FIN,seq,ack）各代表什么意思

SYN，ACK，FIN存放在TCP的标志位，一共有6个字符，这里就介绍这三个：

*SYN*：代表请求创建连接，所以在三次握手中前两次要SYN=1，表示这两次用于建立连接，至于第三次什么用，在疑问三里解答。

*FIN*：表示请求关闭连接，在四次分手时，我们发现FIN发了两遍。这是因为TCP的连接是双向的，所以一次FIN只能关闭一个方向。

*ACK*：代表确认接受，从上面可以发现，不管是三次握手还是四次分手，在回应的时候都会加上ACK=1，表示消息接收到了，并且在建立连接以后的发送数据时，都需加上ACK=1,来表示数据接收成功。

*seq*:序列号，什么意思呢？当发送一个数据时，数据是被拆成多个数据包来发送，序列号就是对每个数据包进行编号，这样接受方才能对数据包进行再次拼接。

初始序列号是随机生成的，这样不一样的数据拆包解包就不会连接错了。（例如：两个数据都被拆成1，2，3和一个数据是1，2，3一个是101，102，103，很明显后者不会连接错误）

*ack*:这个代表下一个数据包的编号，这也就是为什么第二请求时，ack是seq+1，

(这里要吐槽一下，当初不懂的时候查资料，发现好多地方把ACK和ack都搞混了，害的我被坑了好久...)

如果你仔细看了上面对每个字符的解释，那么相信我画的三次握手和四次分手的图你也就明白了。

再复习一遍　　　　

在创建连接时，

1.客户端首先要SYN=1,表示要创建连接，

2.服务端接收到后，要告诉客户端：我接受到了！所以加个ACK=1，就变成了ACK=1,SYN=1

3.理论上这时就创建连接成功了，但是要防止一个意外（见疑问三），所以客户端要再发一个消息给服务端确认一下，这时只需要ACK=1就行了。

三次握手完成！

在四次分手时，

1.首先客户端请求关闭客户端到服务端方向的连接，这时客户端就要发送一个FIN=1，表示要关闭一个方向的连接（见上面四次分手的图）

2.服务端接收到后是需要确认一下的，所以返回了一个ACK=1

3.这时只关闭了一个方向，另一个方向也需要关闭，所以服务端也向客户端发了一个FIN=1 ACK=1

4.客户端接收到后发送ACK=1，表示接受成功

四次分手完成！

我为什么没有在上面的过程中，加入seq和ack呢？就如我对这两个关键字的解释的一样，这两个是数据拆分和组装必备元素，所以所有的请求都需要这两个元素，只要明白了作用，就可以自己举一反三。

关于握手和分手，主要还是SYN,FIN,ACK的变化，这才是重点！

## 疑问二，每次发送请求时为什么ack要+1

关于seq和ack关键字的解释中已经说明了。

## 疑问三，为什么需要三次握手

下面解释明明两次就可以建立连接的为什么还要加第三次的确认。

如果发送两次就可以建立连接话，那么只要客户端发送一个连接请求，服务端接收到并发送了确认，就会建立一个连接。

可能出现的问题：如果一个连接请求在网络中跑的慢，超时了，这时客户端会从发请求，但是这个跑的慢的请求最后还是跑到了，然后服务端就接收了两个连接请求，然后全部回应就会创建两个连接，浪费资源！

如果加了第三次客户端确认，客户端在接受到一个服务端连接确认请求后，后面再接收到的连接确认请求就可以抛弃不管了。

## 疑问四，为什么需要四次分手

TCP是双向的，所以需要在两个方向分别关闭，每个方向的关闭又需要请求和确认，所以一共就4次。
