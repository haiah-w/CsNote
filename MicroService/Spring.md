# Spring/springMVC/SpringBoot

（1）Spring ：是一个开源框架，为简化企业级应用开发而生；

核心：IOC、AOP

（2）SpringMVC：是一个基于MVC开发模式的web开发框架；

（3）SpringBoot：是基于Spring进行二次开发的框架；将基于Spring的开发，更轻量、配置更简化；

spring、springboot区别：

1、SpringBoot嵌入了Tomcat容器；

2、将大量的XML配置简化为Java配置，将Bean的注入方式，简化为注解方式；额外的配置尽可能地浓缩在appliaction.yml配置文件中；

3、提供多种stater简化配置；

4、SpringBoot尽可能自动化配置Spring的功能；

# SpringAOP

（Aspect-Oriented Programming）面向切面编程；使用的是**代理模式**的设计；

SpringAOP是基于动态代理，也有静态代理AspectJ；

执行过程：

> 在内存中临时为方法生成一个AOP对象，这个对象包含目标对象的所有方法，在特定的切点做了增强处理，并回调原来的方法

- 静态代理：AspectJ（编译期织入到字节码文件）

- 动态代理：SpringAOP（运行期临时生成代理对象，必须是代理对象，this调用AOP失效）
  
  - JDK动态代理：需要接口（默认代理方式）
  - CGLib动态代理：不需要接口
  
  对比：
  
  - JDK创建代理对象快，但是运行慢；
  - CGLib创建慢，运行快；

- 动静对比：
  
  - AspectJ性能高，但是需要特定编译器；
  - SpringAOP比较方便；

## AOP名词

```java
// target被代理对象
class UserService{
    public void f1(){}    // joinpoint连接点
    public void f2(){}
}
```

- `Joinpoint`：连接点；被增强的方法即连接点；
- `Pointcut`：切点，就是连接点；
- `Target`：被代理对象；只有通过代理对象，才能织入；
- `Advice`：通知；增强的具体逻辑，叫做通知；
  - 前置通知(Before Advice)
  - 返回后通知(After returning advice)：正常返回结果，没有抛出异常，才会执行；
  - 后置通知(After throwing advice)：无论如何，方法退出后，都会执行；
  - 环绕通知(Finally advice)
  - 抛出异常通知(Around Advice)
- `Weaving`：织入；通过`代理对象`将`Advice`（增强）应用到`Target`（被代理对象）的过程，即织入；

## AOP实现注解记录日志

首先定义一个注解；`LogAnnotation`

在需要记录日志的方法上，加上这个注解；

然后定义一个切面通知类`LogAspect`；

1. 需要切点：即要增强的地方；(切点就是定义的注解)
   
   在执行定义了此注解的方法上，就会织入，我们需要增强的额外的方法；

2. 需要通知：就是具体要增强的内容；

```java
@Aspect
@Component
public class LogAspect {
    // 注入被增强的对象
    @Autowired
    private SysLogService sysLogService;
    // 切点
    @Pointcut("@annotation(pmp.server.annotation.LogAnnotation)")
    public void logPointCut(){

    }
    // 通知
    @Around("logPointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable{
        // 执行原方法
        Object result=point.proceed();
        // 进行增强
        saveLog(point,time);
        // 返回原方法的返回值
        return result;
    }
}
```

## AOP失效

通过`this`调用本类中的方法，如果此方法带有`@Transaction`，`@Cacheable`，`@Asysnc`等注解；注解不会生效；

aop会通过代理对象，调用需要增强的方法，对其进行增强；

但是需要增强的方法内的`this`调用的方法，无法通过代理对象，去调用，也就无法增强；

(前提，this调用的方法是`private`，增强类无法访问`private`方法)

解决：

1、通过ApplicationContext来获得当前this对象的动态代理对象

```java
// xxxxService为当前服务对象
this.applicationContext.getBean(xxxxService.class)
```

2、通过AopContext获取动态代理对象；

```java
((xxxxService) AopContext.currentProxy())
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
       newProxyInstance(Test.class.getClassLoader(), Test.class.getInterfaces(), invokeHandler);
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

通过ASM包，将被代理类的字节码文件，加载进来，修改字节码文件生成一个被代理类的子类；

然后创建代理，这个代理是被代理类的子类对象；

通过CGlib下的方法拦截器，对被代理类的方法进行一个增强；

然后就可以使用代理对象，调用对应的方法，并对其增强丰富等；

# SpringIOC

IOC：

- 工厂模式：由工厂统一创建对象；
- 反射机制：通过反射，来进行依赖注入，将配置的属性，注入到对象中；

IOC容器有两种：BeanFactory、ApplicationContext 

（婚介所！）

我们需要创建对象的时候，不需要考虑对象的创建过程，只需要配置好我们需要一个什么样的对象即可；

然后 创建对象的任务，交由IOC容器来完成；

会根据我们的配置，创建出我们需要的对象；

## 依赖注入

`DI依赖注入`和`控制反转`是同一个概念的不同角度的描述，即：应用程序在运行时依赖IoC容器来动态注入对象需要的外部资源；

- 使用Java的反射机制；

注入方式：

- 构造器注入；
- setter注入；

## BeanFactory/ApplicationContext

共同点：

- 两个都是Spring的IOC容器；
- BeanFactory是ApplicationContext的父接口；ApplicationContext对其进行了扩展；

不同点：

- BeanFactory使用懒加载（Bean使用的时候才加载）；

- ApplicationContext是即时加载（应用启动，就加载所有的Bean），可以通过配置实现懒加载；
  
  配置`lazy-init=true`实现

# Bean的生命周期

## Bean初始化前的准备

（1）资源定位：@ComponentScan、@Component

Spring找到需要加载的资源；

（2）解析Bean信息保存到`beanDefinition`中；

主要是拿到Bean的id、type等属性；

（3）发布到IOC容器；

将Bean的定义发送到SpringIOC容器中，容器中此时只有Bean的定义，没有Bean的实例；

（4）对于BeanFactory来说，到此为止，只有用到某个Bean，才会加载；

（4）对于ApplicationContext来说，会继续完成所有Bean的初始化；

## Bean的初始化过程

实例化——属性赋值——初始化（前置处理，后置处理）——使用Bean——销毁Bean（如果定义了销毁方法）；

1、实例化Bean

通过`BeanFactory`容器，根据请求的Bean，进行实例化，此时并未依赖注入；

2、属性赋值

（1）通过DI将需要的属性注入（设置对象属性）

（2）处理Aware接口

比如BeanNameAware接口，让Bean感知自己的name信息；

3、初始化

到此，Bean对象已经构造完成，如果有额外的自定义处理，就通过`BeanPostProcessor`接口实现；

（1）`BeanPostProcessor`前置处理，初始化Bean之前的处理；

（2）初始化Bean（包括：InitializingBean和自定义的init-method）

（3）`BeanPostProcessor`后置处理，初始化Bean之后的处理；

4、使用Bean

5、销毁Bean，如果实现了`DisposableBean`接口，会执行自定义的`destroy`方法

## Bean的作用域

1、singleton：单例Bean

当前容器只会创建一个Bean实例；

2、prototype：

每次一请求Bean，都会创建一个Bean实例；（可能一次请求，对同一个Bean，创建了多次，也是不同的Bean）；

（可以解决单例Bean的并发安全问题，

也可以通过`ThreadLocal`来解决，每次请求为一个独立线程，创建此线程的独立副本）

3、request：

每一次网络请求，创建一个实例；此Bean仅限与当前请求内；

4、session：

与request类似，一次会话创建一个Bean实例；（同一个会话下，多次请求，只会创建一个Bean）

5、global-session：

全局会话；

## 自动装配

方式：

# Spring解决循环依赖

Spring实例化Bean是通过ApplicationContext.getBean()方法来进行的；

如果创建一个对象，依赖了另一个对象，就会调用ApplicationContext.getBean()方法来获取所依赖的对象，再注入到当前对象中；

## 1、构造器循环依赖

Spring无法解决构造器的循环依赖；

Spring容器会将每一个正在创建的Bean的标识符放在一个池中，如果发现要创建的Bean在这个池中，就会抛出`BeanCurrentylyInCreationException`异常；

这个池子里的Bean创建完成，就会从里面删除；

所以：构造器的循环依赖，Spring会报错；

## 2、setter循环依赖

在单例模式下才存在setter循环依赖；

（1）Spring先用构造器实例化Bean对象，将实例化结束的对象放到一个Map中（就是Spring的缓存中），并且Spring提供获取这个实例化对象的单例工厂方法。（相当于是提前暴露了还未完全创建完成的单例对象，为的就是用于解决循环依赖）

（2）在依赖注入的时候，只需要根据Spring暴露的方法，去拿到这个对象，注入到属性中；

（3）这样就可以互相引用；

setter的循环依赖，Spring会允许通过，不会报错；

# Spring事务

1. 编程式事务；业务事务隔离；
2. 声明式事务；（XML，注解来实现事务）：灵活但是难以维护

## Spring事务隔离级别

**五种**：

| 隔离界别                       |                                         |
| -------------------------- | --------------------------------------- |
| ISOLATION_DEFAULT          | 使用后端数据库默认的隔离级别；                         |
| ISOLATION_READ_UNCOMMITTED | 读未提交：允许读取尚未提交的更改；（存在脏读、不可重复读、幻读）        |
| ISOLATION_READ_COMMITTED   | 读已提交：允许从已经提交的并发事务读取；（存在不可重复读、幻读）        |
| ISOLATION_REPEATABLE_READ  | 可重复读：对相同字段的多次读取的结果是一致的；（存在幻读）MYSQL默认级别； |
| ISOLATION_SERIALIZABLE     | 串行化；完全的隔离；速度最慢；                         |

## Spring事务传播行为

**propagation**

**事务的传播行为：当前的事务下的子事务，是否共用一个事务**

- **调用本类的方法，不能开启新事务！！！！！！**
- **如果要调用本类的方法，并使用事务传播，需要代理对象！！！！！！**

`@Transactional(propagation = Propagation.REQUIRED)`

常用的就前面几个；（**三类七种**）

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

## Spring事务失效情况

Spring中通过注解@Transactional来实现事务，但在以下几种情况时，事务会失效。

- Spring中事务自调用会失效，如果A方法调用B方法，B方法上有事务注解，AB方法在同一个类中，那么B方法的事务就会失效，这是动态代理的原因。
- Spring的事务注解@Transactional只能作用在public修饰的方法上才起作用，如果放在非public（如private、protected）方法上，虽然不报错，但是事务不起作用。
- 如果MySQL用的引擎是MyISAM，则事务会不起作用，原因是MyISAM不支持事务，可以改成InnoDB引擎。
- Spring建议在具体的类（或类的方法）上使用@Transactional注解，而不要使用在类所实现的任何接口上。在接口上使用@Transactional注解，只能当你设置了基于接口的代理时他才会生效，因为注解是不能继承的，这就意味着如果正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装。
- 如果在业务代码中抛出RuntimeException异常，事务回滚；但是抛出Exception，事务不回滚。

需要注意的是，@Transactional也可以作用于类上，放在类级别上等同于该类的每个公有方法都放上了@Transactional。

## 事务的传播行为

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

### 代理对象

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

### 异常回滚策略

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

# SpringMVC工作原理

![image-20200813113847503](image/image-20200813113847503.png)

**①发送请求 :**

用户向服务器发送HTTP请求，请求被Spring MVC的调度控制器DispatcherServlet捕获。

**②映射处理器 :**

DispatcherServlet根据请求URL，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExectuionChain对象的形式返回。

**③处理器适配 :**

DispatcherServlet根据获得Handler，选择一个合适的HandlerAdapter。（如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler()方法）

提取请求Request中的模型数据，填充Handler入参，开始执行Handler（Controller）。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：

- HttpMessageConverter：会将请求信息（如JSON、XML等数据）转换为一个对象。
- 数据转换：对请求消息进行数据转换，如String转换为Integer、Double等。
- 数据格式化：对请求消息进行数据格式化，如将字符串转换为格式化数字或格式化日期等。
- 数据验证：验证数据的有效性（长度、格式等），验证结果存储到BindingResult或error中。（自定义验证机制需要使用注解@InitBinder）

Handler（Controller）执行完成后：

如果判断方法中有@ResponseBody注解，则直接将结果写回用户浏览器。

如果没有@ResponseBody注解，向DispatcherServlet返回一个ModelAndView对象；

**⑤解析试图 :**

根据返回的ModelAndView，选择一个合适的ViewResolver（必须是已经注册到Spring容器中的ViewResolver），解析出View对象，然后返回给DispatcherServlet。

**⑥⑦渲染视图+相应请求 :**

ViewResolver结合Model和View，来渲染视图，并写回给用户浏览器。

# SpringBootApplication

主要包含三个注解：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

### @SpringBootConfiguration

其实就是Spring的@Configuration

标注一个类为配置类；

#### @ComponentScan

将Spring包下的需要的组件，注册到IOC容器中去；

- 扫描注入Spring的Bean的定义到IOC容器中；

@EnableAutoConfiguration：自动配置的核心注解；此注解下还有两个注解：

- @AutoConfigurationPackage：将主配置类所在的包作为自动配置的包进行管理；
- @Import：导入配置；

说人话：

SpringBoot根据配置文件，自动装配需要依赖的类；

再用动态代理的方式，注入到Spring容器里面；

# Spring注解

## @Resource@Autowire

- 都可以用来装配bean，都可以用于字段或setter方法

Bean的自动装配分为：byName（根据Bean的名称），byType（根据Bean的类型）；

`@Autowired`：

默认按**类型装配**，默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false。

可以使用@Qualifier限定装配的Bean的ID；

`@Resource`：

默认按**名称装配**，当找不到与名称匹配的bean时才按照类型进行装配。名称可以通过name属性指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，当注解写在setter方法上时，默认取属性名进行装配。

## @Mapper@Repository

`@Repository`需要额外配置Dao扫描地址；

`@Mapper`=`@Repository`+`@MapperScan`

## @Component和@Bean

@Component和@Bean都是用来：向IOC容器中注入Bean；

@Component：用于标注类；将此类注入IOC容器；

@Bean：用于标注方法，将其返回值注入IOC容器；

## 其他注解

@Controller：

# 设计模式

## 模板模式

SpringBoot通过配置`xxxxTemplate`来整合第三方的工具

**模板模式：**（代码复用强）

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

## 工厂模式

Spring中的` BeanFactory`、ApplicationContext；

- 对象的使用和创建进行分离；

我们在配置Bean的时候，可以给一个唯一标识ID；

然后再需要此对象的时候，通过这个唯一ID，来获取对应的对象；即：DI（依赖注入）

一个接口下可以有多个实现，但是具体，我们需要哪种实现，就交给Spring的一个工厂；

通过这个ID，工厂可以选择，我们需要的那个实现，然后返回给我们一个对象；

## 单例模式

Spring默认使用单例模式来创建对象；

## 代理模式

典型的：AOP实现；

# 线程池配置

比如：商品的详情页；需要查询商品基本属性，SKU属性，配送属性，增值服务等等；

要提高响应速度，用户体验；

多个Service，实现并行执行；

1. 每一个服务创建一个异步线程执行`Callable`；
2. 使用线程池来管理线程；（无限线程，宕机）；
3. 线程池`ThreadLocalPool`：不可以使用无界队列！

**一个服务几个线程池？**

一般2-3个：

- 核心业务线程池（比较大的业务，异步执行）
- 非核心业务线程池（边缘业务：发邮件）

这样做的目的：

- 随时能够释放非核心业务；来增加服务器性能；
- 也能够自定义接口监控线程池；服务器状态；

**线程数设置多少？**

一般是`CPU内核数`；实际：通过压力测试，找到系统的瓶颈；（预估到峰值流量）

实际会设置成：`CPU内核^2-3`（CPU内核数的2-3次方）

**如何做异步线程，感知执行完成，拿到返回结果？**

首先是必须使用`Callable`接口线程；

其次：使用`CompletableFuture`：异步提交线程至执行线程池；

此接口的作用：

- 将多个异步线程的结果合并返回；（多个服务查询结果一并返回）
- 回调：线程执行结束，触发某个动作；
- 感知：异常；

```java
CompletableFuture future1 = CompletableFuture.supplyAsync(() -> {
    // 业务
}, pool).whenComplete((r, e) -> {
    // 线程执行完成，回调处理 返回结果和异常
});
```

**打包多个异步线程的返回结果**

```java
CompletableFuture.allOf(future1, future2, future3).thenRun(() -> {});
```

### 最终代码

**注入线程池：两个**

```java
@Configuration
@PropertySource("classpath:threadPoolProperties.properties")
public class ThreadPoolConfig {
    @Value("${gmall.poll.coreSize}")
    private Integer coreSize;
    @Value("${gmall.poll.maximumPoolSize}")
    private Integer maximumPoolSize;
    @Value("${gmall.poll.queueSize}")
    private Integer queueSize;
    @Bean("mainThreadPool")
    public ThreadPoolExecutor mainThreadPool() {
        LinkedBlockingQueue<Runnable> queue = new LinkedBlockingQueue<>(queueSize);
        ThreadPoolExecutor mainThreadPool = new ThreadPoolExecutor(coreSize, maximumPoolSize, 10, TimeUnit.MINUTES, queue);
        return mainThreadPool;
    }
    @Bean("notMainThreadPool")
    public ThreadPoolExecutor notMainThreadPool() {
        LinkedBlockingQueue<Runnable> queue = new LinkedBlockingQueue<>(queueSize);
        ThreadPoolExecutor notMainThreadPool = new ThreadPoolExecutor(coreSize, maximumPoolSize, 10, TimeUnit.MINUTES, queue);
        return notMainThreadPool;
    }
}
```

服务伪码：

```java
@Autowired
@Qualifier("mainThreadPool")
ThreadPoolExecutor pool;
// 返回结果 model
HashMap<String, Object> json = new HashMap<>();
// 业务1
CompletableFuture future1 = CompletableFuture.supplyAsync(() -> {
    // 查询过程
    String res = "查询SPU完成";
    return res;
}, pool).whenComplete((r, e) -> {
    // 处理返回结果 r 和 异常e
    json.put("f1", r);
});
// 业务2
CompletableFuture future2 = CompletableFuture.supplyAsync(() -> {
    String res = "查询SKU完成";
    return res;
}, pool).whenComplete((r, e) -> {
    json.put("f2", r);
});
// 业务3
CompletableFuture future3 = CompletableFuture.supplyAsync(() -> {
    String res = "查询Sale完成";
    return res;
}, pool).whenComplete((r, e) -> {
    json.put("f3", r);
});
// 阻塞方法 判断是否批量任务执行完毕，执行完毕则回调
CompletableFuture.allOf(future1, future2, future3).thenRun(() -> {
    System.out.println("全部任务完成....");
    // 最终执行结束
    json.forEach((k, v) -> {
        System.out.println(k + "--->" + v);
    });
}).join();
```

# 数据校验

## JSR303

正常：前端+后端进行双重校验

前端校验的缺点：用户浏览器可以将JS功能关闭，不安全；

后端校验：

1. 校验参数的空值（用户名，密码....）
2. 校验是否合法
3. 校验是否存在

SpringBoot提供了：`JSR303`方式进行校验（第三方校验框架：`hibernate-validator`）

比如注册接口：

```java
public Object register(@Valid @RequestBody UmsAdminParam umsAdminParam, BindingResult result)
```

使用JSR303三步：（**在进入方法之前，AOP进行校验逻辑**）

1. 给需要校验数据加上注解（Entity）
   
   (1)`@Length`：校验最大最小长度；
   
   (2)`@NotEmpty`：数据不可为空；（长度为0，一般用于集合，集合长度不为0，也不能为null）
   
   (3)`@Pattern`：符合自定义的正则；
   
   (4)` @NotNull`：不可为null，但可是空（""这就是个空字符）
   
   (5)`@NotBlank`：不可为空白字符，trim（至少一个非空格字符，trim之后不能为空）
   
   ```java
   @ToString
   @Getter
   @Setter
   public class UmsAdminParam {
       @Length(min = 6, max = 18, message = "用户名长度必须是6-18位")
       @ApiModelProperty(value = "用户名", required = true)
       private String username;
       @ApiModelProperty(value = "密码", required = true)
       private String password;
       @NotEmpty
       @ApiModelProperty(value = "用户头像")
       private String icon;
       @Email(message = "邮箱格式不正确，哈哈哈")
       @ApiModelProperty(value = "邮箱")
       private String email;
       @NotNull
       @ApiModelProperty(value = "用户昵称")
       private String nickName;
       @ApiModelProperty(value = "备注")
       private String note;
   }
   ```

2. `@Valid`：接口参数上添加注解；
   
   开启接口的数据校验功能，自动进行上述校验；
   
   **开启之后，保存的数据，都会根据实体类字段上的注解，进行校验！**

3. 感知校验是否成功，接口参数增加：`BindingResult result`（表示校验结果）
   
   需要手动，处理result结果；
   
   ```java
   int errorCount = result.getErrorCount();
   if(errorCount>0){
       // TODO：校验出错
       List<FieldError> fieldErrors = result.getFieldErrors();
       fieldErrors.forEach((fieldError)->{
           String field = fieldError.getField();
           log.debug("属性：{}，传来的值是：{}，校验出错。出错的提示消息：{}",
                     field,fieldError.getRejectedValue(),fieldError.getDefaultMessage());
       });
       // TODO：返回错误响应
   }
   // TODO：校验没出错，完成注册逻辑
   ```

## 统一数据校验

可以将第三步校验结果的处理逻辑，抽出来手写一个切面：（需要导入starter-aop）

**就不需要在接口中进行校验逻辑的处理，统一通过切面方法处理！**

`@RestControllerAdvice`：接口切面通知；拦截对应包下的所有接口；

`MethodArgumentNotValidException`即：校验失败异常

```java
@Slf4j
@RestControllerAdvice(basePackages = "com.muss.product.controller")
public class MussmallExceptionControllerAdvice {

    @ExceptionHandler(value= MethodArgumentNotValidException.class)
    public R handleVaildException(MethodArgumentNotValidException e){
        // 获取校验异常的校验结果对象：BindingResult
        BindingResult bindingResult = e.getBindingResult();
        /**
         * 从校验结果中，取出校验失败的字段，以及对应的失败信息
         * 将对应字段作为key，对应异常作为value
         * 封装进Response的data中
         */
        Map<String,String> errorMap = new HashMap<>();
        bindingResult.getFieldErrors().forEach((fieldError)->{
            errorMap.put(fieldError.getField(),fieldError.getDefaultMessage());
        });
        return R.error(Code,Msg).put("data",errorMap);
    }

    @ExceptionHandler(value = Throwable.class)
    public R handleException(Throwable throwable){
        log.error("错误：",throwable);
        return R.error(Code,Msg);
    }
}
```

## 分组校验

有些字段，只有某些操作下，才需要校验，某些操作，不需要校验；

比如：品牌名字段；

只需要在添加和更新的操作中需要校验品牌名，在删除中不需要校验；

那么就可以进行分组校验！

分组校验实现方式：`@Validated`注解

1、声明两个校验分组接口：

定义两个接口：将不同的操作，进行分组

```java
public interface AddGroup {
}
public interface UpdateGroup {
}
```

2、实体类的校验注解———增加`group`标注；（表示在Add和Update情况下才进行校验）

```java
@NotBlank(message = "品牌名必须提交",groups = {AddGroup.class,UpdateGroup.class})
private String name;
```

表示：此字段品牌名`name`在做添加和修改的时候，才需要进行校验，此注解`@NotBlank`才会进行数据校验；

3、在对应的业务接口处添加需要需要校验的操作：（表示此接口，是新增的操作）

**`@Validated({AddGroup.class})`：表示此接口，只校验AddGroup分组下的字段**

```java
public R save(@Validated({AddGroup.class}) @RequestBody BrandEntity brand){
    //TODO: 新增品牌
}
```

## 自定义校验逻辑

业务复杂，自带的校验无法满足的时候，自定义校验;

导入依赖：

```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

1、编写一个自定义校验注解；

`@Constraint`：指定校验器；

`@Target`：此注解支持的类型：方法，字段....

`@Retention`：注解生效时级

注解内字段，是添加注解时，可以选择的字段；

```java
@Documented
@Constraint(validatedBy = { ListValueConstraintValidator.class })
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
public @interface ListValue {
    // 默认的错误信息
    String message() default "必须提交指定的值";
    Class<?>[] groups() default { };
    Class<? extends Payload>[] payload() default { };
    // 表示标记此注解的字段的值范围
    int[] vals() default { };
}
```

2、编写自定义校验器；

主要校验逻辑：`showStatus`字段的值，是否在我们限制的范围内

```java
public class ListValueConstraintValidator implements ConstraintValidator<ListValue,Integer> {
    private Set<Integer> set = new HashSet<>();
    // 拿到允许的值
    @Override
    public void initialize(ListValue constraintAnnotation) {

        int[] vals = constraintAnnotation.vals();
        for (int val : vals) {
            set.add(val);
        }
    }
    // 判断传入的值，是否在允许范围内
    @Override
    public boolean isValid(Integer value, ConstraintValidatorContext context) {

        return set.contains(value);
    }
}
```

3、字段添加自定义注解：

```java
@ListValue(vals={0,1},groups = {AddGroup.class, UpdateStatusGroup.class})
private Integer showStatus;
```

- 接口接收到此字段，会到自定义的校验器中，进行校验：值是否是0，1中的一个；
- showStatus：只能取0，1值；
- 验证分组为：Add，Update

# 统一异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = {ArithmeticException.class})
    public Object handlerException(Exception exception) {
        log.error("GlobalExceptionHandler：{}", (Object) exception.getStackTrace());
        return new CommonResult().validateFailed("数学没学好");
    }

    @ExceptionHandler(value = {NullPointerException.class})
    public Object handlerException02(Exception exception) {
        log.error("GlobalExceptionHandler：{}", exception.getMessage());
        return new CommonResult().validateFailed("空指针了");
    }
}
```

1. `@RestControllerAdvice`：所有接口如果又异常，即执行此类；（Rest：所有返回Json）
2. `@ExceptionHandler(value = {NullPointerException.class})`：方法上标记，出现空指针异常，即执行此方法；
3. 当存在**环绕通知**的时候，在切面方法中要抛出异常！否则统一异常无法感\！
