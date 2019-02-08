[TOC]



## class类文件结构——属性表集合

### 什么是属性表集合

属性表集合包括Java虚拟机预先规范定义的属性以及用户自定义的属性，对于用户自定义的属性，虚拟机加载的时候会自动忽略掉。class文件、字段表、方法表都可以携带自己的属性表集合，便于描述某些场景专有的信息。

### 单个属性表的结构attribute_info

```
attribute_info{
    attribute_name_index//属性的名称索引（指向常量池）2个字节
    attribute_length//属性长度 4个字节
    info//有attribute_length个字节属性值
}
```



![image](https://ws2.sinaimg.cn/large/005LymWFly1fzs3fusop3j30k60k8dh2.jpg)

### 属性表在方法表中的位置

![image](https://wx1.sinaimg.cn/large/005LymWFly1fzs3ms5iqxj30jw0dhmxi.jpg)

### Java虚拟机中比较常用的属性

#### code属性

code属性在方法表的属性集合中，code属性存储的是方法体中的代码经过javac编译器处理之后的字节码指令

如果一个方法没有方法体，比如抽象类或者接口，那么就没有code属性

#### code属性的结构

我们知道了整个属性表的一个结构，但是对于属性表中的每个属性，他们又会拥有自己的一个结构，那么我们来看下这个code属性的结构长啥样

```
code_attribute_structure{
    attribute_name_index;//指向常量池中“code”的索引值 2个字节 u2
    attribute_length;//code属性的长度 nu4
    max_stack;//操作数栈的最大深度，用于分配栈帧的操作数栈深度的参考值 u2
    max_locals;//局部变量所需的存储空间 u2
    code_length;//机器指令的长度，其值是多少，就向后面数多少个字节表示机器指令 u4
    code;//跟在code_length后面code_length个字节的机器指令的具体的值（JVM最底层的机器指令）u1
    exception_talbe_length;//显示异常长度 u2
    exception_table;显示异常表  exception_info类型(长度为exception_table_length)
    attribute_count;//属性计数器，code属性中还包含有其他子属性的数目 u2
    attribute_info;//属性code的子属性，主要有lindeNumberTable,LocalVariableTable
    
}
```



![image](https://ws1.sinaimg.cn/large/005LymWFly1fzs4eag9bej30la0ew3zf.jpg)

### 实战演练

同样定义一个Java文件

```java
public class HelloWorld{
    
    public static synchronized final void add(){
		int a = 10;
	}

  
}
```

下图展示的是init方法的信息



![image](https://ws4.sinaimg.cn/large/005LymWFly1fzs8ispq3uj319h0lrwgr.jpg)



add方法的信息

![image](https://ws1.sinaimg.cn/large/005LymWFly1fzs98fehj7j314s0mrgnq.jpg)





最后补充一个小知识点🙃

为什么参数的个数为1（因为我们定义的方法add里面根本就没有参数）

![image](https://wx3.sinaimg.cn/large/005LymWFly1fzs9haegi8j30nc0b6mx8.jpg)

![image](https://ws2.sinaimg.cn/large/005LymWFly1fzys9kje7aj30im05ot8j.jpg)