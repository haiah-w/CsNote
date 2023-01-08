SpringCore：IOC + AOP

容器+生态

# SpringIOC

IOC：控制反转，整个对象的生命周期都交给Spring容器；

Spring容器使用Map结构来存放要管理的对象；

Spring容器主要通过工厂模式 + 反射来执行创建对象、依赖注入等操作；

---

BeanFactory：Spring容器最顶层接口，提供了最基本的Bean操作(getBean、isSingleton等等)

DefaultListableBeanFactory：BeanFactory的实现类，是一个完整的Bean工厂，在BeanFactory的基础上，增强了IOC功能：

- 管理BeanDefinition(Bean的元数据)

- 增加了Bean初始化、以及Bean的后置处理能力；

ApplicationContext：一个增强版的IOC容器，提供额外的功能：

- 实现MessageSource，提供国际化功能；

- 实现ResourceLoader，提供加载配置文件、环境变量的能力；

- 实现ApplicationEventPublisher，提供事件发布功能；

# DI依赖注入

三种方式：setter、构造器、字段注入

官方推荐：构造器注入，通过有参构造器创建Bean

- 有参构造器创建时，依赖不可为空，否则创建失败报错；

- 只需要一步就可以完成初始化；不需要先实例化，再初始化；

- 字段可以声明为final，依赖不可变；

## 注解注入

@Autowire、@Resource、@Inject

- @Autowire由Spring提供，@Resource、@Inject是JSR250规范注解；

- @Autowire通过类型注入(byType)，@Resource、@Inject通过名称注入(byName)；

- @Autowire当类型相同，可以结合@Qualifier进行Bean的区分;

# Spring启动流程

1、初始化Spring容器Bean，执行准备工作：

首先初始化Spring容器(BeanFactory)，用于生成Bean；

初始化BeanDefinitionReader，配置读取器，能够读取注解、xml等用户定义的Bean配置，来生成BeanDefinition对象，放入一个Map中，为初始化Bean做准备；

注册BeanPostProcessor，是一批Bean后置处理器，用于对Bean进行增强；

2、开始填充容器：

循环遍历BeanDefinition集合，通过反射实例化Bean对象、进行Bean属性填充、初始化Bean等操作；（非懒加载的Bean）

# SpringBean生命周期

一、加载Bean的定义信息：BeanDefinitionReader.loadBeanDefinition()

BeanDefinitionReader接口定义了各种加载BeanDefinition的方式

主要方式有：Xml、注解等方法，通过这些方式，来定义一个Bean，将需要注册为Bean的对象，先封装为BeanDefinition对象，放入BeanDefinitionMap中；

AbstractBeanDefinition：定义了Bean的元数据信息

```java
AbstractBeanDefinition:
private Boolean lazyInit;        // 是否懒加载
private String[] dependsOn;      // 依赖
private volatile Object beanClass; // beanClass对象
private String initMethodName;   // 初始化方法
private String destroyMethodName;// 销毁方法
....
```

二、创建Bean：

根据已经创建的BeanDifinition，创建完整的Bean对象，总体两个步骤：

- 实例化Bean；

- 初始化Bean；

2.1、实例化对象：

通过BeanDefinition中的BeanClass属性，<mark>反射</mark>获取Bean的构造方法，进行实例化Bean；仅仅是为Bean开辟堆内存空间；Bean的各项属性、依赖，都为<mark>默认值</mark>；

2.2、填充属性、依赖注入：

通过三级缓存，反射调用set方法，填充Bean所有的依赖；

2.3、调用Aware接口方法：

SpringBean分为两类：

- 容器Bean

- 用户自定义Bean

Aware接口：用户可通过Aware接口获取容器Bean，来实现扩展功能；

当用户自定义的Bean，依赖容器Bean时(实现了SpringAware接口)，在此时由IOC处理完成装配工作，向用户Bean中set容器Bean。此时就是实现这个set过程，即调用Aware接口；

比如：

实现ApplicationContextAware接口，来获取容器上下文对象；

实现EnvironmentAware接口，来获取环境变量上下文对象；

2.4、初始化实例：

Bean初始化的过程，包括：

- 前置处理：BeanPostProcessor

- 初始化：从BeanDifinition中获取init方法，初始化Bean；

- 后置处理：BeanPostProcessor

至此Bean创建完成；

2.5、注册销毁：registerDisposableBean()

按照实现了DisposableBean()方法的行为，进行Bean的销毁；

三、销毁Bean：

3.1、销毁的前置处理：postProcessBeforeDestruction()

执行@PreDestroy注解方法；

3.2、销毁Bean：destroyBeans()

通过执行destroy()方法，进行销毁

3.3、自定义的销毁处理：invokeCustomDestroyMethod()

为Bean指定destroyMethod方法，在此处执行；

# Spring三级缓存

```java
/** 一级缓存 */
Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
/** 二级缓存 */
Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
/** 三级缓存 */
Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

- singletonObjects：一级缓存，存放完全初始化的Bean，可以用的Bean；

- earlySingletonObjects：二级缓存，存放代理完成、未填充属性的Bean；

- singletonFactories：三级缓存，不存放Bean，而是存放生成目标Bean的工厂对象，用于生成代理对象；

核心方法：DefaultSingletonBeanRegistry#getSingleton

核心流程：

1、从一级缓存拿；

2、没有则从二级缓存拿，拿到说明已经代理完成，未初始化完成，则处理依赖注入；

3、没有则通过三级缓存的Bean工厂进行创建，创建完成放入二级缓存；

## 为什么要二级缓存？

初始化过程中，总会有未完全初始化完成的bean，如果放在一起，可能存在未初始化完成的bean被提前使用；

- 如果Bean不需要AOP代理，二级缓存就够了

- 并且在没有AOP代理的情况下，二级缓存也可以处理循环依赖问题；

## 为什么要三级缓存？

三级缓存中不存放Bean，而是存放生成目标Bean的工厂对象，用于生成代理对象；

如果单例Bean使用了AOP生成了代理对象，使用三级缓存，防止<mark>代理对象创建完成前</mark>注入到其他Bean中；

## 处理循环依赖

[Spring 解决循环依赖为什么使用三级缓存，而不是二级缓存](https://www.cnblogs.com/grey-wolf/p/13034371.html)

循环依赖：A依赖B，B依赖A

解决循环依赖根本点：

1、set属性注入：构造器注入无法解决循环依赖，死锁；

2、多级缓存：将初始化未完成的Bean，可以提前暴露出来，先用；

- 如果没有代理，理论上两级缓存就够了；

- 通常AOP是开启的，spring无法确定bean是否被代理，就必须用到三级缓存

# SpringAOP

AOP：面向切面编程；

目的：在不侵入业务逻辑的情况下，对原有的方法进行增强；

主流的AOP实现有：SpringAOP、AspectJ；

---

SpringAOP：通过代理的方式，运行时创建代理对象，来实现对原有对象的增强；

- JDK动态代理；

- CGLib静态代理；

AspectJ：通过特殊的编译器，可在编译时、编译后、加载时，完成AOP增强；
