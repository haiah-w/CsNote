# 文件描述符

linux系统一切皆文件；文件描述符就是文件的<mark>索引</mark>；

内核为了高效管理进程，为每个进程为一个打开的文件记录表；

每一个打开的文件，都对应一个索引值，即文件描述符；

# 系统调用

open()：打开、创建文件；

```c
 int open(const char *pathname, int flags, mode_t mode);
```

- pathname：要打开的文件位置

- flags：定义文件行为（阻塞式、）
  
  - O_SYNC：写同步
  
  - O_NONBLOCK：非阻塞式
  
  - O_AYNC：异步

- mode：文件读写方式：O_RDONLY, O_WRONLY, O_RDWR.

read()：

```c
ssize_t read(int fd, void *buf, size_t count);
```

- fd:指定要读取的文件；

- buf：由用户线程设定的缓存区，存放读取的内容；

- count：读取的字节数

- 成功：返回读取到的字节数；错误：-1；无数据：0；

write()

```c
ssize_t write(int fd, const void *buf, size_t nbyte);
```





## aio

Linux-aio接口提供一系列异步接口：

systemcall:aio_read()

```c
int aio_read(struct aiocb *aiocbp);
```

## io_uring
