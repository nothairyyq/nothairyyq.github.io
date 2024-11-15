+++
title = '什么是长连接，为什么需要长连接？'
date = 2024-08-04T03:25:53+08:00
categories = ["八股文"]
tags = ["面试", "HTTP", "计算机网络"]
+++
## 什么是长连接，为什么需要长连接？
HTTP连接是无状态的，也就是说一次请求和一次响应后TCP连接就断开了。

**HTTP长连接就是将这次TCP连接保留下来，请求和响应之后TCP连接仍不断开，后续的请求与响应仍然走这个连接。**

目的：为了避免下次请求和响应还要建立TCP连接，免去建立TCP连接三次握手四次挥手的过程

所谓建立连接，在底层或者说在linux实现上，就是服务端和客户端在彼此的内存中维持着一个对象，记录着对方的信息。

## 长连接分为哪些长连接，有什么区别？
长连接分为TCP长连接，HTTP长连接。

TCP长连接和HTTP长连接关注的点是不一样的。

- TCP长连接是作用在传输层上的，维护的是一个链接。TCP长连接**在意的是对方是否还在线，也就是socket对象是否还存在**
    
    所谓链接，就是客户端和服务端在三次握手过程中分别在自己内存里建立的一个结构体socket对象，这个socket对象记录了双方的信息。
    
- **HTTP长连接是在应用层，关注的是对方是否能提供一个稳定的应用服务**，和TCP长连接关注的问题点不一样。


## HTTP长连接的作用是什么？
HTTP长连接本质上就是`TCP keep alive`，也就是TCP长连接。

通过在header中设置上`Connection:keep-alive` ，使得后续的HTTP请求复用现有的TCP连接。

这样避免了每次HTTP请求，都去建立TCP连接，避免了每次TCP连接的三次握手四次挥手的过程（HTTP1.0版本的实现），加快了效率。

## HTTP1.1引入长连接之后，出现了什么问题？如何解决？
**会产生队头阻塞的问题。**

在HTTP1.1版本引入长连接后，若干资源的请求都是基于同一个TCP连接。因此，当某个资源请求很久都得不到响应时，就会导致后面资源的请求也被阻塞。这件导致了**队头阻塞**的现象。

在HTTP2.0版本中引入了**TCP连接多路复用Multiplexing**，此时，1个TCP连接可以同时服务多个HTTP请求。这样，如果1个资源要加载很久，不会影响别的资源的请求。
## HTTP 1.0和1.1的区别
1. 持久连接
	 HTTP1.0默认开启了短连接，也就是说每次在客户端发送HTTP请求都会开启一个新的TCP连接，在请求完成之后又会断开这个TCP请求。因此在每个资源请求的时候都需要重复建立新的TCP连接，包括3次握手建立连接和4次挥手断开连接。增加了网络延迟和服务器的负载
	 HTTP1.1使用`Connection:keep-alive`字段默认启用了长连接，也就是持久化连接和连接复用。使用`keep-alive`是保持了TCP连接在多次请求之间的有效用，也就是说只有当客户端或服务器端提出断开连接，才会断开TCP连接，那这样就会减少很多网络开销。可以在一个tcp连接上发送多个请求，响应多个请求
2. 管道化传输
	HTTP1.0不支持管道化，也就是每个请求需要等到上一个请求的响应才能发送
	HTTP1.1允许在同一个TCP连接上同时发送多个请求，而不需要等待前一个请求的响应。但是服务器的响应必须按照请求顺序发送。所以也就导致了如果某个请求在服务器端处理时间太久导致后续的响应处理都背堵塞住，这也就是队头堵塞问题
3. 缓存控制
	HTTP1.1新增了`cache-control`字段，而HTTP1.0主要使用`expires`字段，HTTP1.1使服务器端对缓存有更精细化的控制
4. 状态码
	HTTP1.1新增了很多状态码，其中`206:partial content`可以支持范围请求，用于分块资源的传输




## HTTP 2.0 和 HTTP1.1的区别
HTTP1.1存在的问题
1. HTTP1.1虽然使用了`Connection: Keep-alive`键值对保证了能够实现持久化连接和管道化传输。但是只保证了客户端发送请求不会出现队头堵塞，然后服务器处理请求还是会出现对头堵塞，也就是当一条请求处理完成之后，才会处理下一条请求。
2. 除此之外HTTP1.1还存在头部过大的缺点，当客户端向服务器发送多个请求时，可能头部携带的很多信息都是重复的，这样会导致在客户端和服务器互相发送请求和响应时携带很多重复信息导致带宽增加。
3. HTTP1.1服务器只能被动的处理请求，在一些场景下，客户端向服务器端发送请求网页内容，服务器响应了html，但是还需要css渲染，那么这个时候客户端还需要向服务器再次发送请求。这样也会导致延迟和带宽的增加

HTTP2.0做的改进
1. 压缩头部
使用HPACK算法压缩HTTP的头部，压缩的过程是在客户端和服务器共同建立一张动态或静态表，会存储头部并且给一个对应的值，每次请求或响应只会携带表中的key，在接收到请求或响应时会从表中查找对应的头部信息，这样就可以在请求响应的时候减少头部信息，增加效率和带宽。
2. 二进制转化
HTTP2.0将HTTP1.1中的明文的文本格式数据转成了二进制格式信息，虽然说文本信息对于开发者可以在抓包或者进行后续开发时更容易阅读，但是http的后续操作如tcp以及ip以及mac都是使用的二进制信息，因此在http传输时将数据体转化成二进制可以增加解析效率。HTTP2.0会将文本信息转化为帧
3. 多路复用
为了解决队头堵塞问题，http2.1引入了流Stream，每个stream会有唯一标识符，并且每一个http请求和对应的响应都会使用同一个index的stream，但是可以允许在不同时发送，比如客户端先发送stream1，在发送stream3，再发送stream1. 这样当客户端或者服务器接收到流，会根据对应的index进行信息组装，并且在服务器处理请求和客户端发送请求都是可以并发进行的，不会造成队头堵塞问题。
4. 服务器推送
HTTP2允许服务器像客户端主动推送，双方都可以建立stream流，并且客户端建立的stream index是奇数，服务器建立的stream index是偶数


## HTTP 3 
HTTP2.0虽然实现了多路复用，但还是会出现队头堵塞的问题，但是并没有发生在应用层，是由于传输层的tcp连接。因为tcp连接是基于字节流的连接，如果在http连接的传输数据的过程中某一个stream里的字节丢失，会引发tcp的重传机制，这样会导致所有的stream等待tcp的重传。因此就会发生队头堵塞的问题。

HTTP3基于UDP实现了QUIC。
1. 没有队头堵塞，QUIC是基于udp实现的，而且传输的数据还是以流的形式，但是当数据发生丢失时，不会让所有的流都等待重传机制结束，而是数据丢失的流自己等待重传结束后继续传输
2. 建立连接更快，http2和http1如果使用了tls，那么tcp连接和tls连接是相互独立的，也就是先建立tcp三次握手然后再建立tls三次握手，这样会导致有多个RTT。然后http3的tls会成为QUIC的一个信息，直接建立quic连接，其中就包含了tls连接，所有只需要一个rtt，并且在第一次建立连接之后，传输的数据会直接带着quic和tls的信息，可以达到没有rtt的延迟
3. 连接迁移，http1和http2使用的tcp连接，确定一组tcp连接信息是使用的四元组信息源ip+源端口+目的ip+目标端口，这样会导致当客户端进行网络切换时，ip就会变，就会导致tcp连接的重新连接会重新走一遍tcp3次握手，tls三次握手以及tcp的慢启动。然后基于udp的quic双方直接协商好一个连接id，即使连接信息改变了，只要在这个连接中上下文具有quic和tls的信息，就可以无缝切换
然而http3即使这么好用，因为很多网络设备会将quic当成udp，那就会直接丢掉udp包，因此普及并不好


## 发起包含50张图片的http请求，针对HTTP1.0和HTTP1.1的情况下，分别需要多少个http请求和多少次tcp连接？

HTTP（超文本传输协议）主要运行在TCP（传输控制协议）之上，这是因为TCP提供了一种可靠的数据传输方式，确保数据完整性和顺序性，这对于Web页面的加载至关重要。至于UDP（用户数据报协议），由于其不提供数据传输的可靠性保证，通常不用于HTTP。

对于HTTP1.0和HTTP1.1在处理包含50张图片的HTTP请求时的连接情况，这里有一些关键差异需要了解：

### **HTTP 1.0**

**在HTTP 1.0中，默认情况下，每个HTTP请求都会建立一个新的TCP连接，完成数据传输后立即关闭这个连接**。这意味着如果一个HTML页面包含50张图片，浏览器需要为页面本身和每张图片分别建立一个TCP连接，总共需要51个TCP连接。因此，在HTTP 1.0的情况下，你会有：

- **HTTP请求次数**：**51次**（1次是获取HTML页面本身，50次是获取50张图片）
- **TCP连接次数**：**51次**

> **总结：HTTP1.0版本下，TCP连接不复用，用完即弃。**

### **HTTP 1.1**

与HTTP 1.0不同，**HTTP 1.1引入了持久连接（也称为HTTP keep-alive）的概念，允许在一个TCP连接上发送和接收多个HTTP请求/响应，而不是每次请求都打开一个新的连接**。这显著减少了需要建立的连接数，提高了网页的加载速度和效率。

**HTTP 1.1默认开启持久连接，**因此理论上，浏览器可以仅使用一个TCP连接来获取HTML页面和所有50张图片，前提是服务器配置允许足够长的连接保持时间，以及客户端和服务器都支持并正确实现了HTTP 1.1规范。实际上，现代浏览器会针对同一域名并发开启多个TCP连接以优化加载速度，但这仍然远少于HTTP 1.0情况下的连接数。因此，在HTTP 1.1的情况下，可能的情形是：

- **HTTP请求次数**：51次（同上）
- **TCP连接次数**：远少于51次，具体数目取决于浏览器的并发策略和服务器的配置，但**理论上可以为1。**

综上所述，HTTP1.1相较于HTTP1.0在处理包含多个资源请求的页面时，大大减少了必要的TCP连接次数，从而提高了加载效率和性能。


## TCP是按序交付，那是不是50张图片都需要按顺序传播？

不一定。

**在HTTP2.0中，引入了TCP连接多路复用。**

允许在同一个TCP连接上并行发送和接收多个请求/响应，而无需按顺序一一对应。

这意味着，**一个TCP连接可以同时处理多个HTTP请求和响应**，包括对同一个域名下的50张图片的请求，极大地减少了延迟并提高了效率。


