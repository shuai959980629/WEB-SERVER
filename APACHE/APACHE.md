>#源码编译安装Apache2.4.18
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
>	  配置网络yum源
>	  [root@zhoushuai ~]# wget -O /etc/yum.repos.d/CentOS6-Base-163.repo http://mirrors.163.com/.help/CentOS6-Base-163.repo
>	  [root@zhoushuai ~]# sed -i 's/$releasever/6.8/g' /etc/yum.repos.d/CentOS6-Base-163.repo
>	  [root@zhoushuai ~]# yum clean all
>	  [root@zhoushuai ~]# yum list
>	  [root@zhoushuai ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-27.1.el6.x86_64.rpm
>###需要源码编译安装的软件包
>	  httpd-2.4.18.tar.gz			#Apache主程序包
>	  apr-1.5.2.tar.gz				#Apache依赖包
>	  apr-util-1.5.4.tar.gz			#Apache依赖包
>	  pcre-8.38.zip					#Apache依赖包
>###首先下载最新的源码包
>	  安装之前请先安装make、gcc、openssl等编译工具和开发包
>	  [root@zhoushuai ~]# yum -y install make gcc openssl
>###下载源码安装包
>	  http://apache.fayea.com//httpd/httpd-2.4.18.tar.gz
>	  http://archive.apache.org/dist/apr/apr-1.5.2.tar.gz
>	  http://archive.apache.org/dist/apr/apr-util-1.5.4.tar.gz
>	  http://iweb.dl.sourceforge.net/project/pcre/pcre/8.38/pcre-8.38.zip
>	  [root@zhoushuai ~]# cd /usr/local/src/
>	  [root@zhoushuai src]# rz
>###下编译安装依赖包apr-1.5.2.tar.gz
>	  [root@zhoushuai src]# tar zxvf apr-1.5.2.tar.gz
>	  [root@zhoushuai src]# cd apr-1.5.2
>	  [root@zhoushuai apr-1.5.2]# ./configure --prefix=/usr/local/apr
>	  [root@zhoushuai apr-1.5.2]# make && make install
>###编译安装依赖包apr-util-1.5.4.tar.gz
>	  [root@zhoushuai src]# tar zxvf apr-util-1.5.4.tar.gz
>	  [root@zhoushuai src]# cd apr-util-1.5.4
>	  [root@zhoushuai apr-util-1.5.4]# ./configure --prefix=/usr/local/apr-util/ --with-apr=/usr/local/apr/bin/apr-1-config
>	  [root@zhoushuai apr-util-1.5.4]# make && make install
>###编译安装依赖包pcre-8.38.zip
>	  [root@zhoushuai src]# unzip -o pcre-8.38.zip
>	  [root@zhoushuai src]# cd pcre-8.38
>	  [root@zhoushuai pcre-8.38]# ./configure --prefix=/usr/local/pcre
>	  [root@zhoushuai pcre-8.38]# make && make install
>###编译安装Apache
>	  [root@zhoushuai src]# tar zxvf httpd-2.4.18.tar.gz
>	  [root@zhoushuai src]# cd httpd-2.4.18
>	  [root@zhoushuai httpd-2.4.18]# ./configure --prefix=/usr/local/apache2 \
>	  --enable-so \
>	  --enable-rewrite \
>	  --enable-ssl \
>	  --with-apr=/usr/local/apr \
>	  --with-apr-util=/usr/local/apr-util/ \
>	  --with-pcre=/usr/local/pcre/
>	  [root@zhoushuai httpd-2.4.18]# make && make install
>###配置文件
>	  [root@zhoushuai ~]# ls /usr/local/apache2/conf/httpd.conf
>	  /usr/local/apache2/conf/httpd.conf
>###网站根目录
>	  [root@zhoushuai ~]# ls /usr/local/apache2/htdocs/
>	  index.html
>###生成启动脚本
>	  [root@zhoushuai ~]# cp /usr/local/apache2/bin/apachectl /etc/init.d/
>	  [root@zhoushuai ~]# chmod +x /etc/init.d/apachectl
>	  
![20160603132531.png](.\images\20160603132531.png)
>	  
![20160603132617.png](.\images\20160603132617.png)
>###将服务添加到系统启动列表中
>	  [root@zhoushuai ~]# chkconfig --add apachectl
>	  [root@zhoushuai ~]# chkconfig apachectl on
>	  [root@zhoushuai ~]# chkconfig --list apachectl
>	  apachectl       0:关闭  1:关闭  2:启用  3:启用  4:启用  5:启用  6:关闭

>###启动服务
>	  [root@zhoushuai ~]# service apachectl start
>	  错误：How to Fix AH00558 and AH00557 httpd apr_sockaddr_info_get() Error Message-》http://linux.101hacks.com/unix/httpd-apr-sockaddr-info-get-error/

>###测试访问，
>
![20160603134208.png](.\images\20160603134208.png)
>
![20160603134221.png](.\images\20160603134221.png)





