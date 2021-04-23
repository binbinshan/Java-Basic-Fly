# Java 类的加载流程是怎样的？什么是双亲委派机制？

![-w782](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-04-23/16191768660194.jpg)


![-w492](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-04-23/16191768741085.jpg)

## 类加载的过程
* 加载
    * 通过类的全限定名(包名+类名)找到该类的.class文件的二进制文件。
    * 将字节流代表的静态存储结构转化为方法区运行时的数据结构。
    * 在内存中生成一个代表该类的java.lang.Class对象，作为访问入口。

* 连接
    * 验证
        * 确保class文件中的字节流包含的信息，符合当前虚拟机的要求，保证这个被加载的class类的正确性
    * 准备
        * 为类中的静态字段分配内存，并设置默认的初始值，比如int类型初始值是0。被final修饰的static字段不会设置，因为final在编译的时候就分配了
    * 解析
        * 虚拟机将常量池内的符号引用替换为直接引用的过程

* 初始化
    * 初始化就是执行类的构造器方法init()的过程。若该类具有父类，jvm会保证父类的init先执行，然后在执行子类的init。


## 什么是双亲委派机制？
![](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-04-23/16125078893324.jpg)

这里父类加载器并不是通过继承关系来实现的，而是采用组合实现的，也就是强依赖的方式。

类加载器可以大致划分为以下三类 :
1. BootstrapClassLoader(启动类加载器) ：最顶层的加载类，由 C++实现，负责加载 %JAVA_HOME%/lib目录下的 jar 包和类或者或被 -Xbootclasspath参数指定的路径中的所有类。
2. ExtensionClassLoader(扩展类加载器) ：主要负责加载目录 %JRE_HOME%/lib/ext 目录下的 jar 包和类，或被 java.ext.dirs 系统变量所指定的路径下的 jar 包。

3. AppClassLoader(应用程序类加载器) :面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。

4. 自定义加载器：可以自定义加载器，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自java.lang.ClassLoader。自定义加载器的话，需要继承 ClassLoader 。如果不想打破双亲委派模型，就重写 ClassLoader 类中的 findClass() 方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。但是，如果想打破双亲委派模型则需要重写 loadClass() 方法

### 双亲委派机制流程
1. 当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。

2. 当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。

4. 如果BootStrapClassLoader加载失败(例如在$JAVA_HOME/jre/lib里未查找到该class)，会使用ExtClassLoader来尝试加载；

5. 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

