# 基础

## Java引用类型

1、强引用

2、软引用：内存不足时，回收；

3、弱引用：只要GC，就会回收；

4、虚引用：GC

尽量不要重写finazlie()；

- 影响回收速度，重写了finazlie()的对象，不会直接回收，会先入队，再次回收；
- 可能造成对象再次复活：在队列中，如果执行finazlie()，存在强引用再次持有对象，会复活；

## 基本数据类型

| byte    | 8bit  | 1字节 |
|:-------:|:-----:|:---:|
| short   | 16bit | 2字节 |
| int     | 32bit | 4字节 |
| long    | 64bit | 8字节 |
| char    | 16bit | 2字节 |
| float   | 32bit | 4字节 |
| double  | 64bit | 8字节 |
| boolean | 1bit  | 1字节 |

### 机器数

计算机一律用补码来表示和存储数字；

1、原码 = 符号位+真值

byte表示范围：[1111 1111 , 0111 1111]    即：[-127 , 127]

1000 0000：正零，0000 0000：负零（-128）

2、反码

正数：反码 = 原码

负数：反码 = 符号位 + 真值取反

3、补码

正数：补码 = 原码

负数：补码 = 反码 + 1

## 面向对象

### 多态

1、对象引用的具体类型，在运行期才能确定；（存在向上转型）

2、存在继承关系；

3、方法重写；

### 抽象类

1、抽象类不能实例化；

2、至少存在了一个abstract，就必须声明为抽象类，也可以不包含abstract

3、抽象类可以包含实现的方法；

### 重载/重写

1、重写(Override)

存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。

2、重载(Overload)

存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是**参数类型、个数、顺序**至少有一个不同。

**返回值不同，其它都相同不算是重载**。

## swich

```java
public class Test {
    public static void main(String[] args) {
        int i = 1;
        int a = 0;
        switch (i) {
            case 1:
                a = 1;
                //break;
            case 2:
                a = 2;
                //break;
            case 3:
                a = 3;
                break;
            default:
                a = 10;
        }
        System.out.println(a);
    }
}
```

如果不加break；

后续case，直接无视，执行方法体；

# 深拷贝/浅拷贝

对于基本数据类型，不存在深浅拷贝，直接拷贝；

对于引用类型

- 浅拷贝是拷贝了对象地址；
- 深拷贝是复制了一个新的对象过来；

![image-20200629171955272](image/image-20200629171955272.png)

## 浅拷贝

1、=：直接赋值

2、Object的clone()方法；

3、 Arrays.copyOf()方法；

## 如何深拷贝

1、new一个一样的对象；

2、重载clone方法

3、序列化深拷贝：

- Apache Commons Lang序列化
- Gson
- Jackson

# JDK和JRE区别

JDK是整个JAVA的核心，包括了Java运行环境JRE、Java工具、Java基础类库；

可以通过JDK，将Java工程编译成字节码文件；

JRE是Java运行环境，不含开发环境，即没有编译器和调试器；

通过JRE可以运行字节码文件；

# 基本数据类型

- byte         8位（-128~127）（1字节）
- short       16位（2字节）
- int            32位（4字节）
- long         64位（8字节）
- char         16位（2字节）
- float         32位（4字节）
- long         64位（8字节）

# 保证线程安全3要素

线程安全与否的三要素：

1. 原子性：针对多线程下，互斥地访问共享资源；（Atomic类，synchronized，Lock）
2. 可见性：对数据的操作，及时地反映在共享地内存中，让多线程都可见；（volatile）；
3. 有序性：禁止指令地优化重排序；（volatile，锁）；

# Java异常体系

![image-20200731095444844](image/image-20200731095444844.png)

- Error：不可预料的系统级别错误，只能退出运行；

- Exception：需要捕捉或进行处理的异常；
  
  - 运行时异常：不检查异常，程序中可以捕获，可以不处理；
    
    NullPointerException（空指针）、
    
    IndexOutOfBoundsException（数组越界）、
  
  - 编译期异常：必须处理的异常，否则无法通过编译；（抛出异常、捕获）
    
    IOException
    
    FileNotFoundException

# Java反射

首先要了解Java的类加载机制！

<img src="/image/reflect.png" style="zoom:67%;" />

元空间中存储着每个加载过的类的class对象（在Heap中）；

Java的反射就是通过class对象，来获取类的各种属性！

通过这个Class对象，能干嘛？：

```java
public final class Class<T>{
    // 获取属性
    public Field[] getFields() throws SecurityException {}
    // 获取类加载器
    public ClassLoader getClassLoader() {}
    // 获取构造器
    public Constructor<?>[] getConstructors(){}
    // 获取私有方法
    public Method getDeclaredMethod(){ }
    ....
}
```

## 获取Class对象的方式

1、通过对象实例获取

（需要对象实例，不实用）

```java
Student stu=new Student();
Class stuClass1=stu.getClass();
```

2、直接通过类获取

（需要导入类，存在依赖）

```java
Class stuClass2=Student.class;
```

3、通过Class类的`forName`静态方法

（最常用，不需要创建对象，不需要导入类，只需要对象全限定类名）

```java
// 需要全类名
Class stuClass3=Class.forName("com.entity.Student");
```

## 反射创建对象

```java
Student stu = studentClass.newInstance();
```

# 动态代理

## JDK动态代理

利用反射机制！针对接口创建代理对象；(被代理类，必须实现接口！！！！！)

1. 创建代理对象；
   
   需要：被代理类的类加载器，接口和一个`InvocationHandler`；
   
   `InvocationHandler`可以自己定义；
   
   ```java
   Test testProxy 
   =(Test) 
       Proxy.
       newProxyInstance(Test.class.getClassLoader(), Test.class.getInterfaces(), proxyInvokeHandler);
   ```

2. 在自己定义的`InvocationHandler`中，对原有的方法进行织入；
   
   重写`invoke`方法，实现对原方法的丰富；
   
   这里是自定义的，也可以直接在上面方法，`new InvocationHandler(()->{.....})`
   
   ```java
   // 自定义一个，也可以直接在上面方法，new InvocationHandler(()->{.....})
   public class proxyInvokeHandler implements InvocationHandler {
       private Object subject;
       // 构造，传入被代理类的对象 
       proxyInvokeHandler(Object subject){
           this.subject = subject;
       }
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           // 执行方法前
           Object invoke = method.invoke(subject, args);    // 执行方法
           // 执行方法后
           return invoke;
       }
   }
   ```

## CGLib动态代理

利用继承机制实现，无法代理被final的类；

原理是：

通过ASM包，将被代理类的**字节码文件**，加载进来，修改字节码文件生成一个被代理类的子类；

然后创建代理，这个代理是被代理类的子类对象；

通过CGlib下的方法拦截器，对被代理类的方法进行一个增强；

然后就可以使用代理对象，调用对应的方法，并对其增强丰富等；

# 序列化

序列化：将对象转换成字节序列；

反序列化：将字节序列转化成对象；

序列化目的：可以在网络中传输字节序列；

如果想要对象某个字段不参与序列化：

```java
transient
```

# equals / HashCode

## equals

- **对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。**
- **对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。**

```java
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true  两个1是等价的
System.out.println(x == y);      // false 判断两个变量是否引用同一个对象
```

```java
String s1 = "abc";
String s2 = "a";
String s3 = "bc";
String s4 = "a" + "bc";        // 编译期就确定了
String s5 = s2 + s3;        // 不能在编译期确定，所以s5相当于new String
System.out.println(s1 == s4);    // true
System.out.println(s1 == s5);    // false
```

### 重写equals

```java
class Person {
    private String name;
    private int age;
    // 省略getter、setter
    ...
    @Override
    public boolean equals(Object obj) {
        // 1.判断这两个变量是否引用同一个对象
        if (this == obj) {
            return true;
        }
        // 2.判断是否属于同一个类(obj instanceof Person)
        if (obj != null && obj.getClass() == Person.class) {
            Person p = (Person) obj;
            // 3.依次判断属性是否都相同
            return this.getName()==p.getName() && 
                this.getAge()==p.getAge();
        }
        return false;
    }
}
```

## hashCode

hashCode() 返回散列值,等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。

**在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象散列值也相等。**

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```

新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只在集合中添加一个对象，但是因为 EqualExample 没有实现 hasCode() 方法，因此这两个对象的散列值是不同的，最终导致集合添加了两个等价的对象。

#### hashCode重写

理想的散列函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的散列值上。这就要求了散列函数要把所有域的值都考虑进来。

**不相等的对象，体现在对象属性，所以在重写hashCode，需要考虑通过对象属性字段a，来构建散列值**

- 字段为boolean类型，a?1:0
- 字段为byte/short/int/char，强转为(int)a
- 字段long，计算(int)(a^(a>>>32))
- 字段double，先转化为long，Double.doubleToLongBits(a)；在计算long
- 字段为对象引用（数组），直接拿到对象的hash值

```java
class Person {
    private String name;
    private int age;
    // 省略getter、setter
    ...
    @Override
    public int hashCode() {
        // 定义一个初始值，一般17
        int result = 17;
        // 通过属性，构造哈希值
        // 31：可用移位和减法，来代替乘法，提高计算性能；
        result = 31 * result + name.hashCode();
        result = 31 * result + age;
        return result;
    }
    // 一个数与 31 相乘可以转换成移位和减法：31*x == (x<<5)-x，编译器会自动进行这个优化。
}
```

### equals和hashCode

为什么有了equals判断对象是否相等，还需要hashCode？

因为查找对象位置和判断对象是否相同的效率太低，hashCode使用散列表，效率很高！

提高map集合的索引速度；

那为什么有了hashCode，还需要equals？

hashCode不一定完全可靠，不同的对象，哈希值也有可能相同！equals是判断对象是否相同，最可靠的方法！

**重写原则**：

1. equals返回true，那么hashCode也应当返回true！
2. hashCode相同，equals不一定返回true，hashCode方法并不可靠。

**那么这两个方法是怎么运作的？**

1. **在添加元素的时候，先调用hashCode方法（效率大大提升），判断哈希值是否相同，或者说是否已经存在，如果不存在（不相同），就没必要调用eqauls了。**
2. **如果hashCode相同，那么继续调用equals方法判断，两个对象是否相同如果返回true，那就确定这是重复的对象。如果返回false，确定为不同对象，继续存储。比如在HashMap中，就会在同一个bucket中产生链表或红黑树。**

# BigDecimal

- 小数转化为二进制，会不精确，再次使用，就会失真；

```java
0.9*2=1.8  // 取整1
0.8*2=1.6  // 取整1
0.6*2=1.2  // 取整1
0.2*2=0.4  // 取整0
....// 一直计算下去,无法得到精确值
```

所以我们需要BigDecimal类来计算一些商业运算。

BigDecimal构造器：

```java
public BigDecimal(String val) {    // 禁止使用
    this(val.toCharArray(), 0, val.length());
}
// 常用
public BigDecimal(double val) { 
    this(val,MathContext.UNLIMITED);
}
```

建议使用静态方法：

```java
public static BigDecimal valueOf(double val) {
    return new BigDecimal(Double.toString(val));
}
```

使用BIgDecimal来封装加减乘除

```java
public static BigDecimal add(double v1,double v2){
    BigDecimal b1 = BigDecimal.valueOf(v1);
    BigDecimal b2 = BigDecimal.valueOf(v2);
    return b1.add(b2);
}
public static BigDecimal sub(double v1,double v2){
    BigDecimal b1 = BigDecimal.valueOf(v1);
    BigDecimal b2 = BigDecimal.valueOf(v2);
    return b1.subtract(b2);
}
public static BigDecimal mul(double v1,double v2){
    BigDecimal b1 = BigDecimal.valueOf(v1);
    BigDecimal b2 = BigDecimal.valueOf(v2);
    return b1.multiply(b2);
}
public static BigDecimal div(double v1,double v2){
    BigDecimal b1 = BigDecimal.valueOf(v1);
    BigDecimal b2 = BigDecimal.valueOf(v2);
    //除不尽的情况
    return b1.divide(b2,2,BigDecimal.ROUND_HALF_UP);//四舍五入,保留2位小数
}
```

# 单例

```java
public class SingletonDemo {
    private volatile static SingletonDemo SingletonDemo;

    private SingletonDemo() {
    }

    public static SingletonDemo5 newInstance() {
        if (SingletonDemo == null) {
            synchronized (SingletonDemo.class) {
                if (SingletonDemo == null) {
                    SingletonDemo5 = new SingletonDemo();
                }
            }
        }
        return SingletonDemo;
    }
}
```

**volatile必须加！**

`SingletonDemo = new SingletonDemo();`是一个非原子操作，要禁止指令重排序！

# Java集合

顶层接口：

- Collection
  
  - List
    - Vector：线程安全，直接在方法上增加synchronized；
    - ArrayList
    - LinkedList
  - Set
    - HashSet

- Map
  
  - HashMap
  - Hashtable

## ArrayList

- 底层是Object数组，初始容量为10；
- 可以动态扩容（每次扩容1.5倍），使用Arrays.copyOf()进行扩容；
- 可以在O(1)复杂度下完成随即查找；
- 删除、添加操作最坏需要O(N)的复杂度；
- 非线程安全；
- 内存空间存在浪费，一般都会存在预留空间；

## LinkedList

- 底层Node节点的双向链表，并记录首尾节点；
- 无法随机访问，查询需要从头遍历；
- 首尾添加、删除操作可以达到O(1)复杂度；
- 删除指定节点，需要从头遍历查找；

## HashSet

底层使用`HashMap`，元素作为`HashMap`的`key`

- 无序（输出顺序按哈希值）
- 非线程安全
- 元素可以为`null`

## LinkedHashMap

继承HashMap

- 维护了一个双向链表，默认从尾部插入节点，可以保证插入顺序（`accessOrder=true`）；

- 使用Entry节点存储数据，Entry静态内部类继承自HashMap的Node，并扩展为双向链表；

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    // 前后指针
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

### 实现LRU

只需要重写：removeEldestEntry() 方法

```java
public class LRUCacheAPI extends LinkedHashMap<Integer, Integer> {
    private int capacity;
    public LRUCacheAPI(int capacity) {
        // accessOrder=true：按插入顺序输出
        super(capacity, 0.75F, true);
        this.capacity = capacity;
    }
    public int get(int key) {
        return super.getOrDefault(key, -1);// 不存在返回默认-1
    }
    public void set(int key, int value) {
        super.put(key, value);
    }
    // remove最近最少使用的key的时机
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;//当size>容量
    }
}
```

## CopyOnWriteArrayList

解决脏读问题；牺牲写的效率，提高读的效率

CopyOnWriteArrayList是一种读写分离的思想体现的ArrayList；

写的过程中，首先加锁，然后复制出一片新的内存，在新的内存中执行完成写操作，再赋值回去，完成写操作；

读取的话，就直接读取原内存数据；

在写的过程中，可以进行并发的读，因为操作的并不是同一片内存；

（通过开辟新的空间，来避免了java.util.ConcurrentModificationException并发修改异常）

缺陷：

- 需要更多的内存；
- 不满足实时读场景；只能说是作到数据的最终一致性；

### 构造器

直接看源码，不是很复杂；

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    final transient ReentrantLock lock = new ReentrantLock();
    private transient volatile Object[] array;    //数据都会存储在这个array中
    // 提供set，get方法
    final Object[] getArray() {
        return array;
    }
    final void setArray(Object[] a) {
        array = a;
    }
    // ①无参构造器
    public CopyOnWriteArrayList() {
        setArray(new Object[0]); // 初始化一个长度0的array
    }
    // ②将一个Collection转化为CopyOnWriteArrayList
    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        // 此Collection就是CopyOnWriteArrayList，直接赋值
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
        // 其余都转化为Object数组赋值
            elements = c.toArray();
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }
    // ③直接传入一个泛型数组
    public CopyOnWriteArrayList(E[] toCopyIn) {
        setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
    }
```

### add

添加方法，此类的重点方法：即：写操作的实现；

CopyOnWrite容器，即写时复制的容器；

添加元素的时候，并不直接添加，而是先将array复制一份给 Object[ ] newElements，且newElements长度加1，表示要添加一个元素；

添加的操作，都是针对newElements进行的，不对原array进行操作；

这样就将，写操作的内存，与读操作的内存分离开来，写的过程不会影响并发读取；

源码：

```java
// 添加方法
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();// 枷锁
        try {
            Object[] elements = getArray();
            int len = elements.length;
            // 复制出一个newElements:浅层拷贝
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;// 添加元素
            setArray(newElements);// 将添加完成的newElements赋值给原array
            return true;
        } finally {
            lock.unlock();
        }
    }
final Object[] getArray() {
    return array;
}
final void setArray(Object[] a) {
    array = a;
}
```

### get

读取操作，没有做任何的同步措施；

array并不会发生修改，只会在写操作后，直接替换，不存在数据安全问题；

```java
public E get(int index) {
    return get(getArray(), index);
}
private E get(Object[] a, int index) {
    return (E) a[index];
}
final Object[] getArray() {
    return array;
}
```

# IO

## 同步与异步

- **同步：** 同步就是发起一个调用后，被调用者未处理完请求之前，调用不返回。
- **异步：** 异步就是发起一个调用后，立刻得到被调用者的回应表示已接收到请求，但是被调用者并没有返回结果，此时我们可以处理其他的请求，被调用者通常依靠事件，回调等机制来通知调用者其返回结果。

同步和异步的区别最大在于异步的话调用者不需要等待处理结果，被调用者会通过回调等机制来通知调用者其返回结果。

**阻塞和非阻塞**

- **阻塞：** 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
- **非阻塞：** 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。

你烧水，在那里傻等着水开（**同步阻塞**）

时不时来看看水开了没有（**同步非阻塞**）

用上了水开了会发出声音的壶，这样你就只需要听到响声后就知道水开了，在这期间你可以随便干自己的事情，你需要去倒水了（**异步非阻塞**）。

## BIO

**同步阻塞IO**

- jdk1.4之前只有BIO：传统Java的IO模型；
- 即；一个连接一个线程；
- 可通过线程池改善性能；
- 开发简单；

模型：



实现流程：

1. WebServer启动一个ServerSocket
2. 客户端启动Socket，对WebServer进行通信，默认情况下建立一个线程与之通信；
3. 客户端发送请求，先咨询服务器是否有线程响应，如果没有：等待；如果有：响应；
4. 响应之后，客户端线程等待返回结果，继续执行下一个请求；
5. 如果没有数据，线程会阻塞在 read/write方法处；

## NIO

Java-No-blocking IO

同步非阻塞IO



三大核心：

Selector，Channel，Buffer

- **面向缓冲**，**面向块**的IO模型；
- socket线程从通道中进行**请求**或者**获取数据**；
- 数据不可用的时候，线程不会阻塞，可以执行其他任务；

### Buffer

- 是一个内存块，底层是一个数组（最常用`ByteBuffer`）；
- Buffer双向读写，需要`flip()`进行切换；（输入，输出流是单向的）；

```java
public abstract class CharBuffer extends Buffer
{
    final char[] hb;
    final int offset;
    boolean isReadOnly;
    ....
}
```

- buffer可以是只读`buff.asReadOnlyBuffer()`
- `MapByteBuffer`：使用堆外内存；

### Selector（多路复用）

- 一个Selector对应**一个线程**，**可注册多个Channel**；
- Selector切换哪个Channel是由**事件**决定的；
- Selecoter能检测到Channel上的事件，根据不同的事件，在各个通道上切换；
- 多路复用：一个IO线程就可以并发处理多个连接；

```java
public abstract class Selector implements Closeable {
    // 得到一个选择器对象
    public static Selector open();     

    /**
     * 监控所有的注册通道，当其中有IO操作时，将其对应的SelectionKey放入内部集合并返回
     * select是阻塞方法
     * select(long timeout) 阻塞timeout时间后返回
     * selectNow:看一眼，有没有事件，立刻返回
     */
    public abstract int select() throws IOException;
    public abstract int select(long timeout);
    public abstract int selectNow() throws IOException;
    // 从内部集合获得所有SelectionKey
    public abstract Set<SelectionKey> keys();

    // 唤醒selector
    public abstract Selector wakeup();
    ....
}
```

Selector通过`SelectionKey`判断，是`Channel`中是什么事件（读，写，注册）

### SelectionKey

将Channel和Selector进行关联的一个对象；

Channel的事件就存放在这个对象中；

其中事件：由ops来代表；

```java
public static final int OP_READ = 1 << 0;    // 读事件 1
public static final int OP_WRITE = 1 << 2;    // 写事件 4
public static final int OP_CONNECT = 1 << 3;// 注册事件 8
public static final int OP_ACCEPT = 1 << 4;    // 连接事件 16
```

此方法：告诉seletor—此SelectionKey的Channel是哪个；

```java
// 返回关联的selector
public abstract Selector selector();
// 返回关联的channel
public abstract SelectableChannel channel();
```

### Channel

- 一个Channel对应一个Buffer；

- 双向通道，可读可写；流是单向的；

- 异步读写数据；

常用的Channel

```java
FileChannel            // 本地文件读写
DatagramChannel        // UDP读写
ServerSocketChannel    // TCP读写
```

主要方法：

```java
read            // 从buffer读
write            // 向buffer写
transferFrom    // 复制buffer的数据
transferTo        // 复制给buffer数据
```

#### SocketChannel / ServerSocketChannel

ServerSocketChannel：监听客户端连接

SocketChannel：网络IO通道，负责读写缓冲区

都需要向Selector注册；

通过此方法，对selector进行注册；

```java
public final SelectionKey register(Selector sel, int ops)
```

- Selector sel：注册的selector
- ops：`SelectionKey`内部的四种值

### NIO网络模型

1. 客户端连接：
   
   通过`ServerSocketChannel`得到`SocketChannel`（获取一个连接）

2. `SocketChannel`会在`Selector`上进行注册
   
   通过`register(Selector sel, int ops)`进行注册，并返回一个`SelectorKey`；

3. `SelectorKey`会将`SelectableChannel`和`Selelctor`进行关联；
   
   Selector也是通过这个key获取Channel的事件的；

4. Selector进行通过`select()`方法监听Channel
   
   并返回有事件发生的通道的个数；

5. Seletor获取有事件发生的各个`SelectorKey`，进一步获取有事件发生的`SelectableChannel`；

6. 然后依次处理响应的事件；

## BIO/NIO对比

| BIO       | NIO              |
|:---------:|:----------------:|
| 流处理       | 块处理              |
| 阻塞        | 非阻塞              |
| 基于字节流，字符流 | 基于Channel，Buffer |

- NIO数据总是从通道读取到缓冲区，从缓冲区写入到通道；
- Selector监听多个通道的事件（事件驱动模型）；

## 零拷贝

CPU 的所有指令中，有些指令是非常危险的；

所以分用户空间，内核空间，防止指令的乱用；

### 传统IO拷贝

首先看一下**传统IO拷贝**的流程：



1. **用户空间**向**内核空间**发送一个读取数据的系统调用：read()；（上下文切换）

2. 内核空间向磁盘发送读取数据请求；

3. 内核空间通过**DMA**（直接内存访问），将磁盘数据，读取到内核的缓冲区；

4. 内核将数据copy到用户空间缓冲区；（上下文切换）

5. **用户空间**向**内核空间**发送一个写入socket数据的系统调用：write()；（上下文切换）
   
   真正将数据写入网络

6. 内核空间通过硬件写入数据到网络；

7. write返回；（上下文切换）

图中，蓝色框住的操作，是不必要的，浪费的操作；

- 没有必要的两次上下文切换；
- 没有必要的copy的操作；

### 操作系统零拷贝

1. 用户空间使用sendfile的系统调用，而不是read；
2. 数据通过**DMA COPY**拷贝到`内核缓冲区`，在内核中，通过**CPU COPY**到`Socket缓冲区`；
3. 将`Socket缓冲区`数据，通过**DMA COPY**到底层协议中，发送到网络；

数据不再拷贝到用户空间，所以是零拷贝；

# IO多路复用

IO多路复用：一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作；
