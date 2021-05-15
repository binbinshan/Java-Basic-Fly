# JVM中有哪些内存区域，分别都是用来干嘛的？

在讨论了JVM的类加载机制之后，再来看看JVM的内存区域划分

### 1、什么是JVM的内存区域划分？
JVM在运行我们写好的代码时，是必须使用多块内存空间的，不同的内存空间用来放不同的数据， 然后配合我们写的代码流程，才能让我们的系统运行起来。一块内存那也太不方便了，为了更好的使用。
那么现在就来 看一下 JVM 中有哪些内存区域。

### 2、存放类的方法区
方法区在 JDK1.8 之前，代表JVM里一块内存区域，在 JDK1.8 之后，改为叫 元空间 了。当然还是主要是放从 “.class” 文件里加载进来的类，还会有一些类似常量池的东西放在这个区域里。
![-w664](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16208991169747.jpg)


### 3、执行代码指令用的程序计数器
Java代码 被编译成字节码指令，然后字节码指令一定会被一条一条执行，所以当JVM加载类信息到内存之后，实际就会使用自己的字节码执行引擎，去执行我们写的代码编译出来的代码指令。

那么在执行字节码指令的时候，JVM里就需要一个特殊的内存区域了，那就是“程序计数器”，这个程序计数器就是用来记录当前执行的字节码指令的位置的，也就是记录目前执行到了哪一条字节码指令。

另外由于JVM是支持多个线程的，会有多个线程来并发的执行不同的代码指令。所以每个线程都会有自己的一个程序计数器，专门记录当前这个线程目前执行到了哪一条字节码指令了

![-w631](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209001754466.jpg)



### 4、Java虚拟机栈
![-w704](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209776533322.jpg)

在方法里，我们经常会定义一些方法内的局部变量，比如在上面的 main()方法里，其实就有一个“user”局部变量，他是引用一个UserS实例对象的,这里我们先只关注方法和局部变量。

JVM必须有一块区域是来保存每个方法内的局部变量等数据的，这个区域就是Java虚拟机栈。

每个线程都有自己的Java虚拟机栈，比如这里的main线程就会有自己的一个Java虚拟机栈，用来存放自己执行的那些方法的 局部变量。如果线程执行了一个方法，就会对**这个方法调用创建对应的一个栈帧**(有这个方法的局部变量表 、操作数栈、动态链接、方法出口等东西)，我们先只关注局部变量。

![-w405](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209787965500.jpg)
![-w674](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209792606724.jpg)

上面main() 调用 save() , save() 里又去调用了 validation(),栈里的对象如下图：
![-w667](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209794551942.jpg)

JVM中的“Java虚拟机栈”这个组件的作用：调用执行任何方法时，都会给方法创建栈帧然后入栈
。在栈帧里存放了这个方法对应的局部变量之类的数据，包括这个方法执行的其他相关的信息，方法执行完毕之后就出栈。

### 5、Java堆内存
Java堆内存，这里就是存放我们在代码中创建的各种对象的。

![-w405](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209795408688.jpg)
上面的“new UserS()”这个代码就是创建了一个Users类的对象实例，这个对象实例里面会包含一些数据，如 name , age。类似UserS这样的对象实例，就会存放在Java堆内存里。

Java堆内存区域里会放入类似UserS的对象，然后我们因为在main方法里创建了UserS对象的，那么在 线程执行main方法代码的时候，就会在main方法对应的栈帧的局部变量表里，让一个引用类型的“user” 局部变 量来存放UserS对象的地址。可以认为局部变量表里的“user”指向了Java堆内存里的UserS对象。


### 6、核心内存区域的流程
![-w620](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209799651333.jpg)

![-w641](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-05-15/16209800440593.jpg)

* JVM启动，加载ClassLoad到内存中，main线程开始执行main()。

* main线程是关联了一个程序计数器的，他执行到哪一行指令，都会记录在这里。
* main线程在执行main()方法的时候，会在main线程关联的Java虚拟机栈里，压入一个main()方法的栈帧。 
* 接着会发现需要创建一个UserS类的实例对象，此时会加载UserS类到内存里来。创建一个UserS的对象实例分配在Java堆内存里，并且在main()方法的栈帧里的局部变量表引入一个 “user”变量，让他引用UserS对象在Java堆内存中的地址。
* 接着，main线程开始执行UserS对象中的方法，会依次把自己执行到的方法对应的栈帧压入自己的Java虚拟机栈，执行完方法之后再把方法对应的栈帧从Java虚拟机栈里出栈。

### 7、其他内存区域
其实在JDK很多底层API里，可能调用的都是c语言写的方法，或者一些底层类库，比如：
public native int hashCode();

在调用这种native方法的时候，就会有线程对应的本地方法栈，这个里面也是跟Java虚拟机栈类似的，也是存放各种native方 法的局部变量表之类的信息。

还有一个区域，是不属于JVM的，通过NIO中的allocateDirect这种API，可以在Java堆外分配内存空间。然后，通过Java虚拟 机里的DirectByteBuffer来引用和操作堆外内存空间。