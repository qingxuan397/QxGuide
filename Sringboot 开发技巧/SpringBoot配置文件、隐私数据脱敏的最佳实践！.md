这几天公司在排查内部数据账号泄漏，原因是发现某些实习生小可爱居然连带着账号、密码将源码私传到`GitHub`上，导致核心数据外漏，孩子还是没挨过社会毒打，这种事的后果可大可小。\

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de05205a8b774996af7d00a7409ad55d~tplv-k3u1fbpfcp-watermark.image)

说起这个我是比较有感触的，之前[我TM被删库](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzAxNTM4NzAyNg%3D%3D%26mid%3D2247493189%26idx%3D1%26sn%3D5e18220de87c5cc29208ccd595327f22%26scene%3D21%23wechat_redirect)的经历，到现在想起来心里还难受，我也是把数据库账号明文密码误提交到`GitHub`，然后被哪个大宝贝给我测试库删了，后边我长记性了把配置文件内容都加密了，数据安全问题真的不容小觑，不管工作汇还是生活，敏感数据一定要做脱敏处理。

如果对脱敏概念不熟悉，可以看一下我之前写过的一篇[大厂也在用的6种数据脱敏方案](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzAxNTM4NzAyNg%3D%3D%26mid%3D2247489848%26idx%3D1%26sn%3Da4163a7c0914bd8e1ec1f29171ab7c92%26scene%3D21%23wechat_redirect)，里边对脱敏做了简单的描述，接下来分享工作中两个比较常见的脱敏场景。

### 配置脱敏

实现配置的脱敏我使用了`Java`的一个加解密工具`Jasypt`，它提供了`单密钥对称加密`和`非对称加密`两种脱敏方式。

单密钥对称加密：一个密钥加盐，可以同时用作内容的加密和解密依据；

非对称加密：使用公钥和私钥两个密钥，才可以对内容加密和解密；

以上两种加密方式使用都非常简单，咱们以`springboot`集成单密钥对称加密方式做示例。

首先引入`jasypt-spring-boot-starter` jar

```
 <!--配置文件加密-->
 <dependency>
     <groupId>com.github.ulisesbocchio</groupId>
     <artifactId>jasypt-spring-boot-starter</artifactId>
     <version>2.1.0</version>
 </dependency>
复制代码
```

配置文件加入秘钥配置项`jasypt.encryptor.password`，并将需要脱敏的`value`值替换成预先经过加密的内容`ENC(mVTvp4IddqdaYGqPl9lCQbzM3H/b0B6l)`。

这个格式我们是可以随意定义的，比如想要`abc[mVTvp4IddqdaYGqPl9lCQbzM3H/b0B6l]`格式，只要配置前缀和后缀即可。

```
jasypt:
  encryptor:
    property:
      prefix: "abc["
      suffix: "]"
复制代码
```

ENC(XXX）格式主要为了便于识别该值是否需要解密，如不按照该格式配置，在加载配置项的时候`jasypt`将保持原值，不进行解密。

```
spring:
  datasource:
    url: jdbc:mysql://1.2.3.4:3306/xiaofu?useSSL=false&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&ze oDateTimeBehavior=convertToNull&serverTimezone=Asia/Shanghai
    username: xiaofu
    password: ENC(mVTvp4IddqdaYGqPl9lCQbzM3H/b0B6l)

# 秘钥
jasypt:
  encryptor:
    password: 程序员内点事(然而不支持中文)
复制代码
```

秘钥是个安全性要求比较高的属性，所以一般不建议直接放在项目内，可以通过启动时`-D`参数注入，或者放在配置中心，避免泄露。

```
java -jar -Djasypt.encryptor.password=1123  springboot-jasypt-2.3.3.RELEASE.jar
复制代码
```

预先生成的加密值，可以通过代码内调用API生成

```
@Autowired
private StringEncryptor stringEncryptor;

public void encrypt(String content) {
    String encryptStr = stringEncryptor.encrypt(content);
    System.out.println("加密后的内容：" + encryptStr);
}
复制代码
```

或者通过如下Java命令生成，几个参数`D:\maven_lib\org\jasypt\jasypt\1.9.3\jasypt-1.9.3.jar`为jasypt核心jar包，`input`待加密文本，`password`秘钥，`algorithm`为使用的加密算法。

```
java -cp  D:\maven_lib\org\jasypt\jasypt\1.9.3\jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input="root" password=xiaofu  algorithm=PBEWithMD5AndDES
复制代码
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e396b40477146f69a3d83ac4c3ff70a~tplv-k3u1fbpfcp-watermark.image)

一顿操作后如果还能正常启动，说明配置文件脱敏就没问题了。

### 敏感字段脱敏

生产环境用户的隐私数据，比如手机号、身份证或者一些账号配置等信息，入库时都要进行不落地脱敏，也就是在进入我们系统时就要实时的脱敏处理。

用户数据进入系统，脱敏处理后持久化到数据库，用户查询数据时还要进行反向解密。这种场景一般需要全局处理，那么用`AOP`切面来实现在适合不过了。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3ccf388b31f47a4895f45992b289153~tplv-k3u1fbpfcp-watermark.image)

首先自定义两个注解`@EncryptField`、`@EncryptMethod`分别用在字段属性和方法上，实现思路很简单，只要方法上应用到`@EncryptMethod`注解，则检查入参字段是否标注`@EncryptField`注解，有则将对应字段内容加密。

```
@Documented
@Target({ElementType.FIELD,ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface EncryptField {

    String[] value() default "";
}
复制代码
@Documented
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface EncryptMethod {

    String type() default ENCRYPT;
}
复制代码
```

切面的实现也比较简单，对入参加密，返回结果解密。为了方便阅读这里就只贴出部分代码，完整案例Github地址：[github.com/chengxy-nds…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fchengxy-nds%2FSpringboot-Notebook%2Ftree%2Fmaster%2Fspringboot-jasypt)

```
@Slf4j
@Aspect
@Component
public class EncryptHandler {

    @Autowired
    private StringEncryptor stringEncryptor;

    @Pointcut("@annotation(com.xiaofu.annotation.EncryptMethod)")
    public void pointCut() {
    }

    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) {
        /**
         * 加密
         */
        encrypt(joinPoint);
        /**
         * 解密
         */
        Object decrypt = decrypt(joinPoint);
        return decrypt;
    }

    public void encrypt(ProceedingJoinPoint joinPoint) {

        try {
            Object[] objects = joinPoint.getArgs();
            if (objects.length != 0) {
                for (Object o : objects) {
                    if (o instanceof String) {
                        encryptValue(o);
                    } else {
                        handler(o, ENCRYPT);
                    }
                    //TODO 其余类型自己看实际情况加
                }
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public Object decrypt(ProceedingJoinPoint joinPoint) {
        Object result = null;
        try {
            Object obj = joinPoint.proceed();
            if (obj != null) {
                if (obj instanceof String) {
                    decryptValue(obj);
                } else {
                    result = handler(obj, DECRYPT);
                }
                //TODO 其余类型自己看实际情况加
            }
        } catch (Throwable e) {
            e.printStackTrace();
        }
        return result;
    }
    。。。
}
复制代码
```

紧接着测试一下切面注解的效果，我们对字段`mobile`、`address`加上注解`@EncryptField`做脱敏处理。

```
@EncryptMethod
@PostMapping(value = "test")
@ResponseBody
public Object testEncrypt(@RequestBody UserVo user, @EncryptField String name) {

    return insertUser(user, name);
}

private UserVo insertUser(UserVo user, String name) {
    System.out.println("加密后的数据：user" + JSON.toJSONString(user));
    return user;
}

@Data
public class UserVo implements Serializable {

    private Long userId;

    @EncryptField
    private String mobile;

    @EncryptField
    private String address;

    private String age;
}
复制代码
```

请求这个接口，看到参数被成功加密，而返回给用户的数据依然是脱敏前的数据，符合我们的预期，那到这简单的脱敏实现就完事了。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e6dea8dced0498ca843580c5f37f08b~tplv-k3u1fbpfcp-watermark.image)

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96bbf0abe71c4992ba9a7fb8b4b6debc~tplv-k3u1fbpfcp-watermark.image)

### 知其然知其所以然

`Jasypt`工具虽然简单好用，但作为程序员我们不能仅满足于熟练使用，底层实现原理还是有必要了解下的，这对后续调试bug、二次开发扩展功能很重要。

个人认为`Jasypt`配置文件脱敏的原理很简单，无非就是在具体使用配置信息之前，先拦截获取配置的操作，将对应的加密配置解密后再使用。

具体是不是如此我们简单看下源码的实现，既然是以`springboot`方式集成，那么就先从`jasypt-spring-boot-starter`源码开始入手。

`starter`代码很少，主要的工作就是通过`SPI`机制注册服务和`@Import`注解来注入需前置处理的类`JasyptSpringBootAutoConfiguration`。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee307d1e0c0b4b35be2de4b5f5957f7e~tplv-k3u1fbpfcp-watermark.image)

在前置加载类`EnableEncryptablePropertiesConfiguration`中注册了一个核心处理类`EnableEncryptablePropertiesBeanFactoryPostProcessor`。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ce993974c464927aeb1b87818436bb7~tplv-k3u1fbpfcp-watermark.image)

它的构造器有两个参数，`ConfigurableEnvironment`用来获取所有配属信息，`EncryptablePropertySourceConverter`对配置信息做解析处理。

顺藤摸瓜发现具体负责解密的处理类`EncryptablePropertySourceWrapper`，它通过对`Spring`属性管理类`PropertySource<T>`做拓展，重写了`getProperty(String name)`方法，在获取配置时，凡是指定格式如**ENC(x)** 包裹的值全部解密处理。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/896ba64c36534dd6b76e1c0cb53c328c~tplv-k3u1fbpfcp-watermark.image)

既然知道了原理那么后续我们二次开发，比如：切换加密算法或者实现自己的脱敏工具就容易的多了。

> “
>
> 案例Github地址：[github.com/chengxy-nds…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fchengxy-nds%2FSpringboot-Notebook%2Ftree%2Fmaster%2Fspringboot-jasypt)

**PBE算法**

再来聊一下`Jasypt`中用的加密算法，其实它是在JDK的`JCE.jar`包基础上做了封装，本质上还是用的JDK提供的算法，默认使用的是`PBE`算法`PBEWITHMD5ANDDES`，看到这个算法命名很有意思，段个句看看，PBE、WITH、MD5、AND、DES 好像有点故事，继续看。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1951d59c50ce45428597f4dbf96edb70~tplv-k3u1fbpfcp-watermark.image)

`PBE`算法（`Password Based Encryption`，基于口令（密码）的加密）是一种基于口令的加密算法，其特点在于口令是由用户自己掌握，在加上随机数多重加密等方法保证数据的安全性。

PBE算法本质上并没有真正构建新的加密、解密算法，而是对我们已知的算法做了包装。比如：常用的消息摘要算法`MD5`和`SHA`算法，对称加密算法`DES`、`RC2`等，而`PBE`算法就是将这些算法进行合理组合，这也呼应上前边算法的名字。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9028c56a98694fb492a8fcc57e4331e3~tplv-k3u1fbpfcp-watermark.image)

既然PBE算法使用我们较为常用的对称加密算法，那就会涉及密钥的问题。但它本身又没有钥的概念，只有口令密码，密钥则是口令经过加密算法计算得来的。

口令本身并不会很长，所以不能用来替代密钥，只用口令很容易通过穷举攻击方式破译，这时候就得加点**盐**了。

盐通常会是一些随机信息，比如随机数、时间戳，将盐附加在口令上，通过算法计算加大破译的难度。

**源码里的猫腻**

简单了解PBE算法，回过头看看`Jasypt`源码是如何实现加解密的。

在加密的时候首先实例化秘钥工厂`SecretKeyFactory`，生成八位盐值，默认使用的`jasypt.encryptor.RandomSaltGenerator`生成器。

```
public byte[] encrypt(byte[] message) {
    // 根据指定算法，初始化秘钥工厂
    final SecretKeyFactory factory = SecretKeyFactory.getInstance(algorithm1);
    // 盐值生成器，只选八位
    byte[] salt = saltGenerator.generateSalt(8);
    // 
    final PBEKeySpec keySpec = new PBEKeySpec(password.toCharArray(), salt, iterations);
    // 盐值、口令生成秘钥
    SecretKey key = factory.generateSecret(keySpec);

    // 构建加密器
    final Cipher cipherEncrypt = Cipher.getInstance(algorithm1);
    cipherEncrypt.init(Cipher.ENCRYPT_MODE, key);
    // 密文头部（盐值）
    byte[] params = cipherEncrypt.getParameters().getEncoded();

    // 调用底层实现加密
    byte[] encryptedMessage = cipherEncrypt.doFinal(message);

    // 组装最终密文内容并分配内存（盐值+密文）
    return ByteBuffer
            .allocate(1 + params.length + encryptedMessage.length)
            .put((byte) params.length)
            .put(params)
            .put(encryptedMessage)
            .array();
}
复制代码
```

由于默认使用的是随机盐值生成器，导致相同**内容每次加密后的内容都是不同的**。

那么解密时该怎么对应上呢？

看上边的源码发现，最终的加密文本是由两部分组成的，`params`消息头里边包含口令和随机生成的盐值，`encryptedMessage`密文。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09b8cd7152174da9be3edb7aaf854dae~tplv-k3u1fbpfcp-watermark.image)

加密

而在解密时会根据密文`encryptedMessage`的内容拆解出`params`内容解析出盐值和口令，在调用JDK底层算法解密出实际内容。

```
@Override
@SneakyThrows
public byte[] decrypt(byte[] encryptedMessage) {
    // 获取密文头部内容
    int paramsLength = Byte.toUnsignedInt(encryptedMessage[0]);
    // 获取密文内容
    int messageLength = encryptedMessage.length - paramsLength - 1;
    byte[] params = new byte[paramsLength];
    byte[] message = new byte[messageLength];
    System.arraycopy(encryptedMessage, 1, params, 0, paramsLength);
    System.arraycopy(encryptedMessage, paramsLength + 1, message, 0, messageLength);

    // 初始化秘钥工厂
    final SecretKeyFactory factory = SecretKeyFactory.getInstance(algorithm1);
    final PBEKeySpec keySpec = new PBEKeySpec(password.toCharArray());
    SecretKey key = factory.generateSecret(keySpec);

    // 构建头部盐值口令参数
    AlgorithmParameters algorithmParameters = AlgorithmParameters.getInstance(algorithm1);
    algorithmParameters.init(params);

    // 构建加密器，调用底层算法
    final Cipher cipherDecrypt = Cipher.getInstance(algorithm1);
    cipherDecrypt.init(
            Cipher.DECRYPT_MODE,
            key,
            algorithmParameters
    );
    return cipherDecrypt.doFinal(message);
}
复制代码
```

![解密](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0292ff45d6764ce7a2c4185c969678f6~tplv-k3u1fbpfcp-watermark.image)