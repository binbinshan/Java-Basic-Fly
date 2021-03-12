
## 简述动态代理与静态代理

### 静态代理
静态代理，需要代理对象和目标对象**实现同一接口**，可以在不修改目标对象的前提下扩展功能。
缺点：
* 冗余，会产生大量的代理类；
* 不易维护，一旦接口修改，目标对象和代理对象都需要修改。

### 动态代理
动态代理(JDK代理/接口代理)，使用的JDK API，需要目标对象实现了接口才可以。(因为JDK代理需要用构造方法动态获取具体的接口信息，如果不实现接口的话，没法初始化）
JDK中生成代理对象主要涉及的类有java.lang.reflect Proxy，主要方法为
```
//返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h ) 

 ClassLoader loader    //指定当前目标对象使用类加载器
 Class<?>[] interfaces //目标对象实现的接口的类型
 InvocationHandler h   //事件处理器（下方处理方法）
```
java.lang.reflect InvocationHandler，主要方法为
```
// 在代理实例上处理方法调用并返回结果。
Object invoke(Object proxy, Method method, Object[] args) 
```

### CGLIB动态代理
cglib (Code Generation Library )是一个第三方代码生成类库，CGLIB会让生成的代理类继承被代理类，并在代理类中对代理方法进行强化处理，使用cglib代理的对象则无需实现接口，但是无法代理被final修饰的类。
代理类将委托类作为自己的父类，并为其中的非final委托方法创建两个方法：
* 一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法；
* 另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理。

底层将方法全部存入一个数组中，通过数组索引直接进行方法调用。



### 静态代理和动态代理区别？
1. 静态代理在编译时就已经实现编译，完成后代理类是一个实际的class文件。
2. 动态代理则是在运行时动态生成类字节码，并加载到JVM中。
3. JDK动态代理只需要目标对象实现接口，而静态代理则需要代理对象和目标对象都实现接口。
4. CGLIB动态代理目标对象无需实现接口，但是不能对final类以及final方法进行代理。
5. 静态代理会产生很多代理类，JDK动态代理则会比较消耗系统性能，CGLIB动态代理性能比JDK动态代理好。

