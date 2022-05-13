## 一口气说出 6种 @Transactional 注解失效场景

原创 程序员内点事 [程序员内点事](javascript:void(0);) *2020-03-19*

点击“ 程序员内点事 ”关注，选择“ [设置星标](http://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486188&idx=3&sn=f160d91ea23e5113e6077c500a2e30c4&chksm=fa49755dcd3efc4bf4f566fbbbf74c191d0b79f2d3222fd211bc52d80b5ef127f52b1158ed71&scene=21#wechat_redirect) ”

坚持学习，好文每日送达！



**引言**



昨天公众号粉丝咨询了一个问题，说自己之前面试被问`@Transactional`注解哪些场景下会失效，一时语塞致使面试失败。所以今天简单的和大家分享一下`@Transactional`相关的知识。

`@Transactional` 注解相信大家并不陌生，平时开发中很常用的一个注解，它能保证方法内多个数据库操作要么同时成功、要么同时失败。使用`@Transactional`注解时需要注意许多的细节，不然你会发现`@Transactional`总是莫名其妙的就失效了。

**
**

## **一、事务**



事务管理在系统开发中是不可缺少的一部分，`Spring`提供了很好事务管理机制，主要分为`编程式事务`和`声明式事务`两种。

**编程式事务**：是指在代码中手动的管理事务的提交、回滚等操作，代码侵入性比较强，如下示例：

```
1try {
2    //TODO something
3     transactionManager.commit(status);
4} catch (Exception e) {
5    transactionManager.rollback(status);
6    throw new InvoiceApplyException("异常失败");
7}
```

**声明式事务**：基于`AOP`面向切面的，它将具体业务与事务处理部分解耦，代码侵入性很低，所以在实际开发中声明式事务用的比较多。声明式事务也有两种实现方式，一是基于`TX`和`AOP`的xml配置文件方式，二种就是基于@Transactional注解了。

```
1    @Transactional
2    @GetMapping("/test")
3    public String test() {
4
5        int insert = cityInfoDictMapper.insert(cityInfoDict);
6    }
```





***\*
\****

## 二、@Transactional介绍

#### 1、@Transactional注解可以作用于哪些地方？ 

@Transactional 可以作用在`接口`、`类`、`类方法`。

- **作用于类**：当把@Transactional 注解放在类上时，表示所有该类的`public`方法都配置相同的事务属性信息。
- **作用于方法**：当类配置了@Transactional，方法也配置了@Transactional，方法的事务会覆盖类的事务配置信息。
- **作用于接口**：不推荐这种使用方法，因为一旦标注在Interface上并且配置了Spring AOP 使用CGLib动态代理，将会导致@Transactional注解失效

```
 1@Transactional
 2@RestController
 3@RequestMapping
 4public class MybatisPlusController {
 5    @Autowired
 6    private CityInfoDictMapper cityInfoDictMapper;
 7
 8    @Transactional(rollbackFor = Exception.class)
 9    @GetMapping("/test")
10    public String test() throws Exception {
11        CityInfoDict cityInfoDict = new CityInfoDict();
12        cityInfoDict.setParentCityId(2);
13        cityInfoDict.setCityName("2");
14        cityInfoDict.setCityLevel("2");
15        cityInfoDict.setCityCode("2");
16        int insert = cityInfoDictMapper.insert(cityInfoDict);
17        return insert + "";
18    }
19}
```

#### 2、@Transactional注有哪些属性？

##### propagation属性

`propagation` 代表事务的传播行为，默认值为 `Propagation.REQUIRED`，其他的属性信息如下：

- `Propagation.REQUIRED`：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。**(** 也就是说如果A方法和B方法都添加了注解，在默认传播模式下，A方法内部调用B方法，会把两个方法的事务合并为一个事务 **）**
- `Propagation.SUPPORTS`：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
- `Propagation.MANDATORY`：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
- `Propagation.REQUIRES_NEW`：重新创建一个新的事务，如果当前存在事务，暂停当前的事务。**(** 当类A中的 a 方法用默认`Propagation.REQUIRED`模式，类B中的 b方法加上采用 `Propagation.REQUIRES_NEW`模式，然后在 a 方法中调用 b方法操作数据库，然而 a方法抛出异常后，b方法并没有进行回滚，因为`Propagation.REQUIRES_NEW`会暂停 a方法的事务 **)**
- `Propagation.NOT_SUPPORTED`：以非事务的方式运行，如果当前存在事务，暂停当前的事务。
- `Propagation.NEVER`：以非事务的方式运行，如果当前存在事务，则抛出异常。
- `Propagation.NESTED` ：和 Propagation.REQUIRED 效果一样。

##### isolation 属性

`isolation` ：事务的隔离级别，默认值为 `Isolation.DEFAULT`。

- Isolation.DEFAULT：使用底层数据库默认的隔离级别。
- Isolation.READ_UNCOMMITTED
- Isolation.READ_COMMITTED
- Isolation.REPEATABLE_READ
- Isolation.SERIALIZABLE

##### timeout 属性

`timeout` ：事务的超时时间，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

##### readOnly 属性

`readOnly` ：指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。

##### rollbackFor 属性

`rollbackFor` ：用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

##### **noRollbackFor**属性**

`noRollbackFor`：抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。

### ** **



## **二、@Transactional失效场景**



接下来我们结合具体的代码分析一下哪些场景下，@Transactional 注解会失效。

#### 1、@Transactional 应用在非 public 修饰的方法上

如果`Transactional`注解应用在非`public` 修饰的方法上，Transactional将会失效。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aOhsEJ0TtJSM5kcSOKZGJDOJoUPF0IHL2wvUHIzlHVYOib6bEN2tKkCyUn8fOxMVicBxvoKpgXU2HDQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在这里插入图片描述

之所以会失效是因为在Spring AOP 代理时，如上图所示 `TransactionInterceptor` （事务拦截器）在目标方法执行前后进行拦截，`DynamicAdvisedInterceptor`（CglibAopProxy 的内部类）的 intercept 方法或 `JdkDynamicAopProxy` 的 invoke 方法会间接调用 `AbstractFallbackTransactionAttributeSource`的 `computeTransactionAttribute` 方法，获取Transactional 注解的事务配置信息。

```
1protected TransactionAttribute computeTransactionAttribute(Method method,
2    Class<?> targetClass) {
3        // Don't allow no-public methods as required.
4        if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
5        return null;
6}
```

此方法会检查目标方法的修饰符是否为 public，不是 public则不会获取@Transactional 的属性配置信息。

**注意：****`protected`、`private` 修饰的方法上使用 `@Transactional` 注解，虽然事务无效，但不会有任何报错，这是我们很容犯错的一点。**

#### 2、@Transactional 注解属性 propagation 设置错误

这种失效是由于配置错误，若是错误的配置以下三种 propagation，事务将不会发生回滚。

`TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
`TransactionDefinition.PROPAGATION_NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。

#### 3、@Transactional 注解属性 rollbackFor 设置错误

`rollbackFor` 可以指定能够触发事务回滚的异常类型。Spring默认抛出了未检查`unchecked`异常（继承自 `RuntimeException`的异常）或者 `Error`才回滚事务；其他异常不会触发回滚事务。如果在事务中抛出其他类型的异常，但却期望 Spring 能够回滚事务，就需要指定 **rollbackFor**属性。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aOhsEJ0TtJSM5kcSOKZGJDOENdU9CHkPlMgJrCv3Du00eDAiaaqleo6D9Q49dQuwY4lddQOsRRiadPA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在这里插入图片描述

```
1// 希望自定义的异常可以进行回滚
2@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class
```

若在目标方法中抛出的异常是 `rollbackFor` 指定的异常的子类，事务同样会回滚。Spring源码如下：

```
 1private int getDepth(Class<?> exceptionClass, int depth) {
 2        if (exceptionClass.getName().contains(this.exceptionName)) {
 3            // Found it!
 4            return depth;
 5}
 6        // If we've gone as far as we can go and haven't found it...
 7        if (exceptionClass == Throwable.class) {
 8            return -1;
 9}
10return getDepth(exceptionClass.getSuperclass(), depth + 1);
11}
```

#### 4、同一个类中方法调用，导致@Transactional失效

开发中避免不了会对同一个类里面的方法调用，比如有一个类Test，它的一个方法A，A再调用本类的方法B（不论方法B是用public还是private修饰），但方法A没有声明注解事务，而B方法有。则外部调用方法A之后，方法B的事务是不会起作用的。这也是经常犯错误的一个地方。

那为啥会出现这种情况？其实这还是由于使用`Spring AOP`代理造成的，因为只有当事务方法被当前类以外的代码调用时，才会由`Spring`生成的代理对象来管理。

```
 1//@Transactional
 2    @GetMapping("/test")
 3    private Integer A() throws Exception {
 4        CityInfoDict cityInfoDict = new CityInfoDict();
 5        cityInfoDict.setCityName("2");
 6        /**
 7         * B 插入字段为 3的数据
 8         */
 9        this.insertB();
10        /**
11         * A 插入字段为 2的数据
12         */
13        int insert = cityInfoDictMapper.insert(cityInfoDict);
14
15        return insert;
16    }
17
18    @Transactional()
19    public Integer insertB() throws Exception {
20        CityInfoDict cityInfoDict = new CityInfoDict();
21        cityInfoDict.setCityName("3");
22        cityInfoDict.setParentCityId(3);
23
24        return cityInfoDictMapper.insert(cityInfoDict);
25    }
```

#### 5、异常被你的 catch“吃了”导致@Transactional失效

这种情况是最常见的一种@Transactional注解失效场景

```
 1    @Transactional
 2    private Integer A() throws Exception {
 3        int insert = 0;
 4        try {
 5            CityInfoDict cityInfoDict = new CityInfoDict();
 6            cityInfoDict.setCityName("2");
 7            cityInfoDict.setParentCityId(2);
 8            /**
 9             * A 插入字段为 2的数据
10             */
11            insert = cityInfoDictMapper.insert(cityInfoDict);
12            /**
13             * B 插入字段为 3的数据
14             */
15            b.insertB();
16        } catch (Exception e) {
17            e.printStackTrace();
18        }
19    }
```

如果B方法内部抛了异常，而A方法此时try catch了B方法的异常，那这个事务还能正常回滚吗？

答案：不能！

会抛出异常：

```
1org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
```

因为当`ServiceB`中抛出了一个异常以后，`ServiceB`标识当前事务需要`rollback`。但是`ServiceA`中由于你手动的捕获这个异常并进行处理，`ServiceA`认为当前事务应该正常`commit`。此时就出现了前后不一致，也就是因为这样，抛出了前面的`UnexpectedRollbackException`异常。

`spring`的事务是在调用业务方法之前开始的，业务方法执行完毕之后才执行`commit` or `rollback`，事务是否执行取决于是否抛出`runtime异常`。如果抛出`runtime exception` 并在你的业务方法中没有catch到的话，事务会回滚。

在业务方法中一般不需要catch异常，如果非要catch一定要抛出`throw new RuntimeException()`，或者注解中指定抛异常类型`@Transactional(rollbackFor=Exception.class)`，否则会导致事务失效，数据commit造成数据不一致，所以有些时候try catch反倒会画蛇添足。

#### 6、数据库引擎不支持事务

这种情况出现的概率并不高，事务能否生效数据库引擎是否支持事务是关键。常用的MySQL数据库默认使用支持事务的`innodb`引擎。一旦数据库引擎切换成不支持事务的`myisam`，那事务就从根本上失效了。



## 总结

@Transactional 注解的看似简单易用，但如果对它的用法一知半解，还是会踩到很多坑的。

------

**今天就说这么多，如果本文对您有一点帮助，希望能得到您一个点赞👍哦**

**您的认可才是我写作的动力！**

![图片](https://mmbiz.qpic.cn/mmbiz_gif/US10Gcd0tQGXUUoViazicvYYchRc1IJiaPXAVWRtBpmPxW6zARz0aXmhj1l69okNR8eut6Zo8Jiak03rEz8jvjH06w/640?tp=webp&wxfrom=5&wx_lazy=1)