# MySQL物理数据模型(三)
表、行和字段是逻辑上的概念，而表空间、数据区和数据页是物理上的概念。

一行数据的存储格式大致如下所示：
> 变长字段的长度列表，null值列表，数据头，column01的值，column02的值，...，column0n的值

* 定长与变长：
    * Char()就是定长的：在存储时字符串如果未达到指定的长度则会填充空格到指定长度。
    * VarChar()就是变长的：在存储字符串的时候只占用实际字符串长度+1个字节。

* 变长字段的长度列表
    * 变长字段长度列表记录每一个变长字段值的长度，存储的长度是十六进制的。
    * 如果有多个变长字段，那么变长字段长度列表是按**逆序**存储的。
  
        ```
        示例：
        -- 表结构
        create table test(
                  c1 varchar(10) comment '字段1-变长',
                  c2 varchar(5) comment '字段2-变长',
                  c3 varchar(20) comment '字段3-变长',
                  c4 char(1) comment '字段4-定长',
                  c5 char(1) comment '字段5-定长'
        ) ENGINE=InnoDB;
        -- 行数据 
        insert into test values('hello','hi','you','a','a');
        hello十六进制：0x05  hi十六进制：0x02   you十六进制：0x03
    ```
      存储结构：0x03 0x02 0x05 null值列表 数据头 hello hi you a a


* null值列表
    * NULL 值列表记录可为 NULL 的字段的情况（建表时not null 不在该范围）。

    * 用二进制bit位来标识字段值是否为 NULL。1为 NULL，0 不为 NULL。
    * 如果有多个可为 NULL 的字段，那么 NULL 值列表也是按照逆序存储的。
    * 而且 NULL 值列表的位数必须是 8bit 的N倍。例如：列表仅仅只有4个bit，则往高位补0，补到 8个bit。比直接存储null更省空间。
    
        ```
        示例：
        -- 表结构
        create table test(
                  c1 varchar(10) comment '字段1-变长',
                  c2 varchar(5) comment '字段2-变长',
                  c3 varchar(20) comment '字段3-变长',
                  c4 char(1) comment '字段4-定长',
                  c5 char(1) comment '字段5-定长'
        ) ENGINE=InnoDB;
        -- 行数据 
        insert into test values('hello','hi',null,null,'a');
        hello十六进制：0x05  hi十六进制：0x02
        null值：00110 -> 00001100
    ```
      存储结构：0x02 0x05 00001100 数据头 hello hi a


* 数据头：大小为 40 个bit位
    ![-w945](https://github.com/binbinshan/Java-Basic-Fly/blob/master/2021-06-07/16229026130687.jpg)

* 字符集编码
  在建库和建表时，都可以指定字符集编码。所以，数据都会经过数据库指定的字符集编码后，再进行存储的。
       ```
        示例：
        -- 表结构
        create table test(
                  c1 varchar(10) comment '字段1-变长',
                  c2 varchar(5) comment '字段2-变长',
                  c3 varchar(20) comment '字段3-变长',
                  c4 char(1) comment '字段4-定长',
                  c5 char(1) comment '字段5-定长'
        ) ENGINE=InnoDB;
        -- 行数据 
        insert into test values('hello','hi',null,null,'a');
        hello十六进制：0x05  hi十六进制：0x02
        null值：00110 -> 00001100
        hello 转码 606060
        hi 转码 606061
        a 转码 606062
    ```
      存储结构：0x02 0x05 00001100 数据头(40bit) 606060 606061 606062
      
* 隐藏字段
   * DB_ROW_ID 字段：如果我们没有指定主键和unique key唯一索引的时候，他就内部自动加一个ROW_ID作为主键。
    * DB_TRX_ID 字段：事务 ID，标识这是哪个事务更新的数据
    * DB_ROLL_PTR 字段：回滚指针，用来进行事务回滚的
    
    > 0x06 0x08 00000101 40bit位 00000000094C（DB_ROW_ID）00000000032D（DB_TRX_ID） EA000010078E（DB_ROL_PTR） 606060 606061 606062


* 行溢出问题
> 数据页的默认大小是 16kb，但是某些字段的值可以远远大于 16kb。
例如变长字段类型 varchar(N)：N 最大可为 65532（65kb），这就远远大于 16kb。
如果一行数据的大小超过了 16kb，就会出现行溢出的现象。
解决方案：
当一行数据超了 16kb，会在超了大小的那个字段中，可能仅仅包含他的一部分数据，然后同时包含一个20个字节的指针，指向存储了这行数据超了的部分的其他数据页。







