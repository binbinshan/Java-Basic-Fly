# 什么是三次握手？

![](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-21/16215846386591.jpg)

* 第一次握手：Clien发送 SYN=1 和 一个随机产生的Client初始序列号seq 到Server。

* 第二次握手：Server收到 SYN=1 后，知道客户端请求建立连接，发送 SYN=1,ACK=1 和 ack number(Client 发送的seq + 1) 和 一个随机产生的Server初始序列号seq 发送给Client。
* 第三次握手：Client检查接收的 ack number 是不是 自己发送的Client seq + 1 和 SYN=1，检查通过后发送 SYN=1 和 ack number（Server 发送的 seq+1），发送给服务器；服务器检查通过后，连接建立。

### 🏠TCP建立连接可以两次握手吗？为什么?
不可以，原因有二：
1. 失效的连接请求报文段传到服务器，建立垃圾连接。

    > Client发送的连接请求报文，没有丢失，而只是在网络中滞留，直到连接释放以后的某个时间才到达Server，假设只有两次握手，那么服务器接收到失效请求连接后，发出确认后新的连接就建立了。因为Client并没有要建立连接，所以也不会向 Server 发送数据，但是 Server 却以为新的连接已经建立，并一直等待 client 发来数据。这样 Server 的很多资源就白白浪费了。

1. 两次握手则 Server 无法确认 Client 是否收到第二次握手的报文，也无法保证Client和Server之间成功互换初始序列号。

### 🏠TCP建立连接可以四次握手吗？为什么?
可以，但没必要，浪费资源。
四次握手是指，Server 将 ACK =1 和 ack number 单独发一次， 再将 SYN =1 和 seq 发送给client。


### 🏠第三次握手中，如果客户端的ACK未送达服务器，会怎样？
Server：
Server没有收到ACK确认，因此会重发之前的SYN+ACK（默认重发五次，之后自动关闭连接进入CLOSED状态），Client收到后会重新传ACK给Server。

Client：
1. 在Server进行超时重发的过程中，如果Client向服务器发送数据，数据头部的ACK是为1的，所以服务器收到数据之后会读取 ack number ,然后建立连接
2. 在Server进入CLOSED状态之后，如果Client向服务器发送数据，服务器会以RST包应答。


### 🏠如果已经建立了连接，但客户端出现了故障怎么办？
服务器每收到一次客户端的请求后都会重新复位一个计时器，时间通常是设置为2小时，若两小时还没有收到客户端的任何数据，服务器就会发送一个探测报文段，以后每隔75秒钟发送一次。若一连发送10个探测报文仍然没反应，服务器就认为客户端出了故障，接着就关闭连接。7950s


### 🏠初始序列号是什么？
TCP连接的一方A，随机选择一个32位的序列号（Sequence Number）作为发送数据的初始序列号，比如为1000，以该序列号为原点，对要传送的数据进行编号：1001、1002...三次握手时，把这个初始序列号传送给另一方B，以便在传输数据时，B可以确认什么样的数据编号是合法的；同时在进行数据传输时，A还可以确认B收到的每一个字节，如果A收到了B的确认编号（acknowledge number）是2001，就说明编号为1001-2000的数据已经被B成功接受。
