# DAO 进阶



## JDBC

JDBC 的全称是 Java Database Connectivity，是一套面向关系型数据库的规范。每个数据库生厂商都提供了基于 JDBC 规范实现的 JDBC 驱动。开发者只需要面向 JDBC 接口编程，就能在很大程度上规避掉由数据库实现的差异所带来的问题。



~~~java
DriverManagerDataSource dataSource = new DriverManagerDataSource();
dataSource.setDriverClassName("com.mysql.jdbc.Driver");
dataSource.setUrl("jdbc:mysql://localhost:3306/spring-dao?characterEncoding=utf8");
dataSource.setUsername("root");
dataSource.setPassword("123456");

JdbcTemplate jdbcTemplate = new JdbcTemplate();
jdbcTemplate.setDataSource(dataSource);
~~~

`DriverManagerDataSource`是Spring自带的数据源插件，没有连接池的概念。 即每次都会创建新的数据库连接。



### CURD

~~~java
int row = jdbcTemplate.update("insert into tbl_user (name, tel) values (?, ?)", "heihei", "200");

int row = jdbcTemplate.update("update tbl_user set tel = ? where name = ?", "54321", "heihei");

int row = jdbcTemplate.update("delete from tbl_user where name = ?", "heihei");

List<User> userList = jdbcTemplate.query("select * from tbl_user", new BeanPropertyRowMapper<>(User.class));
// 如果没有结果集，那么就返回一个空的List
User user = userList.size() > 0 ? userList.get(0) : null;
~~~

我们看一些`query()`方法的定义：

~~~java
query(String sql, RowMapper<T> rowMapper, Object... args) 。
~~~

`RowMapper`是将 `ResultSet` 的一行数据封装为一个指定的类型，它的定义如下：

~~~java
@FunctionalInterface
public interface RowMapper<T> {
	T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
~~~



## 缓存



## 事务

事务就是一组逻辑操作的组合，它被赋予四个特性：

- **原子性**：一个事务就是一个不可再分解的单位，事务中的操作要么全部做，要么全部不做。
- **一致性**：事务执行后，所有的数据都应该保持一致状态。
- **隔离性**：多个数据库操作并发执行时，一个请求的事务操作不能被其它操作干扰，多个并发事务执行之间要相互隔离。
- **持久性**：事务执行完成后，它对数据的影响是永久性的。

其中，原子性、隔离性、持久性是手段，而一致性是目的

事务并发操作中会出现三种问题：

- 脏读：一个事务读到了另一个事务没有提交的数据
- 不可重复读
- 幻读

针对上述三个问题，由此引出了事务的隔离级别：

- **read uncommitted** 读未提交 —— 不解决任何问题
- **read committed** 读已提交 —— 解决脏读
- **repeatable read** 可重复读 —— 解决脏读、不可重复读
- **serializable** 可串行化 —— 解决脏读、不可重复读、幻读



JDBC中的事务示例：

~~~java
public class JdbcTransactionApplication {
    
    public static void main(String[] args) throws SQLException {
        Connection connection = null;
        try {
            connection = dataSource.getConnection();
            // 开启事务，关闭自动提交
            connection.setAutoCommit(false);
            PreparedStatement statement = connection.prepareStatement(...);
            statement.executeUpdate();
            
            int i = 1 / 0;
            // 提交事务
            connection.commit();
        } catch (Exception e) {
            // 回滚事务
            connection.rollback();
        } finally {
            // 关闭连接
            if (connection != null) {
                connection.close();
            }
        }
    }
}
~~~

当程序运行出现异常时，可以让事务只回滚到某个保存点上

~~~java
try {
    PreparedStatement statement = connection.prepareStatement(...);
    statement.executeUpdate();

    // 事务保存点
    savepoint = connection.setSavepoint();

    statement = connection.prepareStatement("insert into tbl_account (user_id, money) values (2, 123)");
    statement.executeUpdate();

    int i = 1 / 0;
} catch (Exception e) {
    if (savepoint != null) {
        connection.rollback(savepoint);
        connection.commit();
    } else {
        connection.rollback();
    }
}
~~~



**事务（Transaction）**在不同的语境下有着不同的含义

- 在数据库中，事务就是其原子性（Atomic）——操作要么全都执行，要么都不执行。
- 在 Java EE 环境中，事务可以是使用 JTA（Java Transaction API）这样的全局事务，也可以是基于 JDBC 连接的本地事务。
- 分布式事务



为了消除代码对不同事务对的依赖，Spring Framework 对事务管理做了一层抽象。这个抽象的核心是事务管理器，即 `TransactionManager`，它是一个标记接口。而`PlatformTransactionManager`接口继承了 `TransactionManager`接口，定义了获取事务、提交事务和回滚事务的方法：

```java
public interface PlatformTransactionManager extends TransactionManager {
    //获得事务
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
    //提交事务
    void commit(TransactionStatus var1) throws TransactionException;
    //回滚事务
    void rollback(TransactionStatus var1) throws TransactionException;
}
```

`DataSourceTransactionManager`、`JtaTransactionManager` 和 `HibernateTransactionManager` 这些底层事务管理器都实现了上述接口。

在Spring中，事务有两种实现方式：

- **编程式事务管理**：编程式事务管理使用`TransactionTemplate`，或者直接使用底层的`PlatformTransactionManager`。

- **声明式事务管理**： 建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。 通过`@Transactional`就可以进行事务操作。

  默认情况下，声明式事务在遇到 `RuntimeException` 和 `Error` 时才会回滚，对于`checked exception`并不会执行回滚操作。
