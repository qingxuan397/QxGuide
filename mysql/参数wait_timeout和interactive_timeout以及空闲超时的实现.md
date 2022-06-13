###  导读

> 作者：**高鹏（重庆八怪）**
> 原文地址：http://t.cn/Ex3KnFJ
> MySQL版本：**percona 5.7.22**



### 一、参数意思

这里简单解释一下两个参数含义如下：

- interactive_timeout：The number of seconds the server waits for activity on an interactive connection before closing
   it. An interactive client is defined as a client that uses the CLIENT_INTERACTIVE option to mysql_real_connect()
- wait_timeout：The number of seconds the server waits for activity on a noninteractive connection before closing it.
   On thread startup, the session wait_timeout value is initialized from the global wait_timeout value or from the global interactive_timeout value, depending on the type of client (as defined by the CLIENT_INTERACTIVE connect option to mysql_real_connect())

他们都是session/global级别的，简单的说前者用于描述交互式的客户端的空闲超时，后者用于非交互式的客户端的空闲超时，但是这里也揭示了，如果是交互式客户端连接的session那么wait_timeout将被interactive_timeout覆盖掉，换句话说如果是非交互式的客户端连接的session将不会使用interactive_timeout覆盖掉wait_timeout，也就是interactive_timeout没有任何作用了。

### 二、参数内部表示

- interactive_timeout：



```cpp
static Sys_var_ulong Sys_interactive_timeout( vio_io_wait
       "interactive_timeout",
       "The number of seconds the server waits for activity on an interactive "
       "connection before closing it",
       SESSION_VAR(net_interactive_timeout),
       CMD_LINE(REQUIRED_ARG),
       VALID_RANGE(1, LONG_TIMEOUT), DEFAULT(NET_WAIT_TIMEOUT), BLOCK_SIZE(1));
```

- wait_timeout：



```cpp
static Sys_var_ulong Sys_net_wait_timeout(
       "wait_timeout",
       "The number of seconds the server waits for activity on a "
       "connection before closing it",
       SESSION_VAR(net_wait_timeout), CMD_LINE(REQUIRED_ARG),
       VALID_RANGE(1, IF_WIN(INT_MAX32/1000, LONG_TIMEOUT)),
       DEFAULT(NET_WAIT_TIMEOUT), BLOCK_SIZE(1));
```

我们可以看到内部而言参数interactive_timeout表示为net_interactive_timeout，wait_timeout表示为net_wait_timeout。

### 三、interactive_timeout覆盖wait_timeout

实际上这个操作只会在用户登陆的时候才出现函数对应server_mpvio_update_thd，如下：



```php
server_mpvio_update_thd(THD *thd, MPVIO_EXT *mpvio) do_command
{
  thd->max_client_packet_length= mpvio->max_client_packet_length;
  if (mpvio->protocol->has_client_capability(CLIENT_INTERACTIVE)) //这里做判断
    thd->variables.net_wait_timeout= thd->variables.net_interactive_timeout;//这里覆盖
```

这里我们可以明确看到有覆盖操作，并且我们也能看到这里的if条件是如果是CLIENT_INTERACTIVE类型的客户端连接才会做覆盖。

栈帧如下：



```bash
#0  server_mpvio_update_thd (thd=0x7ffe7c012940, mpvio=0x7fffec0f6140) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/auth/sql_authentication.cc:2014
#1  0x0000000000f01787 in acl_authenticate (thd=0x7ffe7c012940, command=COM_CONNECT, extra_port_connection=false)
    at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/auth/sql_authentication.cc:2246
#2  0x0000000001571149 in check_connection (thd=0x7ffe7c012940, extra_port_connection=false)
    at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/sql_connect.cc:1295
#3  0x00000000015712dc in login_connection (thd=0x7ffe7c012940, extra_port_connection=false)
    at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/sql_connect.cc:1352
#4  0x0000000001571bfe in thd_prepare_connection (thd=0x7ffe7c012940, extra_port_connection=false)
    at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/sql_connect.cc:1516
#5  0x000000000170e642 in handle_connection (arg=0x6781c30) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/conn_handler/connection_handler_per_thread.cc:306
```

那么我们这里可以得到一个结论，只在登陆的时候会判断连接是否是交互式的，如果是则覆盖掉参数wait_timeout，但是一旦连接后将不会发生覆盖操作，即便我们再次修改interactive_timeout的值也不会覆盖，后面我们会看到实际上生效的参数只有wait_timeout。

### 四、超时的实现

实际上每次执行任何一个命令都会做一次wait_timeout值的重新检查和赋值给网络read_timeout值。在函数do_command中我们可以发现my_net_set_read_timeout(net, thd->get_wait_timeout());步骤，这个步骤就是将我们的wait_timeout赋值给网络read_timeout值，其中包含片段



```php
  if (net->read_timeout == timeout) //如果read_timeout和wait_timeout相等
    DBUG_VOID_RETURN;//不需要做操作直接return
  net->read_timeout= timeout;//否则进行赋值。
  if (net->vio)
    vio_timeout(net->vio, 0, timeout);//这里会进行net->vio.read_timeout的赋值
```

执行完这个步骤后wait_timeout就生效了，然后就会执行命令，执行完命令后，整个线程会再次回到do_command函数，再做一次my_net_set_read_timeout函数生效其中的wait_timeout参数，中并且堵塞接受命令(后面可以看到是poll实现的)，这个时候wait_timeout就起作用了。整个栈帧如下：



```objectivec
#0  vio_io_wait (vio=0x7ffe7c015520, event=VIO_IO_EVENT_READ, timeout=10000) at /root/mysqlall/percona-server-locks-detail-5.7.22/vio/viosocket.c:1119
#1  0x0000000001e4d5f6 in vio_socket_io_wait (vio=0x7ffe7c015520, event=VIO_IO_EVENT_READ) at /root/mysqlall/percona-server-locks-detail-5.7.22/vio/viosocket.c:116
#2  0x0000000001e4d6d2 in vio_read (vio=0x7ffe7c015520, buf=0x7ffe7c061c10 "\001", size=4) at /root/mysqlall/percona-server-locks-detail-5.7.22/vio/viosocket.c:171
#3  0x00000000014c6ceb in net_read_raw_loop (net=0x7ffe7c028440, count=4) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/net_serv.cc:672
#4  0x00000000014c6ec2 in net_read_packet_header (net=0x7ffe7c028440) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/net_serv.cc:756
#5  0x00000000014c6fcb in net_read_packet (net=0x7ffe7c028440, complen=0x7fffec0c5c58) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/net_serv.cc:822
#6  0x00000000014c715e in my_net_read (net=0x7ffe7c028440) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/net_serv.cc:899
#7  0x00000000014de010 in Protocol_classic::read_packet (this=0x7ffe7c027bf8) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/protocol_classic.cc:808
#8  0x00000000014de514 in Protocol_classic::get_command (this=0x7ffe7c027bf8, com_data=0x7fffec0c5d70, cmd=0x7fffec0c5d98)
    at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/protocol_classic.cc:965
#9  0x00000000015c5699 in do_command (thd=0x7ffe7c0268e0) at /root/mysqlall/percona-server-locks-detail-5.7.22/sql/sql_parse.cc:960
```

最终会调入vio_io_wait函数，如下是其中的部分片段，我们可以清楚看到实际上所谓的空闲超时实际上就是我们的pool实现的。



```cpp
switch ((ret= poll(&pfd, 1, timeout)))  
  {
  case -1:
    /* On error, -1 is returned. */
    break;
  case 0:
    /*
      Set errno to indicate a timeout error.
      (This is not compiled in on WIN32.)
    */
    errno= SOCKET_ETIMEDOUT;
    break;
  default:
    /* Ensure that the requested I/O event has completed. */
    DBUG_ASSERT(pfd.revents & revents);
    break;
  }
```

因此整个步骤就是

- loop
- 做wait_timeout参数检查并且赋值。
- 堵塞接受命令由poll函数实现，通过poll函数的超时参数也实现了空闲等待超时。(如果不发送命令就堵塞在这里)
- 命令来到退出堵塞。
- 再次做wait_timeout参数检查并且赋值。
- 执行命令。
- goto loop

### 五、测试

我这里就用mysql客户端和pymysql进行交互和非交互连接的测试。

- 交互式mysql客户端会话interactive_timeout 参数覆盖wait_timeout参数



```dart
mysql> show variables like 'wait_timeout%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.02 sec)


mysql> show variables like 'interactive_timeout';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| interactive_timeout | 28800 |
+---------------------+-------+
1 row in set (0.01 sec)

mysql> set global interactive_timeout = 20;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
[root@gp1 log]# /mysqldata/mysql3340/bin/mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.22-22-debug-log Source distribution

Copyright (c) 2009-2018 Percona LLC and/or its affiliates
Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like 'interactive_timeout';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| interactive_timeout | 20    |
+---------------------+-------+
1 row in set (0.01 sec)

mysql> show variables like 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 20    |
+---------------+-------+
1 row in set (0.02 sec)
```

- 交互式mysql客户端会话登陆期间修改interactive_timeout不生效，更改wait_timeout生效。



```ruby
mysql> show variables like 'interactive_timeout';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| interactive_timeout | 28800  |
+---------------------+-------+
1 row in set (0.02 sec)

mysql> show variables like 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800  |
+---------------+-------+
1 row in set (0.02 sec)

mysql> set interactive_timeout=5;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800  |
+---------------+-------+
1 row in set (0.01 sec)

mysql> show variables like 'interactive_timeout';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| interactive_timeout | 5     |
+---------------------+-------+
1 row in set (0.02 sec)

等待5秒，并未生效

mysql> select sysdate();
+---------------------+
| sysdate()           |
+---------------------+
| 2019-02-28 17:24:29 |
+---------------------+
1 row in set (0.00 sec)

mysql> set wait_timeout=5;
Query OK, 0 rows affected (0.00 sec)

等待5秒 发现断开了
mysql> show variables like 'wait_timeout';
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    10
```

- 使用python连接非交互式客户端interactive_timeout 参数不会覆盖wait_timeout参数

我们可以简单的写一个python脚本如下：



```python
import socket
import pymysql.cursors
import psutil
import subprocess


mysql_con = {"host":"192.168.99.95","port":3340,"user":"pycon","passwd":"gelc123","db":"test"}

def main():
    sqlwait = "show variables like 'wait_timeout'"
    sqlinter = "show variables like 'interactive_timeout'"
    sql_c_inter = "set global interactive_timeout=10"

    connect = pymysql.Connect(host=mysql_con["host"], port=mysql_con["port"], user=mysql_con["user"],
                              passwd=mysql_con["passwd"], db=mysql_con["db"])
    cursor = connect.cursor()
    ##查看初始值
    cursor.execute(sqlwait)
    ret_wait = cursor.fetchone()
    cursor.execute(sqlinter)
    ret_inter = cursor.fetchone()
    print("before change: {}".format(ret_wait+ret_inter))

    ##更改值
    cursor.execute(sql_c_inter)
    connect.close()##关闭连接

    ##重新登陆开启连接
    connect = pymysql.Connect(host=mysql_con["host"], port=mysql_con["port"], user=mysql_con["user"],
                              passwd=mysql_con["passwd"], db=mysql_con["db"])
    cursor = connect.cursor()

    cursor.execute(sqlwait)
    ret_wait = cursor.fetchone()
    cursor.execute(sqlinter)
    ret_inter = cursor.fetchone()

    print("after change: {}".format(ret_wait+ret_inter))

    ##恢复值
    sql_c_inter = "set global interactive_timeout=28800"
    cursor.execute(sql_c_inter)

    connect.close()#关闭连接

##程序开始
if __name__ == '__main__':
    main()
```

得到的测试结果如下：



```bash
before change: ('wait_timeout', '28800', 'interactive_timeout', '28800')
after change: ('wait_timeout', '28800', 'interactive_timeout', '10')
```

如果是交互是客户端会话的话wait_timeout也应该是10。

### 六、总结

- 内部来讲只有wait_timeout参数会传递到网络层设置，而interactive_timeout参数只会在会话登陆的时候判断是否是交互式客户端会话如果是则进行wait_timeout=interactive_timeout的覆盖，如果不是则不生效的。
- 一旦会话登陆成功如果想要会话级别修改超时参数，不管交互式还是非交互式都是修改wait_timeout(set wait_timeout)参数才会生效。
- 内部实现空闲超时是通过poll函数的超时参数实现的。
- 简单来说interactive_timeout对交互式客户端连接生效，wait_timeout对非交互式客户端连接生效。
- 对于参数生效的过程如下：

1. loop
2. 做wait_timeout参数检查并且赋值。
3. 堵塞接受命令由poll函数实现，通过poll函数的超时参数也实现了空闲等待超时。(如果不发送命令就堵塞在这里)
4. 命令来到退出堵塞。
5. 再次做wait_timeout参数检查并且赋值。
6. 执行命令。
7. goto loop



作者：重庆八怪
链接：https://www.jianshu.com/p/3beba3c8d497
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。