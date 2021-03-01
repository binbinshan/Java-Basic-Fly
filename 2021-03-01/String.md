## String 类能不能被继承？为什么？

String类是被final关键字修饰了，所以无法被继承。
因为将引用声明作final，就不能改变这个引用了，编译器会检查代码，如果试图将变量再次初始化的话，编译器会报编译错误。

### 原理
对于 final 域，编译器和处理器要遵守两个重排序规则：
1. 在构造函数内对一个 final 域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
2. 初次读一个包含 final 域的对象的引用，与随后初次读这个 final 域，这两个操作之间不能重排序。

写 final 域的重排序规则：
编译器要求在 写final域 之后，构造函数return之前插入一个StoreStore障屏，确保屏障前的写操作在屏障后的写操作之前。

读 final 域的重排序规则：
编译器要求在 读final域 之前，插入一个LoadLoad屏障，确保屏障前的读操作在屏障后的读操作之前。

### 我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?
因为String类是final，所以我们无法继承重写，但是可以使用组合的方式来实现，
```
    class MyString{
    
        private String innerString;
    
        // ...init & other methods
    
        // 支持老的方法
        public int length(){
            return innerString.length(); // 通过innerString调用老的方法
        }
    
        // 添加新方法
        public String toMyString(){
            //...
        }
    }
```
