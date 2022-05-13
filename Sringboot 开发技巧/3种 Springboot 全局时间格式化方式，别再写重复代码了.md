## 3种 Springboot 全局时间格式化方式，别再写重复代码了

原创 程序员内点事 [程序员内点事](javascript:void(0);) *2020-09-01*

时间格式化在项目中使用频率是非常高的，当我们的 `API` 接口返回结果，需要对其中某一个 `date` 字段属性进行特殊的格式化处理，通常会用到 `SimpleDateFormat` 工具处理。

```
SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
Date stationTime = dateFormat.parse(dateFormat.format(PayEndTime()));
```

可一旦处理的地方较多，不仅 `CV` 操作频繁，还产生很多重复臃肿的代码，而此时如果能将时间格式统一配置，就可以省下更多时间专注于业务开发了。

可能很多人觉得统一格式化时间很简单啊，像下边这样配置一下就行了，但事实上这种方式只对 `date` 类型生效。

```
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
```

而很多项目中用到的时间和日期`API` 比较混乱， `java.util.Date` 、 `java.util.Calendar` 和 `java.time LocalDateTime` 都存在，所以全局时间格式化必须要同时兼容性新旧 `API`。

------

看看配置全局时间格式化前，接口返回时间字段的格式。

```
@Data
public class OrderDTO {

    private LocalDateTime createTime;

    private Date updateTime;
}
```

很明显不符合页面上的显示要求（**有人抬杠为啥不让前端解析时间，我只能说睡服代码比说服人容易得多~**）

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aMD3GFMzxZk8kOKgFEpN9tMcNeKnpQQrlqRQib69jSQCUjsibNvsHxh9OW5Nviayibe5voMNrVpKYYe3g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)未做任何配置的结果

## 一、@JsonFormat 注解

`@JsonFormat` 注解方式严格意义上不能叫全局时间格式化，应该叫部分格式化，因为`@JsonFormat` 注解需要用在实体类的时间字段上，而只有使用相应的实体类，对应的字段才能进行格式化。

```
@Data
public class OrderDTO {

    @JsonFormat(locale = "zh", timezone = "GMT+8", pattern = "yyyy-MM-dd")
    private LocalDateTime createTime;

    @JsonFormat(locale = "zh", timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    private Date updateTime;
}
```

字段加上 `@JsonFormat` 注解后，`LocalDateTime` 和 `Date` 时间格式化成功。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aMD3GFMzxZk8kOKgFEpN9tMFdLdp1reIKaNHnmQ8mmrpvFs4bxkHT9yoWLnqichb4gxYOC2cHEEDQA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)@JsonFormat 注解格式化

## 二、@JsonComponent 注解（推荐）

这是我个人比较推荐的一种方式，前边看到使用 `@JsonFormat` 注解并不能完全做到全局时间格式化，所以接下来我们使用 `@JsonComponent` 注解自定义一个全局格式化类，分别对 `Date` 和 `LocalDate` 类型做格式化处理。

```
@JsonComponent
public class DateFormatConfig {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    /**
     * @author xiaofu
     * @description date 类型全局时间格式化
     * @date 2020/8/31 18:22
     */
    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilder() {

        return builder -> {
            TimeZone tz = TimeZone.getTimeZone("UTC");
            DateFormat df = new SimpleDateFormat(pattern);
            df.setTimeZone(tz);
            builder.failOnEmptyBeans(false)
                    .failOnUnknownProperties(false)
                    .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                    .dateFormat(df);
        };
    }

    /**
     * @author xiaofu
     * @description LocalDate 类型全局时间格式化
     * @date 2020/8/31 18:22
     */
    @Bean
    public LocalDateTimeSerializer localDateTimeDeserializer() {
        return new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(pattern));
    }

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
        return builder -> builder.serializerByType(LocalDateTime.class, localDateTimeDeserializer());
    }
}
```

看到 `Date` 和 `LocalDate` 两种时间类型格式化成功，此种方式有效。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aMD3GFMzxZk8kOKgFEpN9tMWvBaERoVOtJL9j0jOJWOwiab7oibazATUPVd33HEyaib8A4SGLiaVauSrg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)@JsonComponent 注解处理格式化

但还有个问题，实际开发中如果我有个字段不想用全局格式化设置的时间样式，**想自定义格式怎么办？**

那就需要和 `@JsonFormat` 注解配合使用了。

```
@Data
public class OrderDTO {

    @JsonFormat(locale = "zh", timezone = "GMT+8", pattern = "yyyy-MM-dd")
    private LocalDateTime createTime;

    @JsonFormat(locale = "zh", timezone = "GMT+8", pattern = "yyyy-MM-dd")
    private Date updateTime;
}
```

从结果上我们看到 `@JsonFormat` 注解的优先级比较高，会以 `@JsonFormat` 注解的时间格式为主。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aMD3GFMzxZk8kOKgFEpN9tMZRyZpv1xVy5bqoZlHJcdetU3ATIQ98cfJLibcNxZsXanR8ZeGJDtmMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 三、@Configuration 注解

这种全局配置的实现方式与上边的效果是一样的。

> “
>
> **注意**：在使用此种配置后，字段手动配置`@JsonFormat` 注解将不再生效。
>
> ”

```
@Configuration
public class DateFormatConfig2 {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    public static DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    @Bean
    @Primary
    public ObjectMapper serializingObjectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer());
        javaTimeModule.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer());
        objectMapper.registerModule(javaTimeModule);
        return objectMapper;
    }

    /**
     * @author xiaofu
     * @description Date 时间类型装换
     * @date 2020/9/1 17:25
     */
    @Component
    public class DateSerializer extends JsonSerializer<Date> {
        @Override
        public void serialize(Date date, JsonGenerator gen, SerializerProvider provider) throws IOException {
            String formattedDate = dateFormat.format(date);
            gen.writeString(formattedDate);
        }
    }

    /**
     * @author xiaofu
     * @description Date 时间类型装换
     * @date 2020/9/1 17:25
     */
    @Component
    public class DateDeserializer extends JsonDeserializer<Date> {

        @Override
        public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
            try {
                return dateFormat.parse(jsonParser.getValueAsString());
            } catch (ParseException e) {
                throw new RuntimeException("Could not parse date", e);
            }
        }
    }

    /**
     * @author xiaofu
     * @description LocalDate 时间类型装换
     * @date 2020/9/1 17:25
     */
    public class LocalDateTimeSerializer extends JsonSerializer<LocalDateTime> {
        @Override
        public void serialize(LocalDateTime value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeString(value.format(DateTimeFormatter.ofPattern(pattern)));
        }
    }

    /**
     * @author xiaofu
     * @description LocalDate 时间类型装换
     * @date 2020/9/1 17:25
     */
    public class LocalDateTimeDeserializer extends JsonDeserializer<LocalDateTime> {
        @Override
        public LocalDateTime deserialize(JsonParser p, DeserializationContext deserializationContext) throws IOException {
            return LocalDateTime.parse(p.getValueAsString(), DateTimeFormatter.ofPattern(pattern));
        }
    }
}
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aMD3GFMzxZk8kOKgFEpN9tMrIiaCiaCRYL1sgib9fAXFV7iceLzsN0pica1oyV5KticHo8kxvUBLiaW8xx6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 总结

分享了一个简单却又很实用的 `Springboot` 开发技巧，其实所谓的开发效率，不过是一个又一个开发技巧堆砌而来，聪明的程序员总是能用最少的代码完成任务。

如果对你有用，欢迎 `在看`、`点赞`、`转发` ，您的认可是我最大的动力。