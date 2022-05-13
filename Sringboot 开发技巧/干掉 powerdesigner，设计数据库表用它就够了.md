最近有个新项目刚过完需求，正式进入数据库表结构设计阶段，公司规定统一用数据建模工具 `PowerDesigner`。但我并不是太爱用这个工具，因为它的功能实在是太多了，显得很臃肿繁琐，而平时设计表用的也就那么几个功能。

这里找到一个好用的工具，马不停蹄的分享给大家，`PDMan`一款**国产**开源的数据库模型建模工具，它的功能`PowerDesigner` 均已经实现，但相比于笨重的`PowerDesigner`来说。`PDMan` 专门用于数据表的设计，界面更加清爽漂亮，功能也十分简洁，没多余的设置很容易上手，还提供了 `Windows`，`Mac`，`Linux` 三个平台版本。

`PDMan`保存的是一个`JSON`文件，使用前得先做点准备工作，配置一下 `JDK` 和 `MySQL` 连接，后边的功能会用到。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9liaxj1xJL8okXt3EcwRYbfCqLtAKStH5SUNo8h1zVYOIk0GvrcoavTw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

下边我们来逐一过下`PDMan` 的功能点。

### 生成数据库文档

`PDMan` 支持一键导出数据表结构的`DDL`执行脚本，`JSON`格式数据，还有数据库表结构文档，其中数据库文档又可以生成 `html` 、`word`、`markdown` 三种格式，文档内容包括各个表的字段属性，数据表间的关系图。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9zQd3Oa9H3TpFINUeMWvoCZaJNuMnJP9fuTlI8Y5XnJjdSziaorxwgqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**html 格式**

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9tGbibCeKMA9J7jaS6CPgtibQ4uTmicA9cGndjG2msibH3kPIx95UCO4WrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)表目录

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9zicmxPegoz5fy5OpLAuSGSbXYd7AONq4FLaicOZsBSGSo7FTALDmj6Qg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)表关系图

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9hLxUM57pFYYE0vKlGQF1mjwdZI7ricHPK4QzcsSFDsVQw4wAO6256cA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)表列清单

**word 格式**

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9t4Hffacyj8G2tPR4HD3HcUMOFiblHfmNy0W8COVK11Ig1hibwRpgMzwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)word 格式

**markdown 格式**

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9swxXCtr9CYiaIyXQLtjz0B1o6n9ZuAfsibZLTVW181jVaHS3uKeUO7sA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)markdown 格式

### 数据库逆向解析

前边我们已经配置了数据库信息，这里直接将已有数据库中的数据表，逆向生成表结构，和数据表间的关系图。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9D5hJ3pUlWI2j4zxJ0CJ7FicuLKEA7CgyPdU1Zrd8vrObQo2I9T6YtTQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 版本控制

`PDMan` 可以控制每次修改的版本，对任意版本间的修改进行比对，和`Git`的版本控制类似。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9Oib9aqgRP6tWXyEjgHLoz9ypXPpzVFRHX4e7lyBiaFlvTwlUjxnHEjcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 全局配置

设置表结构的全局通用字段属性，通常在建表的时候，每个表都会有像创建时间 `create_time`、更新时间`update_time`、删除标记`delete_flag`、乐观锁`revision`这类字段，这样设置完以后在建表时会自动生成。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9arQmwjIg83nffGibfOnu459AT4qbicyxADbNLG7n3l9wgsOEtcZGApww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

还可以自定义数据类型，比如：字符型可以自定义 `varchar(10)`、`varchar(20)`、`varchar(30)`，建字段时直接选择对应数据类型即可，一劳永逸。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aPj1Bictf5OldpYdz2hSiaNx9J0KegxeVZKCByklx2lQ3sWCOxicZIq0EzhxURCAO23r6GIFa0mYicTjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> `PDMan`整体功能非常的简洁，不过也足以应对日常开发中数据库设计需求了。
>
> > “
> >
> > 下载地址：http://www.pdman.cn/
> >
> > ”
>
> 
>
> ![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aN0sXSNoiaIdgDia04G2I1anY7vGagHzjYQrSoPxNdb1xhwzeokkRGJPxw46GQI1DabFfek94j2fnIA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

