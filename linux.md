# 基本指令



## ls

ls [options] [路径] 如果没路径  就代表显示当前所在的文件
*  -a   显示指定目录下所有子目录与文件，包括隐藏文件    
* -l   以列表方式显示文件的详细信息    
* -h   配合 -l 以人性化的方式显示文件大小    
* -d   当前目录的属性 

 

### 通配符

| **通配符** | **含义**                                                     |
| :--------- | ------------------------------------------------------------ |
| ls a.?     | 只找只有3个字符，前2字符为a.，最后一个字符任意的文件         |
| []         | [”和“]”将字符组括起来，表示可以匹配字符组中的任意一个。“-”用于表示字符范围 |
| [abc]      | 匹配a、b、c中的任意一个                                      |
| [a-f]      | 匹配从a到f范围内的的任意一个字符                             |



## cp



| 选项 | **含义**                                                     |
| ---- | :----------------------------------------------------------- |
| -a   | 该选项通常在复制目录时使用，它保留链接、文件属性，并递归地复制目录，简单而言，保持文件原有属性。 |
| -n   | 已经存在的目标文件而不提示                                   |
| -i   | 交互式复制，在覆盖目标文件之前将给出提示要求用户确认         |
| -r   | 目标文件必须为一个目录名。                                   |





## mv 移动或重命名



| **选项** | **含义**                               |
| -------- | -------------------------------------- |
| -f       | 禁止交互式操作，如有覆盖也不会给出提示 |
| -v       | 显示移动进度                           |

   

 

## ln 建立链接文件



* 软链接     ln -s 源文件 链接文件
  * 不占用磁盘空间，源文件删除则软链接失效
  * ==软链接必须用绝对路径==

* 硬链接    ln  源文件 链接文件
  * 只能链接普通文件，不能链接目录



 

## find



| **命令**                 | **含义**                              |
| ------------------------ | ------------------------------------- |
| find  -perm 777          | 查找当前目录下权限为 777 的文件或目录 |
| find -size +4k -size -5M | 查找当前目录下大于4k，小于5M的文件    |
| find  [A-Z]*             | 查找当前目录下所有以字母开头的文件    |



## grep



| 选项 | 含义                               |
| ---- | ---------------------------------- |
| -v   | 显示不包含匹配文本的所有行（求反） |
| -n   | 显示匹配行及行号                   |
| -i   | 忽略大小写                         |



### 正则

| **参数**     | **含义**                                                     |
| ------------ | ------------------------------------------------------------ |
| ^a           | 行首,搜寻以 a 开头的行；grep -n '^a' 1.txt                   |
| ke$          | 行尾,搜寻以 ke 结束的行；grep -n 'ke$' 1.txt                 |
| [Ss]igna[Ll] | 匹配 [] 里中一系列字符中的一个；搜寻匹配单词signal、signaL、Signal、SignaL的行；grep -n '[Ss]igna[Ll]' 1.txt |
| .            | (点)匹配一个非换行符的字符；匹配 e 和 e 之间有任意一个字符，可以匹配 eee，eae，eve，但是不匹配 ee，eaae；grep -n  'e.e' 1.txt |



## tar



![img](image.assets/clip_image002-1603294307557.jpg)

 

==f 在最后，其余随意==



## 后台运行jar包



```
java -jar xxx.jar &
```

&代表在后台运行	**当前ssh窗口关闭时，程序中止运行**

 

```
nohup java -jar xxx.jar &
nohup java -jar xxx.jar >/dev/null  &  //输出重定向
```

nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行

缺省情况下该作业的所有输出被重定向到nohup.out的文件中



jobs命令查看后台运行任务	每个作业前面都有个编号

```
jobs
```

如果想将某个作业调回前台控制，只需要 fg + 编号即可。

```
fg 23
```





## 端口



查看端口占用的pid

```shell
#根据进程pid查端口
lsof -i|grep 进程号
netstat -nap |grep 进程号

#根据端口查进程pid
lsof -i:端口
netstat -nap |grep 端口

#根据用户查看进程和端口号
lsof -i|grep user
```



**netstat无权限控制，lsof有权限控制，只能看到本用户**
**losf能看到pid和用户，可以找到哪个进程占用了这个端口**



netstat

- -t : 指明显示TCP端口
- -u : 指明显示UDP端口
- -l : 仅显示监听套接字(LISTEN状态的套接字)
- -p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序
- -n : 不进行DNS解析
- -a 显示所有连接的端口







# 用户/文件管理



* 查看当前用户：whoami

* 退出登录账户： exit
  * 如果是切换后的登陆用户，退出则返回上一个登陆账号。

* 添加用户账号：useradd

| **参数** | **含义**                                                     |
| -------- | ------------------------------------------------------------ |
| -d       | 指定用户登录系统时的主目录，如果不使用该参数，系统自动在/home目录下建立与用户名同名目录为主目录 |
| -m       | 自动建立目录                                                 |
| -g       | 指定组名称                                                   |



* 设置用户密码：passwd

  

## 切换用户：su

| **命令**      | **含义**                                   |
| ------------- | ------------------------------------------ |
| su            | 切换到root用户                             |
| su root       | 切换到root用户                             |
| su -          | 切换到root用户，同时切换目录到/root        |
| su - root     | 切换到root用户，同时切换目录到/root        |
| su 普通用户   | 切换到普通用户                             |
| su - 普通用户 | 切换到普通用户，同时切换普通用户所在的目录 |

 

* 删除用户：userdel

| **命令**                | **含义**                                |
| ----------------------- | --------------------------------------- |
| userdel abc(用户名)     | 删除abc用户，但不会自动删除用户的主目录 |
| userdel -r  abc(用户名) | 删除用户，同时删除用户的主目录          |



* 查看用户组	cat /etc/group

* 修改文件所有者：chown



## 修改文件权限：chmod



| **[ u/g/o/a ]** | **含义**                                                  |
| --------------- | --------------------------------------------------------- |
| u               | user 表示该文件的所有者                                   |
| g               | group 表示与该文件的所有者属于同一组( group )者，即用户组 |
| o               | other 表示其他以外的人                                    |
| a               | all 表示这三者皆是                                        |

 

| **[ +-= ]** | **含义** |
| ----------- | -------- |
| +           | 增加权限 |
| -           | 撤销权限 |
| =           | 设定权限 |

 

| **字母** | **说明**        |
| -------- | --------------- |
| r        | 4               |
| w        | 2               |
| x        | 1               |
| -        | 0  不具任何权限 |





# 系统管理



## ps 

查看进程信息

| **选项** | **含义**                                 |
| -------- | ---------------------------------------- |
| -a       | 显示终端上的所有进程，包括其他用户的进程 |
| -u       | 显示进程的详细状态                       |
| -x       | 显示没有控制终端的进程                   |
| -r       | 只显示正在运行的进程                     |

 

## top



| **按键** | **含义**                           |
| -------- | ---------------------------------- |
| M        | 根据内存使用量来排序               |
| P        | 根据CPU占有率来排序                |
| T        | 根据进程运行时间的长短来排序       |
| U        | 可以根据后面输入的用户名来筛选进程 |
| K        | 可以根据后面输入的PID来杀死进程。  |
| q        | 退出                               |
| d        | 显示信息更新的时间间隔             |



## 关机重启



| **命令**           | **含义**                   |
| ------------------ | -------------------------- |
| reboot             | 重启                       |
| shutdown –r  now   | 重启，会提示其他用户       |
| shutdown -h now    | 立刻关机，now相当于时间为0 |
| shutdown -h  20:25 | 指定时间关机               |
| shutdown -h  +10   | 过十分钟关机               |
| init 0             | 关机                       |
| init 6             | 重启                       |



## df 磁盘占用



| **选项** | **含义**                             |
| -------- | ------------------------------------ |
| -a       | 显示所有                             |
| -m       | 以MB为单位                           |
| -t       | 显示各指定文件系统的磁盘空间使用情况 |
| -T       | 显示文件系统                         |



# Vim



## 插入

| 命令           | 作用                     |
| -------------- | ------------------------ |
| a              | 在光标后附加文本         |
| A（shift + a） | 在本行行末附加文本  行尾 |
| i              | 在光标前插入文本         |
| I(shift+i)     | 在本行开始插入文本  行首 |
| o              | 在光标下插入新行         |
| O(shift+o)     | 在光标上插入新行         |



## 定位



| 命令     | 作用                 |
| -------- | -------------------- |
| :set nu  | 设置行号             |
| :setnonu | 取消行号             |
| gg  G    | 到第一行  到最后一行 |
| nG       | 到第n行              |
| :n       | 到第n行              |
| $        | 移至行尾             |
| 0        | 移至行首             |
|          |                      |
|          |                      |

 



## 删除

| 命令    | 作用                                           |
| ------- | ---------------------------------------------- |
| x       | 删除光标所在处字符  nx 删除光标所在处后n个字符 |
| dd      | 删除光标所在行，ndd删除n行                     |
| :n1,n2d | 删除指定范围的行（eg  :1,3d  删除了123这三行） |
| dG      | 删除光标所在行到末尾的内容                     |
| D       | 删除从光标所在处到行尾                         |



## 复制/剪切



| 命令    | 作用                          |
| ------- | ----------------------------- |
| yy、Y   | 复制当前行                    |
| nyy、nY | 复制当前行以下n行             |
| dd      | 剪切当前行                    |
| ndd     | 剪切当前行以下n行             |
| p、P    | 粘贴在当前光标所在行下 或行上 |

 

## 替换/取消

| 命令         | 作用                                |
| ------------ | ----------------------------------- |
| r            | 取代光标所在处字符                  |
| R(shift + r) | 从光标所在处开始替换字符，按Esc结束 |
| u            | undo,取消上一步操作                 |



## 搜索/替换



| 命令              | 作用                        |
| ----------------- | --------------------------- |
| /string           | 向后搜索  忽略大小写 set ic |
| ?string           | 向前搜索                    |
| :%s/old/new/g     | 全文替换                    |
| :n1,n2s/old/new/g | 范围内替换                  |

==% 指全文，s 指开始，g 指全局替换==



# 定时任务





crontab命令是cron table的简写，它是cron的配置文件，也可以叫它作业列表，我们可以在以下文件夹内找到相关配置文件

- /var/spool/cron/ 目录下存放的是每个用户包括root的crontab任务，以创建者的名字命名
- /etc/crontab 这个文件负责调度各种管理和维护任务
- /etc/cron.d/ 这个目录用来存放任何要执行的crontab文件或脚本。
- 脚本放在/etc/cron.hourly/daily/weekly/monthly目录下，让它每小时/天/星期/月执行一次



### crontab的使用

我们常用的命令如下：

```
crontab [-u username]　　　　//省略用户表表示操作当前用户的crontab
    -e      (编辑工作表)
    -l      (列出工作表里的命令)
    -r      (删除工作作)
```

我们用**crontab -e**进入当前用户的工作表编辑，是常见的vim界面。每行是一条命令。

crontab的命令构成为 时间+动作，其时间有**分、时、日、月、周**五种，操作符有

- ***** 取值范围内的所有数字
- **/** 每过多少个数字
- **-** 从X到Z
- **，**散列数字





### 实例1：每1分钟执行一次myCommand

```
* * * * * myCommand
```

### 实例2：每小时的第3和第15分钟执行

```
3,15 * * * * myCommand
```

### 实例3：在上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * * myCommand
```

### 实例4：每隔两天的上午8点到11点的第3和第15分钟执行

```
3,15 8-11 */2  *  * myCommand
```

### 实例5：每周一上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * 1 myCommand
```

### 实例6：每晚的21:30重启smb

```
30 21 * * * /etc/init.d/smb restart
```

### 实例7：每月1、10、22日的4 : 45重启smb

```
45 4 1,10,22 * * /etc/init.d/smb restart
```

### 实例8：每周六、周日的1 : 10重启smb

```
10 1 * * 6,0 /etc/init.d/smb restart
```

### 实例9：每天18 : 00至23 : 00之间每隔30分钟重启smb

```
0,30 18-23 * * * /etc/init.d/smb restart
```

### 实例10：每星期六的晚上11 : 00 pm重启smb

```
0 23 * * 6 /etc/init.d/smb restart
```





# 安装



```shell
## vim网络ip配置
vim /etc/sysconfig/network-script/ifcfg-ens32			修改onboot yes
systemctl restart sshd

## 安装ssh/gcc/vim/telnet
sudo yum install sshd
service sshd start
yum -y install gcc gcc-c++ autoconf pcre pcre-devel automake
yum install telnet

## 关防火墙
systemctl stop firewalld
systemctl disable firewalld

#可选项
yum -y install bind-utils
```



## 安装docker

yum install docker

配置镜像加速

vim /etc/docker/daemon.json

{

"registry-mirrors":["http://f1361db2.m.daocloud.io"]

}



## 取消前缀显示主机号



```shell
echo $PS1
[\u@\H \W]\$

#其中\u代表用户，\H代表主机，\W代表目录
 export PS1='[\u@ \W]\$'

#或者是修改用户主目录下的bashrc文件,在 ~/.bashrc 文件最下面增加：
export PS1='[\u@ \W]\$'
```





## 安装mysql

tar -xvf 

依次安装

mysql-community-common-5.7.29-1.el7.x86_64.rpm



报错error: Failed dependencies:
	mysql-community-common(x86-64) >= 5.7.9 is needed by mysql-community-libs-5.7.29-1.el7.x86_64
	mariadb-libs is obsoleted by mysql-community-libs-5.7.29-1.el7.x86_64

yum remove mysql-libs清楚之前安装的依赖



 mysql-community-libs-5.7.29-1.el7.x86_64.rpm
 mysql-community-client-5.7.29-1.el7.x86_64.rpm


解决第四个server包的安装依赖

1.确认镜像光盘已连接

![QQ拼音截图20200710145728](image.assets/clip_image002.gif)

2.进行光盘的挂载

mount /dev/cdrom /media/

3.rpm安装net-tools

rpm -ivh net-tools-2.0-0.25.20131004git.el7.x86_64.rpm

4.安装mysql server包

mysql-community-server-5.7.29-1.el7.x86_64.rpm





### 修改初始密码

systemctl start mysqld.service	启动mysql后自动生成mysqld.log

grep 'password' /var/log/mysqld.log		得到初始密码

![](image.assets/clip_image004.gif)

mysql -u root -p



密码初始等级中等,不允许过于简单的密码

alter user 'root'@'localhost' identified by '初始密码';

设置密码等级low

set global validate_password_policy=LOW;

设置最小密码位数6

set global validate_password_length=6;

设置简易密码

ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

使密码立即生效

flush privileges;



### 允许root远程登录

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

修改mysql的字符集为utf8

vi /etc/my.cnf

关闭大小写敏感

lower_case_table_names=1

添加字符配置

character-set-server=utf8

collation-server=utf8_general_ci

关闭ONLY_FULL_GROUP_BY模式(否则select字段不在group中出现会报错)

sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

[client]

default-character-set=utf8



### 重启mysql

systemctl start mysqld.service





# 网络适配器



*  桥接模式：默认使用vmnet0的虚拟网卡使用当有电脑路由器的分配的IP地址，也就是使用这种模式之后虚拟器就相当于当前局域网的一个真正的电脑了

* NAT模式：使用vmnet8的虚拟网卡，就是让虚拟系统借助NAT(网络地址转换)功能，通过宿主机器所在的网络来访问公网。也就是说，使用NAT模式可以实现在虚拟系统里访问互联网。**虚拟系统的TCP/IP配置信息是由VMnet8虚拟网络的DHCP服务器提供**的，无法进行手工修改，因此虚拟系统也就无法和本局域网中的其他真实主机进行通讯。

* 仅主机模式：静态ip设置 ,虚拟机与主机单独组网，安全，其他网络无法访问



 





# 僵尸/孤儿进程



* 僵尸进程
  * 子进程在其父进程没有调用`wait()`或`waitpid()`的情况下退出
  * 如果其父进程存在且一直不调用wait，僵尸进程无法回收，等到其父进程退出后该进程将被init回收。

* 孤儿进程
  * 父进程退出，子进程还在运行
  * 孤儿进程将被init进程（进程号为1）收养，并由init进程对他们完成状态收集工作













![image-20200826214513517](image.assets/image-20200826214513517.png)





# 文件和目录



内核kernel利用文件描述符file descriptor访问文件。文件描述符是非负整数。打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件。



```shell
/bin: (binaries)存放系统命令的目录，所有用户都可以执行。
/sbin: (super user binaries) 保存和系统环境设置相关的命令，只有root可以用，有些命令可以允许普通用户查看
/usr/bin：存放系统命令的目录，所有用户可以执行。这些命令和系统启动无关，单用户模式下不能执行
/usr/sbin：存放根文件系统不必要的系统管理命令，超级用户可执行
/root: 存放root用户的相关文件,root用户的家目录。宿主目录 超级用户
/home：用户缺省宿主目录eg:/home/spark/home/pengpeng
/usr：（unix software resource）系统软件共享资源目录，存放所有命令、库、手册页等
/proc：虚拟文件系统，数据保存在内存中，存放当前进程信息
/boot：系统启动目录
/dev：(devices)存放设备文件
/sys :虚拟文件系统，数据保存在内存中，主要保存于内存相关信息
/lib：存放系统程序运行所需的共享库
/lost+found：存放一些系统出错的检查结果。
/var：(variable)动态数据保存位置，包含经常发生变动的文件，如邮件、日志文件、计划任务等
/mnt：(mount)挂载目录。临时文件系统的安装点，默认挂载光驱和软驱的目录
/media:挂载目录。 挂载媒体设备，如软盘和光盘
/misc:挂载目录。 挂载NFS服务
/opt: 第三方安装的软件保存位置。 习惯放在/usr/local/目录下
/srv : 服务数据目录
```





























