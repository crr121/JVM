[TOC]

### 什么是方法表集合

方法表集合就是对方法的修饰符、返回类型、方法名、参数个数、参数类型、方法体的描述集合

### 方法表集合的结构

#### 总体结构

![image](https://wx3.sinaimg.cn/large/005LymWFly1fzqynrwg9tj30jq0d5aaz.jpg)

#### 单个方法的结构

方法表集合的结构与字段表结构非常的类似，开头也是用了两个字节表示方法的个数，然后接着是每个方法的描述

```
method_info{
    access_flag;//访问修饰符
    name_index；//方法名称（这里是常量池中的index）
    descriptor_index;//参数的类型以及返回值得类型（同样指向常量池得编号）
    attributes_count;//属性个数
    attibutes;//属性表（这里得属性个数及属性表都放在属性表中得‘Code’属性里面）
}
```



对于每个method_info得结构信息：

![image](https://ws2.sinaimg.cn/large/005LymWFly1fzqyvvtr4qj30mi0dcaak.jpg)



#### 方法表集合在class文件中的位置：

![image](https://ws1.sinaimg.cn/large/005LymWFly1fzqyu8c6gaj30nq0cgdh1.jpg)

### 实例练习

```java
public class HelloWorld{
    
    public int add(int a,int b){
        return a + b;
    }
    public String append(String s){
        return s;
    }
}
```

![image](https://wx2.sinaimg.cn/large/005LymWFly1fzqzg2s6vjj30z30hd0up.jpg)

![image](https://wx2.sinaimg.cn/large/005LymWFly1fzqzgrjnc2j30pj0c674k.jpg)





参考博客：

[class类文件结构——方法表集合](https://blog.csdn.net/xiaoqiu_cr/article/details/86742188)