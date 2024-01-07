# JDBC

JDBC 的全称是 Java Database Connectivity，是一套面向关系型数据库的规范。每个数据库生厂商都提供了基于 JDBC 规范实现的 JDBC 驱动。开发者只需要面向 JDBC 接口编程，就能在很大程度上规避掉由数据库实现的差异所带来的问题。

[TOC]

## 配置数据源

基础的 JDBC 查询操作示例片段：

~~~java
Class.forName("org.h2.Driver");
try (
	Connection connection = DriverManager.getConnection("jdbc:h2:mem:test_db");
    Statement statement = connection.createStatement();
    ResultSet resultSet = statement.executeQuery("Select x from system_range(1, 20)")
) {
    while (resultSet.next()) {
        log.info(resultSet.getInt(1));
    }
} catch(Exception e) {
	log.error(e);
}
~~~

在真实的生产环境中，并不推荐像这样直接创建一条连接，因为创建和销毁连接的成本是昂贵的。建议通过数据库连接池来管理连接，它的主要功能有：

- 根据配置，事先创建一定数量的连接放在连接池中，以便在需要的时候直接返回现成的连接；
- 维护连接池中的连接，根据配置，清理已存在的连接。



常用的数据库连接池都实现了 `DataSource` 接口，通过其中的 `getConnection()` 方法即可获得数据库的一个连接。常见的连接池有：HikariCP 、 Druid、 DBCP2 和 C3P0。

~~~java
class DatasourceDempApplicationTests {
    @Autowired
    private ApplicationContext applicationContext;
    
    @Test
    void testDataSource() throws SQLException {
        DataSource dataSource = applicationContext.getBean("dataSource", DataSource.class);
        Connection connection = dataSource.getConnection();
        connection.close();
  
    }
}
~~~

要连接数据库，首先需要在 pom.xml 的 `<dependencies/>` 中加入 MySQL 的 JDBC 驱动：

~~~xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
~~~

随后，我们在 `application.properties` 中添加与数据源相关的配置：

~~~
spring.datasource.url=jdbc:mysql://localhost/binary-tea?useUnicode=true&characterEncoding=utf8

spring.datasource.username=binary-tea
spring.datasource.password=binary-tea
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=10
~~~





Spring Boot 的自动配置机制在检测到 CLASSPATH 中存在 H2 数据库的依赖，且没有配置其他数据库时，就会提供一个内嵌的、基于内存的数据库`H2`。

### 配置

Spring Boot 为连接池提供了一个 `DataSourceProperties` Bean，它绑定了`application.properties`文件中的`spring.datasource` 配置，为连接池提供必要的参数。

~~~java
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {
    
}
~~~

部分常用的 `spring.datasource` 配置项：

| 配置项                                   | 默认值                      | 说明                    |
| :--------------------------------------- | :-------------------------- | :---------------------- |
| `spring.datasource.url`                  |                             | 数据库的 JDBC URL       |
| `spring.datasource.username`             |                             | 连接数据库的用户名      |
| `spring.datasource.password`             |                             | 连接数据库的密码        |
| `spring.datasource.name`                 | 使用内嵌数据库时为 `testdb` | 数据源的名称            |
| `spring.datasource.jndi-name`            |                             | 获取数据源的 JNDI 名称  |
| `spring.datasource.type`                 | 根据 CLASSPATH 自动探测     | 连接池实现的全限定类名  |
| `spring.datasource.driver-class-name`    | 根据 URL 自动探测           | JDBC 驱动类的全限定类名 |
| `spring.datasource.generate-unique-name` | `true`                      | 是否随机生成数据源名称  |

### 数据源自动配置原理

数据源的自动配置是通过`DataSourceAutoConfiguration`这个配置类来实现的：

~~~java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {
    
}
~~~

可以通过@ConditionalOnClass得知，必须加载DataSource、EmbeddedDatabaseType这两个类才能激活这个配置类。其中，`DataSource`是`javax.sql.DataSource`。这个配置类还导入 `DataSourcePoolMetadataProvidersConfiguration` 和 `DataSourceInitializationConfiguration` 两个配置类，前者配置连接池元数据提供者，后者进行数据源初始化配置。



 `DataSourceAutoConfiguration`配置类中有两个内部类，它们都是配置类——内嵌数据库配置类 `EmbeddedDatabaseConfiguration` 和连接池数据源配置类 `PooledDataSourceConfiguration`。

~~~java
@Configuration(proxyBeanMethods = false)
@Conditional(PooledDataSourceCondition.class)
@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })

在这里引入了很多类型数据源的自动配置类
@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
        DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
        DataSourceJmxConfiguration.class })
protected static class PooledDataSourceConfiguration {

}
~~~

`PooledDataSourceConfiguration` 会直接导入 `DataSourceConfiguration` 中关于 HikariCP、DBCP2、Tomcat 和通用数据源的配置类。我们以 HikariCP 的自动配置类 `DataSourceConfiguration.Hikari` 为例

~~~java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(HikariDataSource.class)
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.zaxxer.hikari.
HikariDataSource", matchIfMissing = true)
static class Hikari {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    HikariDataSource dataSource(DataSourceProperties properties) {
        HikariDataSource dataSource = createDataSource(properties, HikariDataSource.class);
        if (StringUtils.hasText(properties.getName())) {
            dataSource.setPoolName(properties.getName());
        }
        return dataSource;
    }
}
~~~

根据`@ConditionalXXX`判断是否返回Hikari这个数据源（例如，没有其他的DataSource类，这一般是我们自己配置的DataSource Bean）。如果满足条件，那么通过`DataSourceProperties`从配置文件中加载配置。这个对象就是 Spring 上下文中的 `DataSource` Bean 了。

### 内嵌数据库

`EmbeddedDatabaseType` 定义了 Spring Boot 内置支持的三种数据库，即 HSQL、H2 和 Derby，`EmbeddedDatabaseConnection` 则分别定义了三者的 JDBC 驱动类和用来创建内存数据库的 JDBC URL。在自动配置时，`DataSourceAutoConfiguration` 会根据 CLASSPATH 来判断是否存在对应的驱动类，如果存在，则它会调用`EmbeddedDataSourceConfiguration.dataSource()` 方法会创建 `DataSource` 对象。

创建完内嵌数据库的 `DataSource` 后，Spring Boot 还会为我们进行数据库的初始化工作，我们可以在这个过程中建表，并导入初始的数据。初始化动作是由 `DataSourceInitializer` 类来实现的，它会根据 `spring.sql.init.schema-locations` 和 `spring.sql.init.data-locations` 这两个属性来初始化数据库中的表和数据，默认通过读取 CLASSPATH 中的 schema.sql 和 data.sql 文件来进行初始化。

| 当前配置项                          | 旧配置项                                | 默认值     | 说明                                                         |
| :---------------------------------- | :-------------------------------------- | :--------- | :----------------------------------------------------------- |
| `spring.sql.init.mode`              | `spring.datasource.initialization-mode` | `embedded` | 何时使用 DDL 和 DML**6**脚本初始化数据源，可选值为 `embedded`、`always` 和 `never` |
| `spring.sql.init.platform`          | `spring.datasource.platform`            | `all`      | 脚本对应的平台，用来拼接最终的 SQL 脚本文件名，例如，schema-{platform}.sql |
| `spring.sql.init.separator`         | `spring.datasource.separator`           | `;`        | 脚本中的语句分隔符                                           |
| `spring.sql.init.encoding`          | `spring.datasource.sql-script-encoding` |            | SQL 脚本的编码                                               |
| `spring.sql.init.continue-on-error` | `spring.datasource.continue-on-error`   | `false`    | 初始化过程中报错是否停止初始化                               |
| `spring.sql.init.schema-locations`  | `spring.datasource.schema`              |            | 初始化用的 DDL 脚本，默认会用 schema.sql                     |
| `spring.sql.init.data-locations`    | `spring.datasource.data`                |            | 初始化用的 DML 脚本，默认会用 data.sql                       |

### HikariCP

Spring Boot 2.*x* 项目的默认数据库连接池是 HikariCP，它有不少配置项，用于调整连接池的大小和各种超时设置。Spring Boot 可以让我们在 `application.properties` 中设置这些配置项，具体参数如下：

| 配置项              | Spring Boot 配置属性                          | 配置含义                             |
| :------------------ | :-------------------------------------------- | :----------------------------------- |
| `maximumPoolSize`   | `spring.datasource.hikari.maximum-pool-size`  | 连接池中的最大连接数                 |
| `minimumIdle`       | `spring.datasource.hikari.minimum-idle`       | 连接池中保持的最小空闲连接数         |
| `connectionTimeout` | `spring.datasource.hikari.connection-timeout` | 建立连接时的超时时间，单位为秒       |
| `idleTimeout`       | `spring.datasource.hikari.idle-timeout`       | 连接清理前的空闲时间，单位为秒       |
| `maxLifetime`       | `spring.datasource.hikari.max-lifetime`       | 连接池中连接的最大存活时间，单位为秒 |

> HikariCP 官方一直将“快”作为自己的亮点。从官方性能测试的结果来看，HikariCP 的性能数倍于 DBCP2、C3P0 和 Tomcat 连接池。
>
> 官方有一篇“Down the Rabbit Hole”的文章，简单说明了 HikariCP 性能出众的原因：
>
> - 通过字节码进行加速，`JavassistProxyFactory` 中使用 `Javassist` 直接生成了大量字节码塞到了 `ProxyFactory` 中，同时还对字节码进行了精确地优化；
> - 使用 `FastList` 代替了 JDK 内置的 `ArrayList`；
> - 从 .NET 中借鉴了无锁集合 `ConcurrentBag`。

### Druid

Druid是阿里巴巴开源的、面向监控的数据库连接池，它提供了很丰富的功能：

- 针对主流数据库的适配，包含驱动、连接检查、异常等；
- 内置 SQL 注入防火墙功能；
- 内置数据库密码非对称加密功能；
- 内置针对数据库异常的 `ExceptionSorter`，可对不同的异常进行区别对待；
- 内置丰富的日志信息；
- ...

依赖如下：

~~~xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.8</version>
</dependency>
~~~

Druid 的常用配置项如下：

| 配置项          | Spring Boot 配置属性                      | 配置含义                                                 |
| :-------------- | :---------------------------------------- | :------------------------------------------------------- |
| `initialSize`   | `spring.datasource.druid.initial-size`    | 初始化连接池时建立的连接数                               |
| `maxActive`     | `spring.datasource.druid.max-active`      | 连接池中的最大连接数                                     |
| `minIdle`       | `spring.datasource.druid.min-idle`        | 连接池中保持的最小空闲连接数                             |
| `maxWait`       | `spring.datasource.druid.max-wait`        | 获取连接的最大等待时间，单位为毫秒                       |
| `testOnBorrow`  | `spring.datasource.druid.test-on-borrow`  | 获取连接时检查连接，会影响性能                           |
| `testOnReturn`  | `spring.datasource.druid.test-on-return`  | 归还连接时检查连接，会影响性能                           |
| `testWhileIdle` | `spring.datasource.druid.test-while-idle` | 检查空闲的连接，具体的检查发生在获取时，对性能几乎无影响 |
| `filters`       | `spring.datasource.druid.filters`         | 要配置的插件过滤器列表                                   |

### 配置一个MySQL的数据源

首先需要在 pom.xml 的 `<dependencies/>` 中加入 MySQL 的 JDBC 驱动：

~~~xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
</dependency>
~~~

随后，我们在 `application.properties` 中添加与数据源相关的配置：

~~~xml
spring.datasource.url=jdbc:mysql://localhost/binary-tea?useUnicode=true&characterEncoding=utf8
spring.datasource.username=binary-tea
spring.datasource.password=binary-tea
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=10
~~~

测试：

~~~java
@SpringBootTest
class DatasourceDemoApplicationTests {
    @Autowired
    private ApplicationContext applicationContext;
    @Value("${spring.datasource.url}")
    private String jdbcUrl;

    @Test
    void testDataSource() throws SQLException {
        assertTrue(applicationContext.containsBean("dataSource"));
        DataSource dataSource = applicationContext.getBean("dataSource", DataSource.class);
        assertTrue(dataSource instanceof HikariDataSource);

        HikariDataSource hikari = (HikariDataSource) dataSource;
        assertEquals(20, hikari.getMaximumPoolSize());
        assertEquals(10, hikari.getMinimumIdle());
        assertEquals("com.mysql.cj.jdbc.Driver", hikari.getDriverClassName());
        assertEquals(jdbcUrl, hikari.getJdbcUrl());

        Connection connection = hikari.getConnection();
        assertNotNull(connection);
        connection.close();
    }
}
~~~



我们还可以自己创建`DataSource` Bean，而不需要Spring Boot为我们自动配置了：

~~~java
@Bean
@ConfigurationProperties("spring.datasource.hikari")
public DataSource dataSource(DataSourceProperties properties) {
    HikariDataSource dataSource = new HikariDataSource();
    dataSource.setJdbcUrl(properties.getUrl());
    dataSource.setUsername(properties.getUsername());
    dataSource.setPassword(properties.getPassword());
    return dataSource;
}
~~~



## 使用JDBC操作数据库

在建立了数据源之后，我们可以通过JDBC接口来操纵数据。JDBC操作的具体流程如下：

1. 获取 `Connection` 连接
2. 通过 `Connection` 创建 `Statement` 或者 `PreparedStatement`
3. 执行具体的 SQL 操作
4. 关闭 `Statement` 或者 `PreparedStatement`
5. 关闭 `Connection`

实际上，只有第三步是和我们业务逻辑相关的。SpringFramework为我们提供了`JdbcTemplate` 和 `NamedParameterJdbcTemplate` 两个模板类，简化了上述操作，使得我们只需关心第三步的实现。



下面我们以一个例子来介绍如何使用`JdbcTemplate`：

首先添加JDBC依赖、H2数据库依赖以及Lombok

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
~~~

接着创建一个Model类，与数据库中的Table对应

~~~java
@Getter
@Setter
@ToString
@Builder
public class MenuItem {
    private Long id;
    private String name;
    private String size;
    private BigDecimal price;
    private Date createTime;
    private Date updateTime;
}
~~~

其中，@Builder为该类生成建造者模式的模板代码。

~~~java
MenuItem.builder().id(1).name("GaoRuofan").build();
~~~

然后，创建一个`Repository`类

~~~java
@Repository
public class MenuRepository {
    private JdbcTemplate jdbcTemplate;

    public MenuRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    // 几个查询方法待说明
}
~~~

Spring 容器发现这个Bean是Repository的话，那么会自动调用构造方法传入所需的`JdbcTemplate` 实例。这和自己定义一个@Autowired `JdbcTemplate` 对象的效果是一样的

要查询的 SQL 只返回一个值，那么可以使用 `queryForObject(String sql, Class<T> requiredType)` 方法，例如：

~~~java
public long countMenuItems() {
    return jdbcTemplate.queryForObject("select count(*) from t_menu", Long.class);
}
~~~

返回的结果有多个字段，可以用 `queryForMap()` 将它们都放到一个 `Map<String, Object>` 中，或者可以通过 `RowMapper` 将字段映射到某个对象上。下面介绍`RowMapper`的方法

~~~java
public MenuItem queryForItem(Long id) {
    return jdbcTemplate.queryForObject("select * from t_menu where id = ?", rowMapper());
}

private RowMapper<MenuItem> rowMapper() {
    return (resultSet, rowNum) -> {
        return MenuItem.builder()
                .id(resultSet.getLong("id"))
                .name(resultSet.getString("name"))
                .size(resultSet.getString("size"))
                .price(BigDecimal.valueOf(resultSet.getLong("price") / 100.0d))
                .createTime(new Date(resultSet.getDate("create_time").getTime()))
                .updateTime(new Date(resultSet.getDate("update_time").getTime()))
                .build();
    }
}
~~~



一个查询操作如果要求返回多条记录，可以使用 `query(String sql, RowMapper<T> rowMapper)`，例如：

~~~java
public List<MenuItem> queryAllItems() {
    return jdbcTemplate.query("select * from t_menu", rowMapper());
}
~~~

包含表结构定义的 schema.sql：

~~~sql
drop table t_menu if exists;

create table t_menu (
    id bigint auto_increment,
    name varchar(128),
    size varchar(16),
    price bigint,
    create_time timestamp,
    update_time timestamp,
    primary key (id)
);
~~~

包含初始数据的 data.sql：

~~~sql
insert into t_menu (name, size, price, create_time, update_time) values ('Java咖啡', '中杯', 1000, now(), now());
insert into t_menu (name, size, price, create_time, update_time) values ('Java咖啡', '大杯', 1500, now(), now());
~~~

除此之外，`JdbcTemplate` 的 `update()` 方法可以用来执行 `INSERT`、`UPDATE` 和 `DELETE` 语句。

~~~java
public static final String INSERT_SQL =
        "insert into t_menu (name, size, price, create_time, update_time) values (?, ?, ?, now(), now())";

public int insertItem(MenuItem item) {
    return jdbcTemplate.update(INSERT_SQL, item.getName(),
            item.getSize(), item.getPrice().multiply(BigDecimal.valueOf(100)).longValue());
}
~~~

其中，SQL 语句后的参数顺序对应了 SQL 中 `?` 占位符的顺序。

在很多时候，数据的 ID 是自增长的主键，如果我们希望在插入记录后能取得生成的 ID，这时可以使用 `KeyHolder` 类来持有生成的键。代码如下：

~~~java
public int insertItemAndFillId(MenuItem item) {
    KeyHolder keyHolder = new GeneratedKeyHolder();
    
    int affected = jdbcTemplate.update(con -> {
        PreparedStatement preparedStatement =
                con.prepareStatement(INSERT_SQL, PreparedStatement.RETURN_GENERATED_KEYS);
        // 也可以用PreparedStatement preparedStatement =
        //            con.prepareStatement(INSERT_SQL, new String[] { "id" });

        preparedStatement.setString(1, item.getName());
        preparedStatement.setString(2, item.getSize());
        preparedStatement.setLong(3, item.getPrice().multiply(BigDecimal.valueOf(100)).longValue());
        return preparedStatement;
    }, keyHolder);
    
    if (affected == 1) {
        item.setId(keyHolder.getKey().longValue());
    }
    
    return affected;
}
~~~

如果SQL 中用到了 `?`的数量一多，就容易在传参时搞错位置。Spring Framework 为我们提供了一个 `NamedParameterJdbcTemplate` 类来避免这个问题。通过它，我们可以为 SQL 中的参数设定名称，然后根据名称进行赋值。

~~~java
public int insertItem(MenuItem item) {
    String sql = 
        "insert into t_menu (name, size, price, create_time, update_time) values (:name, :size, :price, now(), now())";
    MapSqlParameterSource sqlParameterSource = new MapSqlParameterSource();
    
    sqlParameterSource.addValue("name", item.getName());
    sqlParameterSource.addValue("size", item.getSize());
    sqlParameterSource.addValue("price", item.getPrice().multiply(BigDecimal.valueOf(100)).longValue());
    return namedParameterJdbcTemplate.update(sql, sqlParameterSource);
}
~~~

在数据处理时，我们经常会遇到需要插入或更新一大批数据的情况（批操作）。大多数 JDBC 驱动针对批量调用相同 `PreparedStatement` 的情况都做了特殊优化，同时 Spring Framework 中也提供了 `batchUpdate()` 方法来支持为批处理操作

~~~java

public int insertItmes(List<MenuItem> items) {
    int[] count = jdbcTemplate.batchUpdate(
        INSERT_SQL, 		// SQL Insert语句
        new BatchPreparedStatementSetter() {
            
            // 为每条语句的占位符指定参数
        	@Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                MenuItem item = items.get(i);
            	ps.setString(1, item.getName());
            	ps.setString(2, item.getSize());
            	ps.setLong(3, item.getPrice().multiply(BigDecimal.valueOf(100)).longValue());
            }
            
            // 有几条语句
            @Override
            public int getBatchSize() {
                return items.size();
            }
    	});
    return Arrays.stream(count).sum();
}
~~~

`batchUpdate()` 方法还有其他几种形式， `batchUpdate(String sql, List<Object[]> batchArgs)`：

~~~java
public int insertItems(List<MenuItem> items) {
    List<Object[]> batchArgs = items
        .stream()
        .map(item -> new Object[] {
            item.getName(),
            item.getSize(),
            item.getPrice().multiply(BigDecimal.valueOf(100)).longValue()
        })
        .collect(Collectors.toList());
    
    int[] count = jdbcTemplate.batchUpdate(INSERT_SQL, batchArgs);
    return Arrays.stream(count).sum();
}
~~~



下面我们看一下 `JdbcTemplate` 的自动配置类：

~~~java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ 
    DataSource.class, 
    JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ 
    DatabaseInitializationDependencyConfigurer.class, 
    JdbcTemplateConfiguration.class,
    NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {
    
}
~~~

`JdbcTemplateConfiguration` 会在没有配置 `JdbcOperations`  Bean 对象时生效，它的作用是提供一个 `JdbcTemplate` Bean，这个 Bean 会自动注入现有的 `DataSource`，并将 `spring.jdbc.template.*` 的配置项内容设置进来

| 配置项                               | 默认值 | 说明                                                         |
| :----------------------------------- | :----- | :----------------------------------------------------------- |
| `spring.jdbc.template.fetch-size`    | `-1`   | 每次从数据库获取的记录条数，`-1` 表示使用驱动的默认值        |
| `spring.jdbc.template.max-rows`      | `-1`   | 一次查询可获取的最大记录条数，`-1` 表示使用驱动的默认值      |
| `spring.jdbc.template.query-timeout` |        | 查询的超时时间，没有配置的话使用 JDBC 驱动的默认值，如果没有加时间单位，默认为秒 |

## 事务管理

**事务（Transaction）**在不同的语境下有着不同的含义

- 在数据库中，事务就是其原子性（Atomic）——操作要么全都执行，要么都不执行。
- 在 Java EE 环境中，事务可以是使用 JTA（Java Transaction API）这样的全局事务，也可以是基于 JDBC 连接的本地事务。
- 分布式事务

为了消除代码对不同事务对的依赖，Spring Framework 对事务管理做了一层抽象。这个抽象的核心是事务管理器，即 `TransactionManager`，它是一个标记接口。而`PlatformTransactionManager`接口继承了 `TransactionManager`接口，定义了获取事务、提交事务和回滚事务的方法：

~~~java
public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition)throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
~~~

`DataSourceTransactionManager`、`JtaTransactionManager` 和 `HibernateTransactionManager` 这些底层事务管理器都实现了上述接口。

在Spring中，事务有两种实现方式：

- **编程式事务管理**： 编程式事务管理使用`TransactionTemplate`，或者直接使用底层的`PlatformTransactionManager`。
- **声明式事务管理**： 建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。 通过`@Transactional`就可以进行事务操作



如何开启对于声明式事务的支持

- 在配置类上添加 `@EnableTransactionManagement` 注解就能开启事务支持

- 或者在XML配置文件中通过`<tx:annotation-driven/>`来启用注解支持

  ~~~xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:tx="http://www.springframework.org/schema/tx"
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/tx
          https://www.springframework.org/schema/tx/spring-tx.xsd">
  
      <!-- 开启事务注解支持，可以明确设置一个TransactionManager -->
      <tx:annotation-driven transaction-manager="txManager"/>
  </beans>
  ~~~

  



`@EnableTransactionManagement` 和 `<tx:annotation-driven/>` 拥有一些共同的配置：

| 配置项                                           | 默认值                      | 含义                                                         |
| :----------------------------------------------- | :-------------------------- | :----------------------------------------------------------- |
| `mode`                                           | `proxy`                     | 声明式事务 AOP 的拦截方式，默认 `proxy` 是代理方式，也可以改为 `aspectj` |
| `order`                                          | `Ordered.LOWEST_PRECEDENCE` | 声明式事务 AOP 拦截的顺序，值越小，优先级越高                |
| `proxyTargetClass` / `proxy-target-class`（XML） | `false`                     | 是否使用 CGLIB 的方式拦截类 **14**                           |



事务的属性用`TransactionDefinition` 接口来描述：

- 传播性：事务传播性分为 7 个级别：

  | 传播性                         | 值   | 描述                                             |
  | :----------------------------- | :--- | :----------------------------------------------- |
  | `PROPAGATION_REQUIRED`（默认） | `0`  | 当前有事务就用当前事务，没有事务就新启动一个事务 |
  | `PROPAGATION_SUPPORTS`         | `1`  | 事务不是必需的，可以有事务，也可以没有           |
  | `PROPAGATION_MANDATORY`        | `2`  | 一定要存在一个事务，不然就报错                   |
  | `PROPAGATION_REQUIRES_NEW`     | `3`  | 新启动一个事务，如果当前存在一个事务则将其挂起   |
  | `PROPAGATION_NOT_SUPPORTED`    | `4`  | 不支持事务，以非事务的方式运行                   |
  | `PROPAGATION_NEVER`            | `5`  | 不支持事务，如果当前存在一个事务则抛异常         |
  | `PROPAGATION_NESTED`           | `6`  | 如果当前存在一个事务，则在该事务内再启动一个事务 |

- 隔离级别：数据库的事务有 4 种隔离级别

  | 隔离性                       | 值   | 脏读   | 不可重复读 | 幻读   |
  | :--------------------------- | :--- | :----- | :--------- | :----- |
  | `ISOLATION_READ_UNCOMMITTED` | `1`  | 存在   | 存在       | 存在   |
  | `ISOLATION_READ_COMMITTED`   | `2`  | 不存在 | 存在       | 存在   |
  | `ISOLATION_REPEATABLE_READ`  | `3`  | 不存在 | 不存在     | 存在   |
  | `ISOLATION_SERIALIZABLE`     | `4`  | 不存在 | 不存在     | 不存在 |

  `TransactionDefinition` 中的默认隔离级别设置为 `-1`，使用底层数据源的配置，比如，MySQL 默认的隔离级别是 `REPEATABLE_READ`，Oracle 默认的隔离级别则是 `READ_COMMITTED`。

  - **脏读**：事务 A 修改了记录 1 的值但未提交事务，这时事务 B 读取了记录 1 尚未提交的值，但后来事务 A 回滚了，事务 B 读到的值并不会存在于数据库中，这就是脏读。
  - **不可重复读**：事务 A 会读取记录 1 两次，在两次读取之间，事务 B 修改了记录 1 的值并提交了，这时事务 A 第一次与第二次读取到的记录 1 的内容就不一样了，这就是不可重复读。
  - **幻读**：事务 A 以某种条件操作了数据表中的一批数据，这时事务 B 往表中插入并提交了 1 条记录，正好也符合事务 A 的操作条件，当事务 A 再次以同样的条件操作这批数据时，就会发现操作的数据集变了，这就是幻读。以 `SELECT count(*)` 为例，发生幻读时，如果两次以同样的条件来执行，结果值就会不同。

- 超时时间

- 是否只读



Spring Boot 为我们提供了事务的自动配置类，其中主要的是`DataSourceTransactionManagerAutoConfiguration` 和 `TransactionAutoConfiguration`。

~~~java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ 
    JdbcTemplate.class, 
    TransactionManager.class })
@AutoConfigureOrder(Ordered.LOWEST_PRECEDENCE)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceTransactionManagerAutoConfiguration {
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnSingleCandidate(DataSource.class)
    static class DataSourceTransactionManagerConfiguration {
        @Bean
        @ConditionalOnMissingBean(TransactionManager.class)
        DataSourceTransactionManager transactionManager(Environment environment, DataSource dataSource,
            ObjectProvider<TransactionManagerCustomizers> transactionManagerCustomizers) {
            // 省略具体代码
        }
        // 省略部分代码
    }
}
~~~

这里需要特别说明一下 `TransactionManagerCustomizers`，它是为 `DataSourceTransactionManager`自动配置类留下的扩展点



`TransactionAutoConfiguration` 会为事务再提供进一步的配置，它主要做了两件事：

1. 创建了「编程式事务」需要用到的 `TransactionTemplate`；

2. 开启了基于注解的事务支持，这部分是由内部类 `EnableTransactionManagementConfiguration` 来定义的，具体代码如下：

   ~~~java
   @Configuration(proxyBeanMethods = false)
   @ConditionalOnBean(TransactionManager.class)
   @ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class)
   public static class EnableTransactionManagementConfiguration {
       @Configuration(proxyBeanMethods = false)
       @EnableTransactionManagement(proxyTargetClass = false)
       @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false",
           matchIfMissing = false)
       public static class JdkDynamicAutoProxyConfiguration {}
   
       @Configuration(proxyBeanMethods = false)
       @EnableTransactionManagement(proxyTargetClass = true)
       @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
           matchIfMissing = true)
       public static class CglibAutoProxyConfiguration {}
   }
   ~~~



`TransactionProperties` 是事务的属性配置，其中只有两个配置：`spring.transaction.default-timeout` 用于配置默认超时时间，默认单位为秒；`spring.transaction.rollback-on-commit-failure` 配置在提交失败时是否回滚。



### 声明式事务

通常在没有特殊需求的情况下，建议使用 Spring Framework 的声明式事务来管理事务

`@Transactional` 注解可以设置的事务属性：

| 属性                                       | 默认值                                         | 描述                       |
| :----------------------------------------- | :--------------------------------------------- | :------------------------- |
| `transactionManager`                       | 默认会找名为 `transactionManager` 的事务管理器 | 指定事务管理器             |
| `propagation`                              | `Propagation.REQUIRED`                         | 指定事务的传播性           |
| `isolation`                                | `Isolation.DEFAULT`                            | 指定事务的隔离性           |
| `timeout`                                  | `-1`，即由具体的底层实现来设置                 | 指定事务超时时间           |
| `readOnly`                                 | `false`                                        | 是否为只读事务             |
| `rollbackFor` / `rollbackForClassName`     | 空                                             | 指定需要回滚事务的异常类型 |
| `noRollbackFor` / `noRollbackForClassName` | 空                                             | 指定无须回滚事务的异常类型 |

默认情况下，事务只会在遇到 `RuntimeException` 和 `Error` 时才会回滚，对于`checked exception`并不会执行回滚操作。

下面我们通过一个例子，来进一步认识这个声明式事务：

~~~java
@Service
public class DemoService {
    public static final String SQL =
        "insert into t_demo (name, create_time, update_time) values(?, now(), now())";
    
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Transactional(readOnly = true)
    public String showNames() {
        return jdbcTemplate.queryForList("select name from t_demo;", String.class)
            .stream().collect(Collectors.joining(","));
    }

    @Transactional(propagation = Propagation.REQUIRED)
    public void insertRecordRequired() {
        jdbcTemplate.update(SQL, "one");
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void insertRecordRequiresNew() {
        jdbcTemplate.update(SQL, "two");
    }

    @Transactional(propagation = Propagation.NESTED)
    public void insertRecordNested() {
        jdbcTemplate.update(SQL, "three");
        throw new RuntimeException(); // 让事务回滚
    }
}
~~~

事务一般放在内部的领域服务上，也就是 Service 层上会是比较常见的一个做法，其中的一个方法，也就对应了一个业务操作。

下面我们来看如何基于XML来配置事务。Spring Framework 提供了一系列 `<tx/>` 的 XML 来配置与事务相关的 AOP 通知。其中，`<tx:advice/>` 用来配置事务通知，具体的事务属性则通过 `<tx:attributes/>` 和 `<tx:method/>` 来设置。

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <tx:advice id="demoTxAdvice">
        <tx:attributes>
            <tx:method name="showNames" read-only="true"/>
            <tx:method name="insertRecordRequired" propagation="REQUIRED"/>
            <tx:method name="insertRecordRequiresNew" propagation="REQUIRES_NEW"/>
            <tx:method name="insertRecordNested" propagation="NESTED"/>
        </tx:attributes>
    </tx:advice>

    <tx:advice id="mixTxAdvice">
        <tx:attributes>
            <tx:method name="*" />
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="demoServiceMethods"
            expression="execution(* learning.spring.transaction.DemoService.*(..))"/>
        
        <aop:pointcut id="mixServiceMethods"
            expression="execution(* learning.spring.transaction.MixService.*(..))"/>
        
        <aop:advisor advice-ref="demoTxAdvice" pointcut-ref="demoServiceMethods"/>
        
        <aop:advisor advice-ref="mixTxAdvice" pointcut-ref="mixServiceMethods"/>
    </aop:config>
</beans>
~~~

`<tx:method/>` 的属性如下：

| 属性              | 默认值     | 含义                                                         |
| :---------------- | :--------- | :----------------------------------------------------------- |
| `name`            | 无         | 要拦截的方法名称，可以带通配符，是唯一的必选项               |
| `propagation`     | `REQUIRED` | 事务传播性                                                   |
| `isolation`       | `DEFAULT`  | 事务隔离性                                                   |
| `timeout`         | `-1`       | 事务超时时间，单位为秒                                       |
| `read-only`       | `false`    | 是否是只读事务                                               |
| `rollback-for`    | 无         | 会触发回滚的异常清单，以逗号分隔，可以是全限定类名，也可以是简单类名 |
| `no-rollback-for` | 无         | 不触发回滚的异常清单，以逗号分隔，可以是全限定类名，也可以是简单类名 |

### 编程式事务

在上面我们提到过`TransactionAutoConfiguration`会创建`TransactionTemplate`，这是通过它的内部类`TransactionTemplateConfiguration`来实现的：

~~~java
@Configuration
public class TxConfiguration {
    @Bean
    public TransactionTemplate transactionTemplate(PlatformTransactionManager transactionManager) {
        return new TransactionTemplate(transactionManager);
    }
}
~~~

我们主要使用TransactionTemplate对象的`execute()` 和 `executeWithoutResult()` 方法：

~~~java
public <T> T execute(TransactionCallback<T> action) throws TransactionException;
public void executeWithoutResult(Consumer<TransactionStatus> action) throws TransactionException;
~~~

使用示例：

~~~java
// Lambda形式
public String showNamesProgrammatically() {
    return transactionTemplate.execute(
            status -> jdbcTemplate.queryForList("select name from t_demo;", String.class)
                    .stream().collect(Collectors.joining(",")));
}

// 匿名类形式
public String showNamesProgrammatically() {
    return transactionTemplate.execute(new TransactionCallback<String>() {
        @Override
        public String doInTransaction(TransactionStatus status) {
            return jdbcTemplate.queryForList("select name from t_demo;", String.class)
                    .stream().collect(Collectors.joining(","));
        }
    });
}
~~~

如果希望修改事务的属性，可以直接调用 `TransactionTemplate` 的对应方法，例如使用 `setPropagationBehavior()` 和 `setIsolationLevel()` 方法来设置传播性与隔离性

## 异常处理

不同数据库的 JDBC 驱动中会定义一些自己的 `SQLException` 子类，而且不同数据库对于同一种错误可能返回不同的错误码。为了避免业务层代码和持久层代码的耦合，Spring Framework 为我们提供了统一的数据库异常类`DataAccessException`。它支持绝大多数常用数据库，将不同数据库的返回码翻译成特定的异常类型。例如，违反了唯一性约束就会抛出的 `DataIntegrityViolationException`；针对主键冲突的异常，还有一个 `DuplicateKeyException` 子类

![{%}](assets/017.jpg)



这背后的核心接口就是 `SQLExceptionTranslator`，它负责将驱动所抛出的 `SQLException` 转换为 `DataAccessException`。`SQLExceptionTranslator` 及其重要实现类的关系如下图所示

<img src="assets/018.jpg" alt="{%}" style="zoom: 33%;" />

其中，`SQLExceptionSubclassTranslator`和`SQLErrorCodeSQLExceptionTranslator`作为备用转换器，当`SQLErrorCodeSQLExceptionTranslator`无法转换时，将降级处理

`JdbcTemplate` 中会创建一个的 `SQLErrorCodeSQLExceptionTranslator`，根据数据库类型选择不同配置来进行实际的异常转换。具体来说就是：

1. `SQLErrorCodeSQLExceptionTranslator` 会通过 `SQLErrorCodesFactory` 加载特定数据库的错误码信息，

2. `SQLErrorCodesFactory` 默认从 CLASSPATH 的 org/springframework/jdbc/support/sql-error-codes.xml 文件中加载错误码配置，这是一个 Bean 的配置文件，其中都是 `SQLErrorCodes` 类型的 Bean。这个文件中包含了 MySQL、Oracle、PostgreSQL、MS-SQL 等 10 余种常见数据库的错误码信息。

   ~~~xml
   <bean id="MySQL" class="org.springframework.jdbc.support.SQLErrorCodes">
       <property name="databaseProductNames">
           <list>
               <value>MySQL</value>
               <value>MariaDB</value>
           </list>
       </property>
       <property name="badSqlGrammarCodes">
           <value>1054,1064,1146</value>
       </property>
       <property name="duplicateKeyCodes">
           <value>1062</value>
       </property>
       <property name="dataIntegrityViolationCodes">
           <value>630,839,840,893,1169,1215,1216,1217,1364,1451,1452,1557</value>
       </property>
       <property name="dataAccessResourceFailureCodes">
           <value>1</value>
       </property>
       <property name="cannotAcquireLockCodes">
           <value>1205,3572</value>
       </property>
       <property name="deadlockLoserCodes">
           <value>1213</value>
       </property>
   </bean>
   ~~~

   



**SQLErrorCodeSQLExceptionTranslator**的转换逻辑在`doTranslate`中，具体流程如下：

- 尝试调用customTranslate方法（留给用户覆写的），若成功则直接放回

- 获取SQLErrorCodes对象

- 尝试调用 `SQLErrorCodes` 中的 `customSqlExceptionTranslator` 方式来转换

- 再尝试调用 `SQLErrorCodes` 中的 `customTranslations`方式来转换

- 最后再根据配置的错误码来判断

  ~~~java
  if (Arrays.binarySearch(sqlErrorCodes.getBadSqlGrammarCodes(), errorCode) >= 0) {
      this.logTranslation(task, sql, sqlEx, false);
      return new BadSqlGrammarException(task, sql != null ? sql : "", sqlEx);
  }
  
  if (Arrays.binarySearch(sqlErrorCodes.getInvalidResultSetAccessCodes(), errorCode) >= 0) {
      this.logTranslation(task, sql, sqlEx, false);
      return new InvalidResultSetAccessException(task, sql != null ? sql : "", sqlEx);
  }
  
  if (Arrays.binarySearch(sqlErrorCodes.getDuplicateKeyCodes(), errorCode) >= 0) {
      this.logTranslation(task, sql, sqlEx, false);
      return new DuplicateKeyException(this.buildMessage(task, sql, sqlEx), sqlEx);
  }
  
  if (Arrays.binarySearch(sqlErrorCodes.getDataIntegrityViolationCodes(), errorCode) >= 0) {
      this.logTranslation(task, sql, sqlEx, false);
      return new DataIntegrityViolationException(this.buildMessage(task, sql, sqlEx), sqlEx);
  }
  
  //...
  ~~~

- 如果最后还是匹配不上，就降级到其他 `SQLExceptionTranslator` 上。



Spring 给我们留了以下三处扩展点来自定义异常转换，首先定义一个自定义`DataAccessException`：

~~~java
public class CustomSQLException extends DataAccessException {
	public CustomSQLException(String msg, Throwable cause) {
		super(msg, cause);		
	}
}
~~~

1. 继承 **SQLErrorCodeSQLExceptionTranslator**，重写 **customTranslate**。

   ~~~dart
   public  class CustomSQLErrorCodeTranslator extends SQLErrorCodeSQLExceptionTranslator {
       @Override	
       protected DataAccessException customTranslate(
           String task, 
           String sql,
           SQLException sqlEx) {		
           return new CustomSQLException("扩展方法一") ;
       }
   }
   
   // 注册
   jdbcTemplate.setExceptionTranslator(new CustomSQLErrorCodeTranslator());
   ~~~

   

2. 继承 **SQLExceptionTranslator**，重写 **translate**。

   ~~~java
   public  class CustomSQLErrorCodeTranslator implements  SQLExceptionTranslator {
   	public DataAccessException translate(
           String task, 
           String sql,	
           SQLException ex) {		
   		return new CustomSQLException("扩展方法二",ex);
   	}
   }
   
   
   // 注册
   SQLErrorCodeSQLExceptionTranslator translator=(SQLErrorCodeSQLExceptionTranslator) jdbcTemplate.getExceptionTranslator();
   
   translator.getSqlErrorCodes().setCustomSqlExceptionTranslatorClass(CustomSQLErrorCodeTranslator.class);
   ~~~

   

3. 使用 **SQLErrorCodes#customTranslations** 

   ~~~java
   SQLErrorCodeSQLExceptionTranslator translator=(SQLErrorCodeSQLExceptionTranslator) jdbcTemplate.getExceptionTranslator();
   			
   CustomSQLErrorCodesTranslation  tran=new CustomSQLErrorCodesTranslation();
   tran.setErrorCodes("8152");							// 设置错误码
   tran.setExceptionClass(CustomSQLException.class);	  // 设置异常类型
   translator.getSqlErrorCodes().setCustomTranslations(tran);	
   ~~~

   

4. 扩展 sql-error-codes.xml

   ~~~xml
   <bean id="MySQL" class="org.springframework.jdbc.support.SQLErrorCodes">
       <property name="databaseProductNames">
           <list>
               <value>MySQL</value>
               <value>MariaDB</value>
           </list>
       </property>
       <property name="badSqlGrammarCodes">
           <value>1054,1064,1146</value>
       </property>
       <property name="duplicateKeyCodes">
           <value>1062</value>
       </property>
       <property name="dataIntegrityViolationCodes">
           <value>630,839,840,893,1169,1215,1216,1217,1364,1451,1452,1557</value>
       </property>
       <property name="dataAccessResourceFailureCodes">
           <value>1</value>
       </property>
       <property name="cannotAcquireLockCodes">
           <value>1205,3572</value>
       </property>
       <property name="deadlockLoserCodes">
           <value>1213</value>
       </property>
       
       
       <property name="customTranslations">
           <!--这里就是我们自定义的异常转换-->
           <bean class="org.springframework.jdbc.support.CustomSQLErrorCodesTranslation">
               <property name="errorCodes" value="123456" />
               <property name="exceptionClass" value="learning.spring.data.DbSwitchingException" />
           </bean>
           
       </property>
   </bean>
   ~~~

