## 前提

```shell
# 确定服务器的基础框架是arm架构还是x86架构
[root@iZbp15nk6nlw6o25xej77gZ opt]# uname -m
x86_64
# 确定出你需要的系统版本。
# 以我的为例，是CentOS 7系统。（CentOS 版本是基于Red Hat 版本开发的）
[root@iZbp15nk6nlw6o25xej77gZ opt]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core) 

```



## 下载mysql5.7的安装包

**下载地址：https://dev.mysql.com/downloads/mysql/5.7.html**

![image-20220601142201808](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220601142201808.png)

![image-20220601142146163](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220601142146163.png)



## wget mysql5.7安装包上传到linux服务器

```sh
[root@iZbp15nk6nlw6o25xej77gZ]# cd /opt
[root@iZbp15nk6nlw6o25xej77gZ opt]# wget -i -c https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.38-el7-x86_64.tar
```

## 检查系统是否安装过mysql

```sh
[root@iZbp15nk6nlw6o25xej77gZ opt]# rpm -qa|grep mysql
# 如果系统自带mysql，查询所有mysql 对应的文件夹，全部删除
[root@iZbp15nk6nlw6o25xej77gZ opt]# whereis mysql
mysql: /usr/lib64/mysql /usr/share/mysql
[root@iZbp15nk6nlw6o25xej77gZ opt]# rm -rf /usr/lib64/mysql /usr/share/mysql

[root@iZbp15nk6nlw6o25xej77gZ opt]# find / -name mysql
/etc/selinux/targeted/active/modules/100/mysql
[root@iZbp15nk6nlw6o25xej77gZ opt]# rm -rf /etc/selinux/targeted/active/modules/100/mysql

[root@iZbp15nk6nlw6o25xej77gZ opt]# rm /etc/my.cnf

```



## 卸载CentOS系统自带mariadb

```shell
# 查看一下是否已经安装了mariadb
[root@iZbp15nk6nlw6o25xej77gZ opt]# rpm -qa|grep mariadb
mariadb-libs-5.5.60-1.el7_5.x86_64
# 删除mariadb
[root@iZbp15nk6nlw6o25xej77gZ opt]# rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64



```

## 检查有无安装过mysql 用户组，没有的话创建

```sh
# 检查mysql 用户组是否存在
cat /etc/group | grep mysql 
cat /etc/passwd |grep mysql
# 创建mysql 用户组和用户
groupadd mysql
useradd -r -g mysql mysql
```



## 安装mysql5.7步骤

```shell
[root@iZbp15nk6nlw6o25xej77gZ opt]# tar -xvf mysql-5.7.38-el7-x86_64.tar 
mysql-test-5.7.38-el7-x86_64.tar.gz
mysql-5.7.38-el7-x86_64.tar.gz
[root@iZbp15nk6nlw6o25xej77gZ opt]# ll
total 1517420
drwx--x--x 4 root root       4096 Nov 17  2021 containerd
drwxr-xr-x 2 root root       4096 Oct  7  2021 frps
-rw-r--r-- 1 root root  776489472 Mar 22 01:13 mysql-5.7.38-el7-x86_64.tar
-rw-r--r-- 1 7161 31415 741502789 Mar 22 02:11 mysql-5.7.38-el7-x86_64.tar.gz
-rw-r--r-- 1 7161 31415  34984391 Mar 22 02:09 mysql-test-5.7.38-el7-x86_64.tar.gz
drwxr-xr-x 6 root root       4096 Oct  7  2021 nginx
drwxr-xr-x 9 1001  1001      4096 Oct  7  2021 nginx-1.8.0
-rw-r--r-- 1 root root     832104 Oct  7  2021 nginx-1.8.0.tar.gz
[root@iZbp15nk6nlw6o25xej77gZ opt]# tar -zxvf mysql-5.7.38-el7-x86_64.tar.gz 
mysql-5.7.38-el7-x86_64/bin/myisam_ftdump
mysql-5.7.38-el7-x86_64/bin/myisamchk
mysql-5.7.38-el7-x86_64/bin/myisamlog
mysql-5.7.38-el7-x86_64/bin/myisampack
mysql-5.7.38-el7-x86_64/bin/mysql
mysql-5.7.38-el7-x86_64/bin/mysql_client_test_embedded
mysql-5.7.38-el7-x86_64/bin/mysql_config_editor
mysql-5.7.38-el7-x86_64/bin/mysql_embedded
mysql-5.7.38-el7-x86_64/bin/mysql_install_db
mysql-5.7.38-el7-x86_64/bin/mysql_plugin
...
    
[root@iZbp15nk6nlw6o25xej77gZ opt]# ll
total 1517428
drwx--x--x 4 root root       4096 Nov 17  2021 containerd
drwxr-xr-x 2 root root       4096 Oct  7  2021 frps
drwxr-xr-x 9 root root       4096 May 31 21:25 mysql-5.7.38-el7-x86_64
-rw-r--r-- 1 root root  776489472 Mar 22 01:13 mysql-5.7.38-el7-x86_64.tar
-rw-r--r-- 1 7161 31415 741502789 Mar 22 02:11 mysql-5.7.38-el7-x86_64.tar.gz
-rw-r--r-- 1 7161 31415  34984391 Mar 22 02:09 mysql-test-5.7.38-el7-x86_64.tar.gz
drwxr-xr-x 6 root root       4096 Oct  7  2021 nginx
drwxr-xr-x 9 1001  1001      4096 Oct  7  2021 nginx-1.8.0
-rw-r--r-- 1 root root     832104 Oct  7  2021 nginx-1.8.0.tar.gz
[root@iZbp15nk6nlw6o25xej77gZ opt]# rm -rf mysql-test-5.7.38-el7-x86_64.tar.gz mysql-5.7.38-el7-x86_64.tar
[root@iZbp15nk6nlw6o25xej77gZ opt]# mv mysql-5.7.38-el7-x86_64 mysql5.7
[root@iZbp15nk6nlw6o25xej77gZ opt]# chown -R mysql:mysql /opt/mysql5.7
[root@iZbp15nk6nlw6o25xej77gZ opt]# chmod -R 755 /opt/mysql5.7
[root@iZbp15nk6nlw6o25xej77gZ opt]# cd mysql5.7/bin
[root@iZbp15nk6nlw6o25xej77gZ bin]# ./mysqld --initialize --user=mysql --datadir=/opt/mysql5.7/data --basedir=/opt/mysql5.7
2022-05-31T13:29:49.149233Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2022-05-31T13:29:50.170143Z 0 [Warning] InnoDB: New log files created, LSN=45790
2022-05-31T13:29:50.341187Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2022-05-31T13:29:50.401219Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: bf585347-e0e5-11ec-957b-00163e0c0071.
2022-05-31T13:29:50.402673Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2022-05-31T13:29:50.623910Z 0 [Warning] A deprecated TLS version TLSv1 is enabled. Please use TLSv1.2 or higher.
2022-05-31T13:29:50.623925Z 0 [Warning] A deprecated TLS version TLSv1.1 is enabled. Please use TLSv1.2 or higher.
2022-05-31T13:29:50.624504Z 0 [Warning] CA certificate ca.pem is self signed.
2022-05-31T13:29:50.732265Z 1 [Note] A temporary password is generated for root@localhost: osPFiar)d4eX    

    
[root@iZbp15nk6nlw6o25xej77gZ opt]# vi /etc/my.cnf
[mysqld]
datadir=/xz/mysql5.7/data
port = 3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
symbolic-links=0
max_connections=400
innodb_file_per_table=1
#表名大小写不明感，敏感为
lower_case_table_names=1


[root@iZbp15nk6nlw6o25xej77gZ opt]# chmod -R 775 /etc/my.cnf
# 修改/opt/mysql5.7/support-files/mysql.server文件，如下图中5个位置的/usr/local/mysql全部修改成/opt/mysql5.7。因为lz没有安装下/usr/local/mysq目录下，所以需要修改成lz安装的/opt/mysql5.7目录。
[root@iZbp15nk6nlw6o25xej77gZ opt]# vi /opt/mysql5.7/support-files/mysql.server        
   
    


```

![在这里插入图片描述](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/707c9150af394e23be771d0c6bc0752e.png)



**启动mysql 服务器**

```shell

# 查询服务
[root@iZbp15nk6nlw6o25xej77gZ opt]# ps -ef|grep mysql
root     10712  6064  0 21:59 pts/0    00:00:00 grep --color=auto mysql
[root@iZbp15nk6nlw6o25xej77gZ opt]# ps -ef|grep mysqld
root     10720  6064  0 21:59 pts/0    00:00:00 grep --color=auto mysqld

# 启动服务
[root@iZbp15nk6nlw6o25xej77gZ opt]# /opt/mysql5.7/support-files/mysql.server start
Starting MySQL.Logging to '/opt/mysql5.7/data/iZbp15nk6nlw6o25xej77gZ.err'.
                                                           [  OK  ]
# 添加软连接                                                           
[root@iZbp15nk6nlw6o25xej77gZ opt]# ln -s /opt/mysql5.7/support-files/mysql.server /etc/init.d/mysql
[root@iZbp15nk6nlw6o25xej77gZ opt]# ln -s /opt/mysql5.7/bin/mysql /usr/bin/mysql
# 重启mysql服务
[root@iZbp15nk6nlw6o25xej77gZ opt]# service mysql restart
Shutting down MySQL..                                      [  OK  ]
Starting MySQL.                                            [  OK  ]

# 登录mysql ，密码就是初始化时生成的临时密码
[root@iZbp15nk6nlw6o25xej77gZ support-files]#  mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.38

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

# 修改密码为root
mysql> set password for root@localhost = password('root');
Query OK, 0 rows affected, 1 warning (0.00 sec)

# 开放远程连接
mysql> use mysql;
mysql> update user set user.Host='%' where user.User='root';
mysql> flush privileges;
mysql> exit;

# 设置开机自启
# 将服务文件拷贝到init.d下，并重命名为mysql
[root@iZbp15nk6nlw6o25xej77gZ support-files]# cp /opt/mysql5.7/support-files/mysql.server /etc/init.d/mysqld
# 赋予可执行权限
[root@iZbp15nk6nlw6o25xej77gZ support-files]# chmod +x /etc/init.d/mysqld
# 添加服务
[root@iZbp15nk6nlw6o25xej77gZ support-files]# chkconfig --add mysqld
# 显示服务列表
[root@iZbp15nk6nlw6o25xej77gZ support-files]# chkconfig --list

Note: This output shows SysV services only and does not include native
      systemd services. SysV configuration data might be overridden by native
      systemd configuration.

      If you want to list systemd services use 'systemctl list-unit-files'.
      To see services enabled on particular target use
      'systemctl list-dependencies [target]'.

frps           	0:off	1:off	2:on	3:on	4:on	5:on	6:off
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
netconsole     	0:off	1:off	2:off	3:off	4:off	5:off	6:off
network        	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@iZbp15nk6nlw6o25xej77gZ support-files]# 

# 开放3306端口，测试本地客户端是否连接成功
# 开放3306端口命令
firewall-cmd --zone=public --add-port=3306/tcp --permanent
# 配置立即生效
firewall-cmd --reload
```

