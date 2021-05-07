# Class.forName()和ClassLoader.loadClass()区别?

* Class.forName(): 将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块； 

* ClassLoader.loadClass(): 只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。 

* Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象。