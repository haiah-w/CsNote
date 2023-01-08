# Redis布隆过滤器

[Redis 中的布隆过滤器](https://segmentfault.com/a/1190000016721700)

# Redis实现幂等

# Redis消息队列

# Redisson

分布式锁：自动续时，只解当前线程的锁

```java
RLock rlock = Redisson.getLock("redisson-product-1");
rlock.lock(3,TimeUnit.SECONDS);
// 业务
rlock.unlock();
```

lock()

是一个阻塞方法，其他线程会阻塞住；

拿锁线程，如果解锁，会通过发布订阅模型，通知其他线程；

`lock(10,TimeUnit.SECONDS)`：10秒解锁；

tryLock()

非阻塞；

`tryLock()`：没拿到锁的线程，立即返回`false`；

`tryLock(100,10,TimeUnit.SECONDS)`：尝试100s，如果拿到锁，设置10s超时时间；

读写锁

```java
RReadWriteLock rwlock = redisson.getReadWriteLock("anyRWLock");
// 最常见的使用方法
rwlock.readLock().lock();
// 或
rwlock.writeLock().lock();
```

信号量

```java
RSemaphore semaphore = redisson.getSemaphore("lock-semaphore");
// 获取,可设置过期时间
semaphore.acquire();
String permitId = semaphore.acquire(2, TimeUnit.SECONDS);
// 释放
semaphore.release();
// 非阻塞获取
semaphore.tryAcquire();
```

- acquire()：阻塞方法；
- release()：释放

闭锁（CountDownLatch）

await的线程必须等到count为0；

```java
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.trySetCount(10);
latch.await();
// ...
latch.countDown();
```
