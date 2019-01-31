[TOC]

### class类文件结构——字段表结构

#### 什么是字段表

描述类或者接口中声明的变量（**不包括方法内部声明的变量，也就是说字段表中的字段信息只包含成员变量和静态变量，不包含局部变量**）****

 <fieldset><legend><span style="font-size:14px;"><span style="font-family:'Microsoft YaHei';">ps</span></span>😋</legend>

<p><span style="font-family:'Microsoft YaHei';font-size:12px;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="font-size:14px;"></span></span><span style="font-family:'KaiTi_GB2312';font-size:14px;">字段表集合中不会出现从超类或者父接口继承而来的字段，但是可能会出现原本Java代码之中不存在的字段，比如在内部类中为了保持对外部类的访问性会自动添加执行外部实例的字段</span>
<span style="font-size:14px;"></span></span><span style="font-family:'KaiTi_GB2312';font-size:14px;">另外由于Java中字段是无法重载的，所以无论两个字段数据类型，修饰符是否相同，字段名称一定不能相同</span></p>
<p><br></p>

</fieldset>

#### 从哪些方面可以进行描述

修饰符：**（通过布尔值来描述）**

+ 作用域：public,private,protected
+ 实例变量or类变量：static
+ 可变性：final
+ 并发可见性：volatile

**字段名和字段的数据类型不能通过字段表来表述，因为是不确定的，只能通过常量池来描述，不能用布尔值来描述**

#### 字段表结构

##### 字段计数器

表示有多少个字段，占2个字节，16位

参考博客：<https://blog.csdn.net/luanlouis/article/details/41046443#commentBox>

![image](https://ws1.sinaimg.cn/large/005LymWFly1fzpy4xqgljj30em08uwev.jpg)

---

字段表结构在class文件中的位置：

![image](https://wx4.sinaimg.cn/large/005LymWFly1fzq0d2rpw7j30mq0fct9x.jpg)

---



每个field_info的结构：

![image](https://ws2.sinaimg.cn/large/005LymWFly1fzpy9xu64aj30ks0fz750.jpg)

---

##### 访问标志access_flag

其中access_flag占2个字节，16位，具体信息如下：

![image](https://ws4.sinaimg.cn/large/005LymWFly1fzq045n4uij30kw0gawfa.jpg)

---



将每种访问标志用十六进制表示为：

![image](https://ws3.sinaimg.cn/large/005LymWFly1fzq0535ghfj30hg0a6t94.jpg)

---

下面我们采用具体的例子演示一下

写一个Java类

```java
public class HelloWorld{
    private int a = 0;
}
```

编译成class文件并查看字段表中的字段计数器和访问标志

![image](https://wx1.sinaimg.cn/large/005LymWFly1fzq0tn689cj30uw0fuq40.jpg)

如果字段有多个修饰符的情况

```java
public class HelloWorld{
    private static int a = 0;
}
```



![image](https://wx1.sinaimg.cn/large/005LymWFly1fzq145t9swj30v90gtgmv.jpg)

##### 字段的数据类型,字段名，属性长度

在class文件中的顺序为：访问标志（修饰符）->字段名->数据类型->属性长度

![image](https://ws3.sinaimg.cn/large/005LymWFly1fzq17vf1pgj30jh0h3754.jpg)

注意对象的完全限定名就是将“.” => "/" 

例如：java.lang.Object 的完全限定名为：Ljava/lang/Object

![image](https://ws2.sinaimg.cn/large/005LymWFly1fzq21px6zbj310y0hjac0.jpg)



 <fieldset><legend><span style="font-size:14px;"><span style="font-family:'Microsoft YaHei';">小贴士</span></span>😛</legend>

<p><span style="font-family:'Microsoft YaHei';font-size:12px;">&nbsp;&nbsp;&nbsp;&nbsp;<span style="font-size:14px;"></span></span><span style="font-family:'KaiTi_GB2312';font-size:14px;">如果您觉得收获了一点点，那么记得点个赞噢！您的鼓励，是我继续分享知识的强大动力！😛</span></p>
<p><br></p>

</fieldset>









