windows版本

Erlang安装包下载地址:https://erlang.org/download/otp_versions_tree.html

RabbitMQ安装包下载地址:https://www.rabbitmq.com/install-windows-manual.html

安装包与Erlang版本关系:https://www.rabbitmq.com/which-erlang.html

安装Erlang：

原因在于RabbitMQ服务端代码是使用并发式语言erlang编写的，下载地址：http://www.erlang.org/downloads，双击.exe文件进行安装就好，安装完成之后创建一个名为ERLANG_HOME的环境变量，其值指向erlang的安装目录，同时将%ERLANG_HOME%\bin加入到Path中，最后打开命令行，输入erl，如果出现erlang的版本信息就表示erlang语言环境安装成功；

![image-20210908171203761](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210908171203761.png)                     



![image-20210908171239949](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210908171239949.png)



安装插件:rabbitmq-plugins.bat enable rabbitmq_management

启动:rabbitmq-server.bat

浏览访问:[http://localhost:15672](http://localhost:15672/)  guest/guest





https://blog.csdn.net/weixin_44356673/article/details/107223182

1. 安装Erlang
1.1 步骤一：下载
下载Erlang

我是windows 64位的,根据自己的情况来
https://www.erlang.org/downloads

百度网盘下载地址：
版本为：Erlang22.3      RabbitMQ3.8.3
链接：https://pan.baidu.com/s/1Vx_5RS153moaRDMtNHzTSA 
提取码：k1ub


1.2 步骤二
以管理员运行安装包



1.3 步骤三
点击next



1.4 步骤四
选择安装路径



1.5 步骤五
等待安装完毕



2. 安装RabbitMQ
官网: https://www.rabbitmq.com/
2.1 步骤一：下载
注意了,安装RabbitMQ电脑用户名不能使用中文,跳进去了。当然，如果你的是中文，也是有解决方案的，具体看后面3目录

下载方式多种 下载地址 https://www.rabbitmq.com/download.html

1.exe方式安装包下载： https://www.rabbitmq.com/install-windows.html

2.解压缩方式安装包下载： http://www.rabbitmq.com/install-windows-manual.html

这里使用exe方式 

2.2 步骤二


2.3 步骤三


2.4 步骤四：安装完毕


2.5 步骤五：
执行如下命令

安装插件
rabbitmq-plugins.bat enable rabbitmq_management
2.6 步骤六
执行如下命令:启动

rabbitmq-server.bat

访问: http://localhost:15672/
2.7 启动成功,登录
默认用户名: guest
默认密码: guest


2.8 哦吼,大功告成


2.9 错误,没有配置Erlang的环境变量
(有的默认安装时自己就配置了,如果你的没有配置,看后面4目录配置环境变量,配置后再来尝试)



2.10 启动方式
以应用方式启动
rabbitmq-server -detached 后台启动
rabbitmq-server 直接启动，如果你关闭窗口或者需要在改窗口使用其他命令时应用就会停止
关闭:rabbitmqctl stop

以服务方式启动（安装完之后在任务管理器中服务一栏能看到RabbtiMq）
rabbitmq-service install 安装服务
rabbitmq-service start 开始服务
rabbitmq-service stop  停止服务
rabbitmq-service enable 使服务有效
rabbitmq-service disable 使服务无效
rabbitmq-service help 帮助
3. 中文用户名解决方案
你也可以看这个博客，单独抽取出去了：https://my.oschina.net/manlu/blog/4276649

需要使用管理员权限的cmd, 进入到sbin目录下
第1步: (删除本来的)
执行 rabbitmq-service.bat remove
第2步: (路径自己定义(D:\rabbitmq\data),不要有中文和空格。这里应该就是表示一个绕过C:\User\用中文用户名\AppData\Roaming\RabbitMQ 这个文件夹，将config文件更改位置)
执行 set RABBITMQ_BASE=D:\rabbitmq\data
第3步: (重新安装)
执行 rabbitmq-service.bat install
第4步: (安装管理插件，就是可视化)
执行 rabbitmq-plugins enable rabbitmq_management


之后你会发现多出这么几个文件，再对比一下你本来C:\User\用户名\AppData\Roaming\RabbitMQ文件夹下的东西，结果是一样的。之后就会使用该文件夹下的了，然后就不会有中文的情况了



3.1 重新启动rabbitmq即可
启动完成访问: http://localhost:15672/

4 配置Erlang环境变量
4.1 步骤一


4.2 步骤二


4.3 步骤三


4.4 步骤四


4.5 步骤五
安装路径,点击浏览目录选择即可,避免输入错误



4.6 步骤六


4.7 步骤七
复制内容,避免出错

%ERLANG_HOME%\bin



4.8 测试是否OK
打开cmd输入erl。和我效果一样表示OK



5. 配置RabbitMQ环境变量
5.1 步骤一
打开环境变量的步骤我就不截图了

5.2 步骤二


5.3 步骤三


5.4 步骤四


5.5 步骤五
复制即可,避免敲错

%RABBITMQ_SERVER%\sbin

————————————————
版权声明：本文为CSDN博主「兮赫」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_44356673/article/details/107223182
