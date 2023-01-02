SpringCore：IOC + AOP

容器+生态

# Spring启动流程

1、服务构建



2、环境准备



3、容器创建



4、填充容器







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

- 前置处理：

- 初始化：从BeanDifinition中获取init方法，初始化Bean；

- 后置处理：

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



# 处理循环依赖

循环依赖：A依赖B，B依赖A

解决循环依赖根本点：set属性注入+三级缓存；



三级缓存：

```java

/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);


/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);


/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

```
