---
title: linux 常见命令汇总
date: 2019-05-03 13:21:59
tags: linux
---
# 记录日期
* date: 获取当前日期
* cal: 获取日历

# 记录文件跳转
* pwd: 列出当前所在的路径
* cd: 更改当前所在目录
* ls: 列出系统文件内容

## 注意
### ls
* 以 .开头的文件名是隐藏文件。这表示 ls 命令不能列出它们。而 ls -a 就可以，而很多配置文件就是以这个形式隐藏在 home 目录下的
* 文件名和目录名是大小写敏感的
* linux 文件名支持空格和标点符号，但标点符号仅允许 '.', '-' 和下划线，此外，不要在文件名中使用 空格，而是用下划线来替代它。
* 常见命令选项:
    -a	--all	列出所有文件，甚至包括文件名以圆点开头的默认会被隐藏的隐藏文件。
    -d	--directory	通常，如果指定了目录名，ls 命令会列出这个目录中的内容，而不是目录本身。 把这个选项与 -l 选项结合使用，可以看到所指定目录的详细信息，而不是目录中的内容。
    -F	--classify	这个选项会在每个所列出的名字后面加上一个指示符。例如，如果名字是 目录名，则会加上一个'/'字符。
    -h	--human-readable	当以长格式列出时，以人们可读的格式，而不是以字节数来显示文件的大小。
    -l		以长格式显示结果。
    -r	--reverse	以相反的顺序来显示结果。通常，ls 命令的输出结果按照字母升序排列。
    -S		命令输出结果按照文件大小来排序。
    -t		按照修改时间来x排序。
    -i  展示文本节点号

# linux 系统中的目录
* /: 根目录
* /bin: 包含系统启动和运行所必须的二进制程序
* /etc: 所有系统层面的配置文件。也包含一些脚本，这些脚本应在系统启动时运行，其中你每个文件都应该是可读的文件
* /home: 系统给每个用户分配一个目录。普通用户只能在自己的目录下写文件
* /lib: 包含核心系统程序所使用的共享库文件
* /opt: 这个/opt 目录被用来安装“可选的”软件。这个主要用来存储可能安装在系统中的商业软件产品
* /tmp: 这个/tmp 目录，是用来存储由各种程序创建的临时文件的地方。一些配置导致系统每次 重新启动时，都会清空这个目录
* /usr: 在 Linux 系统中，/usr 目录可能是最大的一个。它包含普通用户所需要的所有程序和文件
* /usr/local: 这个/usr/local 目录，是非系统发行版自带程序的安装目录。 通常，由源码编译的程序会安装在/usr/local/bin 目录下。新安装的 Linux 系统中会存在这个目录， 并且在管理员安装程序之前，这个目录是空的。
* /var: 除了/tmp 和/home 目录之外，相对来说，目前我们看到的目录是静态的，这是说， 它们的内容不会改变。/var 目录存放的是动态文件。各种数据库，假脱机文件， 用户邮件等等，都位于在这里。


# 文件链接
* ln file link(ln -s file link)
## 硬连接
最初的链接方式，直接连接一个文件，会导致 ls -l 时的连接数增加，不能跨文件系统、不能关联目录
## 符号链接
多个文件名指向的实际是同一个文件，该方法可以很好的管理一些文件的版本问题。
很好的解决了硬连接的问题，但是如果目标文件被删除，那么会产生坏连接。

# 通配符
* \* 匹配任意多个字符（包括零个或一个）
* ?	匹配任意一个字符（不包括零个）
* [characters]	匹配任意一个属于字符集中的字符
* [!characters]	匹配任意一个不是字符集中的字符
* [[:class:]]	匹配任意一个属于指定字符类中的字符

## 字符类含义
*  [:alnum:]	匹配任意一个字母或数字
*  [:alpha:]	匹配任意一个字母
*  [:digit:]	匹配任意一个数字
*  [:lower:]	匹配任意一个小写字母
*  [:upper:]	匹配任意一个大写字母

# 操作文件和目录
* cp — 复制文件和目录
* mv — 移动/重命名文件和目录
* mkdir — 创建目录
* rm — 删除文件和目录

##  cp
* cp item1 item2   把 item1 复制到 item2
* cp item... directionary   复制多个文件到目录下
* 当出现 cp -r dir1 dir2 时，若存在 dir2，则 dir1 及其内容会出现在 dir2 内，否则会创建一个 dir2，其内容与 dir1 相同

### 可选项
*    -a, --archive	复制文件和目录，以及它们的属性，包括所有权和权限。 通常，副本具有用户所操作文件的默认属性。
*    -i, --interactive	在重写已存在文件之前，提示用户确认。如果这个选项不指定， cp 命令会默认重写文件。
*    -r, --recursive	递归地复制目录及目录中的内容。当复制目录时， 需要这个选项（或者-a 选项）
*    -u, --update	当把文件从一个目录复制到另一个目录时，仅复制 目标目录中不存在的文件，或者是文件内容新于目标目录中已经存在的文件。
*    -v, --verbose	显示翔实的命令操作信息

## mv
* mv item1 item2
* mv item... directory
* mv dir1 dir2（同样存在 dir2 是否存在的问题）

## rm
* -i, --interactive	在删除已存在的文件前，提示用户确认信息。 如果不指定这个选项，rm 会默默地删除文件
* -r, --recursive	递归地删除文件，这意味着，如果要删除一个目录，而此目录 又包含子目录，那么子目录也会被删除。要删除一个目录，必须指定这个选项。
* -f, --force	忽视不存在的文件，不显示提示信息。这选项覆盖了“--interactive”选项。
* -v, --verbose	在执行 rm 命令时，显示翔实的操作信息。

# 命令
## 分类
命令分为以下四种：
1. 是一个可执行程序，就像我们所看到的位于目录/usr/bin 中的文件一样。 这一类程序可以是用诸如 C 和 C++语言写成的程序编译的二进制文件, 也可以是由诸如shell，perl，python，ruby等等脚本语言写成的程序 
2. 是一个内建于 shell 自身的命令。bash 支持若干命令，内部叫做 shell 内部命令 (builtins)。例如，cd 命令，就是一个 shell 内部命令
3. 是一个 shell 函数。这些是小规模的 shell 脚本，它们混合到环境变量中。 在后续的章节里，我们将讨论配置环境变量以及书写 shell 函数。但是现在， 仅仅意识到它们的存在就可以了
4. 是一个命令别名。我们可以定义自己的命令，建立在其它命令之上

## 具体
* type   type command 用来分辨该命令是哪种类型
* which  判断该命令的来源（只对可执行程序有效）
* help   获取文档
* --help 用来显示用法信息
* alias
    * alias name='string'  用于生成命令的别名
    * alias 显示目前的 alias
    * unalias foo 解除某个 alias


# 重定向
首先，我们要明白标准输入、标准输出和错误输出的概念，通常而言，标准输入连接在键盘，输出连接显示器，但这是可以改变的。

我们可以通过一些符号来完成重定向。
```
ls -l /usr/bin > ls-output.txt
```
要注意的是， > 会覆盖之前的内容，因此，当我们需要清空某个文件的内容时，可以使用 > a.txt 的方式来解决。而如果我们要做的是跟在后面，那么应当使用 >>。

标准错误重定向使用的符号是 2>, 且必须放在 > 的后面。
```
ls -l /bin/usr 2> ls-error.txt
```
< 符号可以使后面的文件内容作为前面命令的输入

##  管道线
管道线的符号为 |，其作用是使得前一个命令的结果作为下一个命令的输出。
在管道线中，有以下命令经常被使用

* cat  查看文件的内容（也可以将多个文件的内容归一）
* sort 排序
* uniq 报道或忽略重复航（使用 -d 时可以看到重复行）
* wc   统计文件的行数、字数、字节数，使用 -l 可以只统计行数
* grep grep pattern [file...] 所谓 pattern 就是正则
* head / tail -n  打印开头或结尾前几行
* tee  从 Stdin 读取数据，并同时输出到 Stdout 和文件（保证流不中断）

```
ls /usr/bin | tee ls.txt | grep zip
```

# 展开符
## 花括号展开
* {A,B,C} : A,B,C
* {1..5} : 从 1 - 5

## $ 或 `
$(x) 可以让我们把 x 当作变量来读取，也可以
```
file $(ls /usr/bin/* | grep zip)
```

## “”
在双引号中，参数展开、算数表达式展开和命令替换仍然有效（但是禁止花括号展开）。
```
    [me@linuxbox ~]$ echo "$USER $((2+2)) $(cal)"
    me 4    February 2008
    Su Mo Tu We Th Fr Sa
```
此外，空格不会成为分隔符，只有换行符才会进行分割

## ‘
单引号中禁止所有的展开,而转义符也会失去它的含义
```
[me@linuxbox ~]$ echo text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER
text /home/me/ls-output.txt a b foo 4 me
[me@linuxbox ~]$ echo "text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER"
text ~/*.txt   {a,b} foo 4 me
[me@linuxbox ~]$ echo 'text ~/*.txt {a,b} $(echo foo) $((2+2)) $USER'
text ~/*.txt  {a,b} $(echo foo) $((2+2)) $USER
```

# 命令行常见技巧
* ctrl-a : 移动光标到行首
* ctrl-e:  移动光标到行尾
* ctrl-l:  同 clear
* ctrl-k:  剪切从光标位置到行尾的文本
* ctrl-u:  剪切从光标位置到行首的文本
* ctrl-y:  把剪切环中的文本粘贴到光标位置
* ctrl-r:  反向搜索包含该字符串的命令
* ctrl-o:  执行历史列表中的当前项，并移到下一个。如果你想要执行历史列表中一系列的命令，这很方便

# 权限
## 第一个字符含义
* \- : 一个普通的文件
* d : 一个目录
* l : 一个 symbol link
* c : 一个字符设备文件
* b : 一个块设备文件

## 不同权限含义
* r	允许打开并读取文件内容	允许列出目录中的内容，前提是目录必须设置了可执行属性（x）
* w	允许写入文件内容或截断文件。但是不允许对文件进行重命名或删除，重命名或删除是由目录的属性决定的	允许在目录下新建、删除或重命名文件，前提是目录必须设置了可执行属性（x）
* x	允许将文件作为程序来执行，使用脚本语言编写的程序必须设置为可读才能被执行	允许进入目录，例如：cd directory 

## 改变文件权限(chmod)
* 数字表示法: 从 0 开始，八进制的顺序为 rwx
* 符号表示法: u:user, g: group, o:others, a:all
    * u+r user 增加 r 权限
    * a-x 所有人减少 x 权限
    * go=rw g 和 o 权限为读写
* 默认权限设置: unmask, 使用的是八进制掩码(要注意，从 0 开始，一共四位)，且只对当前 shell 有效

## 更改用户身份
* su -l 要求以某用户身份开启窗口，用户为空，则是超管，然后要求输入指定用户的密码
* sudo 以另一个用户的身份执行命令，输入自己的密码

## 改变文件所有者和用户(chown)
```
chown [owner][:[group]] file...
```
* bob	把文件所有者从当前属主更改为用户 bob
* bob:users	把文件所有者改为用户 bob，文件用户组改为用户组 users
* :admins	把文件用户组改为组 admins，文件所有者不变
* bob:	文件所有者改为用户 bob，文件用户组改为用户 bob 登录系统时所属的用户组

## 更改用户密码(passwd)
只输入 passwd 可以更改自己的密码，如果有管理员权限，则可以改其他人的

# 进程
当系统启动的时候，内核先把一些它自己的活动初始化为进程，然后运行一个叫做 init 的程序。init， 依次地，再运行一系列的称为 init 脚本的 shell 脚本（位于/etc），它们可以启动所有的系统服务。 其中许多系统服务以守护（daemon）程序的形式实现，守护程序仅在后台运行，没有任何用户接口(User Interface)。 这样，即使我们没有登录系统，至少系统也在忙于执行一些例行事务。

## ps(报告当前进程快照，使用 x 可以看全部进程)
* TTY: 控制终端
* TIME: 消耗 cpu 时间vim
* CMD: 启动命令
* STAT: R(运行中或准备运行)、S(睡眠，进程没有运行，在等待事件)、D(不可中断睡眠，等待 IO 事件)、T（已停止）、Z（已终止子进程，但是还没有被父进程清空）、< (高优先级进程)、N（低优先级进程）

## top
可以动态的观察系统的状态，以下参数:
* Load Avg: 等待运行的进程并共享CPU的个数，三个值分别是 60s, 5min, 15min 的平均值
* CPUS: us(用户)、sy（系统内核）、ni（低优先级）、ideal（空闲）、wa（等待 IO）

## 后台运行
在命令之后加 & 即为后台运行
如果要终止一个进程，应当使用 ctrl-c,如果只是停止，那么应该用 ctrl-z

## jobs
查看当前活跃任务
参数以 %n 的形式指出
* bg 将某个任务转为后台
* fg 将某个任务转为前台

## 常用信号
* 1  HUP   挂起，可以重新进行初始化
* 2  INT   中断，同 ctrl-c
* 9  KILL  杀死，内核级的处理，不会给程序收尾的机会
* 15 TEAM  终止，kill 的默认信号
* 18 CONT  继续，恢复进程运行
* 19 STOP  停止，停止运行
* 3  QUIT  退出
* 20 TSTP  终端终止，同 ctrl-z

## killall(杀死全部)

## 其他
* pstree: 输出一个树型结构的进程列表(processtree)，这个列表展示了进程间父/子关系
* vmstart: 输出一个系统资源使用快照，包括内存，交换分区和磁盘 I/O,后面可跟数字表明更新时间
* xload: 一个图形化界面程序，画出系统负载随时间变化的图像
* tload: 在终端内画出图像

# shell 环境
shell 在环境中存储了两种基本类型的数据，虽然 bash 几乎无法分辨这些数据的类型。 它们是环境变量和 shell 变量。Shell 变量是 bash 存放的少量数据。剩下的都是 环境变量。除了变量，shell 也存储了一些可编程的数据，即别名和 shell 函数。

## 查看环境变量
* printenv [变量名]
* echo $变量名
* alias: 查看所有 alias 

## shell 启动文件
### 登陆
* /etc/profile	应用于所有用户的全局配置脚本
* ~/.bash_profile	用户个人的启动文件。可以用来扩展或重写全局配置脚本中的设置
* ~/.bash_login	如果文件 ~/.bash_profile 没有找到，bash 会尝试读取这个脚本
* ~/.profile	如果文件 ~/.bash_profile 或文件 ~/.bash_login 都没有找到，bash 会试图读取这个文件。 这是基于 Debian 发行版的默认设置，比方说 Ubuntu

### 非登陆
* /etc/bash.bashrc	应用于所有用户的全局配置文件。
* ~/.bashrc	用户个人的启动文件。可以用来扩展或重写全局配置脚本中的设置

# vim
## 移动光标
```
^:       行首第一个非空字符
$:       本行行末
ctrl-f:  向下翻一页
ctrl-b:  向上翻一页
gg:      全文第一行
G:       全文最后一行
ngg:     全文第 n 行
nH:      当前页面第几行
M:       跳转当前页面中间
nL:      跳转当前页面倒数第几行
```
## 删除
```
(n)dd:      删除以下几行
d$^:        删除光标到行首或行尾
dG:     	从当前行到文件的末尾
d20G:   	从当前行到文件的第20行
J:          合并当前行和下一行
```

## 复制粘贴撤销重做
```
p               粘贴至光标之后
P               粘贴至光标之前
(n)yy           复制到内存中
u               撤销
.               重复上一步操作
```
## 文件相关
```
:w filename     另存为filename
:n              跳转到第n行
:m,nw filename  第m-n行写入文件
:.,$w filename  当前行到最后一行写入文件
:f              显示当前信息，比如文件共几行，光标在第几行
```

##  搜索
```
:/str/          正向搜索
:?str?          逆向搜索
:%s/Line/line/g 替换
```

## 打开多个文件
```
vi file1 file2 file3...   同时开多个
:e file2                  追加打开
:buffers                  查看目前已打开的文件
:buffer n                 切到第 n 个 
```

# 包管理系统
## 大致分类
* Debian Style (.deb)	Debian, Ubuntu, Xandros, Linspire
* Red Hat Style (.rpm)	Fedora, CentOS, Red Hat Enterprise Linux, OpenSUSE, Mandriva, PCLinuxOS

## 查找某个包
* Debian	apt-get update; apt-cache search search_string
* Red Hat	yum search search_string

## 从资源库安装某个包
* Debian	apt-get update; apt-get install package_name
* Red Hat	yum install package_name

## 通过软件包文件安装文件
* Debian	dpkg --install package_file
* Red Hat	rpm -i package_file

## 卸载文件包
* Debian	apt-get remove package_name
* Red Hat	yum erase package_name

## 更新软件包
* Debian	apt-get update; apt-get upgrade
* Red Hat	yum update
* Debian	dpkg --install package_file
* Red Hat	rpm -U package_file

## 查询软件包
### 查询列表
* Debian	dpkg --list
* Red Hat	rpm -qa
### 确认是否安装
* Debian	dpkg --status package_name
* Red Hat	rpm -q package_name
### 确认文件对应的软件包
* Debian	dpkg --search file_name
* Red Hat	rpm -qf file_name


# 网络系统
## ping
ping 用来确认网络通畅，该命令发送一个特殊的网络数据包，为 ICMP ECHO_REQUEST，大多数服务器都会回复它。

## traceroute
追踪连接到某服务器的网络跳转

## netstat
统计各种各样的网络路由，-r检测内核路由

## 网络中传输文件
### ftp
* ftp 通常是是匿名的，用户名为 anonymous，密码随意
* lcd 用于切换本地的路径
* get ... 获取某文件
* bye 退出 ftp
### wget
* 可用于递归下载，可以下载多个文件

## 与远程主机通信
### ssh
用来进行安全连接
OpenSSH 软件包也包含两个程序，它们可以利用 SSH 加密通道在网络间复制文件。第一个，scp（安全复制）被用来复制文件，与熟悉的 cp 程序非常相似。第二个 SSH 文件复制程序是 sftp，顾名思义，它是 ftp 程序的安全替代品。

```
scp remote-sys:document.txt .
```

# 查找文件
## locate
用于在文件数据库中查找数据

## find
* -type d、f、l
* -size 判断大小
* 可以使用逻辑操作符 -and,-or,-not,()（注意转义）

## xargs
这个 xargs 命令会执行一个有趣的函数。它从标准输入接受输入，并把输入转换为一个特定命令的 参数列表。

## stat
* ls 的加强版

# 压缩和归档
## gzip
一种 linux 下非常常见的压缩操作，后缀为 .gz
* -d  解压缩
* -r  递归

## bzip2
类似于 gzip，但是压缩算法不同，舍弃了压缩速度实现了更高层级的压缩，扩展名为 .bz2

## tar
tar 是一种归档命令，它有非常多的模式
* c: creative,创建归档文件
* v: 显示操作过程
* f: 指定归档后的文件名
* g: 以 gzip 进行压缩
* t: 列出归档内容
* x: 提取相关文件
* p: 保留原文件权限

```
tar -zcvf test.tar.gz  test/*    //压缩
tar -ztvf test.tar.gz      // 查看内容
tar -zxvf test.tar.gz       //解压缩
```

## zip
本身既是归档又是压缩，但是用得很少，主要用于和 Windows 交互
zip -r ; unzip

## rsync
用于远程同步（也可以本地同步，但是至少要有一个在本地）

# linux 里的正则
## grep
* i 忽略大小写
* v 不匹配
* l 输出匹配的文件名，而不是内容
* n 打印出对应文件中的行号

## POSIX 字符集
* [:alnum:]	字母数字字符。在 ASCII 中，等价于：[A-Za-z0-9]
* [:word:]	与[:alnum:]相同, 但增加了下划线字符。
* [:alpha:]	字母字符。在 ASCII 中，等价于：[A-Za-z]
* [:blank:]	包含空格和 tab 字符。
* [:digit:]	数字0到9

## BRE
在 BRE 中，( ) { } ? + | 不被认为是元字符，需要手动转义（grep 和 vim 中都是BRE）
find 命令中不是 BRE，也就是说，不需要转义任何东西

# 文本处理
## cat
将多个文件连接，-n 显示行号

## sort
对进行排序（默认是每一行）
* -f: 不区分大小写
* -r: 相反顺序
* -n: 按照数值排序，而不是字符串值

## uniq
* -c: 输出所有重复行，并且显示每行重复的次数
* -d: 只输出重复行
* -u: 只输出独有行，默认
* -f n: 忽略每行开头的 n 个字符

## cut
从文件中抽取一些内容
```
cut -d ':' -f 1 /etc/passwd | head
```
* -d: 代表分隔符是什么，默认是单个 tab
* -f: 抽取第几段
* -c: 抽取定义的 chat_list 文本

## paste
将某一列内容插入进去

## comm
comm 命令产生了三列输出。第一列包含第一个文件独有的文本行；第二列， 文本行是第二列独有的；第三列包含两个文件共有的文本行
* -n: 显示行数

## diff
diff -u 显示两个文件
* -:	    删除行。这一行将会出现在第一个文件中，而不是第二个文件内。
* +:        添加行。这一行将会出现在第二个文件内，而不是第一个文件中。
* !:	    更改行。将会显示某个文本行的两个版本，每个版本会出现在更改组的各自部分。

## tr
运行时编辑，用于转换
-s 则用于挤压内容，消除连续重复的文本
```
echo "lowercase letters" | tr a-z A-Z
LOWERCASE LETTERS

echo "aaabbbccc" | tr -s ab
abccc
```

## sed
用于查找替换
```
echo "front" | sed '2s/front/back/' //将第二行的front替换为back
sed -n '/SUSE/p' distros.txt   // 正则进行查找
SUSE         10.2     12/07/2006
SUSE         11.0     06/19/2008
SUSE         10.3     10/04/2007
SUSE         10.1     05/11/2006
sed -n '/SUSE/!p' distros.txt   // 进行否定
```

# 格式化输出
## nl
打印行号
## fold
限制文件列宽 -w 指定参数

# 编译软件
## ./configure
这个 configure 程序是一个 shell 脚本，由源码树提供。它的工作是分析程序构建环境。大多数源码会设计为可移植的。 也就是说，它被设计成能够在不止一种类 Unix 系统中进行构建。但是为了做到这一点，在建立程序期间，为了适应系统之间的差异， 源码可能需要经过轻微的调整。configure 也会检查是否安装了必要的外部工具和组件。让我们运行 configure 命令。 因为 configure 命令所在的位置不是位于 shell 通常期望程序所呆的地方，我们必须明确地告诉 shell 它的位置，通过 在命令之前加上 ./ 字符，来表明程序位于当前工作目录.
我们看到在我们的源码目录中 configure 命令创建了几个新文件。最重要一个是 Makefile。Makefile 是一个配置文件， 指示 make 程序究竟如何构建程序。没有它，make 程序就不能运行。Makefile 是一个普通文本文件，所以我们能查看它：
## make
make 命令会执行 Makefile 文件，make 命令会保证目标文件是最新的，目标文件必然会新于依赖文件
## make install
打包良好的源码经常包括一个特别的 make 目标文件，叫做 install。这个目标文件将在系统目录中安装最终的产品，以供使用。 通常，这个目录是 /usr/local/bin，为在本地所构建软件的传统安装位置。然而，通常普通用户不能写入该目录，所以我们必须变成超级用户， 来执行安装操作
```
sudo make install
```

# 编写 bash 脚本
## 变量和常量声明
* 变量: title="bac"  // 要注意中间不能有空格
* title1 = ${title}1 //这样可以局部使用变量
* 常量: declare -r TITLE=”Page Title” // 习惯上将常量名大写
## 函数
```
function name {
    commands
    return
}

name () {
    commands
    return
}
```
## 局部变量
local foo
## 流程控制之条件
```
if commands; then
     commands
[elif commands; then
     commands...]
[else
     commands]
fi
```
一个简单的 bash 脚本如下
```
#!/bin/bash
# test-string: evaluate the value of a string
ANSWER=maybe
if [ -z "$ANSWER" ]; then
    echo "There is no answer." >&2
    exit 1
fi
if [ "$ANSWER" = "yes" ]; then
    echo "The answer is YES."
elif [ "$ANSWER" = "no" ]; then
    echo "The answer is NO."
elif [ "$ANSWER" = "maybe" ]; then
    echo "The answer is MAYBE."
else
    echo "The answer is UNKNOWN."
fi
```
更现代的条件版本是 [[]] 和 (())

## read
* read variable : 读取某变量
* 若变量个数少于输入，则多出的都会在最后一个变量上
* 若只有 read，则全部输入在 REPLY 变量上
* 变量 IFS 用来说明用什么分隔参数
```
#!/bin/bash
# read-ifs: read fields from a file
FILE=/etc/passwd
read -p "Enter a user name > " user_name
file_info=$(grep "^$user_name:" $FILE)
if [ -n "$file_info" ]; then
    IFS=":" read user pw uid gid name home shell <<< "$file_info"
    echo "User = '$user'"
    echo "UID = '$uid'"
    echo "GID = '$gid'"
    echo "Full Name = '$name'"
    echo "Home Dir. = '$home'"
    echo "Shell = '$shell'"
else
    echo "No such user '$user_name'" >&2
    exit 1
fi
```

## 循环
while commands; do commands; done
break 和 continue 一样可用
until 和 while 类似，除了条件相反

## case
就是 switch 匹配, ;;& 类似于 switch 的 break
```
case word in
    [pattern [| pattern]...) commands ;;]...
esac
```

```
#!/bin/bash
read -p "enter word > "
case $REPLY in
    [[:alpha:]])        echo "is a single alphabetic character." ;;
    [ABC][0-9])         echo "is A, B, or C followed by a digit." ;;
    ???)                echo "is three characters long." ;;
    *.txt)              echo "is a word ending in '.txt'" ;;
    *)                  echo "is something else." ;;
esac
```

## 访问 bash 脚本参数
* $n: 获取第 n 个参数
* shift: 所有参数向前移动一位
* $0,$*: 拿到所有参数

## for 循环
传统 for 循环是以下的样子,可用使用路径名展开，命令替换或者花括号占考
```
for variable [in words]; do
    commands
done
```
```
for i in distros*.txt; do echo $i; done
```
新 for 循环
```
for (( expression1; expression2; expression3 )); do
    commands
done
```

## 管理空变量展开
### ${parameter:-word}
本方法为一次性，若 parameter 没有设置（例如，不存在）或者为空，展开结果是 word 的值。若 parameter 不为空，则展开结果是 parameter 的值。
### ${parameter:=word}
可以理解为赋默认值
### ${parameter:?word}
若 parameter 没有设置或为空，这种展开导致脚本带有错误退出，并且 word 的内容会发送到标准错误。若 parameter 不为空， 展开结果是 parameter 的值。
### ${parameter:+word}
若 parameter 没有设置或为空，展开结果为空。若 parameter 不为空， 展开结果是 word 的值会替换掉 parameter 的值；然而，parameter 的值不会改变。

## 变量名展开
${!prefix*}、${!prefix@}：列出以 prefix 开头的变量名
### 查看长度
```
${#parameter}
```
展开成由 parameter 所包含的字符串的长度。通常，parameter 是一个字符串；然而，如果 parameter 是 @ 或者是 * 的话， 则展开结果是位置参数的个数。
### ${parameter:offset:length}
进行一个截断，length默认是全部，offset若是负值就倒着算
### 定点清楚
```
${parameter#pattern}、${parameter##pattern}、${parameter%pattern}、${parameter%%pattern}
```
进行清除，分别是前面最短、最长，后面

## 大小写处理
### declare
* declare -u upper
* declare -l lower
### 大小写展开
* ${parameter,,}	把 parameter 的值全部展开成小写字母。
* ${parameter,}	仅仅把 parameter 的第一个字符展开成小写字母。
* ${parameter^^}	把 parameter 的值全部转换成大写字母。
* ${parameter^}	仅仅把 parameter 的第一个字符转换成大写字母（首字母大写）。

## 数组
创建一个数组
```
a[1] = '333'
declare -a a
name=(value1 value2 ...)    // 多赋值
```
${!array[@]}用引号引起来的 @ 格式是最有用的， 因为它能展开成分离的词。
```
[me@linuxbox ~]$ foo=([2]=a [4]=b [6]=c)
[me@linuxbox ~]$ for i in "${foo[@]}"; do echo $i; done
a
b
c
[me@linuxbox ~]$ for i in "${!foo[@]}"; do echo $i; done
2
4
6
```
在数组末尾追加元素: foo+=(d e f)
删除数组使用 unset 命令
关联数组
```
declare -A colors
colors["red"]="#ff0000"
colors["green"]="#00ff00"
colors["blue"]="#0000ff"
```

# 参考资源
* [bash 教程](http://billie66.github.io/TLCL/book/index.html)