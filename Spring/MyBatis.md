# MyBatis

ORM框架；

我们不需要手动加载驱动，创建连接，创建`statement`操作；

只需要专注于SQL编写；

- 大量减少代码量；
- 与Spring集成很好；
- 性能好；

# 生命周期

### SqlSessionFactoryBuilder

用于构建者模式，通过SqlSessionFactoryBuilder来创建`SqlSessionFactory`，创建完成，就没用了；

构建者模式：通过一系列的参数，创建出要构造的对象；

`SqlSessionFactoryBuilder`就是读取配置，然后创建出我们需要的`SqlSessionFactory`（sqlSession工厂）

就是配置好连接池；

### SqlSessionFactory

`SqlSessionFactory`就是一个数据库连接池；

可以创建一个个的数据库连接`SqlSession`，进行数据库操作；

- 一个程序，一旦创建`SqlSessionFactory`就一直存在；

### SqlSession

每次要查询数据库，或者开启一个事务，就要先创建一个`SqlSession`，然后执行对应的操作，

操作完成，归还连接；

### Sql Mapper

由`SqlSession`创建，代表了一次数据库的业务请求；

MyBatis就是通过Mapper直接执行SQL；提供了多种操作方法；

# # / $

#{}是预编译处理，${}是字符串替换。

Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；

Mybatis在处理${}时，就是把${}替换成变量的值。

使用#{}可以有效的防止SQL注入，提高系统安全性。

- $：拼接字符串的方式传参；可以SQL注入；（静态赋值）

- #：（占位符）参数传值，不会SQL注入；（动态赋值）
  
  会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；

比如：我们通过`#`传入一个数组，执行批量操作，最终只会操作第一个元素；

```sql
==>  Preparing: DELETE FROM sys_post WHERE post_id IN (?)
==> Parameters: 
```

# 动态SQL执行原理

if、foreach、when、where

动态SQL是在XML映射文件的SQL中添加各种标签来实现；

在执行SQL的时候，通过逻辑判断，将对象的标签，转换成SQL的语法；

### if

```xml
<select id="findActiveBlogLike"
        resultType="Blog">
    SELECT * FROM BLOG
    WHERE
    <if test="state != null">
        state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
</select>
```

当所有条件失效， 会变成：

```sql
SELECT * FROM BLOG
WHERE
```

出现异常，查询失败；

### where标签

解决所有条件失效的问题；

```xml
<select id="findActiveBlogLike"
        resultType="Blog">
    SELECT * FROM BLOG
    <where>
        <if test="state != null">
            state = #{state}
        </if>
        <if test="title != null">
            AND title like #{title}
        </if>
        <if test="author != null and author.name != null">
            AND author_name like #{author.name}
        </if>
    </where>
</select>
```

### foreach标签批量查询

可以两种逻辑：

- 手动拆分，通过`$`传参**（效率高！比foreach高很多！！！！）**

比如：参数为`Long[ ] ids`；

我们需要手动拆分，拼接为字符串，再通过`$`传参，查询；

```sql
==>  Preparing: DELETE FROM sys_post WHERE post_id IN (10,12,13)
```

- 使用标签foreach标签：**（弊端：耗时！！占内存！！！数据量大内存溢出！！）**

collection：可以是array数组，也可以是list

```xml
<select id="findMenuName" resultType="java.lang.String" parameterType="java.util.List">
    select menu_name
    from menu
    where menu_id in
<foreach collection="array" item="valueList" open="(" close=")" separator=",">
    #{valueList}
</foreach>
</select>
```

# ResultMap

数据库字段和JavaBean字段，进行绑定映射；

## association嵌套查询

对象中，还有对象字段：用`association`；

表结构：

```sql
Student表：
    id,name,tid(关联Teacher表的id)
Teacher表：
    id,name
```

Pojo：

```java
public class Student {
    private int id;
    private String name;
    // 对象字段
    private Teacher teacher;
}
```

```xml
<select id="selectStuById" resultMap="StudentMap">
    select * from student
</select>
<resultMap type="com.pojo.Student" id="StudentMap">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <!--通过selectTeacherById,这条sql，使用tid，查询出Teacher对象-->
    <association property="teacher" column="tid" select="selectTeacherById" >            </association>
</resultMap>

<select id="selectTeacherById" resultType="com.pojo.Teacher">
    select * from teacher where id = #{id}
</select>
```

## collection嵌套结果

对象中，存在另一个对象的容器，返回多个映射：用`collection`；

```java
public class Teacher {
    private int id;
    private String name;
    // 集合
    private List<Student> students;
}
```

```xml
<select id="selectTeacher" resultMap="TeacherMap">
    select s.id sid, s.name sname, t.name tname,t.id tid
    from teacher t,student s
    where s.tid = t.id and t.id = #{tid}
</select>
<resultMap type="Teacher" id="TeacherMap">
    <id column="tid" property="id"/>
    <result column="tname" property="name"/>
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    <collection/>  
</resultMap>
```

# 两级缓存

MyBatis的两级缓存：（底层hashmap存储）

- 一级缓存（同一个SqlSession下，默认是开启的）
  
  开启了一级缓存后：在同一个Sqlsession下，连续使用同一个SQL进行查询，第一次是查询数据库，然后会将数据放入缓存，之后每次，都会直接从MyBatis缓存中直接取数据；
  
  - 只能是在SQL和参数都一致的情况下，完全是同一个SQL，才能使用到一级缓存；
  - 不同的SqlSession之间的一级缓存，是隔离的；

- 二级缓存（生命周期是：SqlSessionFactory下，需要手动开启）
  
  就说明二级缓存，可以跨SqlSession；
  
  也就是，二级缓存是一个所有SqlSession都可以共享访问的一片内存区域；
  
  首先要开启：
  
  ```xml
  <!--配置开启-->
  <settings>
      <setting name = "cacheEnabled" value = "true" />
  </settings
  ```
  
  如果要使用，在需要缓存的Mapper.xml中加上`<cache>`标签：来配置缓存：
  
  (1)如果只是在当前Mapper开启二级缓存，不配置，那只需要添加：
  
  ```xml
  <cache/>
  ```
  
  (2)多种配置二级缓存：
  
  ```xml
  <cache
         eviction = "FIFO"
         flushInterval = "60000"
         size="512"
         readOnly = "true"/>
  ```
  
  - eviction：缓存回收策略：
    - LRU：最近最少使用；（默认）
    - FIFO：先进先出；按顺序回收；
    - SOFT：
    - WEAK
  - flushInterval：缓存刷新间隔；
  - size：缓存存放元素个数；
  - readOnly：只读；

# XML与接口对应

Mapper接口———XML

Mapper接口的全限定名，对应XML映射文件的`namespace`；

Mapper接口的方法名，对应XML文件的Statement语句的`id`；

接口方法参数，对应SQL语句的参数；

### 工作原理

Mapper接口工作原理是JDK动态代理；

1、MyBatis会使用JDK动态代理为Mapper接口生成代理对象；

2、代理对象 拦截接口方法，并执行XML映射文件的SQL；

3、再返回SQL执行结果；

# MyBatis分页原理

MyBatis是使用`RowBounds`对象进行分页，是针对结果集的**内存分页**；

原理是：通过插件接口完成的；

插件的拦截方法里面是 等待执行的SQL，插件会对SQL进行重写，改写成分页的逻辑；返回分页的结果；

# SQL执行结果如何封装为对象

（1）ResultMap标签，字段与对象属性一一对应

（2）SQL的别名，别名就是对象属性名；

# 获取主键ID

直接通过insert插入，返回int类型的行数；

即可获得主键ID；

# MyBatisPlus

## 配置SQL日志

```yaml
# mp日志：显示详细日志
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 时间字段自动填充

### 数据库级别

```sql
CREATE TABLE `mytest` (
    `text` varchar(255) DEFAULT '' COMMENT '内容',
    `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

添加了这样的策略，在 插入数据 / 更新数据时，自动更新时间；

### 代码级别

1、对应字段加上注解：

`@TableField(fill = FieldFill.INSERT_UPDATE)`

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    @TableId(type = IdType.AUTO)
    Long id;
    @TableField(fill = FieldFill.INSERT)
    Date createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    Date updateTime;
}
```

2、实现`MetaObjectHandler`接口：

- 在插入记录的时候，会调用`insertFill`；
  
  执行`"createTime"`，`"updateTime"`两个字段的插入；

- 在更新数据的时候，会调用`updateFill；`
  
  执行`"updateTime"`两个字段的插入；

就不需要在业务逻辑中，额外的维护时间字段！

```java
@Component
@Slf4j
public class TimeHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject,"createTime",LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject,"updateTime",LocalDateTime.class, LocalDateTime.now());
    }
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject,"updateTime",LocalDateTime.class, LocalDateTime.now());
    }
}
```

## 乐观锁插件

乐观锁：加上version字段；

在插入的时候，version字段作为比较条件；

MP为乐观锁提供了插件：

1、 实体类对应的version字段，加上注解

```java
@Version    //识别乐观锁
Integer version;
```

2、添加MP配置

```java
@EnableTransactionManagement
@MapperScan("com.mp.mapper")
@Configuration
public class MyBatisPlusConfig {
    // 注册乐观锁插件
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor(){
        return new OptimisticLockerInterceptor();
    }
}
```

3、测试

我们不需要手动写sql维护version字段

```java
// 测试乐观锁
@Test
void testVersion(){
    // 查询
    User user = userMapper.selectById(1L);
    // 更新
    user.setName("kkkk");
    user.setEmail("kkkk@baomidou.com");
    userMapper.updateById(user);
}
```

4、查看执行日志：

自动加上了version字段！

![1587765864050](C:/Users/whr/Desktop/notes/面试/image/1587765864050.png)

## 树形查询

```xml
<resultMap id="BaseResultMap" type="com.gmall.pms.entity.ProductCategory">
    <id column="id" property="id"/>
    <result column="parent_id" property="parentId"/>
    <result column="name" property="name"/>
</resultMap>
<sql id="Base_Column_List">
    id, parent_id, name
</sql>

<resultMap id="ExtendResultMap"
           type="com.gmall.vo.product.PmsProductCategoryWithChildrenItem"
           extends="BaseResultMap">
    <collection property="children" select="listCatelogWithChilder" column="id"></collection>
</resultMap>

<select id="listCatelogWithChilder" resultMap="ExtendResultMap">
    select * from pms_product_category where parent_id = #{i}
</select>
```

这里引入了一个新的`resultMap`

返回类型是：重新封装的List结果集：`PmsProductCategoryWithChildrenItem`

```java
public class PmsProductCategoryWithChildrenItem extends ProductCategory implements Serializable {
    private List<ProductCategory> children;
}
```

重点是

```xml
<collection property="children" select="listCatelogWithChilder" column="id"></collection>
```

当select查询语句执行结束，返回`resultMap`的时候，会执行`collection`

`property="children"`：新增一个集合字段；(就是结果json中的children字段)

**collection标签：将返回结果的每一条记录的`id`作为参数，再次执行`select语句`！！！！**

## 分页查询

配置：

```java
// 分页插件,中间都可以不配置，直接return也可以用
@Bean
public PaginationInterceptor paginationInterceptor() {
    PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
    // 设置最大单页限制数量，默认 500 条，-1 不受限制
    paginationInterceptor.setLimit(20);
    // 开启 count 的 join 优化,只针对部分 left join
    paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
    return paginationInterceptor;
}
```

代码：

```java
// 分页查询
// new Page<>(1, 5);前端传值
@Test
void testPage(){
    // 第2页，3条记录
    Page<User> page = new Page<>(2, 3);
    userMapper.selectPage(page, null);
    page.getRecords().forEach(System.out::println);
}
```

## 逻辑删除

物理删除：从数据库中直接移除！

逻辑删除：数据库中并没有被删除！而是通过一个变量，让他失效；让查询逻辑，查询不到；（delete字段）

场景：

- 管理员可以查看被删除的操作，或者记录等；

首先：

1. 要有一个逻辑删除的字段：`delete`
   
   加上MP提供的注解

```java
@TableLogic //逻辑删除字段
Integer delete;
```

2. 配置逻辑删除的值

```yaml
# 逻辑删除
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1         # 删除了：1
      logic-not-delete-value: 0        # 没删除：0
```

3. 测试
   
   ```java
   @Test
   void testDeleteLogic(){
       userMapper.deleteById(1L);
   }
   ```
   
   走了update操作

![1587768795999](C:/Users/whr/Desktop/notes/面试/image/1587768795999.png)

4. 查询的时候，自动拼接deleted字段，查询不到数据；
   
   ```java
   userMapper.selectById(1L);
   ```

## 代码生成器

依赖：

```xml
<!-- 代码生成器 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.3.1</version>
</dependency>
<!-- 第三方模板引擎 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<!-- swagger -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```

主程序：

三步走

最后启动

```java
public class CodeGenerator {
    public static void main(String[] args) {
        /**
         * 代码生成器 对象
         */
        AutoGenerator autoGenerator = new AutoGenerator();

        /**
         * 1. 全局配置
         */
        GlobalConfig globalConfig = new GlobalConfig();
        // 项目路径
        String property = System.getProperty("user.dir");
        globalConfig.setOutputDir(property+"/src/main/java");
        globalConfig.setAuthor("whr");
        globalConfig.setOpen(false);
        globalConfig.setFileOverride(false);// 是否覆盖
        globalConfig.setServiceName("%sService"); // 去Service的I前缀
        globalConfig.setIdType(IdType.ID_WORKER);
        globalConfig.setDateType(DateType.ONLY_DATE);
        globalConfig.setSwagger2(true);

        /**
         * 2. 设置数据源
         */
        DataSourceConfig dataSourceConfig = new DataSourceConfig();
        dataSourceConfig.setUrl("jdbc:mysql://localhost:3306/mybatis_plus?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8");
        dataSourceConfig.setUsername("root");
        dataSourceConfig.setPassword("root");
        dataSourceConfig.setDriverName("com.mysql.cj.jdbc.Driver");
        dataSourceConfig.setDbType(DbType.MYSQL);

        /**
         * 生成的：包路径配置
         */
        PackageConfig packageConfig = new PackageConfig();
        packageConfig.setModuleName("generator");
        packageConfig.setParent("com.mp");
        packageConfig.setEntity("entity");
        packageConfig.setMapper("mapper");
        packageConfig.setService("service");
        packageConfig.setController("controller");
        /**
         * 3. 配置策略
         */
        StrategyConfig strategyConfig = new StrategyConfig();
        // 需要映射的表名,不写表明，默认全部表
        //strategyConfig.setInclude("user");
        // 表前缀不生成
        //strategyConfig.setTablePrefix("user_");
        // 驼峰命名策略
        strategyConfig.setNaming(NamingStrategy.underline_to_camel);
        strategyConfig.setColumnNaming(NamingStrategy.underline_to_camel);
        strategyConfig.setEntityLombokModel(true); // entity自动使用lombok
        strategyConfig.setLogicDeleteFieldName("deleted");  //逻辑删除字段 注解
        strategyConfig.setVersionFieldName("version");  // 乐观锁
        strategyConfig.setRestControllerStyle(true);
        // 自动填充的注解配置
        TableFill createTime = new TableFill("create_time", FieldFill.INSERT);
        TableFill updateTime = new TableFill("update_time", FieldFill.INSERT_UPDATE);
        ArrayList<TableFill> list = new ArrayList<>();
        list.add(createTime);
        list.add(updateTime);
        strategyConfig.setTableFillList(list);

        /**
         * 最后
         * 代码生成器 对象
         * 导入所有配置
         */
        autoGenerator.setGlobalConfig(globalConfig);
        autoGenerator.setPackageInfo(packageConfig);
        autoGenerator.setDataSource(dataSourceConfig);
        autoGenerator.setStrategy(strategyConfig);
        // 额外模板引擎
        autoGenerator.setTemplateEngine(new FreemarkerTemplateEngine());
        // 执行
        autoGenerator.execute();
    }
}
```
