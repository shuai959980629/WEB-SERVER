#DDos
>###使用DDOS脚本防止DDOS攻击

>###使用DDOS脚本防止DDOS攻击：
>###DDOS概述：
	分布式拒绝服务(DDoS:Distributed Denial of Service)攻击，指借助于客户/服务器技术，将多个计算机联合起来作为攻击平台，对一个或多个目标发动DDoS攻击，从而成倍地提高拒绝服务攻击的威力。
>###攻击原理如图：
![1.png](.\images\1.png)
![2.png](.\images\2.png)
>###如何查是否受到DDOS攻击？
>	 通过：netstat  查看网络连接数。如果一个IP地址对服务器建立很多连接数（比如一分钟产生了100个连接），就认为发生了DDOS
>	 [root@xuegod63 html]# vim  ddos-test.sh   #写入以下内容
>	 #!/bin/bash
>	 netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
>注释：
>	 #!/bin/bash
>	 netstat -ntu | awk '{print $5}' | cut -d: -f4    | sort | uniq -c | sort -n            
>              截取外网IP和端口     截取外网的IP以：为分隔符  ｜排序 ｜ 排除相同的记录  ｜ 排序并统计
>###实战：模拟DDOS攻击

>###安装web服务器：
>	 [root@xuegod63 ~]# yum install httpd
>	 >	 生成默认首页：
>	 [root@xuegod63 ~]# cp /etc/passwd /var/www/html/index.html
>	 启动web服务器
>	 [root@xuegod63 ~]# service httpd restart
>	 Stopping httpd:                                            [FAILED]
>	 Starting httpd:                                            [  OK  ]
>	 [root@xuegod63 ~]#


>###测试，模拟DDOS

>#####ab命令：做压力测试的工具和性能的监控工具
>	 扩展：
>	 ie6 ，  每个页面，都一个进程。 

>	 语法： ab  -n 要产生的链接数总和   -c 同时打的客户端数量  http：//链接
>	 安装ab命令：
>	 [root@xuegod63 ~]# rpm -qf `which  ab `
>	 httpd-tools-2.2.15-15.el6.x86_64

>	 登录xuegod64,：
>	 [root@xuegod64 ~]# ab -n 1000 -c 10 http://192.168.1.63/index.html  #访问一个页面比较大，页面越大，消耗服务器带宽就越大
>	 [root@xuegod63 ~]# ./ddos-test.sh    #检查DDOS
>	       1 Address
>	       1 servers)
>	       5 
>	    4025 192.168.1.64


>####防止DDOS：
>	 方法一： 手动写iptables 规则，ip地址数比较少时
>	 方法二: 检测到访问次数比较多的ip地址后，自动添加iptables规则。
>	 如fail2ban或linux+DDoS deflate
>	 DDoS deflate介绍
>	 DDoS deflate是一款免费的用来防御和减轻DDoS攻击的脚本。它通过netstat监测跟踪创建大量网络连接的IP>	 地址，在检测到某个结点超过预设的限制时，该程序会通过APF或IPTABLES禁止或阻挡这些IP.
>	 DDoS deflate官方网站：http://deflate.medialayer.com/     被屏蔽，有可能打不开。

>####实战： 使用DDoS deflate 解决服务器被DDOS攻击的问题
>####检测是否有DDOS攻击
>	 执行：
>	 netstat -ntu | awk '{print $5}' | cut -d: -f4 | sort | uniq -c | sort -n
>	 如果发现某个IP连接数据上百的链接，说明就有DDOS攻击。

>	 IDC  机房，你们家服务器，大量往外发送数据包
>	 阿里云主机有监测机制，如果你的云主机大量对外发包，阿里云会给你发短信，处理不及时，把你的云主机就封了。


>####下面开始安装DDos deflate
>	 1、安装DDoS deflate
>	 [root@xuegod63 ~]#wget http://www.inetbase.com/scripts/ddos/install.sh
>	 [root@xuegod63 ~]# chmod 700 install.sh    //添加权限
>	 [root@xuegod63 ~]#./install.sh             //安装
>####Installing DOS-Deflate 0.6


>	 Downloading source files.........done

>	 >	 Creating cron to run script every minute.....(Default setting).....done

>	 Installation has completed.
>	 Config file is at /usr/local/ddos/ddos.conf
>	 >	 Please send in your comments and/or suggestions to zaf@vsnl.com

>	 ##############################################################################
>	 ##############################################################################
>	 #                       "Artistic License"                                   #
>	 #                                                                            #
>	 #                           Preamble                                         #
>	 #                                                                            #
>	 # The intent of this document is to state the conditions under which a       #
>	 # Package may be copied, such that the Copyright Holder maintains some       #
>	 >	 q 输入q 退出。

>	 DDoS deflate安装路径： 
>	 [root@xuegod63 ~]# ls /usr/local/ddos/
>	 配置文件：
>	 [root@xuegod63 ~]# ls /usr/local/ddos/ddos.conf 
>	 >	 /usr/local/ddos/ddos.conf

>	 [root@xuegod63 ~]# cat /usr/local/ddos/ignore.ip.list    #IP地址白名单
>	 127.0.0.1

>	 [root@xuegod63 ~]# vim /usr/local/ddos/ddos.conf   #查看
>	 ##### Paths of the script and other files
>	 PROGDIR="/usr/local/ddos"
>	 PROG="/usr/local/ddos/ddos.sh"  #要执行的DDOS脚本
>	 IGNORE_IP_LIST="/usr/local/ddos/ignore.ip.list"  //IP地址白名单，注：在这个文件中IP不受控制。
>	 >	 CRON="/etc/cron.d/ddos.cron"    //定时执行程序

>	 查看定时任务：
>	 [root@xuegod63 ~]# cat  /etc/cron.d/ddos.cron 
>	 SHELL=/bin/sh
>	 0-59/1 * * * * root /usr/local/ddos/ddos.sh >/dev/null 2>&1
>	 >	 注：每分钟查看一下，是不是有ddos攻击，如果发现就开始拒绝

>	 扩展：
>	 install.sh  #DDOS deflate安装脚本的功能：1，自动下载文件进行安装 2，自动执行。 
>	 联想：木马程序。 变种成木马程序。
>	 最重要的是：他的这个计划任务，你查不到。 

>	 实战： 如果1分钟内，一个IP地址对我们服务器访问150次以上，就认为发生DDOS，使用iptables把这个IP地址自动屏蔽掉。

>	 [root@xuegod63 ~]# vim /usr/local/ddos/ddos.conf
>	 配置文件中的注释如下：
>	 ##### frequency in minutes for running the script
>	 ##### Caution: Every time this setting is changed, run the script with --cron
>	 #####          option so that the new frequency takes effect
>	 >	 FREQ=1   //检查时间间隔，默认1分钟

>	 ##### How many connections define a bad IP? Indicate that below.
>	 NO_OF_CONNECTIONS=150     //最大连接数，超过这个数IP就会被屏蔽，一般默认即可

>	 ##### APF_BAN=1 (Make sure your APF version is atleast 0.96)
>	 ##### APF_BAN=0 (Uses iptables for banning ips instead of APF)
>	 APF_BAN=1        //使用APF还是iptables。推荐使用iptables,将APF_BAN的值改为0即可。
>	 改：19 APF_BAN=1
>	 为：19 APF_BAN=0
>	 ##### KILL=0 (Bad IPs are'nt banned, good for interactive execution of script)
>	 ##### KILL=1 (Recommended setting)
>	 KILL=1   //是否屏蔽IP，默认即可

>	 ##### An email is sent to the following address when an IP is banned.
>	 ##### Blank would suppress sending of mails
>	 EMAIL_TO=mk@xuegod.cn   //当IP被屏蔽时给指定邮箱发送邮件报警，换成自己的邮箱即可

>	 ##### Number of seconds the banned ip should remain in blacklist.
>	 BAN_PERIOD=600    //禁用IP时间，默认600秒，可根据情况调整
>	 用户可根据给默认配置文件加上的注释提示内容，修改配置文件。

>	 注：安装后，不需要手动运行任何软件，因为有crontab计划任务，每过一分钟，会行自动执行一次。检查是否有不正常的访问量

>	 测试：
>	 在xuegod64上模拟DDOS
>	 [root@xuegod64 html]# ab -n 1000 -c 10 http://192.168.1.63/index.html 
>	 立即查看防火墙规则：
>	 [root@xuegod63 ~]# iptables -L -n
>	 Chain INPUT (policy ACCEPT)
>	 target     prot opt source               destination         

>	 Chain FORWARD (policy ACCEPT)
>	 >	 target     prot opt source               destination         

>	 Chain OUTPUT (policy ACCEPT)
>	 target     prot opt source               destination         
>	 [root@xuegod63 ~]#

>	 等待一分钟后查看结果。
>	 改：
>	 [root@xuegod63 ~]# vim /usr/local/ddos/ddos.sh  #把原来的f1改为f4 ，第四列

>	 一键卸载：
>	 [root@xuegod63 ~]#wget http://www.inetbase.com/scripts/ddos/uninstall.ddos
>	 [root@xuegod63 ~]# chmod  +x uninstall.ddos
>	 [root@xuegod63 ~]# ./uninstall.ddos 

>	 Uninstalling DOS-Deflate
>	 Deleting script files.........done

>	 Deleting cron job.......done

>	 Uninstall Complete

>	 排查系统级别crontab：
>	 黑客：高级crontab ，篡改一个系统级别的计划任务

>	 提前成生成md5数据库文件：
>	 对/etc/cron*下所有文件都生成md5值
>	 [root@xuegod63 ~]# find /etc/cron* -type f -exec md5sum {} \; > /usr/share/file_md5.v1
>	 [root@xuegod63 ~]# cp /etc/passwd /etc/cron.d/ddos.sh
>	 [root@xuegod63 ~]# find /etc/cron* -type f -exec md5sum {} \; >/usr/share/file_md5.v2
>	 [root@xuegod63 ~]# diff /tmp/file_md5.v1 /tmp/file_md5.v2   #对比

>###重启：/usr/local/ddos/ddos.sh -c
>
>###如以上方法都无效可安装国外大牛修改版本 
>	 wget http://www.mattzuba.com/wordpress/wp-content/uploads/2011/02/ddos_deflate-0.7.tar.gz
>	 tar -xf ddos_deflate-0.7.tar.gz
>	 cd ddos_deflate-0.7
>	 sudo ./install.sh
>	 卸载的话直接sudo ./uninstall.sh 
>	 网盘备用地址：http://pan.baidu.com/s/1sjsed2X
>	 重启：/usr/local/ddos/ddos.sh -c
>###实验
>	 [root@zhoushuai69 ~]# ab -n 1000 -c 10 http://192.168.0.66/index.php
![20160604224900.png](.\images\20160604224900.png)

>	 [root@zhoushuai66 ~]# iptables -L -n
![20160604225023.png](.\images\20160604225023.png)

>###实验结果
![111120160604230354.png](.\images\111120160604230354.png)

>	 [root@zhoushuai66 ~]# service crond restart
>	 [root@zhoushuai66 ~]# cat /usr/local/ddos/ignore.ip.list



