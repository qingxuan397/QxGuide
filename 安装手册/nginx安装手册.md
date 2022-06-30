# **安装环境**

nginx是C语言开发，建议在linux上运行，本教程使用Centos6.5作为安装环境。

-  gcc：安装nginx需要先将官网下载的源码进行编译，编译依赖gcc环境，如果没有gcc环境，需要安装gcc：

```shell
# yum install gcc-c++
```

-  pcre：PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。

```shell
# yum install -y pcre pcre-devel
```

注：pcre-devel是使用pcre开发的一个二次开发库。nginx也需要此库。

-  zlib：zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。

```shell
# yum install -y zlib zlib-devel
```

-  openssl：OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。

```shell
# yum install -y openssl openssl-devel
```

 

# **编译安装**

1、下载nginx-1.8.0.tar.gz安装包

```shell
# wget http://nginx.org/download/nginx-1.8.0.tar.gz
```

2、解压，随便解压到什么位置都可，最终的安装是靠configure的--prefix

```shell
# tar -zxvf nginx-1.8.0.tar.gz
# cd nginx-1.8.0
./configure \
--prefix=/opt/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```

 **注意：上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录**

 

3、 编译安装

```shell
# make
# make install
```

4、安装成功查看安装目录

![img](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/wps11.jpg) 

 

5、创建 /opt/nginx/vhost目录并在 /opt/nginx/conf/nginx.conf文件中http块增加以下这行代码

```shell
# mkdir vhost
# vi /opt/nginx/conf/nginx.conf
```

 

```shell
http{

  ...

  include /opt/nginx/vhost/*.conf;

}
```

 

效果如图

![img](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/wps12.jpg) 

 

6、以后就直接在vhost目录下创建各种xx.conf, conf文件内容例子

```shell
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

# **启动nginx**

```shell
# cd /usr/local/nginx/sbin/
# ./nginx
```

查询nginx进程：

```shell
# ps aux|grep nginx
```

 

![img](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/wps13.jpg) 

10402是nginx主进程的进程id，10294是nginx工作进程的进程id

 

注意：执行./nginx启动nginx，这里可以-c指定加载的nginx配置文件，如下：

**./nginx -c /usr/local/nginx/conf/nginx.conf**

**如果不指定-c，nginx在启动时默认加载conf/nginx.conf文件，此文件的地址也可以在编译安装nginx时指定./configure的参数（--conf-path= **指向配置文件（nginx.conf））**

 

**进入/opt/nginx目录**

```shell
# ./sbin/nginx -s reload #重启
# ./sbin/nginx -s stop #关闭
```

 