# 深拷贝与浅拷贝区别是什么？


* 浅拷贝
	- 拷贝对象和原始对象的引用类型引用同一个对象

* 深拷贝
	- 拷贝对象和原始对象的引用类型引用不同对象

	
	clone() 是 Object 的 protected 方法 , 无论是浅拷贝还是深拷贝，都需要实现 clone() 方法，来完成操作。而实现 clone() 必须实现 Cloneable 接口.
	
![](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-11/16207418065691.jpg)

![](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-11/16207418114905.jpg)


但是使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。