>#系统环境准备
>###1）清空iptables
>	 [root@zhou ~]# iptables -F
>	 [root@zhou ~]# /etc/init.d/iptables save
>	 iptables：将防火墙规则保存到 /etc/sysconfig/iptables：     [确定]
>	 [root@zhou ~]# chkconfig iptables off


>###2）关闭selinux
>	 查看selinux状态
>	 [root@zhou ~]# getenforce
>	 Enforcing
>	 [root@zhou ~]# vim /etc/sysconfig/selinux 
>	 改：
>	 # SELINUX= can take one of these three values:
>	 #>	      enforcing - SELinux security policy is enforced.
>	 #     permissive - SELinux prints warnings instead of enforcing.
>	 #     disabled - No SELinux policy is loaded.
>	 SELINUX=enforcing
>	 为：
>	 # SELINUX= can take one of these three values:
>	 #     enforcing - SELinux security policy is enforced.
>	 #     permissive - SELinux prints warnings instead of enforcing.
>	 #     disabled - No SELinux policy is loaded.
>	 SELINUX=disabled
>	 [root@zhou ~]# reboot			#重启后生效

>###3）配置好静态IP

>###4）配置主机和ip映射关系
>	 [root@zhou ~]# vim /etc/hosts

>###5）修改主机名
>	 [root@zhou ~]# vim /etc/sysconfig/network
>	 NETWORKING=yes
>	 HOSTNAME=zhoushuai69.cn
>	 GATEWAY=192.168.0.1
>	 [root@zhou ~]# reboot			#重启后生效
>	 [root@zhoushuai69 ~]# hostname
>	 zhoushuai69.cn

>###配置yum源
>RHEL系统本身光盘做成的yum源所提供的软件包有限，在实际使用过程中经常会出现缺包的现象，本文中CentOS源作为替代，CentOS的软件包和RHEL系统是互相兼容的，而且CentOS公司已被RHEL收购，所以不会出现兼容性的问题。
>>注：配置网络yum时，为了避免之前的yum文件相互冲突
>>建议删除之间的配置文件可直接执行 `rm -rf /etc/yum.repos.d/*`在执行相应配置
>
>####RHEL6.5
>	 本地yum源
>     [root@zhoushuai ~]# mount /dev/sr0  /mnt/
>     [root@zhoushuai ~]# echo "/dev/sr0 /mnt iso9660 defaults 0 0" >> /etc/fstab
>     [root@zhoushuai ~]# rm -rf /etc/yum.repos.d/*
>     [root@zhoushuai ~]# cat > /etc/yum.repos.d/rhel6.repo <<EOF
>     [rhel6-source]
>     name=rhel6-source
>     baseurl=file:///mnt
>     enabled=1
>     gpgcheck=0
>     EOF
>     [root@zhoushuai ~]# yum clean all
>     [root@zhoushuai ~]# yum list
_________________________________________________________________________
>	 配置网络yum源
>	  [root@zhoushuai ~]# wget -O /etc/yum.repos.d/CentOS6-Base-163.repo http://mirrors.163.com/.help/CentOS6-Base-163.repo
	 [root@zhoushuai ~]# sed -i 's/$releasever/6.8/g' /etc/yum.repos.d/CentOS6-Base-163.repo
	 [root@zhoushuai ~]# yum clean all
	 [root@zhoushuai ~]# yum list
>####上传软件包到服务器
>	 [root@zhoushuai ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-27.1.el6.x86_64.rpm
>	 rz 	上传
>	 sz	下载
>	 [root@zhoushuai ~]# cd usr/local/src/
>	 [root@zhoushuai ~]# rz
>###时间同步
>#####硬件BIOS时间：
>	  hwclock -r    :读出BIOS的时间参数
>	  hwclock -w    :将当前系统时间写入BIOS中。
>	  [root@zhoushuai ~]# date -s "2016-3-27 22:00"
>	  2016年 03月 27日 星期日 22:00:00 CST
>	  [root@zhoushuai ~]# hwclock -r
>	  2016年03月26日 星期六 21时16分19秒  -1.172966 seconds
>	  [root@zhoushuai ~]# date
>	  [root@zhoushuai ~]# hwclock -w
>	  [root@zhoushuai ~]# hwclock -r
>	  2016年03月27日 星期日 22时00分33秒  -0.797470 seconds
>	  
>#####客户端配置：
>	  不同机器之间的时间同步 
>	  为了避免主机时间因为长期运作下所导致的时间偏差，进行时间同步(synchronize)的工作是非常必要的。
>#####同步时间，可以使用ntpdate命令，也可以使用ntpd服务。 
>	  方法一：使用ntpdate比较简单。格式如下： 
>	  [root@zhoushuai ~]# ntpdate 192.168.0.66
>	  4 Jun 15:56:09 ntpdate[2239]: step time server 192.168.0.66 offset -28786.259500 sec
>	  [root@zhoushuai ~]# ntpdate 192.168.0.66
>	  4 Jun 15:56:09 ntpdate[2239]: step time server 192.168.0.66 offset -28786.259500 sec
>	  [root@zhoushuai ~]# ntpdate 192.168.0.66
>	  4 Jun 15:56:37 ntpdate[2240]: adjust time server 192.168.0.66 offset -0.000741 sec
>	  [root@zhoushuai ~]# date
>	  2016年 06月 04日 星期六 15:56:49 CST
>	  [root@zhou ~]# hwclock -r
>	  2016年06月04日 星期六 23时56分41秒  -0.423922 seconds
>	  [root@zhoushuai ~]# hwclock -w
>	  [root@zhoushuai ~]# hwclock -r
>	  2016年06月04日 星期六 15时57分07秒  -0.985232 seconds
>	  [root@zhoushuai ~]# 
>#####但这样的同步，只是强制性的将系统时间设置为ntp服务器时间。只是治标不治本。所以，一般配合cron命令，来进行定期同步设置。比如，在crontab中添加： 
>	  [root@zhoushuai ~]# crontab -e
>	  0 12 * * * /usr/sbin/ntpdate 192.168.0.66
>	  方法2：使用ntpd服务进行同步。
>	  要注意的是，ntpd 有一个自我保护设置: 如果本机与上源时间相差太大, ntpd 不运行. 所以新设置的时间服务器一定要先 ntpdate 从上源取得时间初值, 然后启动 ntpd服务。 
>	  
>####RHEL6.5引导配置
>	 [root@zhoushuai ~]# vim /etc/inittab
>	 id:3:initdefault:

>####主机网卡类型：桥接

>####开机配置成： init 3  模式
>	 reboot
>####创建一个快照。
>####每做一个新的服务时，使用一个全新的快照。
>
>	 台Linux服务器之间复制数据
>	 同的Linux之间copy文件常用有多种方法：
>	 三种就是利用scp命令来进行文件复制。
>	 scp基于ssh登录并复制数据。远程复制过程中很安全。操作起来比较方便。
>	 例：要把当前一个文件copy到远程另外一台主机上
>	 [root@zhoushuai ~]# scp /etc/passwd root@192.168.1.64:/tmp
>	 然后会提示你输入另外那台192.168.1.64主机的root用户的登录密码，接着就开始copy了。
>	 例：把文件从远程主机copy到当前系统
>	 [root@zhoushuai ~]# scp root@192.168.1.63:/etc/passwd /root/
>	 在Linux之间复制目录： 加`-r`参数









