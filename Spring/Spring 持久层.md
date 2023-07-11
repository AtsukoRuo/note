Spring 持久层

![image-20230704171938287](C:\Users\AtsukoRuo\Desktop\note\Spring\assets\image-20230704171938287.png)





H2Database只在内存中运行，关闭连接后数据库将被清空，适合测试环境。



在application.properties中添加

```properties
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb
```

然后可以在浏览器http://localhost:8080/h2-console/中访问

springboot每次启动时都会读取resources/.sql到h2中，







Spring JDBC简化了JDBC

Spring JDBC的例子：

~~~java
@Component
public class CourseJdbcCommandLineRunner implements CommandLineRunner {

    @Autowired
    private CourseJdbcRepository repository;

    @Override
    public void run(String... args) throws Exception {
        repository.insert(new Course(2, "Learn AWS Now!", "in28minutes"));
        repository.insert(new Course(3, "Learn Azure Now!", "in28minutes"));
        repository.insert(new Course(4, "Learn DevOps Now!", "in28minutes"));
        repository.deleteById(2);
        System.out.println(repository.findById(3));
    }
}


@Repository
public class CourseJdbcRepository {

    @Autowired
    private JdbcTemplate springJdbcTemplate;

    private static String INSERT_QUERY =
            """
                insert into course (id, name, author)
                values(?, ?, ?);
            """;

    private static String DELETE_QUERY =
            """
                delete from course where id = ?
            """;

    private static String SELECT_QUERY =
            """
                select * from course where id = ?
            """;
    public void insert(Course course) {
        springJdbcTemplate.update(INSERT_QUERY, course.getId(), course.getName(), course.getAuthor());
    }

    public void deleteById(long id) {
        springJdbcTemplate.update(DELETE_QUERY, id);
    }

    public Course findById(long id) {
        return springJdbcTemplate.queryForObject(SELECT_QUERY, new BeanPropertyRowMapper<>(Course.class), id);
    }
}

public class Course {
    private long id;
    private String name;
    private String author;
	//setter、getter、constructor、toString
}
~~~

> CommandLineRunner是Spring Boot框架中的一个接口，用于在Spring Boot应用程序启动时运行一些代码。它提供了一个run方法，可以在应用程序启动时自动运行。





而Spring JPA进一步简化了Spring JDBC

~~~java
@Component
public class CourseJpaCommandLineRunner implements CommandLineRunner {

    @Autowired
    private CourseJpaRepository repository;

    @Override
    public void run(String[] args) {
        repository.insert(new Course(2, "Learn AWS Now!", "in28minutes"));
        repository.insert(new Course(3, "Learn Azure Now!", "in28minutes"));
        repository.insert(new Course(4, "Learn DevOps Now!", "in28minutes"));
        repository.deleteById(2);
        System.out.println(repository.findById(3));

    }
}

@Repository
@Transactional
public class CourseJpaRepository {
    @PersistenceContext     //与@Autowired作用差不多
    private EntityManager entityManager;

    public void insert(Course course) {
        entityManager.merge(course);
    }

    public Course findById(long id) {
        return entityManager.find(Course.class, id);
    }

    public void deleteById(long id) {
        entityManager.remove(entityManager.find(Course.class, id));
    }

}

@Entity(name="course")
public class Course {

    @Id     //主键
    private long id;

    //@Column(name="name")      名字一样可以省略@Column或者name
    private String name;

    @Column(name="author")
    private String author;

    //setter、getter、constructor、toString
}
~~~





Spring Data JPA更进一步简化Spring JPA
~~~java
@Component
public class CourseSpringDataCommandLineRunner implements CommandLineRunner {

    @Autowired
    private CourseSpringDataJpaRepository repository;

    @Override
    public void run(String... args) throws Exception {
        repository.save(new Course(2, "Learn AWS Now!", "in28minutes"));
        repository.save(new Course(3, "Learn Azure Now!", "in28minutes"));
        repository.save(new Course(4, "Learn DevOps Now!", "in28minutes"));
        repository.deleteById(2l);
        System.out.println(repository.findById(3l));
    }
}

public interface CourseSpringDataJpaRepository extends JpaRepository<Course, Long> {
		//这意味着JpaRepository接口中定义的方法将针对Course实体类进行操作，并且使用Long类型的ID唯一标识Course实体类的每个记录。
    	//按照命名约定，Spring Data JPA会自动为你实现这些方法
    	List<Course> findByAuthor(String author);
    	List<Course> findByName(String name);
}

public class Course {
    private long id;
    private String name;
    private String author;
	//setter、getter、constructor、toString
}

~~~

JPA（Java Persistence API）是Java EE 5规范中定义的一套API，用于管理Java对象与关系型数据库之间的映射关系和持久化操作。JPA提供了一种标准化的方式来进行ORM（对象关系映射）操作，使得开发者可以通过一套API来访问不同的ORM框架。

Hibernate是一个开源的、基于JPA规范的ORM框架，它实现了JPA的所有API，并提供了一些扩展功能。







H2数据库会根据@Entity Bean对象创建一张表（其他数据库并不会这样），这是在SpringBoot启动阶段完成的。此时可以@Entity(name="")指定表名，默认使用类名作为表名。@Column(name="name")指定表中的属性名，默认使用字段名作为属性名。

~~~java
@Entity(name = "TodoABC") 
public class Todo {
    @Column("id")
    private int ID;
}
~~~

默认情况下，SpringBoot会先读取resource下的sql文件，再扫描Entity创建表，但是可以通过以下配置改变这一行为，即先创建表，再读取文件

~~~properties
spring.jpa.defer-datasource-initialization=true
~~~

可以通过以下配置在终端查看执行了什么SQL语句

~~~properties
spring.jpa.show-sql=true;
~~~







通过Docker启动MySQL，在终端中输入

~~~shell
docker run --detach --env MYSQL_ROOT_PASSWORD=root --env MYSQL_USER=todos-user --env MYSQL_PASSWORD=dummytodos --env MYSQL_DATABASE=todos --name mysql --publish 3310:3310 mysql:8-oracle
~~~

在依赖中

~~~xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
~~~

在配置文件中（这个配置对本地数据库也适用）

~~~properties
spring.datasource.url=jdbc:mysql://localhost:3310/todos
spring.datasource.username=todos-user
spring.datasource.password=dummytodos
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
~~~

