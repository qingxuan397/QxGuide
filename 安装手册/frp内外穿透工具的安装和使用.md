**一、安装frps服务端：**

1、新建一个存放目录

```shell
# cd /opt
# mkdir frps
```

2、拉取一键搭建脚本 

```shell
# cd frps
# wget --no-check-certificate https://raw.githubusercontent.com/clangcn/onekey-install-shell/master/frps/install-frps.sh -O ./install-frps.sh
```

3、增加脚本权限

```shell
# chmod 700 ./install-frps.sh
```

4、安装

```shell
# ./install-frps.sh install
```

之后他会让你输一些参数，全部参数都有默认值，直接回车就是输入默认值：

```shell
Please input frps bind_port [1-65535](Default Server Port: 5443): #输入frp提供服务的端口，用于服务器端和客户端通信，默认即可
Please input frps vhost_http_port [1-65535](Default vhost_http_port: 80): #输入frp进行http穿透的http服务端口，建议选择其他端口，默认的80端口给Nignx，然后用Nginx代理frp的http端口
Please input frps vhost_https_port [1-65535](Default vhost_https_port: 443): #输入frp进行https穿透的https服务端口，同上面的80端口类似，建议分配其他端口，然后通过Nginx代理此端口
Please input frps dashboard_port [1-65535](Default dashboard_port: 6443):#输入frp的控制台服务端口，用于查看frp工作状态，默认即可

Please input dashboard_user (Default: admin):#登录控制台的用户名，默认即可
Please input dashboard_pwd (Default: arepR7VZ):#登录控制台的密码，如果记不住默认的建议修改


Please input privilege_token (Default: 9e2UAeWa6hxrwdc):#输入frp服务器和客户端通信的密码，默认是随机生成的，默认即可
Please input frps max_pool_count [1-200](Default max_pool_count: 50):#设置每个代理可以创建的连接池上限，默认50


##### Please select log_level #####
1: info
2: warn
3: error
4: debug
#####################################################
Enter your choice (1, 2, 3, 4 or exit. default [1]): 默认即可
Please input frps log_max_days [1-30](Default log_max_days: 3 day): 默认即可
##### Please select log_file #####
1: enable
2: disable
#####################################################
Enter your choice (1, 2 or exit. default [1]):默认即可

安装完毕后会弹出以下内容，标明了具体信息，到此服务端操作全部完成。
==============================================
You Server IP      : X.X.X.X
Bind port          : 5443
KCP support        : true
vhost http port    : 11000
vhost https port   : 4435
Dashboard port     : 6443
token              : 9e2UAeWa6hxrwdc
tcp_mux            : true
Max Pool count     : 50
Log level          : info
Log max days       : 3
Log file           : enable
==============================================
frps Dashboard     : http://X.X.X.X:6443/
Dashboard user     : admin
Dashboard password : admin
==============================================


frps status manage : frps {start|stop|restart|status|config|version}
Example:
  start: frps start # 启动frps命令
   stop: frps stop # 停止frps命令
restart: frps restart # 重启frps命令

[root@root frps]#
```

5、添加二级域名

```shell
# frps config
在行末尾添加
subdomain_host = qingxuan397.com
```

6、重启frps

```shell
# frps restart
```

至此，安装完成，可以访问ip地址+控制台端口查看（如果是阿里云腾讯云的服务器或者服务器已安装宝塔面板的，记得在安全组放行以上的配置的端口，否则无法访问），见下图

![image-20220602110652155](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602110652155.png)

**二、安装frpc客户端：**

1、下载[frp_0.33.0_windows_amd64.zip](https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_windows_amd64.zip) 并解压至D:\develop\frp_0.20.0_windows_386

2、删除除frpc.ini和frpc.exe外的文件，最终保留下来的文件如图所示

![image-20220602110711481](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602110711481.png)

3、修改frpc.ini

```
[common]
server_addr = 47.75.168.28 #frps服务器地址
server_port = 5443 #frps service端口
token = PLIIGXsnT50TnBnf #frps token

[web_qx]
type = http
local_port = 11091
subdomain = test #效果如test.qingxuan397.com
#custom_domains = test.qingxuan397.com
```

通过在 frps 的配置文件中配置 subdomain_host，就可以启用该特性。之后在 frpc 的 http、https 类型的代理中可以不配置 custom_domains，而是配置一个 subdomain 参数。

只需要将 *.{subdomain_host} 解析到 frps 所在服务器。之后用户可以通过 subdomain 自行指定自己的 web 服务所需要使用的二级域名，通过 {subdomain}.{subdomain_host} 来访问自己的 web 服务。

注意：域名需解析到frps服务器

   ![image-20220602110745384](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602110745384.png)

4、创建frpc启动批处理.bat

```
cd D:\develop\frp_0.20.0_windows_386
d:
start /b frpc -c frpc.ini
```

​         

5、查看端口映射情况

![image-20220602110814577](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602110814577.png)

6、测试

http://test.qingxuan397.com:11000(frps的http端口)就会穿透到启动frpc的11091端口的服务

7、配置域名访问

上面我们发现连接内网还是需要 输入 8085端口，而且域名是nginx上配置的三级域名，其实我们可以通过Nginx来反向代理一下，这样就可以自己的顶级域名，并通过80端口来访问了。

1、Nginx新建一个虚拟主机配置，或者直接改默认的主机配置也可以

在Nignx 的listen 后 增加 default_server 配置，用来监听默认情况下所有来自解析到该服务器的域名

然后在 server 段内增加 反向代理配置

在/opt/nginx/vhost目录下新增一个名为：frps.conf文件

```
vim frps.conf
```

```
# frps内网穿透服务--测试服务为mock服务
server{
    listen 80;
    server_name test.qingxuan397.com;
    location / {
            proxy_redirect off;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://47.75.168.28:11000;
    }
}
```

这样Nginx 将接管域名的绑定工作，监听到80端口的网站后，会将网站转发到 本机的frp 11000端口

配置域名后：

http://test.qingxuan397.com

  ![image-20220602111948871](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602111948871.png)

上面用不了了，下面是新的方式

（Frp第一篇）Frp内网穿透安装教程#Frps服务端一键安装脚本#_sugoods的博客-CSDN博客

- 系统：CentOS7
- 内存：1G
- CPU：单核1G

客户端安装教程：[（Frp第二篇）Frp内网穿透安装教程#Frp OpenWrt客户端安装#图形化管理](https://blog.csdn.net/sugoods/article/details/108839840)

**注意事项：记得给使用的端口开放防火墙，开放防火墙，开放防火墙**

Frps服务端一键配置脚本地址：https://github.com/MvsCode/frps-onekey

脚本有两个源，国外VPS可以用Github的源，国内的VPS建议使用Aliyun的源，要不可能很慢。本教程使用的就是Aliyun的源

```
#下载脚本
wget https://code.aliyun.com/MvsCode/frps-onekey/raw/master/install-frps.sh -O ./install-frps.sh
#设置脚本运行权限
chmod 700 ./install-frps.sh
#执行脚本
./install-frps.sh install
```

1、选择源，1是Aliyun，2是Github。我们选1

![image-20220602112418324](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112418324.png)

2、选择服务端口。默认是5443。这个端口的作用是在客户端连接服务端时是通过这个端口连接的。可以不用修改

![image-20220602112428772](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112428772.png)

3、设置http连接的端口。默认80，没有被占用就默认

![image-20220602112443779](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112443779.png)

4、设置https连接的端口。默认443，没有被占用就默认

![image-20220602112454366](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112454366.png)

5、设置面板的端口。直接用默认端口6443

![image-20220602112504151](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112504151.png)

6、设置登录面板的用户名和密码,根据个人喜好设置

![image-20220602112513900](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112513900.png)

![image-20220602112521234](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112521234.png)

7、设置token。客户端需要填写的

![image-20220602112529989](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112529989.png)

8、设置域名，如果有就填写，没有就默认IP

![image-20220602112541100](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112541100.png)

**其他的配置就默认就好**

安装好之后可以通过 frps config 指令修改或者查看配置。所以忘记了不怕

9、启动服务

```
frps start
```

​        

 

最后，在浏览器中输入 http://ip:6443。如果打不开，请看看是不是VPS的防火墙没有开放6443端口。

如果成功就能看到如下的界面

![image-20220602112604634](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220602112604634.png)

**附录：**

[1.frp中文文档](https://gofrp.org/docs/) 或 https://github.com/fatedier/frp/blob/master/README_zh.md

[2.frp文件下载地址](https://github.com/fatedier/frp/releases)

3.[【frp配置教程】frp内网穿透服务端frps.ini各配置参数详解](http://www.chuantou.org/49.html)

**参考文档：**

1[.大牛的一键安装脚本](https://github.com/clangcn/onekey-install-shell/tree/master/frps)

[2.centos7搭建frps内网穿透服务](https://www.cnblogs.com/longweiqiang/p/11877096.html)

3[.如何安装frp 和使用frp (linux /windows)](https://blog.csdn.net/zxf1242652895/article/details/93472012)