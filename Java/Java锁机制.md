# Synchronized

分两种情况：

1、`Synchronized`直接加到方法上：

```java
public synchronized void test(){ }
```

![img](https://img-blog.csdnimg.cn/20190310221443337.jpg?x-oss-process=../.images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODQ4MTk2Mw==,size_16,color_FFFFFF,t_70)

- 方法结构中，会存在一个标志`ACC_SYNCHRONIZED`；一个flag；
- 当线程尝试调用此方法，会在此标志位设置值，如果设置成功，则此线程获取`monitor`监视器；

2、`Synchronized`同步代码块：

```java
public void test(){
    synchronized(this){
    }
}
```

`Synchronized`加锁是通过线程持有或释放`monitor`监视器来完成的；

- 在同步代码块的入口，执行指令：`monitorenter`监视器入口，线程尝试获取锁对象；
  
  - 获取成功，计数器加+1；
  - 获取不成功，根据当前锁级别，自旋或者阻塞；

- 执行同步代码，结束时，会调用指令`monitorexit`释放锁；
  
  - 此时，计数器减一；当计数器为0，才会释放锁；即可重入；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190311182248435.jpg?x-oss-process=../.images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODQ4MTk2Mw==,size_16,color_FFFFFF,t_70)

3、wait/notify只能用在同步代码块中；

否则抛出：java.lang.IllegalMonitorStateException（非法监视器状态异常）

wait/notify需要依赖monitor对象；

3、`Synchronized`下的锁优化：

偏向锁—轻量级锁—重量级锁

# JVM优化锁

Epoch：偏向时间戳；

## 偏向锁

轻量级锁的获取和释放都依赖于CAS原子操作，而偏向锁是在无竞争的情况下，连CAS操作都不做了。只有在需要置换ThreadID的时候，执行一次CAS原子操作。在只有一个线程执行同步块的情况下，进一步提高性能。

偏向锁的获取过程：

1. 在对象第一次被线程获取的时候，查看Mark Word中偏向锁标志是否为1，锁标志是否为01。是否已经确定Thread ID，如果已经确定，则不需要CAS操作，直接执行同步代码块。
2. 如果没有确定Thread ID，则通过CAS操作竞争锁，成功：则将Mark Word中的Thread ID设置为当前线程ID，然后执行同步代码块
3. 如果竞争失败：则说明有线程一同竞争，偏向锁会升级为轻量级锁。竞争失败的线程自旋。

偏向锁的释放：

如果没有竞争，线程不会主动释放偏向锁，只有在有竞争的情况下，偏向锁会被撤销，提升为轻量级锁。

## 轻量级锁

当偏向锁存在竞争的时候，当前持锁线程会在安全点暂停，检查当前持锁线程，的同步代码是否执行完成：

- 执行完成，则释放锁，偏向锁依然是偏向锁，想要拿锁的线程尝试替换ThreadID即可；
- 没有执行完，进行锁升级，升级为轻量级锁；

**升级过程**：（此时持锁线程正在被暂停）

1. 在当前线程栈帧中分配`锁记录空间`；
2. 将锁对象的MarkWord拷贝到琐记录空间中；
3. 使用CAS操作，将MarkWord修改为一个指针（指向持锁线程的琐记录空间）；
4. 修改对象头的锁标志位为`"00"`

升级完成，唤醒当前线程，从安全点继续执行；

- 竞争的线程会进行自适应的`自旋`；

达到一定自旋次数，或者竞争线程数增多，会发生锁膨胀，继续升级为重量级锁；

## 重量级锁

内置锁在Java中被抽象为监视器（monitor）；

当有其他线程占用锁时，当前线程会进入阻塞状态。

## 自旋

线程的阻塞，挂起，唤醒，都会给系统性能带来压力。

有时候线程只需要等待很短的时间，这个时间将线程挂机，就很不值得。

- 在只有两个线程竞争的情况下，让等待线程自旋；

所以自旋技术：让线程执行一个忙循环（自旋），稍等一下。这就是自旋锁。

（do...while不断尝试替换对象头信息，即不断尝试拿锁）

-XX:+UseSpinning开启自旋锁。JDK1.6之后，默认开启。

自旋锁的效果：避免了线程切换的开销，但是在自旋过程中要占用CPU。如果锁被占用很短，自旋时间很短，自旋等待的效果就会很好，反之，如果锁占用时间长，自旋就会很浪费资源。

JDK1.6之前自旋默认循环次数为10次。可以使用参数-XX:PreBlockSpin来修改。

JDK1.6之后，次数不在固定，可以自适应，由同一个锁的前一个自旋时间来决定。比如：上一个等待此锁的线程自旋了多久，这次系统就会让此线程自旋多久。

缺点

- 占用CPU处理时间；

## 锁消除

对于堆内存中永远不会逃逸，不会被其他线程访问到的变量，就认为是线程私有的，无需同步加锁。

虚拟机即时编译器在运行时，如果检测到不可能存在共享数据竞争的锁，说明这个锁无意义，就会自动进行消除这个锁。这就是锁消除。

```java
// 看起来没有同步的方法
public String concatString(String str1,String str2,String str3){
    return str1 + str2 + str3;
}
// 在JDK1.5之前，会自动转化为StringBuffer进行append（）操作
public String concatString(String str1,String str2,String str3){
    StringBuffer sb = new StringBuffer();
    sb.append(str1);
    sb.append(str2);
    sb.append(str3);
    return sb.toString();
}
```

## 锁粗化

原则上，将同步块的作用范围限制的尽量小，在多线程中的互斥同步的操作数量就会尽可能的小，如果存在锁竞争，那等待锁的线程也能尽快拿到锁。

大部分情况下，这是正确的。但是，如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作在循环体中，即使没有竞争，频繁互斥同步也会导致不必要的性能损耗。

锁粗化：如果虚拟机检测到有这样一连串零碎操作（如上面的代码）都对同一个对象加锁，将会把加锁同步的范围扩大到整个操作序列的外部。

# 乐观锁与悲观锁

数据库下高并发下写测试两种锁：

https://www.cnblogs.com/mussessein/p/12078074.html

## 悲观锁（多写场景）

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。

Java中 `synchronized`和 `ReentrantLock`等独占锁就是悲观锁思想的实现。

数据库：for update

## 乐观锁（多读场景）

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用**版本号机制和CAS算法实现。**

在Java中 `java.util.concurrent.atomic`包下面的原子变量类就是使用了乐观锁的一种实现方式**CAS**实现的。

两种乐观锁的常用实现方式：

#### 版本号机制（数据库）

一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，提交版本必须大于记录的当前版本。

举一个简单的例子：假设数据库中帐户信息表中有一个 version 字段，当前值为 1 ；而当前帐户余额字段（ balance ）为 $100 。

1. 操作员 A 此时将其读出（ version=1 ），并从其帐户余额中扣除 50（ 100-$50 ）。
2. 在操作员 A 操作的过程中，操作员B 也读入此用户信息（ version=1 ），并从其帐户余额中扣除 20 （ 100-$20 ）。
3. 操作员 A 完成了修改工作，将数据版本号加一（ version=2 ），连同帐户扣除后余额（ balance=$50 ），提交至数据库更新，此时由于提交数据版本大于数据库记录当前版本，数据被更新，数据库记录 version 更新为 2 。
4. 操作员 B 完成了操作，也将版本号加一（ version=2 ）试图向数据库提交数据（ balance=$80 ），但此时比对数据库记录版本时发现，操作员 B 提交的数据版本号为 2 ，数据库记录当前版本也为 2 ，不满足 “ 提交版本必须大于记录当前版本才能执行更新 “ 的乐观锁策略，因此，操作员 B 的提交被驳回。

这样，就避免了操作员 B 用基于 version=1 的旧数据修改的结果覆盖操作员A 的操作结果的可能。

#### CAS算法（compare and swap比较再交换）（Unsafe，AQS类）

Compare and Swap，即比较再交换；

区别于synchronouse同步锁的一种乐观锁（是一种无锁算法）

CAS有3个操作数，

- 内存地址，以直接从内存中获取旧值；
- 旧的预期值A，代码中的旧值；
- 要修改的新值B

当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做；

CAS是CPU级别的指令操作，上述操作是一步原子操作；

缺陷：ABA问题

有可能value被改了，又被改回来了，那么CAS算法是无法发现value已经被修改过了，误认为没有被修改；

最多的就是Unsafe类下的CAS操作：

`getIntVolatile`，`getIntVolatile`都是Unsafe类下的native的CAS操作；

```java
// 再调用Unsafe类下的getAndAddInt方法
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    // CAS操作
    do {
        /**
         * var1：我们创建的AtomicInteger对象
         * var2：valueOffset
         * 就是找到此对象下，value属性在内存中的偏移量，在内存级别直接拿到value的值
         */
        var5 = this.getIntVolatile(var1, var2); 
    } while(!this.getIntVolatile(var1, var2, var5, var5 + var4));
    // 返回旧值
    return var5;
}
```

此方法会调用Unsafe类下的多个Native方法

`getIntVolatile`和`compareAndSwapInt`都是Unsafe类下的native方法；

`getIntVolatile`：就是CAS中的根据内存地址，直接在内存级别获取当前旧值；

`compareAndSwapInt`：就是CAS的比较修改的操作，修改成功，返回true，那么跳出循环；修改失败，继续尝试获取内存值，进行修改；
