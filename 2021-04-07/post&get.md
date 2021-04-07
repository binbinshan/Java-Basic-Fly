# HTTP 中 GET 和 POST 区别

后退/刷新：get是无害的，Post会导致重新提交
书签：get可保存为书签，Post不会保存成书签
缓存：get可以被缓存，Post不会被缓存
数据长度：get对数据长度有限制，Post对数据长度无限制
安全性：get数据在URL中对所有人都是可见的，Post数据不会显示在 URL 中

GET 和 POST 方法没有实质区别，只是报文格式不同。GET 和 POST 只是 HTTP 协议中两种请求方式，而 HTTP 协议是基于 TCP/IP 的应用层协议，无论 GET 还是 POST，用的都是同一个传输层协议，所以在传输上，没有区别。


