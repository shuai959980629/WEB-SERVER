>#安装fail2ban
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
>####下载软件包：
>####官方地址：
>####http://www.fail2ban.org
>####上传软件包到服务器
>	 [root@zhoushuai ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-27.1.el6.x86_64.rpm
>	 rz 	上传
>	 sz	下载
>	 [root@zhoushuai ~]# cd usr/local/src/
>	 [root@zhoushuai ~]# rz
>	 [root@zhoushuai ~]# tar zxvf fail2ban-0.8.14.tar.gz
>	 [root@zhoushuai fail2ban-0.8.14]# vim README.md  #查看以下内容
![1000.png](.\images\1000.png)

>	 需要安装python开发环境，并且版本要大于2.4
>	 查看当前系统中python的版本：
>	 [root@zhoushuai fail2ban-0.8.14]# python -V
>	 Python 2.6.6
>	 
>####安装：
>	 [root@zhoushuai fail2ban-0.8.14]# python setup.py install

>####生成服务启动脚本
>	 [root@zhoushuai fail2ban-0.8.14]# cp files/redhat-initd /etc/init.d/fail2ban
>	 [root@zhoushuai fail2ban-0.8.14]# chkconfig --add fail2ban
>	 [root@zhoushuai fail2ban-0.8.14]# chkconfig --list fail2ban
>	 fail2ban        0:关闭  1:关闭  2:关闭  3:启用  4:启用  5:启用  6:关闭

>>####互动： 你怎么知道要复制这个文件？ 一个新的软件包，后期怎么可以知道哪个文件是启动脚本文件？
>####这就要找服务器启动脚本文件中有什么特点，然后过滤出来对应的文件名。
>####[root@zhoushuai fail2ban-0.8.14]# grep chkconfig ./* -R --color
>####./files/redhat-initd:# chkconfig: - 92 08
>
>####相关主要文件
>	 [root@zhoushuai ~]# ls /etc/fail2ban/
>	 action.d  fail2ban.conf  fail2ban.d  filter.d  jail.conf  jail.d
>	 /etc/fail2ban/action.d  #动作文件夹，内含默认文件。iptables以及mail等动作配置
>	 /etc/fail2ban/fail2ban.conf    #定义了fai2ban日志级别、日志位置及sock文件位置
>	 /etc/fail2ban/filter.d                     #条件文件夹，内含默认文件。过滤日志关键内容设置
>	 /etc/fail2ban/jail.conf     #主要配置文件，模块化。主要设置启用ban动作的服务及动作阀值   
>	 # jail   [dʒeɪl]  监狱
>	 /etc/rc.d/init.d/fail2ban                #启动脚本文件
>###etc/fail2ban/jail.conf及说明如下：
>	 应用实例：
>	 设置条件：ssh远程登录5分钟内3次密码验证失败，禁止用户IP访问主机1小时，1小时该限制自动解除，用户可>	 重新登录。
>	 因为动作文件（action.d/iptables.conf）以及日志匹配条件文件（filter.d/sshd.conf ）安装后是默认存在的。基本不用做任何修改。所有主要需要设置的就只有jail.conf文件。启用sshd服务的日志分析，指定动作阀值即可。实例文件/etc/fail2ban/jail.conf及说明如下：
>	 [DEFAULT]               #全局设置
>	 ignoreip = 127.0.0.1/8       #忽略的IP列表,不受设置限制
>	 bantime  = 600             #屏蔽时间，单位：秒
>	 findtime  = 600             #这个时间段内超过规定次数会被ban掉
>	 maxretry = 3                #最大尝试次数
>	 backend = auto            #日志修改检测机制（gamin、polling和auto这三种）

>	 [sshd]                   #单个服务检查设置，如设置bantime、findtime、maxretry和全局冲突，服务优先级大于全局设置。
>	 enabled  = true             #是否激活此项（true/false）修改成 true
>	 filter   = sshd              #过滤规则filter的名字，对应filter.d目录下的sshd.conf
>	 action   = iptables[name=SSH, port=ssh, protocol=tcp]             #动作的相关参数，对应action.d/iptables.conf文件
>	 sendmail-whois[name=SSH, dest=you@example.com, sender=fail2ban@example.com, sendername="Fail2Ban"]   #触发报警的收件人
>	 logpath  = /var/log/secure   #检测的系统的登陆日志文件。这里要写sshd服务日志文件。 默认为logpath  = /var/log/sshd.log
>	 #5分钟内3次密码验证失败，禁止用户IP访问主机1小时。 配置如下
>	 bantime  = 3600   #禁止用户IP访问主机1小时
>	 findtime  = 300    #在5分钟内内出现规定次数就开始工作
>	 maxretry = 3    #3次密码验证失败

>	 启动服务
>	 [root@xuegod63 ~]# service fail2ban start
>	 启动fail2ban:                                              [确定]

>####测试：
>	 [root@xuegod63 fail2ban]# > /var/log/secure  #清日志。 从现在开始
>	 >	 [root@xuegod63 fail2ban]# /etc/init.d/fail2ban restart
>	 >	 Stopping fail2ban:                                         [  OK  ]
>	 Starting fail2ban:                                         [  OK  ]
>	 [root@xuegod63 fail2ban]# iptables -L -n  
>	 
![20160604201318.png](.\images\20160604201318.png)
>	 [root@zhoushuai69 ~]# ssh root@192.168.0.66
>	 root@192.168.0.66's password: 
>	 Permission denied, please try again.
>	 root@192.168.0.66's password: 
>	 Permission denied, please try again.
>	 root@192.168.0.66's password: 
>	 Permission denied (publickey,password).
>	 [root@zhoushuai69 ~]# ssh root@192.168.0.66
>	 ssh: connect to host 192.168.0.66 port 22: Connection refused
>	 [root@zhoushuai69 ~]# 
>	 
![20160604201148.png](.\images\20160604201148.png)
>	 [root@zhoushuai66 ~]#  iptables -nL | tail -4
>	 Chain fail2ban-SSH (1 references)
>	 target     prot opt source               destination         
>	 REJECT     all  --  192.168.0.69         0.0.0.0/0           reject-with icmp-port-unreachable 
>	 RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
>	 [root@zhoushuai66 ~]# 
>###查看fail2ban工作状态
>	 [root@xuegod63 ~]# fail2ban-client  status
>	 Status
>	 |- Number of jail:      1
>	 `- Jail list:           ssh-iptables

>	 查看具体某一项的工作状态
>	 [root@xuegod63 ~]# fail2ban-client status ssh-iptables
>	 >	 Status for the jail: ssh-iptables
>	 |- filter
>	 |  |- File list:        /var/log/secure
>	 |  |- Currently failed: 0
>	 |  `- Total failed:     3
>	 `- action
>	    |- Currently banned: 1
>	    |  `- IP list:       192.168.1.64
>	    `- Total banned:     1
![20160604201912.png](.\images\20160604201912.png)
>###解除Bantime
>	 [root@xuegod63 fail2ban]# > /var/log/secure  #清日志。 从现在开始
>	 [root@xuegod63 fail2ban]# /etc/init.d/fail2ban restart
>	 Stopping fail2ban:                                         [  OK  ]
>	 Starting fail2ban:                                         [  OK  ]
>	 [root@xuegod63 fail2ban]# iptables -L -n  
>	 启动服务
>	 [root@xuegod63 ~]# service fail2ban start
>	 启动fail2ban:                                              [确定]

