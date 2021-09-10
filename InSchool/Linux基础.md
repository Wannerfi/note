**Linux世界里，一切皆为文件**



# Linux目录



* /bin 常用指令
* /boot 启动Linux时的核心文件，包括一些连接文件及镜像文件
* /dev  device（设备），存放Linux的外部设备，Linux中访问设备的方式和访问文件的方式相同
* /etc etcetera（等等），存放所有的系统管理所需要的配置文件和子目录
* /home 用户的主目录
* /lib library（库），存放系统最基本的动态链接共享库
* /lost+found 一般情况下为空，当系统非法关机后，就存放了文件
* /media 把识别的设备挂载在这个目录下，例如U盘、光驱等
* /mnt 用户临时挂载别的文件系统
* /opt optional（可选），给主机额外安装软件所摆放的目录
* /proc processes（进程），虚拟文件系统，存储当前内核运行状态的一系列特殊文件，系统内存的映射
* /root 系统管理员，超级权限者用户的主目录
* /sbin s是super user，存放系统管理员使用的系统管理程序
* /selinux Redhat/CentOS所特有的目录，selinux是一个安全机制
* /srv 存放一些服务启动后需要提取的数据
* /sys Linux2.6内核的巨变，该目录下安装了2.6内核中新出现的一个文件系统sysfs。sysfs文件系统集成了：针对进程信息的proc文件系统、针对设备的devfs文件系统、针对伪终端的devpts文件系统；该文件系统是内核设备树的直观反映；当一个内核对象被创建，对应的文件和目录也在对象子系统中被创建。
* /tmp 存放临时文件
* /usr unix shared resources（共享资源）
* /usr/bin 系统用户使用的应用程序
* /usr/sbin 超级用户使用的比较高级的管理系统和系统守护程序
* /usr/src 内核源代码默认的放置目录
* /var variable（变量），放置经常修改的目录，包括各种日志文件
* /run 临时文件系统，存储系统启动以来的信息

![](../picture/Linux基础-Linux目录树.png)





