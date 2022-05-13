# IDE Eval Reset无限重置30天

https://zhile.io/2020/11/18/jetbrains-eval-reset-da33a93d.html

##### 0x1. 如何安装

###### 1). 插件市场安装：

- 在`Settings/Preferences...` -> `Plugins` 内手动添加第三方插件仓库地址：`https://plugins.zhile.io`
- 搜索：`IDE Eval Reset`插件进行安装。如果搜索不到请注意是否做好了上一步？网络是否通畅？
- 插件会提示安装成功。

###### 2). 下载安装：

- 点击这个[链接(v2.2.3)](https://plugins.zhile.io/files/ide-eval-resetter-2.2.3-031813.zip)下载插件的`zip`包（macOS可能会自动解压，然后把`zip`包丢进回收站）
- 通常可以直接把`zip`包拖进IDE的窗口来进行插件的安装。如果无法拖动安装，你可以在`Settings/Preferences...` -> `Plugins` 里手动安装插件（`Install Plugin From Disk...`）
- 插件会提示安装成功。

##### 0x2. 如何使用

- 一般来说，在IDE窗口切出去或切回来时（窗口失去/得到焦点）会触发事件，检测是否长时间（`25`天）没有重置，给通知让你选择。（初次安装因为无法获取上次重置时间，会直接给予提示）
- 也可以手动唤出插件的主界面：
  - 如果IDE没有打开项目，在`Welcome`界面点击菜单：`Get Help` -> `Eval Reset`
  - 如果IDE打开了项目，点击菜单：`Help` -> `Eval Reset`
- 唤出的插件主界面中包含了一些显示信息，`2`个按钮，`1`个勾选项：
  - 按钮：`Reload` 用来刷新界面上的显示信息。
  - 按钮：`Reset` 点击会询问是否重置试用信息并**重启IDE**。选择`Yes`则执行重置操作并**重启IDE生效**，选择`No`则什么也不做。（此为手动重置方式）
  - 勾选项：`Auto reset before per restart` 如果勾选了，则自勾选后**每次重启/退出IDE时会自动重置试用信息**，你无需做额外的事情。（此为自动重置方式）

##### 0x3. 如何更新

###### 1). 插件更新机制（推荐）：

- IDE会自行检测其自身和所安装插件的更新并给予提示。如果本插件有更新，你会收到提示看到更新日志，自行选择是否更新。
- 点击IDE的`Check for Updates...` 菜单手动检测IDE和所安装插件的更新。如果本插件有更新，你会收到提示看到更新日志，自行选择是否更新。
- 插件更新可能会需要**重启IDE**。

###### 2). 手动更新：

- 从本页面下载最新的插件`zip`包安装更新。参考本文：`下载安装`小节。
- 插件更新需要**重启IDE**。

##### 0x4. 一些说明

- 本插件默认不会显示其主界面，如果你需要，参考本文：`如何使用`小节。

- 市场付费插件的试用信息也会**一并重置**。

- ```
  MyBatisCodeHelperPro
  ```

  插件有两个版本如下，功能完全相同，安装时须看清楚！

  - [MyBatisCodeHelperPro (Marketplace Edition)](https://plugins.jetbrains.com/plugin/14522-mybatiscodehelperpro-marketplace-edition-)，可重置！
  - ~~[MyBatisCodeHelperPro](https://plugins.jetbrains.com/plugin/9837-mybatiscodehelperpro)，不可重置！~~

- 对于某些付费插件（如: `Iedis 2`, `MinBatis`）来说，你可能需要去取掉`javaagent`配置（如果有）后重启IDE：

  - 如果IDE没有打开项目，在`Welcome`界面点击菜单：`Configure` -> `Edit Custom VM Options...` -> 移除 `-javaagent:` 开头的行。
  - 如果IDE打开了项目，点击菜单：`Help` -> `Edit Custom VM Options...` -> 移除 `-javaagent:` 开头的行。

- 重置需要**重启IDE生效**！

- 重置后并不弹出`Licenses`对话框让你选择输入License或试用，这和之前的重置脚本/插件不同（省去这烦人的一步）。

- 如果长达`25`天不曾有任何重置动作，IDE会有**通知询问**你是否进行重置。

- 如果勾选：`Auto reset before per restart` ，重置是静默无感知的。

- 简单来说：勾选了`Auto reset before per restart`则无需再管，一劳永逸。

# GenerateAllSetter

实际的开发中，可能会经常为某个对象中多个属性进行 `set` 赋值，尽管可以用`BeanUtil.copyProperties()`方式批量赋值，但这种方式有一些弊端，存在属性值覆盖的问题，所以不少场景还是需要手动 `set`。如果一个对象属性太多 `set` 起来也很痛苦，`GenerateAllSetter`可以一键将对象属性都 `set` 出来。

快捷键：`Alt+Enter`![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jvJtowaYibfS8RPl0ZLz6DLCSMRhFIibDscl8wPM2xmpNY6OibPoxtOIrw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)



# Lombok

`Lombok` 插件应该比较熟，它替我们解决了那些繁琐又重复的代码，比如`Setter`、`Getter`、`toString`、`equals`等方法。![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jmOUib6fwHcicmxvaXsjptnxroDZPSRCb1VVc1YicwZfgk0D0MyqLyDuvg/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)



# Maven Helper

`Maven Helper` 是解决`Maven`依赖冲突的利器，可以快速查找项目中的依赖冲突。安装后打开`pom`文件，底部有 `Dependency Analyzer` 视图。显示红色表示存在依赖冲突，点进去直接在包上右键`Exclude`排除，`pom`文件中会做出相应排除包的操作。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jEBlkCMNAe78x4n6myzHarX8OhayUc5jibmsxp6ehAFQyOa5CpQJXZ6w/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)在这里插入图片描述

- Conflicts(冲突)
- All Dependencies as List(列表形式查看所有依赖)
- All Dependencies as Tree(树结构查看所有依赖)，并且这个页面还支持搜索。



# Alibaba Java Coding Guidelines

阿里出品的《Java 开发手册》时下已经成为了很多公司新员工入职必读的手册，前一段阿里发布了《Java 开发手册(泰山版)》， 又一次对`Java`开发规范做了完善。不过，又臭又长的手册背下来是不可能的，但集成到`IDEA`开发工具中就方便很多。

举个栗子：开发手册上不允许用`Executors`去创建线程池，而是通过`ThreadPoolExecutor`的方式。![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jkqwO3cjiaZzx0TU56d7bvJGxqJTLw7cpuSsJbdHL58AWfqRCAD0CGeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)集成插件后会再去使用`Executors`去创建线程池会有如下的提示。![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jRvLKQ56QrrgqHFklu5oWMRWX1nk3mGkGt7u8nmAL1ib3lSKK5jibrwaQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

# MyBatisCodeHelper

MyBatisCodeHelper-Pro是IDEA下的一个插件，功能类似mybatis plugin。

最新版本请使用：[MyBatisCodeHelperPro (Marketplace Edition)](https://plugins.jetbrains.com/plugin/14522-mybatiscodehelperpro-marketplace-edition-/versions) 配合 [激活码](https://zhile.io/jetbrains-paid-plugins-license.html) 使用，完美和谐。

# GsonFormat

`GsonFormat` 个人觉得是一个非常非常实用的插件，它可以将`JSON`字符串自动转换成`Java`实体类。特别是在和其他系统对接时，往往以`JSON`格式传输数据，而我们需要用`Java`实体接收数据入库或者包装转发，如果字段太多一个一个编写那就太麻烦了。

快捷键：`Alt+ S`

![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jn8bSQh4ibJs8jgnzYdibicSr5vibNv0rMpZ9Fibkia8Yyk85sWeFuXEIT54w/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)在这里插入图片描述



# FindBugs



# Properties to YAML Converter

将`Properties` 配置文件一键转换成`YAML` 文件，很实用的一个插件。**「注意：要提前备份原`Properties` 文件」**![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jUatibmXLZDbCrraesJiaib67oJdWhCyCIQ5ibJWqJyiakBHyfuGB3RJhYew/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)



# CodeGlance 

`CodeGlance` 是一款代码编辑区迷你缩放图插件，可以很方便的知道我们方法大致在什么位置。![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aP8BzcrS7KTgLndFGth9F2jHcpnChqm8zqJEiayKxHgC7DbYKzmFBtVdUN9Xd6v4fEZ0y8WGmZb9bg/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)



# Cloud Toolkit alibaba快速部署应用



# VisualVM Launcher

运行java程序的时候启动visualvm，方便查看jvm的情况 比如堆内存大小的分配
某个对象占用了多大的内存，jvm调优必备工具



# IntelliJad

`IntelliJad`是一个Java class文件的反编译工具，需要在 `setting` 中设置本地`Java` `jad.exe`工具的地址。随便找个`Jar`架包选择`class`文件右键`Decompile`，会出现反编译的结果。