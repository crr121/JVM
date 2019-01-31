[TOC]

### Java内存模型介绍

在了解volatile到底是什么东西之前，我们先来了解一下Java的内存模型

参考博客<https://segmentfault.com/a/1190000014903099>

[这篇博客总结的非常通俗易懂]: https://segmentfault.com/a/1190000014903099	"Java并发编程-volatile可见性的介绍"

![image](https://ws4.sinaimg.cn/large/005LymWFly1fzpkuw4ohkj30k406pdgp.jpg)

简单来说，Java的内存模型就是

+ 每个线程都有一个自己独占的工作内存
+ 所有线程工作的时候都需要和主内存进行交互，这个交互的过程就是load和store的过程，load就是从主内存中取数据，store呢就是将数据存储到主内存中

对于交互的过程我们还可以详细的划分为8个操作

- lock( 锁定 )：作用于**主内存**的变量，把一个变量标识为一条线程独占的状态。（相当于大boss（主内存）将某个小兵（变量）指定给某个线程独占）
- unlock（解锁）：作用于**主内存**的变量，把一个处于锁定的变量释放出来，释放变量才可以被其他线程锁定。（相当于大boss主内存将某个小兵从某个线程中释放出来，这个时候又可以重新分配给其他线程）
- read（读取）：作用于**<u>主内存</u>**的变量，把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。（也就是从shared memory读入CPU local memory，相当从主内存中读取数据放入cache中）

```
这里补充一下什么是共享内存shared memory以及本地内存CPU local memory
-共享内存 shared memory 
    SM（SM = streaming multiprocessor）中的内存空间；
    最大48KB；
    作用域是线程块
-本地内存 local memory
    位于堆栈中，不在寄存器中的所有内容
    作用域为特定线程
    存储在global内存空间中，速度比寄存器慢很
```

- load（载入）：作用于**工作内存**的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。(相当于变量从CPU local memory读入JVM stack，你可以认为它是把数据从CPU cache读入到“JVM寄存器”)
- use（使用）：作用于**工作内存**中的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。（工作内存使用对象）
- assign（赋值）：作用于**工作内存**中的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。（ 对工作内存中的对象进行赋值）
- store（存储）：作用于**工作内存**的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用（将工作内存中的对象传送到主内存当中，这里也是相当于从JVMstack放入local memory中，也就是先暂时放入cache中）
- write（写入）：作用于**<u>主内存</u>**的变量，它把store操作从工作内存中得到的值放入主内存的变量中。（从local memory写入到share memory，即从cache写入到主存）

使用这个8个操作的注意点：

- read 和load，store和write 必须成对出现，即从主内存中读数据的数据工作内存**必须**接受；传递到主内存的数据，也不**可以被拒绝**写入。
- assign后的对象**必须回写到缓存**
- 未进行新赋值的对象不允许回写到主内存
- 新的变量只能在主内存产生，且未完成初始化的对象不允许在工作内存中使用
- 对象只允许被**一条线程锁定**，且可以被此线程多次锁定
- 未被锁定的对象不允许执行unlock操作
- 对一个对象执行unlock之前，必须将对象回写到主内存（也就是说释放某个小兵之前必须先交给大boss）

### volatile变量怎么用

大概了解了Java内存模型之后，我们就可以开始了解volatile变量了

参考博客：volatile变量与普通变量的区别<https://juejin.im/post/59cf5bedf265da06710dd337>

[volatile变量与普通变量的区别]: https://juejin.im/post/59cf5bedf265da06710dd337

首先我们先定义这样一个场景：创建两个线程1和线程2，他们都想要操作对一个int a进行自增的操作

```java
public class Test6 {

    public static int a = 0;

    public static void increase(){
        a++;
    }

    public static void main(String[] args){

       Thread t1 =  new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i = 0; i < 10; i++){
                    increase();
                    System.out.println("i:t1->"+i);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

            }
        });

        Thread t2 =  new Thread(new Runnable() {
           @Override
           public void run() {
               for(int i = 0; i < 10; i++){
                   increase();
                   System.out.println("i:t2->"+i);
                   try {
                       Thread.sleep(1000);
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }

               }
           }
       });

        t1.start();
        t2.start();


    }
}

```

```java
i:t1->0
i:t2->0//这里t2应该为1
i:t2->1//这里应该为2
i:t1->1
i:t2->2
i:t1->2
i:t2->3
i:t1->3
i:t1->4
i:t2->4
i:t2->5
i:t1->5
i:t1->6
i:t2->6
i:t2->7
i:t1->7
i:t2->8
i:t1->8
i:t1->9
i:t2->9
```

通过以上案例可以发现，线程1对i执行了increase()操作后，线程2取到的i并不是最新的值，那么是为什么呢

我们可以通过Java的内存模型来分析一下原因

![image](https://wx3.sinaimg.cn/large/005LymWFly1fzpnaadcgzj30gw0knq44.jpg)

从以上图可以看出，线程2在从主存中read数据i时，可能发生在线程1执行read、load、use、assign、store、write的任何时刻。假设线程1已经更新了i的值，正处在assign之后，此时正打算执行store操作，即往cache里面写数据，（还没有写回主存）那么这时线程2从主存中read的数据肯定不是最新的。

下面我们将a设置为volatile变量，将会发生什么呢？

首先我们先搞清楚volatile的工作机制：

![image](https://ws1.sinaimg.cn/large/005LymWFly1fzpnpsv37gj30hr0ku3zt.jpg)

可以看出volatile的工作机制是：当一个线程正处于read、load、use操作时，其他线程是不能执行read操作的，只有当该线程执行完use操作后，其他线程才能读取主内存中的数据。同时volatile机制还规定执行了assign操作之后，必须紧跟store和write操作。也就是说volatile将read，load、use操作封装为一个原子操作，将assign、store、write封装为一个原子操作，那么在原子操作的外部的任何一个时间节点，工作内存和主内存的数据是同步的。



修改了数据a,还没来得及写回主存，此时其他线程读取数据a时就不是最新的数据。所以这样看，volatile并不是完美的。

而下面的操作也证明了这一点

```java
 public static volatile int a = 0;
```

```java
i:t2->0
i:t1->0//可以看出进程t2仍然没有获取到最新的值
i:t2->1
i:t1->1
i:t1->2
i:t2->2
i:t1->3
i:t2->3
i:t2->4
i:t1->4
i:t2->5
i:t1->5
i:t2->6
i:t1->6
i:t1->7
i:t2->7
i:t1->8
i:t2->8
i:t1->9
i:t2->9
```

这里线程2读到的数据仍然为脏数据，那么是为什么呢？事实上volatile并不总是线程安全的

我们知道只有当当前线程处于assign、store、以及write操作时，其他线程才能执行read操作。但是细心的盆友会发现，就算在这三个操作中，也会发生脏读数据的可能性。比如当前线程1取到的a值为0，正打算执行assign-store-write原子操作，此时线程2读取了数据a（a=0）,那么等到当前线程1更新了数据a(a = 1)并写回了主存中，此时线程2也更新了a(a = 1)，并且同样执行了assign-store-write原子操作并也写回了主存，那么实际上是两个线程在对a执行简单的更新操作而已，并没有进行所谓的自增操作。

那么volatile一般怎么用呢？通常volatile用做**保存某个状态**的boolean值。并不会受到当前值的影响

[volatile 的线程安全是有条件的](https://juejin.im/post/59cf5bedf265da06710dd337)

所有的对象都是相对线程安全的，也就是有条件的。volatile的线程安全当然也是有条件的，它是对synchronized这一重量级线程同步的一种补充，其整体性能上优于synchronized。那volatile的线程安全的条件是什么呢？适合使用在哪些场景？《java虚拟机》给出两个条件: 

- 运算结果**并不依赖变量的当前值**（即结果对产生中间结果不依赖），或者能够确保只有**单一的线程**修改变量的值（比如自增运算就不适合volatile）
- 变量不需要与其它的状态变量共同参与不变约束