## 如何指定id策略

在[JPA](https://so.csdn.net/so/search?q=JPA&spm=1001.2101.3001.7020)中，我们是通过`@id`和`@GeneratedValue`来指定id[主键](https://so.csdn.net/so/search?q=主键&spm=1001.2101.3001.7020)和id策略的，比如：

```java
@Id 
@GeneratedValue(strategy = GenerationType.AUTO) 
@Column(name = "id") 
private String id;
```

这样也就指定了id和生成id所使用的策略，下面我们来看一下都有哪些策略呢

## 4种JPA策略用法

我们点进`@GeneratedValue`源码里可以看到，`strategy`属性是由`GenerationType`指定的，我们点进`GenerationType`里面可以看到这里定义了四种策略：
\- **TABLE**：使用一个特定的数据库表格来保存主键
\- **SEQUENCE**：根据底层数据库的序列来生成主键，条件是数据库支持序列，如Oracle的序列
\- **IDENTITY**：主键由数据库自动生成（主要是自动增长型），如MySQL的auto_increment
\- **AUTO**：默认值。主键由程序控制、程序自动帮我们选择主键生成策略（TABLE/SEQUENCE/IDENTITY）

这些策略也不是所有数据库都支持的，支持情况如下表：

| 策略\数据库 | mysql  | oracle | postgreSQL | kingbase |
| ----------- | ------ | ------ | ---------- | -------- |
| TABLE       | 支持   | 支持   | 支持       | 支持     |
| SEQUENCE    | 不支持 | 支持   | 支持       | 支持     |
| IDENTITY    | 支持   | 不支持 | 支持       | 支持     |
| AUTO        | 支持   | 支持   | 支持       | 支持     |

### TABLE

创建表

| SEQUENCE_TABLE |                 |
| :------------- | --------------- |
| ***SEQ_NAME*** | ***SEQ_COUNT*** |
| EMP_SEQ        | 13              |
| PROJ_SEQ       | 570             |

```java
...
@Entity
public class Employee {
    @Id
    @TableGenerator(name="TABLE_GEN", table="SEQUENCE_TABLE", pkColumnName="SEQ_NAME",
        valueColumnName="SEQ_COUNT", pkColumnValue="EMP_SEQ")
    @GeneratedValue(strategy=GenerationType.TABLE, generator="TABLE_GEN")
    private long id;
    ...
}
```

### SEQUENCE

Sequence objects use special database objects to generate ids. Sequence objects are only supported in some databases, such as Oracle, DB2, and Postgres. Usually, a SEQUENCE object has a name, an INCREMENT, and other database object settings. Each time the `<sequence>.NEXTVAL` is selected the sequence is incremented by the INCREMENT.

Sequence objects provide the optimal sequencing option, as they are the most efficient and have the best concurrency, however they are the least portable as most databases do not support them. Sequence objects support sequence preallocation through setting the INCREMENT on the database sequence object to the sequence preallocation size.

```java
...
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy=GenerationType.SEQUENCE, generator="EMP_SEQ")
    @SequenceGenerator(name="EMP_SEQ", sequenceName="EMP_SEQ", allocationSize=100)
    private long id;
   
}
```

### IDENTITY

```java
...
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private long id;
    ...
}
```



官方wiki：https://en.wikibooks.org/wiki/Java_Persistence/Identity_and_Sequencing#Sequencing

hibernate：https://docs.jboss.org/hibernate/orm/4.2/manual/en-US/html/ch05.html#mapping-declaration-id

## Hibernate拓展id策略

当然，很多时候，这么几种策略并不够用，这里hibernate也拓展了JPA的id策略，我们可以在`org.hibernate.id.IdentifierGeneratorFactory`中看到，主要提供了这么些策略：

1. **native**: 对于oracle采用Sequence方式，对于MySQL和SQL Server采用identity(自增主键生成机制)，native就是将主键的生成工作交由数据库完成，hibernate不管(很常用)。

2. **uuid**: 采用128位的uuid算法生成主键，uuid被编码为一个32位16进制数字的字符串。占用空间大(字符串类型)。

3. **hilo**: 使用hilo生成策略，要在数据库中建立一张额外的表，默认表名为hibernate_unique_key,默认字段为Integer类型，名称是next_hi(比较少用)。

4. **assigned**: 在插入数据的时候主键由程序处理(很常用)，这是`generator`元素没有指定时的默认生成策略。等同于JPA中的AUTO。

5. **identity**: 使用SQL Server和MySQL的自增字段，这个方法不能放到Oracle中，Oracle不支持自增字段，要设定sequence(MySQL和SQL Server中很常用)。等同于JPA中的IDENTITY。

6. **select**: 使用触发器生成主键(主要用于早期的数据库主键生成机制，少用)。

7. **sequence**: 调用底层数据库的序列来生成主键，要设定序列名，不然hibernate无法找到。

8. **seqhilo**: 通过hilo算法实现，但是主键历史保存在Sequence中，适用于支持Sequence的数据库，如Oracle(比较少用)。

9. **increment**: 插入数据的时候hibernate会给主键添加一个自增的主键，但是一个hibernate实例就维护一个计数器，所以在多个实例运行的时候不能使用这个方法。

10. **foreign**: 使用另外一个相关联的对象的主键。通常和联合起来使用。

11. **guid**: 采用数据库底层的guid算法机制，对应MYSQL的uuid()函数，SQL Server的newid()函数，ORACLE的rawtohex(sys_guid())函数等。

12. **uuid.hex**: 看uuid，建议用uuid替换。

13. **sequence-identity**: sequence策略的扩展，采用立即检索策略来获取sequence值，需要JDBC3.0和JDK4以上（含1.4）版本 。
    具体使用就是多了一个`@GenericGenerator`注解，指定策略，指定自定义名称，然后在`@GeneratedValue`中使用该策略，比如：

```java
@Id
@GeneratedValue(generator  = "myIdStrategy")
@GenericGenerator(name = "myIdStrategy", strategy = "uuid")
@Column(name = "id")
private String id;
```



其他的类似，就不再多举例了

到这里已经有很多策略供我们使用了，但是呢，有时候比如分布式系统中要求全局id唯一，或者其他一些场景，要求我们有自己的策略，那么该怎么做呢？

## 使用的id策略(以snowflake为例)

在看其他策略源码的时候，我们发现他们实现了这样一个接口`IdentifierGenerator`，他位于`org.hibernate.id`包下，我们进入该类，就可以看到源码注释上写着用户实现此接口可以实现自定义id策略
那么就很简单了，我们实现一下这个接口：

```java
public class SnowflakeId implements IdentifierGenerator{
    .
    .
    .
    @Override
    public Serializable generate(SessionImplementor s, Object obj) {
        return getId() + "";
    }
}
```

实现`generate`方法，方法体调用我们生成id的方法就好了，这里省略了生成过程，有需要可以去我代码里找一下。

***注意\***：`IdentifierGenerator`接口里面还写了一句注释，必须要实现一个默认的无参构造。当时实现的就少看了这一句，折腾了好久(手动捂脸)。所以大致是这样的：

```java
public class SnowflakeId implements IdentifierGenerator{

    public SnowflakeId() {
    }
    .
    .
    .

    @Override
    public Serializable generate(SessionImplementor s, Object obj) {
        return getId() + "";
    }
}
```

自定义完了之后，我们只需要在指定策略的时候使用我们自定义的就好了，`@GenericGenerator`注解的`strategy`属性上说了，使用非默认策略的时候，需要使用全类名，即：

```java
@Id
@GeneratedValue(generator = "snowFlakeId")
@GenericGenerator(name = "snowFlakeId", strategy = "top.felixu.idworker.SnowflakeId")
@Column(name = "id")
private String id;
```

这样我们就实现了自己的id策略了





## 常见问题

### 使用数据库自增

Entity

```java
@Id 
@GeneratedValue(strategy = GenerationType.IDENTITY) 
@Column(name = "id") 
private Long id;
```

数据库设置主键自增

```mysql
 `id` bigint(20) NOT NULL AUTO_INCREMENT
```















前置条件

数据库：mysql5.7，使用spring-data-jpa查询没问题，save报错，entity如下图

![这里写图片描述](https://img-blog.csdn.net/20180726203121616?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4ODQ0MDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



分析原因及就解决方法

原因：jpa默认是auto的模式，也就是主键序列化；而mysql是不支持的，oracle支持的；

解决方法：使用mysql时需要主动设置id的策略

![这里写图片描述](https://img-blog.csdn.net/20180726203421216?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4ODQ0MDQw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)