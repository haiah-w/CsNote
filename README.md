工单策略：
工单模块，是一个commons功能，提供定时拉取数据库工单，进行业务处理的功能；

并提供重试、可配置重试间隔，类似于JDK提供的调度策略的线程池；

提供一个接口，我们实现周期任务的业务逻辑，在配置文件中配置线程池的参数、调度策略等等，比如每秒拉取一次工单，失败30s后重试，重试1个小时等等策略；

执行流程：

1、项目启动，读取策略配置，如：配置的所有任务类别、线程池参数、重试策略等；
2、初始化worker，业务方自己实现的业务逻辑，放到一个Map中，key用来标识任务类别；value就是执行此任务的worker；
3、初始化线程池，我们对接每一个业务，单独一个线程池处理，便于管理、调节，业务之间的业务量是不同的；
4、初始化调度策略，每一个策略包括：执行任务的间隔、重试间隔、重试时长、指定调度的数据表；
5、初始化

总体：根据配置，在启动时，不同的调度任务有各自的调度策略、线程池，不同的业务互相隔离；


执行过程：
一个轮询DB的线程池，将数据放入延迟队列，一个单线程消费此队列
消费到任务，则根据任务的类型，交给对应的线程池执行任务；

spring的发布订阅




java DelayQueue take阻塞
