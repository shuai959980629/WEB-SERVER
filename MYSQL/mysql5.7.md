>#RHEL6.5源码安装mysql-5.7.11
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
>###添加用户和组
>	 [root@zhoushuai ~]# groupadd mysql
>	 [root@zhoushuai ~]# useradd -M -s /sbin/nologin -r -g mysql mysql
>###创建安装目录和数据存放目录
>添加设备  分区  格式化（创建文件系统）  创建挂载点   挂载    修改配置文件
>
>	 [root@zhoushuai ~]# fdisk -l		#查看可用存储设备
>	 [root@zhoushuai ~]# fdisk /dev/sdb
>	 [root@zhoushuai ~]# mkfs.ext4 /dev/sdb1　　	或mkfs -t ext4 /dev/sdb1	#RHEL6 格式化
>	 [root@zhoushuai ~]# mkdir /server
>	 [root@zhoushuai ~]# mount /dev/sdb1 /server/
>	 开机自动挂载
>	 [root@zhoushuai ~]# echo "/dev/sdb1 /server ext4 defaults 0 0"  >> /etc/fstab
>	 注：mysql-5.7.11.tar.gz安装时占用空间比较大，虚拟机环境下建议新添加一块硬盘进行安装，真实服务器中不需要
>	 [root@zhoushuai ~]# mkdir -p /server/mysql/data
>###下载和安装
>####下载地址：
http://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.11.tar.gz
http://liquidtelecom.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
>>注意：MySQL从5.7版本之后，boost是必须的，建议把系统自带的boost库卸载，源码编译安装高版本
>>
>>	 [root@zhoushuai ~]# yum -y remove boost-*
>>卸载系统自带的mysql
>>
>>	 [root@zhoushuai ~]# yum -y remove mysql
>>####安装必要的资源包
>>建议使用网络yum源，RHEL6.5光盘中自带的软件包版本不够，mysql-5.7.11.tar.gz的编译对软件包的版本要求比较高，其中cmake的版本要不低于2.8
>>
>>	 [root@zhoushuai ~]# yum -y install gcc gcc-c++ autoconf automake zlib* libxml* ncurses-devel libtool-ltdl-devel* make cmake
>####上传源码包包到/usr/local/src/目录下或者直接wget进行下载源码包
>	 [root@zhoushuai ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-27.1.el6.x86_64.rpm
>####解压源码包
>	 [root@zhoushuai ~]# cd /usr/local/src/
>	 [root@zhoushuai src]# tar zxf boost_1_59_0.tar.gz
>	 [root@zhoushuai src]# tar zxf mysql-5.7.11.tar.gz
>####安装
>	 [root@zhoushuai src]# mv boost_1_59_0 boost    #修改boost解压目录名称
>	 [root@zhoushuai src]# cd mysql-5.7.11
>	 [root@zhoushuai mysql-5.7.11]# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
>	 -DMYSQL_DATADIR=/server/mysql/data/ \ #数据文件目录
>	 -DSYSCONFDIR=/etc \ #初始化参数文件目录
>	 -DWITH_MYISAM_STORAGE_ENGINE=1 \
>	 -DWITH_INNOBASE_STORAGE_ENGINE=1 \
>	 -DWITH_MEMORY_STORAGE_ENGINE=1 \
>	 -DWITH_READLINE=1 \
>	 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \ #socket文件路径，默认/tmp/mysql.sock
>	 -DMYSQL_TCP_PORT=3306 \
>	 -DENABLED_LOCAL_INFILE=1 \
>	 -DWITH_PARTITION_STORAGE_ENGINE=1 \
>	 -DEXTRA_CHARSETS=all \
>	 -DDEFAULT_CHARSET=utf8 \
>	 -DDEFAULT_COLLATION=utf8_general_ci \
>	 -DDOWNLOAD_BOOST=1 \
>	 -DWITH_BOOST=/usr/local/src/boost \
>	 -DMYSQL_USER=mysql
___________________________________________________
>	 [root@zhoushuai mysql-5.7.11]# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql  -DMYSQL_DATADIR=/server/mysql/data/  -DSYSCONFDIR=/etc -DWITH_MYISAM_STORAGE_ENGINE=1  -DWITH_INNOBASE_STORAGE_ENGINE=1  -DWITH_MEMORY_STORAGE_ENGINE=1  -DWITH_READLINE=1  -DMYSQL_UNIX_ADDR=/tmp/mysql.sock  -DMYSQL_TCP_PORT=3306  -DENABLED_LOCAL_INFILE=1  -DWITH_PARTITION_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all  -DDEFAULT_CHARSET=utf8  -DDEFAULT_COLLATION=utf8_general_ci  -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/src/boost -DMYSQL_USER=mysql
>	 编译
>	 mysql-5.7.11.tar.gz编译时会占用大量的系统资源，建议使用多个核心同时进行编译，否则可能会编译失败
>	 [root@zhoushuai mysql-5.7.11]# make -j 4
>	 查看服务器CPU核心数
>	 [root@zhoushuai ~]# grep processor /proc/cpuinfo | wc -l # 4
>	 安装
>	 [root@zhoushuai mysql-5.7.11]# make install
>	 修改目录权限
>	 [root@zhoushuai ~]# chown -R mysql:mysql /usr/local/mysql/
>	 [root@zhoushuai ~]# chown -R mysql:mysql /server/mysql/data/
>	 [root@zhoushuai ~]# chmod 777 /tmp

>	 生成配置文件
>	 [root@zhoushuai ~]# mv /etc/my.cnf{,.bak}
>	 [root@zhoushuai ~]# cp /usr/local/mysql/support-files/my-default.cnf  /etc/my.cnf
>	 生成服务启动脚本
>	 [root@zhoushuai ~]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
>	 [root@zhoushuai ~]# chmod +x /etc/init.d/mysqld
>	 [root@zhoushuai ~]# chkconfig --add mysqld
>	 [root@zhoushuai ~]# chkconfig mysqld on
>	 [root@zhoushuai ~]# chkconfig --list mysqld
>	 mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
>	 初始化数据库
>	 [root@zhoushuai ~]# /usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/server/mysql/data
>	 启动服务
>	 [root@zhoushuai ~]# service mysqld start
>	 Starting MySQL                                             [  OK  ]
>	 添加path路径： vim /etc/profile 添加下面2行 在文件的结尾
>	 export MYSQL_HOME=/usr/local/mysql
>	 export PATH=$PATH:$MYSQL_HOME/bin
>	 使修改生效
>	 source /etc/profile
>	 或
>	 [root@zhoushuai ~]# ln -s /usr/local/mysql/bin/* /usr/local/bin/
>	 修改mysql密码：
>	 [root@zhoushuai ~]# mysqladmin -u root password "123456"
>	 [root@zhoushuai ~]# mysql
>	 mysql> set password=password('123456');
>




