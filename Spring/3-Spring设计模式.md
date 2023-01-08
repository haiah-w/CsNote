# 模板模式

SpringBoot通过配置`xxxxTemplate`来整合第三方的工具

父类定义了骨架（定义了方法的执行顺序：方法1，方法2....）

方法的具体实现，交给子类；

子类只要实现方法，方法的执行顺序由父类预先写好了；

典型的：

抽象父类：`AbstractPlatformTransactionManager`

父类下定义了多个抽象方法：

```java
// 开始事务
protected abstract void doBegin(Object transaction, TransactionDefinition definition)
// 拿到当前的事务的对象
protected abstract Object doGetTransaction()
// 提交
protected abstract void doCommit(DefaultTransactionStatus var1);
// 回滚
protected abstract void doRollback(DefaultTransactionStatus var1);
```

然后其子类`DataSourceTransactionManager`

实现了具体的开始事务，提交，回滚，拿到事务对象等方法；

# 工厂模式

Spring中的 `BeanFactory`、ApplicationContext；

- 对象的使用和创建进行分离；

我们在配置Bean的时候，可以给一个唯一标识ID；

然后再需要此对象的时候，通过这个唯一ID，来获取对应的对象；即：DI（依赖注入）

一个接口下可以有多个实现，但是具体，我们需要哪种实现，就交给Spring的一个工厂；

通过这个ID，工厂可以选择，我们需要的那个实现，然后返回给我们一个对象；

# 单例模式

Spring默认使用单例模式来创建对象；

# 代理模式

典型的：AOP实现；
