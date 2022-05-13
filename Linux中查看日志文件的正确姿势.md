作为一个后端开发工程师，在Linux中查看查看文件内容是基本操作了。尤其是通常要**分析日志文件排查问题**，那么我们应该如何正确打开日志文件呢？对于笔者这种小菜鸡来说，第一反应就是 cat，tail，vi（或vim）了，是的，我曾经用过好多次vim编辑器来查看日志文件。

**千万不要使用vi命令来查看大文件内容**， 尤其对于那些几十G的大文件。因为vi仅仅是一个编辑器（可以理解为windows中的记事本），使用vi命令后则会把文件所有内容加载到内存中，如果内存不够大的话，则可能会导致服务器瘫痪。

为了生成测试数据，笔者抓心挠肝，东拼西凑，写了一个生成测试文件的shell脚本，方便下文的命令演示，复制到linux命令行执行即可。

```shell
Copy# 生成10行测试数据（可根据需求自行修改）
for ((i=1;i <= 10; i++));
    do 
        echo "第$i行" >> test.txt
        if [[  `expr $i % 2` -eq 0 ]]
        then
            echo -e  >> test.txt
        fi
    done
```

## 直接查看文件内容[#](https://www.cnblogs.com/liqiangchn/p/12343833.html#2139240881)

查看整个文件的内容的命令一共有三个，cat/tac/nl，nl命令笔者用的比较少，所以此处就不再演示了，感兴趣的小伙伴可以去百度一哈。

> cat [-AbEnTv]

**选项与参数**：
**-A** ：相当于-vET的整合， 课列出一些特殊字符而不是空白而已
**-b** ：列出行号，进针对非空白行做行号显示，空白行不会标记
**-E** ：将结尾换行符$显示出来
**-n** ：打印出行号，连同空白行也会有行号，与-b的选项不同
**-T** : 将[tab]键以^I显示出来
**-v** : 列出一些看不出来的特殊字符

**范例1**：查看test.txt文件的内容

```
cat test.txt
```

[![img](http://source.mycookies.cn/202002212119_188.png?ERROR)](http://source.mycookies.cn/202002212119_188.png?ERROR)

**范例2**：查看test.txt文件的内容， 并展示行号

```
cat -n test.txt
```

[![img](http://source.mycookies.cn/202002212203_865.png?ERROR)](http://source.mycookies.cn/202002212203_865.png?ERROR)

**范例3**: 不推荐使用cat查看大文件

```
cat -n test.txt
```

[![img](http://source.mycookies.cn/202002212119_209.gif?ERROR)](http://source.mycookies.cn/202002212119_209.gif?ERROR)

**cat仅仅适合查看行数比较少的文件**， 如果文件比较大则没有什么意义了，文件会快速翻到最后一行。如果文件中有特殊符号，比如[Tab], 换行等要显示出来，就必须加上-A之类的选项。当然cat也可以通过管道符配合more或less使用也可以达到比较好的效果。

> tac（和cat打印顺序相反）

tac学过之后笔者从来没有实际应用过，由于用的比较少，所以大家知道就行了。不过这个命令比较有意思，和cat拼写相反，所以他们的打印顺序也相反，将最后一行作为第一行输出。
**范例1**：倒叙查看文件的内容

[![img](http://source.mycookies.cn/202002212125_329.gif?ERROR)](http://source.mycookies.cn/202002212125_329.gif?ERROR)

## 翻页查看[#](https://www.cnblogs.com/liqiangchn/p/12343833.html#189754086)

> more

more命令了解一下就行了，功能太少，笔者一般都用less命令。

**按键/命令**
**空格键（Space）**：向下翻页
**回车（Enter）**：向下翻行
**/字符串**：在当前显示的内容（翻页进度位置），向下查找这个字符串关键字
**:f**：立刻显示文件名以及目前位置的行号
**q:** ：退出当前文件的浏览
**b或ctrl+b**：往回翻页
**范例1**：翻行后，查看行号

[![img](http://source.mycookies.cn/202002212126_572.gif?ERROR)](http://source.mycookies.cn/202002212126_572.gif?ERROR)

> less

less命令比more更加有弹性，可以前后翻页，不止可以向上查找，也可以向下查找。
**按键/命令**
**[pagedown]** ：向下翻页
**[pageup]** ：向上翻页
**/字符串**：在当前显示的内容（翻页进度位置），向下查找这个字符串关键字
**?字符串**：向上查找字符串
**n** ：重复前一个查找，与/或?有关， 比如前一个命令是？表示向上查找，此时n会向上查找
**N**: 反向的重复前一个查找
**g** ：跳转到当前文件数据的第一行
**G** ：跳转到当前文件数据的最后一行
**q** ：退出当前文件的浏览

**范例演示**

[![img](http://source.mycookies.cn/202002212126_323.gif?ERROR)](http://source.mycookies.cn/202002212126_323.gif?ERROR)

## 数据截取[#](https://www.cnblogs.com/liqiangchn/p/12343833.html#1361124996)

> head

head命令用来提取文件的前n行，一般配合使用-n选项。当指定的行数为负数-x时，则会打印出除了后面x行的其他所有数据。
**范例1**：查看前10行数据

[![img](http://source.mycookies.cn/202002212132_459.png?ERROR)](http://source.mycookies.cn/202002212132_459.png?ERROR)


**范例2（一共10000行，没有空行）**：head -n -9989 test.txt



[![img](http://source.mycookies.cn/202002212130_584.png?ERROR)](http://source.mycookies.cn/202002212130_584.png?ERROR)



> tail

从文件尾部截取数据。tail也是工作中最常用的命令，因为可以利用-f选项，一直刷新获取文件尾部最新数据。

**选项与参数**
**-n** ： 查看后n行数据，注意当n后面值**带“+”号表示从第x行开始**， 如 `tail -n +1000 test.txt`
**-f** : 展示文件后面
**范例1**：查看尾部5行数据【tail -n 5 test.txt】

[![img](http://source.mycookies.cn/202002212129_251.png?ERROR)](http://source.mycookies.cn/202002212129_251.png?ERROR)

**范例2**：查看文件尾部数据，并实时刷新数据

[![img](http://source.mycookies.cn/202002212131_445.gif?ERROR)](http://source.mycookies.cn/202002212131_445.gif?ERROR)

**范例3**：查看文件尾部5行数据，并实时刷新数据

[![img](http://source.mycookies.cn/202002212130_863.gif?ERROR)](http://source.mycookies.cn/202002212130_863.gif?ERROR)

## 通用命令[#](https://www.cnblogs.com/liqiangchn/p/12343833.html#1098851186)

**管道**：Shell 还有一种功能，就是可以将两个或者多个命令（程序或者进程）连接到一起**，把一个命令的输出作为下一个命令的输入**，以这种方式连接的两个或者多个命令就形成了管道（pipe），管道命令**用"|"来表示**。

**范例**：查看ll命令输出的前10行

```
ll | head -n 3
```

[![img](http://source.mycookies.cn/202002212201_844.png?ERROR)](http://source.mycookies.cn/202002212201_844.png?ERROR)

**grep** ： 命令用于**查找文件里符合条件的字符串**，这两个命令也是linux中最常用的的，而在查看日志文件也通常会结合这两个命令一起使用。

**范例**：查看文件文件中那些行包含‘999’

```
cat -n test.txt | grep '999'
```

[![img](http://source.mycookies.cn/202002212147_381.png?ERROR)](http://source.mycookies.cn/202002212147_381.png?ERROR)

**>>** : **文件追加重定向命令**，可以往文件末尾追加数据，正如上文 `echo "第$i行" >> test.txt`。

**范例**：将一个文件的最后10行复制到helloworld.txt中

```
tail -n 10 >> helloworld.txt
```

[![img](http://source.mycookies.cn/202002212155_9.png?ERROR)](http://source.mycookies.cn/202002212155_9.png?ERROR)

**wc**：**文件字节数，字数，行数查看**`wc [-clw] [文件...]`,
**-c**或--bytes或--chars 只显示Bytes数。
**-l**或--lines 只显示行数。
**-w**或--words 只显示字数。
**范例**：查看文件行数
`wc -l`

## 案例实战[#](https://www.cnblogs.com/liqiangchn/p/12343833.html#2793958320)

**案例1**：打印日志文件中第11到20行。
思路：首先获取前20行，然后在获取20行的后10行即可，需要使用管道命令。
`head -n 20 text.txt | tail -n 10`

[![img](http://source.mycookies.cn/202002212133_273.png?ERROR)](http://source.mycookies.cn/202002212133_273.png?ERROR)


`cat -n test.txt | head -n 20 | tail -n 10`（如果需要行号）



[![img](http://source.mycookies.cn/202002212134_402.png?ERROR)](http://source.mycookies.cn/202002212134_402.png?ERROR)

## 总结[#](https://www.cnblogs.com/liqiangchn/p/12343833.html#3181258861)

Linux的命令实在太多了，对于开发来讲要用到的也有很多，不过笔者认为**首先要知道是否存在相关命令**，然后分**类掌握最常用**的，需要时再**查表**即可。没有必要去纠结命令记不记得住，毕竟这些东西决定不了你的上限。