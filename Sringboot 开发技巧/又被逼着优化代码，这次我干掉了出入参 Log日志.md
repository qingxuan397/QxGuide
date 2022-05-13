## 又被逼着优化代码，这次我干掉了出入参 Log日志

原创 程序员内点事 [程序员内点事](javascript:void(0);) *2020-07-20*

最近技术部突然刮起一阵 `review` 代码的小风，挨个项目组过代码，按理说这应该是件挺好的事，让别人指出自己代码中的不足，查缺补漏，对提升自身编码能力有很大帮助，毕竟自己审查很容易“`陶醉`”在自己写的代码里。![图片](https://mmbiz.qpic.cn/mmbiz_jpg/0OzaL5uW2aPnwzAPGIfufkedNmnrdPAogfPAMGQoWJDYqKwIGexQD0lqQvPfvWnGDwzicLc5byAI2viaib5g2619w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

不过，代码 `review` 的详细程度令人发指，一行一行的分析，简直就是个培训班啊。不夸张的说，如果我村里仅有县重点小学学历的四大爷，来听上一个月后，保证能上手开发，666~

既然组内气氛到这了，咱也得行动起来，要不哪天评审到我的代码，让人家指指点点的心里多少有点不舒服，与其被动优化代码不如主动出击~

选优化代码的方向，方法入参和返回结果日志首当其冲，每个方法都会有这两个日志，一大堆冗余的代码，而且什么样的打印格式都有，非常的杂乱。

```
public OrderDTO getOrder(OrderVO orderVO, String name) {

        log.info("订单详情入参：orderVO={},name={}", JSON.toJSONString(orderVO), name);

        OrderDTO orderInfo = orderService.getOrderInfo(orderVO);

        log.info("订单详情结果：orderInfo={}", JSON.toJSONString(orderInfo));

        return orderInfo;
}
```

下边我们利用 `AOP` 实现请求方法的入参、返回结果日志统一打印，避免日志打印格式杂乱，同时减少业务代码量。

### 一、自定义注解

自定义切面注解`@PrintlnLog` 用来输出日志，注解权限 `@Target({ElementType.METHOD})` 限制只在方法上使用，注解中只有一个参数 `description` ，用来自定义方法输出日志的描述。

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Documented
public @interface PrintlnLog {

    /**
     * 自定义日志描述信息文案
     *
     * @return
     */
    String description() default "";
}
```

### 二、切面类

接下来编写`@PrintlnLog` 注解对应的切面实现，`doBefore(）`中输出方法的自定义描述、入参、请求方式、请求url、被调用方法的位置等信息，`doAround()` 中打印方法返回结果。

`注意：` 如何想指定切面在哪个环境执行，可以用`@Profile` 注解，只打印某个环境的日志。

```
@Slf4j
@Aspect
@Component
//@Profile({"dev"}) //只对某个环境打印日志
public class LogAspect {

    private static final String LINE_SEPARATOR = System.lineSeparator();

    /**
     * 以自定义 @PrintlnLog 注解作为切面入口
     */
    @Pointcut("@annotation(com.chengxy.unifiedlog.config.PrintlnLog)")
    public void PrintlnLog() {
    }

    /**
     * @param joinPoint
     * @author fu
     * @description 切面方法入参日志打印
     * @date 2020/7/15 10:30
     */
    @Before("PrintlnLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {

        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        String methodDetailDescription = this.getAspectMethodLogDescJP(joinPoint);

        log.info("------------------------------- start --------------------------");
        /**
         * 打印自定义方法描述
         */
        log.info("Method detail Description: {}", methodDetailDescription);
        /**
         * 打印请求入参
         */
        log.info("Request Args: {}", JSON.toJSONString(joinPoint.getArgs()));
        /**
         * 打印请求方式
         */
        log.info("Request method: {}", request.getMethod());
        /**
         * 打印请求 url
         */
        log.info("Request URL: {}", request.getRequestURL().toString());

        /**
         * 打印调用方法全路径以及执行方法
         */
        log.info("Request Class and Method: {}.{}", joinPoint.getSignature().getDeclaringTypeName(), joinPoint.getSignature().getName());
    }

    /**
     * @param proceedingJoinPoint
     * @author xiaofu
     * @description 切面方法返回结果日志打印
     * @date 2020/7/15 10:32
     */
    @Around("PrintlnLog()")
    public Object doAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {

        String aspectMethodLogDescPJ = getAspectMethodLogDescPJ(proceedingJoinPoint);

        long startTime = System.currentTimeMillis();

        Object result = proceedingJoinPoint.proceed();
        /**
         * 输出结果
         */
        log.info("{}，Response result  : {}", aspectMethodLogDescPJ, JSON.toJSONString(result));

        /**
         * 方法执行耗时
         */
        log.info("Time Consuming: {} ms", System.currentTimeMillis() - startTime);

        return result;
    }

    /**
     * @author xiaofu
     * @description 切面方法执行后执行
     * @date 2020/7/15 10:31
     */
    @After("PrintlnLog()")
    public void doAfter(JoinPoint joinPoint) throws Throwable {
        log.info("------------------------------- End --------------------------" + LINE_SEPARATOR);
    }

    /**
     * @param joinPoint
     * @author xiaofu
     * @description @PrintlnLog 注解作用的切面方法详细细信息
     * @date 2020/7/15 10:34
     */
    public String getAspectMethodLogDescJP(JoinPoint joinPoint) throws Exception {
        String targetName = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        Object[] arguments = joinPoint.getArgs();
        return getAspectMethodLogDesc(targetName, methodName, arguments);
    }

    /**
     * @param proceedingJoinPoint
     * @author xiaofu
     * @description @PrintlnLog 注解作用的切面方法详细细信息
     * @date 2020/7/15 10:34
     */
    public String getAspectMethodLogDescPJ(ProceedingJoinPoint proceedingJoinPoint) throws Exception {
        String targetName = proceedingJoinPoint.getTarget().getClass().getName();
        String methodName = proceedingJoinPoint.getSignature().getName();
        Object[] arguments = proceedingJoinPoint.getArgs();
        return getAspectMethodLogDesc(targetName, methodName, arguments);
    }

    /**
     * @param targetName
     * @param methodName
     * @param arguments
     * @author xiaofu
     * @description 自定义注解参数
     * @date 2020/7/15 11:51
     */
    public String getAspectMethodLogDesc(String targetName, String methodName, Object[] arguments) throws Exception {
        Class targetClass = Class.forName(targetName);
        Method[] methods = targetClass.getMethods();
        StringBuilder description = new StringBuilder("");
        for (Method method : methods) {
            if (method.getName().equals(methodName)) {
                Class[] clazzs = method.getParameterTypes();
                if (clazzs.length == arguments.length) {
                    description.append(method.getAnnotation(PrintlnLog.class).description());
                    break;
                }
            }
        }
        return description.toString();
    }
}
```

### 三、应用

我们在需要打印入参和返回结果日志的方法，加上`@PrintlnLog`注解，并添加自定义方法描述。

```
@RestController
@RequestMapping
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PrintlnLog(description = "订单详情Controller")
    @RequestMapping("/order")
    public OrderDTO getOrder(OrderVO orderVO, String name) {

        OrderDTO orderInfo = orderService.getOrderInfo(orderVO);

        return orderInfo;
    }
}
```

代码里去掉 `log.info`日志打印，加上 `@PrintlnLog` 看一下效果，清晰明了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPnwzAPGIfufkedNmnrdPAoltVVDOmkLa7d4q2ibas3jz7fUQKPNtrkeP3QQ9g2CHkp5q6n6Fow8pw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Demo  `GitHub`地址：https://github.com/chengxy-nds/Springboot-Notebook/tree/master/springboot-aop-unifiedlog**