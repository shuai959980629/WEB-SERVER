>#RHEL6.5源码安装mysql-5.5.32
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
>	 [root@zhoushuai ~]# mkdir -p /server/mysql/data
>###下载和安装
>####下载地址：
>>wget http://ftp.ntu.edu.tw/pub/MySQL/Downloads/MySQL-5.5/mysql-5.5.32.tar.gz
>>
>>卸载系统自带的mysql
>>
>>	 [root@zhoushuai ~]# yum -y remove mysql
>>####安装必要的资源包
>>
>>	 [root@zhoushuai ~]# yum -y install gcc gcc-c++ autoconf automake zlib* libxml* ncurses-devel libtool-ltdl-devel* make cmake
>####上传源码包包到/usr/local/src/目录下或者直接wget进行下载源码包
>	 [root@zhoushuai ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-27.1.el6.x86_64.rpm
>####解压源码包
>	 [root@zhoushuai ~]# cd /usr/local/src/
>	 [root@zhoushuai src]# tar zxf mysql-5.5.32.tar.gz
>####安装
>	 [root@zhoushuai src]# cd mysql-5.5.32
>	 [root@zhoushuai mysql-5.5.32]# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DENABLED_LOCAL_INFILE=1 -DMYSQL_DATADIR=/server/mysql/data -DMYSQL_USER=mysql
>	 编译
>	 [root@zhoushuai mysql-5.5.32# make -j 4
>	 查看服务器CPU核心数
>	 [root@zhoushuai ~]# grep processor /proc/cpuinfo | wc -l # 4
>	 安装
>	 [root@zhoushuai mysql-5.5.32]# make install
>	 修改目录权限
>	 [root@zhoushuai ~]# chown -R mysql:mysql /usr/local/mysql/
>	 [root@zhoushuai ~]# chown -R mysql:mysql /server/mysql/data/
>	 [root@zhoushuai ~]# chmod 777 /tmp

>>	 生成配置文件
>	 [root@zhoushuai ~]# mv /etc/my.cnf{,.bak}
>	 [root@zhoushuai mysql-5.5.32]# cp /usr/local/mysql/support-files/my-large.cnf /etc/my.cnf

>>	 生成服务启动脚本
>	 [root@zhoushuai ~]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
>	 [root@zhoushuai ~]# chmod +x /etc/init.d/mysqld
>	 [root@zhoushuai ~]# chkconfig --add mysqld
>	 [root@zhoushuai ~]# chkconfig mysqld on
>	 [root@zhoushuai ~]# chkconfig --list mysqld
>	 mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
>	 启动服务
>	 [root@zhoushuai ~]# service mysqld start
>	 Starting MySQL                                             [  OK  ]
>	 设置环境变量
>	 [root@zhoushuai ~]# echo 'export PATH=/usr/local/mysql/bin:$PATH' >>/etc/profile
     source !$
>	 设置环境变量
>	 添加path路径： vim /etc/profile 添加下面2行 在文件的结尾
>	 export MYSQL_HOME=/usr/local/mysql
>	 export PATH=$PATH:$MYSQL_HOME/bin
>	 使修改生效
>	 source /etc/profile
>	 或
>	 [root@zhoushuai ~]# ln -s /usr/local/mysql/bin/* /usr/local/bin/
>	 初始化数据库
>	 [root@zhoushuai ~]# chmod +x scripts/mysql_install_db
>	 [root@zhoushuai ~]# /usr/local/mysql/scripts/mysql_install_db --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/server/mysql/data --user=mysql
>>	 初始化安全配置
>	 [root@zhoushuai ~]# mysql_secure_installation  #安全初始化配置
>	 排错
    出现这种错误
    Enter current password for root (enter for none):
    ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
    干掉mysql进程
    pkill mysqld
    rm -rf  /data/*
    重新初始化
******************************************************
>>	 MySQL安全优化小配置
>	 用户安全
>	 mysql> select user,host from mysql.user;
>	 mysql> delete from mysql.user where user='';
>	 mysql> delete from mysql.user where host='server01.cn';
>	 mysql> delete from mysql.user where host='::1';
>	 mysql> select user,host from mysql.user;
    +------+-----------+
    | user | host      |
    +------+-----------+
    | root | 127.0.0.1 |
    | root | localhost |

>> 	或者把用户都删了，添加一个额外的管理员
    mysql> delete from mysql.user;
    mysql> grant all privileges on *.* to system@'localhost' identified by '123456' with grant option;
	mysql> flush privileges;
    mysql> select user,host from mysql.user;
    +--------+-----------+
    | user   | host      |
    +--------+-----------+
    | system | localhost |
    +--------+-----------+
    1 row in set (0.00 sec)

>> 	mysql> drop database test;






