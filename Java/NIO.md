# NIO

同步非阻塞IO

三大核心：

Selector，Channel，Buffer

IO多路复用：基于epoll实现

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
| --------- | ---------------- |
| 流处理       | 块处理              |
| 阻塞        | 非阻塞              |
| 基于字节流，字符流 | 基于Channel，Buffer |

- NIO数据总是从通道读取到缓冲区，从缓冲区写入到通道；
- Selector监听多个通道的事件（事件驱动模型）；
