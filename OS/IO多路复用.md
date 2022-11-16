# I/O Multiplexing

1、针对要想内核获取事件这一接口来说，是<mark>应该阻塞</mark>的；不占用CPU，并且能第一时间收到就绪事件；

2、对于读写事件来说，<mark>应该使用非阻塞IO</mark>，因为只有真正能够读写的时候，才去进行读写，不会出现阻塞IO在数据未就绪时也“空等”的情况；

<mark>IO多路复用</mark>单独使用一个阻塞线程，专门监听事件的状态，当事件就绪，交由其他非阻塞线程处理；

IO多路复用的实现：select、poll、epoll

# select/poll

多路复用线程调用select、poll，阻塞等待事件集合返回，拿到fd集合，根据集合中的就绪事件，处理不同的事件（可由多线程处理）；

区别：

1、返回大小不同

- selec使用fd_set，默认1024，可调整，但是有限；（现在好像不限制了）

- poll时使用pollfd，基本无限制，基于内存；

2、fd_set返回后处理机制不同：

- select会返回所有fd，具体是否就绪，什么事件就绪，由用户线程遍历处理；

- poll仅返回就绪事件，直接处理即可；

select/poll缺陷：

1、大量的用户、内核间拷贝；

2、需要对返回结果遍历；

# epoll

参考：[刘丹冰—epoll的API及内部机制](https://www.bilibili.com/video/BV1jK4y1N7ST/?p=3&spm_id_from=pageDriver&vd_source=ce67cf212f4a949cf75348b5404c5e27)

epoll接口的核心是在内核创建一个epoll数据结构：**一个包含两个列表的容器**；

能够处理的最大请求，取决与系统可打开的文件描述符个数；

> `cat /proc/sys/fs/file-max`查询可得

## epll接口

1、epoll_create

```c
int epoll_create(int size);
```

- size：需要监听的socket数量；现在已经不需要了，内核会动态调整，但在调用时，仍需要传递大于0的一个数；

创建epoll实例，返回一个文件描述符指向创建的epoll实例；

此fd用于后续所有对epoll接口的调用，不需要时则删除此fd，内核则会销毁epoll实例，释放所有关联的资源；

本质是创建了一颗红黑树，size为节点个数；

2、epoll_ctl

```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

- epfd：指向epoll实例的fd，由create创建；

- fd：需要执行以上动作的文件描述符（红黑树中的一颗节点）

- epoll_event：想要关联到fd上的事件；
  
  - EPOLLIN：对fd绑定读事件，可以执行read()系统调用；
  
  - EPOLLOUT：对fd绑写读事件，可以执行write()系统调用；
  
  - .....

event是一个位掩码，可以绑定多个：

`epoll_event.events = EPOLLIN | EPOLLOUT`

- op：需要对fd执行的动作：
  
  - EPOLL_CTL_ADD：将此fd添加进红黑树，并绑定event；
  
  - EPOLL_CTL_MOD：将event绑定到fd上；
  
  - EPOLL_CTL_DEL：注销、删除fd，此时event参数被忽略，可以为null；

epoll_ctl就是在操作已经创建完成的epoll实例中的节点，添加、修改、删除指定fd上的监听事件；

执行过程：

用户创建epoll_event结构体，并添加事件、用户数据、指定fd，然后调用epoll_ctl；内核会在epfd红黑树中的对应fd上进行事件绑定；当此fd事件就绪时，就触发对应的事件；

3、epoll_wait

```c
int epoll_wait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout);
```

- epfd：监听等待的epoll实例；

- events：是一个epoll_event空列表，有就绪fd时，内核会将对应epoll_event结构体，放入数组中，返回后，用户线程进行处理；

- maxevents：大于等于events的大小；

- timeout：超时时间；
  
  - timeout=-1，永久阻塞，直到有时间返回；
  
  - timeout=0，非阻塞，立即返回，无论有无就绪IO；
  
  - timeout>0，阻塞指定时间后返回；

epoll_wait就是在等待epoll实例上的事件发生，如果没有事件，则阻塞；

## epoll实现

```c
// 创建epoll
int epfd = epoll_create(1000);
// 添加事件
epoll_ctl(epfd, EPOLL_CTL_ADD, listen_fd, &listen_event);
// 循环监听
while(1){
    int active_cnt = epoll_wait(epfd, events, 1000, -1);
    for(i=0; i<active_cnt; i++){
        if (event[i].data.fd & EPOLLIN){
            // TODO read
        ) else if (event[i].data.fd & EPOLLOUT) {
            // TODO write
        }
    }
}
```

## epoll水平触发、边沿触发

数字电路中由电平、边沿触发

- 电平触发（水平触发）：当处于高电平时，则保持触发，处理完成降为低电平；

- 边沿触发：只有从低电平到高电平变化的瞬间，才会输出，输出完成，不会再次输出；

epoll中则是：

- 水平触发：数据未处理，则下次调用wait，仍然可以返回未处理的数据；
  
  - 容错高；

- 边沿触发：只会返回一次，无论是否处理；
  
  - 效率高；

# netpoll

# IO多路复用模型

## 单线程

```java
while(channel=Selector.select()){   // 阻塞
  if(channel.event==accept){
      // accept
  }
  if(channel.event==write){
      // write 非阻塞
  }
  if(channel.event==read){
      // read 非阻塞
  }
}
```
