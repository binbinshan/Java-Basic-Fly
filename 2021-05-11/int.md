# Java 中 int 的最大值是多少？

A constant holding the maximum value an int can have, 2$^{31}$-1.
  
A constant holding the minimum value an int can have, -2$^{31}$.




### 原码、反码、补码
在计算机中，数据是由二进制补码进行存储的，在 Java 代码中我们看到的 “0x80000000”、“0x7fffffff”，这些非10进制的数，都是以补码的形式存在的，通过转换成原码，我们才能知道其真实的值。

原码转换成补码的公式：（用“|“来分隔每个字节，8位）

1. 当原码为正数时，反码和补码与原码相同。
    ```
    正数：1
    原码：0000 0000 | 0000 0000 | 0000 0000 | 0000 0001
    反码：0000 0000 | 0000 0000 | 0000 0000 | 0000 0001
    补码：0000 0000 | 0000 0000 | 0000 0000 | 0000 0001
    ```

2. 当原码为负数时，反码为其绝对值按位全部取反（包括符号位），补码为反码加1。
    ```
    负数：-1
    原码：1000 0000 | 0000 0000 | 0000 0000 | 0000 0001
    反码：1111 1111 | 1111 1111 | 1111 1111 | 1111 1110
    补码：1111 1111 | 1111 1111 | 1111 1111 | 1111 1111

    ```

因此在程序中，我们定义16进制整形数时，0x00000001表示1，0xffffffff表示-1

### 为什么是31次方？

因为int类型是4个字节，共32位，最大值用二进制表示就是， 0111...(总共31个1)

这里为什么第一位是0？ 二进制里，最高位(第一位)表示符号：0表示正，1表示负。


### 为什么最大值是 2^31 -1  而不是 2^31 
计算机存储数字时，第一位是标志位，只有 31 位用来存储数字的值。所以最大表示的正数为 0111 1111 1111 1111 1111 1111 1111 1111，转换为10进制数为2^31-1 = 2147483647


### 最小值为什么是 -2^31，而不是 -2^31 -1

计算机中可表示的整数最小值的负数原码为 1111 1111 1111 1111 1111 1111 1111 1111
对应的补码为 1000 0000 0000 0000 0000 0000 0000 0001 ,此时 值为 2^{31}-1

但是如果这样的话，1000 0000 0000 0000 0000 0000 0000 0000 这个数字就被浪费了，所以 1000 0000 0000 0000 0000 0000 0000 0000 这个数字就被规定为表示 -2^31，
所以 int 型数据取值范围是  -2$^{31}$ , 2$^{31}$-1 

