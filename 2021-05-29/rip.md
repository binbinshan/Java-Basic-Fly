# 什么是RIP (Routing Information Protocol, 距离矢量路由协议)? 算法是什么？
每个路由器维护一张表，记录该路由器到其它网络的”跳数“，路由器到与其直接连接的网络的跳数是1，每多经过一个路由器跳数就加1；更新该表时和相邻路由器交换路由信息；路由器允许一个路径最多包含15个路由器，如果跳数为16，则不可达。交付数据报时优先选取距离最短的路径。

* 实现简单，开销小
* 随着网络规模扩大开销也会增大；
* 最大距离为15，限制了网络的规模；
* 当网络出现故障时，要经过较长的时间才能将此信息传递到所有路由器