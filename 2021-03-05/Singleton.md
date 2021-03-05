饿汉式和懒汉式的区别：

1）饿汉式是空间换时间，懒汉式是空间换时间。

2）在多线程访问的时候，懒汉式可能会创建多个对象，而饿汉式不会。

```
//懒汉，顾名思义比较懒，在用的时候才实例化 
public class Singleton { 
    //创建实例，注意，此时没有new 
    private static volatile Singleton instance = null; 
    //构造方法私有化，无法在外部获取实例，只能通过下方的公有静态方法 
    private Singleton() {} 
    //公有的静态方法，返回实例对象 
    public static synchronized Singleton getInstance() { 
    //先看下是否存在实例，有的话就不再new了 
    if (instance == null) { 
    //这里才new 
        instance = new Singleton(); 
    } 
    return instance;
 }
```

```
//饿汉，顾名思义很饥饿，创建对象的时候就直接new 
public class Singleton { 
    //创建实例的时候就new 
    private static Singleton instance = new Singleton(); 
    // 私有化构造方法，外部不能new 
    private Singleton() {} 
    //公有的静态方法，返回实例对象 
    public static Singleton getInstance() { 
    //直接将事先new好的实例返回 
    return instance; 
    } 
}
```
