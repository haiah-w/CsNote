Spring事务的核心是：传播特性；

事务传播特性解决两个问题：

1、事务嵌套时，事物之间是共用事务，还是开启新的事务

2、事务嵌套时，如果发生异常，是回滚，还是提交；

---

# 事务的分类

1. 编程式事务；业务事务隔离；
2. 声明式事务；（XML，注解来实现事务）：灵活但是难以维护

# Spring事务隔离级别

| 隔离界别                       |                                         |
| -------------------------- | --------------------------------------- |
| ISOLATION_DEFAULT          | 使用后端数据库默认的隔离级别；                         |
| ISOLATION_READ_UNCOMMITTED | 读未提交：允许读取尚未提交的更改；（存在脏读、不可重复读、幻读）        |
| ISOLATION_READ_COMMITTED   | 读已提交：允许从已经提交的并发事务读取；（存在不可重复读、幻读）        |
| ISOLATION_REPEATABLE_READ  | 可重复读：对相同字段的多次读取的结果是一致的；（存在幻读）MYSQL默认级别； |
| ISOLATION_SERIALIZABLE     | 串行化；完全的隔离；速度最慢；                         |

# Spring事务传播行为

事务的传播行为：当前的事务下的子事务，是否共用一个事务

- 调用本类的方法，不能开启新事务
- 如果要调用本类的方法，并使用事务传播，需要代理对象

`@Transactional(propagation = Propagation.REQUIRED)`

常用的就前面几个；（三类七种）

```properties
################## 共用一个事务 ##################
# 共用一个事务，如果没有事务，就开启一个新的
PROPAGATION_REQUIRED
# 共用一个事务，如果没有事务，就不用事务
PROPAGATION_SUPPORTS
# 共用一个事务，如果没有事务，就抛出异常
PROPAGATION_MANDATORY
################## 不共用一个事务 ##################
# 新建一个事务，并把父事务挂起
PROPAGATION_REQUIRES_NEW
# 不使用事务，并把父事务挂起
PROPAGATION_NOT_SUPPORTED
# 不使用事务，如果存在父事务，就抛出异常
PROPAGATION_NEVER
#################### 其他情况 ####################
PROPAGATION_NESTED
```

场景：

```java
//事务A REQUIRED
Service.A(){ 
    ServiceB.B();// 事务B REQUIRES_NEW
    ServiceC.C();// 事务C REQUIRED
    ServiceD.D();// 事务D REQUIRES_NEW
    try{
        ServiceE.E() // 事务C REQUIRED
    }catch(Exception e){ }
}
```

- 事务B失败，C成功，那么：A成功；D也不受影响；
- 事务C失败，那么：A失败；（AC共用事务），B不受影响；D无法执行；
- 事务E失败，不会影响别的，因为catch了；（编译时异常）

商户添加商品的接口的问题：

1. 操作的数据表太多，回滚问题；

商品的数据涉及很多个表！如果某个数据出了问题，插入失败，只能全部回滚；

如果在添加商品的`Service`上直接加上`@Transactional`，就会使上面的情况，全部回滚；

2. 并发访问的问题：很多商户同时操作的线程安全问题；

这种做法不可取！

# Spring事务失效情况

Spring中通过注解@Transactional来实现事务，但在以下几种情况时，事务会失效。

- Spring中事务自调用会失效，如果A方法调用B方法，B方法上有事务注解，AB方法在同一个类中，那么B方法的事务就会失效，这是动态代理的原因。
- Spring的事务注解@Transactional只能作用在public修饰的方法上才起作用，如果放在非public（如private、protected）方法上，虽然不报错，但是事务不起作用。
- 如果MySQL用的引擎是MyISAM，则事务会不起作用，原因是MyISAM不支持事务，可以改成InnoDB引擎。
- Spring建议在具体的类（或类的方法）上使用@Transactional注解，而不要使用在类所实现的任何接口上。在接口上使用@Transactional注解，只能当你设置了基于接口的代理时他才会生效，因为注解是不能继承的，这就意味着如果正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装。
- 如果在业务代码中抛出RuntimeException异常，事务回滚；但是抛出Exception，事务不回滚。

需要注意的是，@Transactional也可以作用于类上，放在类级别上等同于该类的每个公有方法都放上了@Transactional。

# 事务例子

解决第一个问题：将大事务拆分为小事务，分别回滚；

即：商品数据分离：核心数据，非核心数据，商品打折信息 .................

将多个数据的保存方法，分离出来；

如果说折扣信息的数据录入失败，回滚的只是折扣信息；可以让商户只需要重新录入折扣信息；

（下面代码行不通！！！！！！！！！）

```java
@Transactional(propagation = Propagation.REQUIRED)
@Override
public void saveProduct(PmsProductParam productParam) 
    // TODO: 保存商品核心数据
    Product product = saveBaseInfo(productParam);
    // TODO: 保存商品折扣信息，需要上一步创建完成的商品
    saveProductLadder(product);
    ...
}
// 新建事务：挂起父事务
// TODO: 保存商品核心数据
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveBaseInfo(PmsProductParam productParam) {
    ...
}
// 新建事务：挂起父事务
// TODO: 保存商品折扣信息
@Transactional(propagation = Propagation.REQUIRES_NEW,
               rollbackFor = FileNotFoundException.class,
               noRollbackFor = {ArithmeticException.class, NullPointerException.class})
public void saveProductLadder(PmsProductParam productParam) {
    ...
}
```

## 代理对象

解决：调用本类方法无法开启事务

1. 导入AOP，开启代理对象；
   
   ```xml
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```

2. 启动类加上注解：
   
   ```java
   @EnableTransactionManagement // 显式开启事务
   @EnableAspectJAutoProxy(exposeProxy = true)// 暴漏代理对象
   ```

3. 获取本类的代理对象，去调用本类的方法；
   
   ```java
   @Transactional(propagation = Propagation.REQUIRED)
   @Override
   public void saveProduct(PmsProductParam productParam) 
     // 获取本类代理对象
     ProductServiceImpl proxy = (ProductServiceImpl) AopContext.currentProxy();
     // TODO: 保存商品核心数据
     Product product = proxy.saveBaseInfo(productParam);
     // TODO: 保存商品折扣信息，需要上一步创建完成的商品
     proxy.saveProductLadder(product);
     ...
   }
   ```

## 异常回滚策略

```java
@Transactional(propagation = Propagation.REQUIRES_NEW,
               rollbackFor = FileNotFoundException.class,
               noRollbackFor = {ArithmeticException.class, NullPointerException.class})
```

- rollbackFor：指定异常一定回滚；
- noRollbackFor：指定异常不回滚；

## ThreadLocal

用户请求流程：

Tomcat接受请求，创建一个请求线程

请求线程依次调用：Controller(`productService`) ---> Service(`productMapper`) ---> Mapper

在每一层使用的Bean都是单例！

在高并发的情况下，多个请求线程，在不断的从单例对象中，拿数据，存数据的时候，无疑会出现共享数据的线程安全问题；

比如说：我们分离了保存商品的逻辑

1. 线程A：第一次保存商品的核心数据，需要在数据库中创建一个商品A；
2. 线程B：第一次保存商品的核心数据，创建商品B
3. 线程A：第二次保存非核心数据，拿到上一个创建的商品的`id`，就拿到了商品B的id；然后将数据存入了商品B；

上面的情况，就出现了错误；

解决：

使用`ThreadLocal`：保证每个线程的数据安全；

1. 线程A：第一次保存，创建商品A，在`ThreadLocal`中保存商品A的`id`；
2. 线程B：第一次保存，创建商品B，在`ThreadLocal`中保存商品B的`id`；
3. 线程A：第二次保存非核心数据，从`ThreadLocal`，拿到商品A的`id`；
