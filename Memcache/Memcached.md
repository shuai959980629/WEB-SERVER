>#PHP7安装Memcache+Memcached
>###1、安装Memcache扩展
>	 [root@zhoushuai src]# wget -c https://github.com/websupport-sk/pecl-memcache/archive/php7.zip
>	 [root@zhoushuai src]# unzip php7.zip
>	 [root@zhoushuai src]# cd pecl-memcache-php7/
>	 [root@zhoushuai pecl-memcache-php7]# /server/php/bin/phpize
>	 [root@zhoushuai pecl-memcache-php7]# ./configure --with-php-config=/server/php/bin/php-config
>	 [root@zhoushuai pecl-memcache-php7]# make -j 4 && make install
>	 [root@zhoushuai pecl-memcache-php7]# ls /server/php/lib/php/extensions/no-debug-zts-20151012/
>	 memcache.so  opcache.a  opcache.so  redis.so
>###2、修改php.ini文件，加载memcache组件。
>	 [root@zhoushuai src]# vim /server/php/php.ini
>	 [memcache]
>	 extension_dir = "/server/php/lib/php/extensions/no-debug-zts-20151012/"
>	 extension=memcache.so
>###1.安装Memcached扩展
>	 [root@zhoushuai src]# wget -c https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
>	 [root@zhoushuai src]# tar -zvxf libmemcached-1.0.18.tar.gz
>	 [root@zhoushuai src]# cd libmemcached-1.0.18/
>	 [root@zhoushuai libmemcached-1.0.18]# ./configure --prefix=/usr/local/libmemcached
>	 [root@zhoushuai libmemcached-1.0.18]# make -j 4 && make install
>	 
>	 [root@zhoushuai src]# wget -c https://github.com/shuai959980629/php-memcached/archive/php7.zip
>	 [root@zhoushuai src]# unzip php7.zip
>	 [root@zhoushuai src]# cd php-memcached-php7/
>	 [root@zhoushuai php-memcached-php7]# /server/php/bin/phpize
>	 [root@zhoushuai php-memcached-php7]# ./configure --with-php-config=/server/php/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached --disable-memcached-sasl
>	 [root@zhoushuai php-memcached-php7]# make -j 4 && make install
>	 [root@zhoushuai php-memcached-php7]# ls /server/php/lib/php/extensions/no-debug-zts-20151012/
>	  memcached.so  memcache.so  opcache.a  opcache.so  redis.so
>	 [root@zhoushuai php-memcached-php7]# 
>	 
>###2、修改php.ini文件，加载memcached组件。
>	 [root@zhoushuai src]# vim /server/php/php.ini
>	 [memcached]
>	 extension_dir = "/server/php/lib/php/extensions/no-debug-zts-20151012/"
>	 extension=memcached.so
>	 
>###3、重启。
>	 [root@zhoushuai src]# /etc/init.d/php-fpm restart
>###4、配置Memcached的步骤，首先安装Libevent事件触发管理器。
>	 [root@zhoushuai src]# tar vxf libevent-2.0.22-stable.tar.gz
>	 [root@zhoushuai src]# cd libevent-2.0.22-stable/
>	 [root@zhoushuai libevent-2.0.22-stable]# ./configure --prefix=/usr/local/libevent && make -j 4 && make install
>###5、编译Memcached
>	 [root@zhoushuai src]# tar vxf  memcached-1.4.29.tar.gz
>	 [root@zhoushuai src]# cd memcached-1.4.29/
>	 [root@zhoushuai memcached-1.4.29]# ./configure --prefix=/usr/local/memcached --with-libevent=/usr/local/libevent
>	 [root@zhoushuai memcached-1.4.29]# make -j 4 && make install
>###6、启动Memcached
>	 [root@zhoushuai memcached-1.4.29]# /usr/local/memcached/bin/memcached -u root -p 11211 -l 192.168.137.62 -P /var/run/memcached.pid -m 128m -c 2048 -d 
>	 //u user  ； -p port  ；  -l listen  ；  -P pid  ；  -m 内存缓存大小 ；-c 最大并发 ； -d 作为守护进程在后台启动
>###7、查看是否生效
>	 [root@zhoushuai memcached-1.4.29]# ps -aux | grep memcached
>	 root      84713  0.0  0.0 323136  1076 ?        Ssl  13:29   0:00 /usr/local/memcached/bin/memcached -u root -p 11211 -l 192.168.137.62 -P /var/run/memcached.pid -m 128m -c 2048 -d
>	 root      84728  1.0  0.0 112660   964 pts/2    S+   13:33   0:00 grep --color=auto memcached
>	 [root@zhoushuai memcached-1.4.29]# 
>	 [root@zhoushuai memcached-1.4.29]# netstat  -antup | grep 11211
>	 tcp        0      0 192.168.137.62:11211    0.0.0.0:*               LISTEN      84713/memcached     
>	 udp        0      0 192.168.137.62:11211    0.0.0.0:*                           84713/memcached     
>	 [root@zhoushuai memcached-1.4.29]# 
>###8、开机/重启后生效，编辑 /etc/rc.d/rc.local 文件，添加以下内容
>	 [root@zhoushuai memcached-1.4.29]# echo "/usr/local/memcached/bin/memcached -u root -p 11211 -l 192.168.137.62 -P /var/run/memcached.pid -m 128m -c 2048 -d" >> /etc/rc.local
>###9.测试：
>		<?php
            $mem = new Memcache;
            $mem->connect('192.168.137.62',11211) or die("Could not connet");
            $key = md5("ilove");
            $mem->add($key,"iloveyou forever",0,30);
       ?>
       string(16) "iloveyou forever"
>###10完成。







