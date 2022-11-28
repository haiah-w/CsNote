一、CompletableFuture:

1. 多任务并行
2. 顺序任务存在依赖：单依赖、双依赖、多依赖；

二、HttpClient客户端：

1. Apache HttpClient
2. OKHttp
3. Spring5 WebClient

三、MDC：映射调试上下文
Log4j、Logback提供的多线程条件下记录日志的功能；
MDC为当前线程上下文中的一个map，向其添加键值对，以在并发环境下对单个线程做到链路追踪；

四、InheritableThreadLocal/ThreadLocal
InheritableThreadLocal：子线程继承父线程的变量；
https://zhuanlan.zhihu.com/p/131299779

五、websockets
HTML5支持的一种新协议：基于TCP，支持服务端推送消息、支持长连接；

设计模式：
发布订阅（观察者模式）
责任链
