# @Resource、@Autowire、@Inject

- @Autowire由Spring提供，@Resource、@Inject是JSR250规范注解；

- @Autowire通过类型注入(byType)，@Resource、@Inject通过名称注入(byName)；

- @Autowire当类型相同，可以结合@Qualifier进行Bean的区分;

# @Mapper、@Repository

`@Repository`需要额外配置Dao扫描地址；

`@Mapper`=`@Repository`+`@MapperScan`

# @Component、@Bean

@Component和@Bean都是用来：向IOC容器中注入Bean；

@Component：用于标注类；将此类注入IOC容器；

@Bean：用于标注方法，将其返回值注入IOC容器；
