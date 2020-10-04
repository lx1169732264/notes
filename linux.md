# 初步安装



## vim网络ip配置

vim /etc/sysconfig/network-script/ifcfg-ens32			修改onboot yes

systemctl restart sshd

## 安装ssh/gcc/vim/telnet

sudo yum install sshd

service sshd start

yum -y install gcc gcc-c++ autoconf pcre pcre-devel automake

yum install telnet



## 安装docker

yum install docker

配置镜像加速

vim /etc/docker/daemon.json

{

"registry-mirrors":["http://f1361db2.m.daocloud.io"]

}



## 前缀不显示主机号

echo $PS1
[\u@\H \W]\$

其中\u代表用户，\H代表主机，\W代表目录

 export PS1='[\u@ \W]\$'




或者是修改用户主目录下的bashrc文件

在 ~/.bashrc 文件最下面增加：

export PS1='[\u@ \W]\$'



# 安装mysql

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

## 关闭并设置开机不启动防火墙

systemctl stop firewalld

systemctl disable firewalld

## 获取mysql的root用户的初始密码

systemctl start mysqld.service	启动mysql后自动生成mysqld.log

grep 'password' /var/log/mysqld.log

![4444444](image.assets/clip_image004.gif)

mysql -u root -p

## 修改初始密码

密码初始等级中等,不允许过于简单的密码

alter user 'root'@'localhost' identified by '2020root$PWD';

设置密码等级low

set global validate_password_policy=LOW;

设置最小密码位数6

set global validate_password_length=6;

设置简易密码

ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

使密码立即生效

flush privileges;

## 允许root远程登录mysql

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

修改mysql的字符集为utf8

vi /etc/my.cnf

关闭大小写敏感

lower_case_table_names=1

添加字符配置

character-set-server=utf8

collation-server=utf8_general_ci

[client]

default-character-set=utf8

## 重启mysql服务

systemctl start mysqld.service

## 执行sql脚本

source /xxx.sql;





 

 # 网络适配器

*  桥接模式：默认使用vmnet0的虚拟网卡使用当有电脑路由器的分配的IP地址，也就是使用这种模式之后虚拟器就相当于当前局域网的一个真正的电脑了

* NAT模式：使用vmnet8的虚拟网卡，就是让虚拟系统借助NAT(网络地址转换)功能，通过宿主机器所在的网络来访问公网。也就是说，使用NAT模式可以实现在虚拟系统里访问互联网。**虚拟系统的TCP/IP配置信息是由VMnet8虚拟网络的DHCP服务器提供**的，无法进行手工修改，因此虚拟系统也就无法和本局域网中的其他真实主机进行通讯。

* 仅主机模式：静态ip设置 ,虚拟机与主机单独组网，安全，其他网络无法访问





# 进程/端口查询

 















 

# 后台运行jar包

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

查看某端口占用的线程的pid

```
netstat -nlp |grep :9181
```



































![image-20200826214513517](image.assets/image-20200826214513517.png)

![image-20200826214527561](image.assets/image-20200826214527561.png)

# Linux版本

### 内核版本

又分为稳定版和开发版，两种版本是相互关联，相互循环：

·     稳定版：具有工业级强度，可以广泛地应用和部署。新的稳定版相对于较旧的只是修正一些bug或加入一些新的驱动程序。

·     开发版：由于要试验各种解决方案，所以变化很快。

内核源码网址：http://www.kernel.org 所有来自全世界的对Linux源码的修改最终都会汇总到这个网站，由Linus领导的开源社区对其进行甄别和修改最终决定是否进入到Linux主线内核源码中。

### 发行版本

Linux发行版 (也被叫做 GNU/Linux 发行版) 通常包含了包括桌面环境、办公套件、媒体播放器、数据库等应用软件。

### 服务器领域

linux免费、稳定、高效等特点在这里得到了很好的体现，但早期因为维护、运行等原因同样受到了很大的限制，但近些年来linux服务器市场得到了飞速的提升，尤其在一些高端领域尤为广泛

典型代表：

Red Hat公司的AS系列

完全开源的debian系列

suse EnterPrise 11系列等

### 嵌入式领域

·     近些年来linux在嵌入式领域的应用得到了飞速的提高

·     linux运行稳定、对网络的良好支持性、低成本，且可以根据需要进行软件裁剪，内核最小可以达到几百KB等特点，使其近些年来在嵌入式领域的应用得到非常大的提高

·     主要应用：机顶盒、数字电视、网络电话、程控交换机、手机、PDA、等都是其应用领域，得到了摩托罗拉、三星、NEC、Google等公司的大力推广



# 文件和目录



/bin: (binaries)存放系统命令的目录，所有用户都可以执行。

/sbin: (super user binaries) 保存和系统环境设置相关的命令，只有超级用户可以使用这些命令，有些命令可以允许普通用户查看。（root）

/usr/bin：存放系统命令的目录，所有用户可以执行。这些命令和系统启动无关，单用户模式下不能执行

/usr/sbin：存放根文件系统不必要的系统管理命令，超级用户可执行

/root: 存放root用户的相关文件,root用户的家目录。宿主目录 超级用户

/home：用户缺省宿主目录eg:/home/spark/home/pengpeng

/tmp：(temporary)存放临时文件

/etc：(etcetera)系统配置文件

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

## 文件权限

文件权限就是文件的访问控制权限，即哪些用户和组群可以访问文件以及可以执行什么样的操作。

Unix/Linux系统是一个典型的多用户系统，不同的用户处于不同的地位，对文件和目录有不同的访问权限。为了保护系统的安全性，Unix/Linux系统除了对用户权限作了严格的界定外，还在用户身份认证、访问控制、传输安全、文件读写权限等方面作了周密的控制。

在 Unix/Linux中的每一个文件或目录都包含有访问权限，这些访问权限决定了谁能访问和如何访问这些文件和目录。

### 访问权限

用户能够控制一个给定的文件或目录的访问程度，一个文件或目录可能有读、写及执行权限：

·     读权限（r） 对文件而言，具有读取文件内容的权限；对目录来说，具有浏览目录的权限。

·     写权限（w） 对文件而言，具有新增、修改文件内容的权限；对目录来说，具有删除、移动目录内文件的权限。

·     可执行权限（x） 对文件而言，具有执行文件的权限；对目录了来说该用户具有进入目录的权限。





















