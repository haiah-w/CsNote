CompletableFuture:
1. 多任务并行
2. 顺序任务存在依赖：单依赖、双依赖、多依赖；



HttpClient客户端：
1. Apache HttpClient
2. OKHttp
3. Spring5 WebClient


MDC：映射调试上下文
Log4j、Logback提供的多线程条件下记录日志的功能；
MDC为当前线程上下文中的一个map，向其添加键值对，以在并发环境下对单个线程做到链路追踪；
