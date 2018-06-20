
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [linux的根目录文件](#linux的根目录文件)
* [vi编辑器](#vi编辑器)
* [解压命令](#解压命令)
* [编码命令](#编码命令)
* [搜索相关](#搜索相关)
* [分区相关](#分区相关)
* [用户相关](#用户相关)
* [用户组相关](#用户组相关)
* [Linux 命令](#linux-命令)
* [文件相关](#文件相关)
* [文本命令](#文本命令)
* [挂载](#挂载)
* [ssh](#ssh)
		* [密钥验证概念](#密钥验证概念)
		* [ssh免密登录配置](#ssh免密登录配置)
		* [ssh常用命令](#ssh常用命令)
* [防火墙设置](#防火墙设置)
		* [iptables](#iptables)
		* [firewalld](#firewalld)
* [任务调度的使用](#任务调度的使用)
* [RMP,Samba相关](#rmpsamba相关)
* [进程相关](#进程相关)
* [时间相关](#时间相关)
* [监控网络状态](#监控网络状态)
* [修改配置文件来转换界面](#修改配置文件来转换界面)
* [修改环境变量](#修改环境变量)
* [修改ip地址方法](#修改ip地址方法)
* [yum](#yum)
		* [安装和卸载yum](#安装和卸载yum)
		* [yum常用命令](#yum常用命令)
* [历史命令](#历史命令)
* [Linux 快捷键和使用技巧](#linux-快捷键和使用技巧)
	* [快捷键](#快捷键)
	* [使用技巧](#使用技巧)
* [其余常用命令](#其余常用命令)
* [shell](#shell)
	* [变量](#变量)
	* [常用shell](#常用shell)
	* [变量相关](#变量相关)
	* [shell脚本执行方式的区别](#shell脚本执行方式的区别)

<!-- /code_chunk_output -->


# linux的根目录文件
`root`: 存放root用户相关文件
`home`:存放普通用户的相关文件
`bin`:存放常用命令
`sbin`:存放要具有一定权限才可以使用的命令
`mnt`:默认挂载光驱和软驱的目录
`boot`:存放引导相关的文件
`etc`:存放配置相关文件
`var`:存放经常变化的文件
`usr`:默认软件安装文件夹，类似windows里面的Program Files

# vi编辑器
`vi XXX文件名`  使用vi编译器创建一个文件
1. 输入i进入编辑模式
2. 输入esc进入命令模式
3. wq 保存后退出
4. q!退出不保存

建议使用`vim`，显示跟更好

其余查看命令：
`cat 文件路径` ：查看一个文件
`more 文件路径`：分页查看一个文件
`less 文件路径`：分页查看一个文件（可前翻可后翻）

# 解压命令
`tar -zxvf ???.tar.gz` 解压targz文件
`tar -zxvf ???.tar.gz -C /usr/xxx` 解压targz文件到指定目录
`tar -xf ???.tar.gz` 常用命令，解压targz文件

`zip 文件1 文件2` 把文件1压缩成文件2
`zip 文件1 文件2 文件3` 把文件1，文件2压缩成文件3
`zip -r a.zip /*` 把`/*`后的所有文件压缩成a.zip
`unzip 文件` 解压文件

`gzip –v` 文件路径 压缩文件成gz文件
`gzip –d` 文件路径 解压文件

# 编码命令
当显示中文乱码时，可修改系统默认语言为英文：`LANG=en_US.UTF-8`(可以`echo $LANG`查看当前系统语言)

`locale` 查看编码

# 搜索相关
`grep 内容 文件路径`:查询文件里面的关键字内容
`grep -n 内容 文件路径`: 查询文件里面的内容,显示在那一行
`grep -n 内容 文件路径 文件路径`: 查询多个文件里面的内容,显示在那一行

`grep wsz a.txt`: 查找a.txt文件里面wsz所在那行
`grep 'wsz' a.txt`: 查找a.txt文件里面wsz所在那行
`grep `pwd` ` --查找命令pwd返回的目录下的进程
（`mkfs -t ext3 /dev/xx` 格式化某分区为ext3格式）

`find 目录 -name 文件名` 在某一目录下查找该名称的文件或目录
`find /home -amin -10` 十分钟内存取过的文件或目录
`find /home -atime -10` 十小时内存取的文件过目录
`find /home cmin -10` 十分钟内更改过的文件或目录
`find /home ctime +10` 十小时前更改过的文件或目录

更高效的查找：
`which 命令名` 根据PATH查找文件名（可用来查找命令）
`type 命令名` 用来显示命令是外部命令还是bash内置命令，也可用于查找命令位置
`whereis 文件或目录名` 只能找二进制，man说明文件，源文件（可用来查找命令）
`locate 文件或目录名`（使用前先更新数据库，updatedb）

通配符：
`*` 多个字母或数字
`？` 一个字母或数字
`[1-9]` 1到9的一个数字

# 分区相关
硬盘分区分为：
1）主分区（Primary partion）：可马上使用不需分区。类似于C盘，一般为系统所在区
，一般只有一个
2） 扩展分区(Extension partion)：必须进行二次分区才能使用，二次分区为逻辑分区，类似于D盘，E盘。逻辑分区没有数量限制
注意：主分区和扩展分区两者的数目和不能超过4个

以windows系统作为例子：
![](.png)

C盘为主分区；绿色框为扩展分区，其中又分为了D,E,F三个逻辑分区。注意还可能存在为空的分区的空间存在。

`fdisk -l` 查看linux系统分区具体情况
`fdisk /dev/sda` 创建一个叫sda的新分区,后按m可以选择进一步的操作
`-sda` 代表挂载的硬盘为sata接口硬盘（更好）
`-hda` 代表ide接口硬盘

`df`或`df –k`  以K为单位查看磁盘使用情况
`df –h`   以G为单位查看磁盘使用情况
`df –m`  以M为单位查看磁盘使用情况df -l 目录
`df` 目录 查看该目录分区的使用情况

`mkfs -t ext3 /dev/xx` 格式化某分区为ext3格式
`du` 查看本目录每个文件和目录的空间使用情况

# 用户相关
`useradd xxx` 新增用户
`userdel xxx` 删除用户
`userdel -r xxx` 删除用户以及用户主目录
`cat /etc/passwd` 查看用户列表
`passwd xxx` 修改用户密码，需要root用户

`su -` 切换到root用户
`who` 查看当前在线用户

`passwd` 修改当前用户密码
`logout` 注销

# 用户组相关
groupapp xxx：添加用户组
groupdel xxx：删除一个用户组
cat /etc/group：查看用户组（用vi也可以，但有修改风险）
cat /etc/passwd： 查看用户组以及里面的用户
useradd -g 组名 用户名： 创建用户，同时指定将用户放进哪个组
usermod -g 组名 用户名： 修改用户所在组
chown 用户名 文件名： 修改文件所有者
chown 用户名：用户组 文件名 修改文件的所有者以及所在组

# Linux 命令
`.` 当前目录
`..` 上级目录
`-`	前一个工作目录
`~` 目前用户所在主文件夹

# 文件相关
ls -l 后
drwx------ 第一个代表文件类型，第二个代表文件的所有者对该文件的权限，第三个文件所在组对该文件的权限，第四个代表其他组的用户对该文件的权限

who am i 查看当前用户
chmod 777 用户名 数字分别代表什么参考上面(r 4,w 2,x 1  即读写执行)
chown -R 用户名 目录 修改该目录的所有者为指定用户

`mkdir xxx` 创建文件夹
`rmdir xxx` 删除文件夹
`touch xxx` 创建文件
`rm -rf` 不带提示的级联删除文件或文件夹
`cp –a` 源文件 目标文件 复制文件过去且权限不变

# 文本命令
cut -d: -f1 /etc/passwd 以“：”为分隔符（默认为空格）查看第一个字段
sort 文件路径 根据第一个字符的ascii码进行排序
uniq -c 文件路径 显示文件中行重复的次数
uniq -d 文件路径 只显示文件中重复的行
wc 文件路径 统计文本文件中行数，字节数等信息
tr 'ab' 'AB' < /etc/passwd 把该文件中的ab换为AB
`sed -i "s/查找字段/替换字段/g" xxx` 替换xxx文件的内容

# 挂载
mount  /mnt/cdrom/  挂载光驱
umount  /mnt/cdrom/ 卸载光驱
mount  /dev/sda? 目录  挂载指定分区到指定目录
umount  目录 卸载目录分区



# ssh
### 密钥验证概念
先来弄清楚两个相关概念：公匙和密匙。简单来说：公匙用来加密，私匙用来解密。
>
>公匙可被广泛传播，甚至保存在公共密匙数据库中以被其他Internet用户查阅。私匙属于个人信息，绝不应该泄漏给其他人。
公匙和私匙相互作用对数据进行加密及解密。被公匙加密的数据只能被私匙解密，被私匙加密的数据也只能被一个公匙解密。这样就可以实现双重认证。
>
>用户在发送关键信息给指定人前，首先使用该用户的公匙对信息进行加密。因为只有使用该用户的私匙才能对发送信息进行解密，所以就保证了没有私匙的其他人不会解密信息。
>
>另外，用户也可以使用他的私匙来加密信息，然后发送给许多人。因为只有使用发送者的公匙才能对接收信息进行解密，这样接收者就能确信信息的确来自某个人。

注意：linux中ssh监听端口为22

### ssh免密登录配置
1. `ssh-keygen -t rsa`，用`rsa`算法生成公钥和密钥。（生成路径会有提示，CentOS7中是`/root/.ssh`）
2. 将生成的公钥`id_rsa.pub`复制到对应机器的`./ssh`目录
3. 在对应机器的`.ssh`目录手动生成密钥权限文件：`touch authorized_keys`
4. 在对应机器的权限文件中添加公钥：`cat id_rsa.pub >> authorized_keys`
5. 在原机器验证：`ssh 对应机器ip`

以上步骤可在生成公匙后用一句命令解决：
`ssh-copy-id 对应机器ip或host`
### ssh常用命令
- `ssh root@192.168.88.1` 用`root`用户登录`192.168.88.1`机器
- `ssh 192.168.88.1 ls /` 在`192.168.88.1`机器执行`ls /`命令
- `scp /usr/a.txt root@192.168.88.2:/usr/a.txt` 将本机文件复制到目标机，采用root登录目标机，需要输入密码
- `scp -r /usr/a.txt root@192.168.88.2:/usr/a.txt` 将本机**文件夹**复制到目标机，采用root登录目标机，需要输入密码

# 防火墙设置

### iptables
1）重启后生效
开启： chkconfig iptables on
关闭： chkconfig iptables off 或者 /sbin/chkconfig --level 2345 iptables off

2) 即时生效，重启后失效
service 方式
开启： service iptables start
关闭： service iptables stop
iptables方式
查看防火墙状态：
/etc/init.d/iptables status
暂时关闭防火墙：
/etc/init.d/iptables stop
重启iptables:
/etc/init.d/iptables restart

### firewalld
`systemctl stop firewalld.service` 停止firewall
`systemctl disable firewalld.service` 禁止firewall开机启动


# 任务调度的使用
crontab -e  指定什么时间执行某一命令，精确最高只能到分钟
* * * * * 分 时 月 年 星期
* * * * * 命令 指定什么时间重复执行这一命令
为了使命令调度列表不那么乱，可以采用shell脚本形式
即创建一个.sh文件，把要调度的命令都放里面
再添加任务
* * * * * 文件路径 指定什么时间重复执行这一脚本

crontab -r 终止任务调度（所有任务都消除，慎用）
crontab -l 列出当前有那些任务调度

# RMP,Samba相关
RPM包格式
apache-1.3.23-11.i386.rpm
apache 软件名称
1.3.23.11-11 软件版本号，主版本和次版本
i386 软件运行的硬件平台

rpm -qa 查询所有的rpm包软件
rpm -q 软件名 查询指定名字的rpm软件
rpm -i rpm包全路径 安装包到当前系统
rpm -ivh rpm包全路径 安装包到当前系统，有提示信息
rpm -e rpm包的名称 删除
rpm -e --nodeps 软件名 强制删除，不管依赖软件，不建议使用

cat /etc/passwd | mksmbpasswd.sh > /etc/samba/smbpasswd把/etc/passwd里的用户放入smbpasswd中
smbpasswd 用户名 给用户名设置samba的密码
service smb start 启动samba服务器
service smb stop  停止samba服务器
service smb restart 启动samba服务器

# 进程相关
进程：正在执行的程序
线程：轻量级进程
      线程没有分配内存，进程分配
      线程由进程创建

ps -A / ps -e  显示系统中所有进程信息
ps -a 显示同一终端下的所有程序,即其他用户的进程
ps -u 显示当前用户创建的所有进程信息
ps -x 显示后台进程运行参数
推荐使用 ps -aux 显示全面
pgrep 查看进程号

top 实时显示系统中各个进程的资源占用情况
top -d 10 10秒刷新一下信息
（按u后提示输入用户，可观察用户相关进程）

kill 进程号 发送终止信号，但该进程不一定被立即终止
kill -9 进程号 强制立即终止某个进程

# 时间相关
date 显示当前系统时间
date -s 11:11:00 修改时间为11：11
date -s 06/18/14 修改日期为2014-6-18
date +%y%m%d 以规定格式显示日期：151203
date +%Y%m%d  20151203
例如：tail -f -n 0 ../logs/catalina.out.`date +%Y-%m-%d` 动态查看今天的日志文件

cal 2015 查看2015日历
cal 11 2015 查看2015 11月的日历

# 监控网络状态
`netstat`  netstat是在内核中访问网络及相关信息的程序，它能提供TCP连接，TCP和UDP监听，进程内存管理的相关报告。
`netstat -an` 按照端口号排序显示所有
`netstat -anp` 排列时显示进程号
`netstat -nltp | grep 端口号` 用于查看指定端口号的进程情况

`traceroute 网址` 追踪路由，用来检测数据包在网络上传输的过程
route 显示本机路由表


# 修改配置文件来转换界面
1.`vi /etc/inittab`
2. `init:5:initdefault`   5为图形化界面，修改为3为命令行
Linux有7个运行级别：`init[0123456]`
- 0：关机
- 1：单用户
- 2：多用户状态没有网络服务
- 3：多用户状态有网络服务
- 4：系统未使用保留给用户
- 5：图形界面
- 6：重新启动

可用`init 数字` 来暂时切换各模式

注意：Centos7 中已经放弃修改配置文件的的方式
转换为图形化界面：
`systemctl set-default graphical.target`
转换为命令行界面：
`systemctl set-default multi-user.target`

# 修改环境变量
env 显示当前操作系统的环境变量
`vi /etc/profile`
或者
`vi /etc/profile.d/development.sh`  新建一个sh文件专门存储软件的环境变量，原理是因为系统会自动读取`profile.d`文件夹下的文件并加载
`vi /root/.bash_profile` 可在里面配置用户环境变量

`export PATH=$PATH:/root` 临时添加环境变量（可以在别的目录运行/root下的sh命令）
`source /etc/profile` 使环境变量立即生效
# 修改ip地址方法
1）`setup` 进入配置界面（未发现怎样修改）
2）在图形界面直接点右上角的网络进行设置
3）`ifconfig 网卡号 修改的ip地址`立即生效，重启后恢复原本ip
4）`vim /etc/sysconfig/network-scripts/ifcfg-eth0`（CentOS7中为`ifcfg-ens33`） 最底层的方法，修改文件，在里面修改ip,网关等
（静态为指定的ip，动态为系统可跟你分配ip）
/etc/rc.d/init.d/network restart 或 service network restart 重启网卡服务

# yum
### 安装和卸载yum
- 卸载yum：rpm -aq | grep yum|xargs rpm -e --nodeps
- 下载yum：
`Wget	http://mirror.centos.org/centos/6.7/os/x86_64/Packages/python-iniparse-0.3.1-2.1.el6.noarch.rpm`
`Wget	http://mirror.centos.org/centos/6.7/os/x86_64/Packages/yum-metadata-parser-1.1.2-16.el6.x86_64.rpm`
`wget		http://mirror.centos.org/centos/6.7/os/x86_64/Packages/yum-3.2.29-69.el6.centos.noarch.rpm`
`wget http://mirror.centos.org/centos/6.7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.30-30.el6.noarch.rpm`
(yum 的基础安装包包括：
`yum`　　//RPM installer/updater`
`yum-fastestmirror`　　//Yum plugin which chooses fastest repository from a mirrorlist
`yum-metadata-parser`　　//A fast metadata parser for yum)

### yum常用命令
`yum list | grep telnet`：去大黄狗应用市场搜索`telnet`关键字的应用
`yum install xxx` 安装`xxx`应用

# 历史命令
history 显示最近使用的命令
history 数字 显示最近使用的指定数目的命令
history -c 清楚缓存命令

!n 执行命令历史中第n条命令
！！ 执行上一条命令
！ta 执行命令历史里面以ta开头的命令，按最近的执行（也可按下esc松开再按.）
！$ 引用上一个命令的最后一个参数

# Linux 快捷键和使用技巧
## 快捷键
ctrl+a 跳到命令行首
ctrl+e 跳到命令行尾
ctrl+u 删除光标至命令行首的内容
ctrl+k 删除光标至命令行尾的内容
ctrl+l 清屏，等同clear

## 使用技巧
1.	输命令后按两下tab会出现以xx开头的命令列表
2.	Linux默认开了6个终端，alt+ctrl+【F1~F6】进行切换。默认进入的终端是F1，如果为命令行界面，可用startx进入xwindos界面
3.	命令太长，可用”\+回车键”进行转义回车，使命令在下行输入
4.	查看命令帮助可以man 命令，或者命令 –help
5.	在一串命令中需要通过其他命令提供的信息用：`命令`或者$(命令)

# 其余常用命令

ifconfig linux下查看ip地址
ipconfig windows下查看ip地址
tracert 网址 追踪路由（windows）
回送地址除了127.0.0.1外，以127开头的都可以，访问tomcat都可以成功

startx 开启图形化界面
exit 注销（可用快捷键ctrl+d）
reboot 重启
shutdown -h now  立刻关机
shutdown -r now /reboot  立刻重新启动

ls 浏览文件
ls -a 显示隐藏文件
ls -l 浏览文件详细信息

pwd 显示当前目录
./XXX文件名 运行一个文件

gcc XXX.cpp 编译cpp文件
gcc -o 文件名 XXX.cpp 指定编译后的文件名


more 分页
| 管道命令（把上一个命令的结果交给|后面的命令处理， 如分页浏览文件：ls -l /etc | more）
grep -n "关键字" 文件 查看文件中的关键字所在行信息

find / -name xxx  从根目录查找xxx文件是否存在

`> a.txt` 把xx结果写入一个文件（覆盖），可以是浏览结果等
`>> a.txt` 追加写入一个文件

mv 旧名字 新名字 重命名
./xxx & 以后台形式运行程序（即控制台仍然能够使用）

ll会列出该文件下的所有文件信息，包括隐藏的文件，而ls -l只列出显式文件

cd 直接到用户目录下，如root用户登录到/root目录下

`ln -s 源 目标`  即生成源的快捷方式

`alias 别名=‘命令’` 给命令取别名

`jps`查看所有的java进程

`date --date='2 days ago' +%Y%m%d` 输出两天前的日期“20171207”
`date +%Y%m%d` 输出当天的日期“20171209”

# shell
##变量
${}中放的是变量名，其中双引号保存变量，单引号全是文本。
``或$()为执行命令后的文本。

举例：
`test="PATH is $PATH "`输出PATH变量
`test='PATH is $PATH'`输出文本PATH is $PATH

## 常用shell
/bin/sh
/bin/csh
/bin/ksh
chsh -s 输入新的shell

## 变量相关
- `echo $PATH` 用于显示变量
`echo -e "xxx"` 处理特殊字符输出，例如字符串中的“\n”会被处理为换行
<br/>
- `echo $?`
这是一个shell的变量，意思是返回上一步执行任务是否成功。
如果为 0 说明执行成功。
如果非 0 说明执行失败。
<br/>
- 定义加减乘除变量的方法：
`declare -i c=1+2` 输出`$c`为3
`declare -i c=$((1+2))` 输出`$c`为3
<br/>
- linux 启动会自动运行一些shell文件，所以想要使一些程序开机时自动启动可以在这些shell文件里面配置
例如：vi /root/bashrc 修改root下的bashrc，使每次root用户登录时会执行你想要启动的程序
<br/>
- linux中的三元表达式：因为linux中所以命令都有返回值（执行成功会返回0），所以可用如下方式代替三元表达式：`test -e /usr && echo "exist" || echo "not exist"`。即“/usr”目录存在时输出"exist"，反之输出"not exist"

## shell脚本执行方式的区别
1. 普通方式执行（`./`或者bash）：开启子进程执行shell命令，定义的变量在父进程不会保存
2. source shell脚本，在父进程执行，定义的变量会保存
