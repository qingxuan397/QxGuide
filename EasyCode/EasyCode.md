[原文地址](https://blog.csdn.net/qq_39720208/article/details/104583761?utm_term=easycode%E7%BB%A7%E6%89%BFswagger&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-0-104583761&spm=3001.4430)

## 简介

EasyCode是基于IntelliJ IDEA开发的代码生成插件，通过自定义生成模板可以完成定制化的 Mapper Service Controller 生成，结合数据库 Comment还可以实现从数据库到 [Swagger](https://so.csdn.net/so/search?q=Swagger&spm=1001.2101.3001.7020) 的一键配置，非常的强大与方便，项目地址：[EasyCode--码云](https://gitee.com/makejava/EasyCode) 这里推荐大家使用

## 安装

和一般的[Idea插件](https://so.csdn.net/so/search?q=Idea插件&spm=1001.2101.3001.7020)安装方式一样，点击 File -> Setting -> Plugins 搜索 EasyCode 点击 Install 安装即可，安装之后需要重启，当然如果是Idea最新的2019.3版本支持插件热安装就不需要重启了。

![image-20191201215718734](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY1NDE4OS8yMDE5MTIvMTY1NDE4OS0yMDE5MTIwMjA4NDMzNjMxMi0xNDA0NzMwODk5LnBuZw?x-oss-process=image/format,png)

## 连接数据库

安装之后需要使用Idea连接数据库，在Idea的右侧有个DataBase选项卡，点击之后选择对应的数据库。这边我使用的是 Mysql 数据库

![image-20191201215907808](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY1NDE4OS8yMDE5MTIvMTY1NDE4OS0yMDE5MTIwMjA4NDMzNTkzOS0zNzY1OTkzMTMucG5n?x-oss-process=image/format,png)

 

配置好连接名称，连接路径，账号密码和数据库测试连接，测试通过后点击OK，就可以成功的连接到数据库，这里Idea的数据库图形化界面做的也挺好的。

![image-20191201220127365](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY1NDE4OS8yMDE5MTIvMTY1NDE4OS0yMDE5MTIwMjA4NDMzNTYyMy05MzE1NjEyMDIucG5n?x-oss-process=image/format,png)

## 配置EasyCode的模板

### 1. 配置作者名称

同样是 File -> Settings -> other Settings 选择 EasyCode 或者直接搜索 EasyCode 进行编辑，首先键入作者名称，这样在生成的类上面就会加上你的名字，时间等信息。

![image-20191201220618628](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY1NDE4OS8yMDE5MTIvMTY1NDE4OS0yMDE5MTIwMjA4NDMzNTI0NC0xMDE1NDIzMTk1LnBuZw?x-oss-process=image/format,png)

### 2. Type Manager 映射类型管理

此页面是用来建立数据库字段类型与Java变量类型关系的，其中已经预先定义好了很多对应关系，但对于 **tinyint((\d+))? unsigned** (无符号的byte)类型却没有进行预定义，如果不进行手动配置，在进行逆向生成的时候会将其映射成 Java Object 类型，所以需要我们进行手动的**添加关联关系**

![image-20191201223534232](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY1NDE4OS8yMDE5MTIvMTY1NDE4OS0yMDE5MTIwMjA4NDMzNDg4MS0xNzYzNjg1NTUzLnBuZw?x-oss-process=image/format,png)

### 3. Template Setting 模板设置

![image-20191201221538903](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTY1NDE4OS8yMDE5MTIvMTY1NDE4OS0yMDE5MTIwMjA4NDMzNDQ4NC0xMTg5NDY3NDQ4LnBuZw?x-oss-process=image/format,png)

这个页面就是我们主要需要配置的页面了，我们可以自己新建一个模板组，也可以直接在原来模板文件的基础上进行修改。这里我已经对原有的模板进行了自定义的修改，保留了 entity.java mapper.java mapper.xml service.java controller.java 去掉了原有的 dao serviceImpl.具体的模板内容如下，需要的朋友可以直接复制修改。

**当然也可以点击配置作者名称页面的导入模板按钮，输入对应的 Token 进行一键替换**由于token只能保持6个小时，所以我就不在这里贴上了。



### 对象说明文档

```
属性
$author 设置中的作者 java.lang.String
$encode 设置的编码 java.lang.String
$modulePath 选中的module路径 java.lang.String
$projectPath 项目绝对路径 java.lang.String

对象
$tableInfo 表对象
    obj 表原始对象 com.intellij.database.model.DasTable
    name 表名（转换后的首字母大写）java.lang.String
    comment 表注释 java.lang.String
    fullColumn 所有列 java.util.List<ColumnInfo>
    pkColumn 主键列 java.util.List<ColumnInfo>
    otherColumn 其他列 java.util.List<ColumnInfo>,除主键以外的列
    savePackageName 保存的包名 java.lang.String
    savePath 保存路径 java.lang.String
    saveModelName 保存的model名称 java.lang.String
columnInfo 列对象
    obj 列原始对象 com.intellij.database.model.DasColumn
    name 列名（首字母小写） java.lang.String
    comment 列注释 java.lang.String
    type 列类型（类型全名） java.lang.String
    shortType 列类型（短类型） java.lang.String
    custom 是否附加列 java.lang.Boolean
    ext 附加字段（Map类型） java.lang.Map<java.lang.String, java.lang.Object>
$tableInfoList java.util.List<TableInfo>所有选中的表
$importList 所有需要导入的包集合 java.util.Set<java.lang.String>

回调
&callback        
	setFileName(String) 设置文件储存名字
    setSavePath(String) 设置文件储存路径，默认使用选中路径

工具
$tool
    firstUpperCase(String name) 首字母大写方法
    firstLowerCase(String name) 首字母小写方法
    getClsNameByFullName(String fullName) 通过包全名获取类名
    getJavaName(String name) 将下划线分割字符串转驼峰命名(属性名)
    getClassName(String name) 将下划线分割字符串转驼峰命名(类名)
    append(Object... objs) 多个数据进行拼接
    newHashSet(Object... objs) 创建一个HashSet对象
    newArrayList(Object... objs) 创建一个ArrayList对象
    newLinkedHashMap() 创建一个LinkedHashMap()对象
    newHashMap() 创建一个HashMap()对象
    getField(Object obj, String fieldName) 获取对象的属性值,可以访问任意修饰符修饰的属性.配合debug方法使用.
    call(Object... objs) 空白执行方法,用于调用某些方法时消除返回值
    debug(Object obj) 调式方法,用于查询对象结构.可查看对象所有属性与public方法
    serial() 随机获取序列化的UID
    service(String serviceName, Object... param)远程服务调用
    parseJson(String) 将字符串转Map对象
    toJson(Object, Boolean) 将对象转json对象，Boolean：是否格式化json，不填时为不格式化。
$time
    currTime(String format) 获取当前时间，指定时间格式（默认：yyyy-MM-dd HH:mm:ss）
$generateService
    run(String, Map<String,Object>) 代码生成服务，参数1：模板名称，参数2：附加参数。
```

