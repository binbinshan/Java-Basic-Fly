

# 什么是四次挥手？
![](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-21/16215895455361.jpg)

* 第一次挥手：Client将 FIN = 1 和 一个随机序列号 seq 发送给Server。

* 第二次挥手：Server收到 FIN 之后，发送一个 ACK=1 和 ack number(client seq + 1)，此时客户端已经没有要发送的数据了，但仍可以接受服务器发来的数据。
* 第三次挥手：Server将 FIN = 1 和 一个新的随机序列号 seq 给 Client。
* 第四次挥手：Client 收到服务器的 FIN 后，进入 TIME_WAIT 状态；接着将 ACK = 1 和 ack number(Server序列号+1) 发送给服务器；服务器收到后，确认ack number后，变为 CLOSED 状态，不再向客户端发送数据。客户端等待 2 * MSL（报文段最长寿命）时间后，也进入 CLOSED 状态。完成四次挥手。


### 为什么不能把服务器发送的ACK和FIN合并起来，变成三次挥手？
因为服务器收到客户端断开连接的请求时，可能还有一些数据没有发完，这时先回复ACK，表示接收到了断开连接的请求。等到数据发完之后再发FIN，断开服务器到客户端的数据传送。

### 如果第二次挥手时服务器的ACK没有送达客户端，会怎样？
客户端没有收到ACK确认，会重新发送FIN请求。

### 客户端TIME_WAIT状态的意义是什么？
第四次挥手时，客户端发送给服务器的ACK有可能丢失，TIME_WAIT状态就是用来重发可能丢失的ACK报文。如果Server没有收到ACK，就会重发FIN，如果Client在 2 * MSL 的时间内收到了 FIN，就会重新发送 ACK 并再次等待 2MSL，防止 Server 没有收到 ACK 而不断重发FIN。

MSL(Maximum Segment Lifetime)，指一个片段在网络中最大的存活时间，2MSL就是一个发送和一个回复所需的最大时间。如果直到2MSL，Client都没有再次收到FIN，那么Client推断ACK已经被成功接收，则结束TCP连接。





