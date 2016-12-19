>#编译安装PHP7并安装Redis
>###方法一.下载[redis-3.0.0](http://pecl.php.net/get/redis-3.0.0.tgz)
>	 [root@zhoushuai ~]# cd /usr/local/src/
>	 [root@zhoushuai src]# tar xvf redis-3.0.0.tgz && cd redis-3.0.0
>	 [root@zhoushuai redis-3.0.0]# /server/php/bin/phpize
>	 [root@zhoushuai redis-3.0.0]# ./configure --with-php-config=/server/php/bin/php-config
>	 [root@zhoushuai redis-3.0.0]# make -j 4 && make install
>	 [root@zhoushuai redis-3.0.0]# ls /server/php/lib/php/extensions/no-debug-zts-20151012/
>	 memcache.so  opcache.a  opcache.so  redis.so
>###1、编译redis
>	 [root@zhoushuai src]# wget -c https://github.com/phpredis/phpredis/archive/php7.zip
>	 [root@zhoushuai src]# unzip php7.zip
>	 [root@zhoushuai src]# cd phpredis-php7/
>	 [root@zhoushuai phpredis-php7]# /server/php/bin/phpize
>	 [root@zhoushuai phpredis-php7]# ./configure --with-php-config=/server/php/bin/php-config
>	 [root@zhoushuai phpredis-php7]# make -j 4 && make install
>	 [root@zhoushuai phpredis-php7]# ls /server/php/lib/php/extensions/no-debug-zts-20151012/
>	 memcache.so  opcache.a  opcache.so  redis.so
>###2、修改php.ini文件，加载Redis组件。
>	 [root@zhoushuai src]# vim /server/php/php.ini
>	 [redis]
>	 extension_dir = "/server/php/lib/php/extensions/no-debug-zts-20151012/"
>	 extension=redis.so
>###3、重启。
>	 [root@zhoushuai src]# /etc/init.d/php-fpm restart





wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz  
sudo tar xzvf tcl8.6.1-src.tar.gz  -C /usr/local/  
cd  /usr/local/tcl8.6.1/unix/  
sudo ./configure  
sudo make  
sudo make install   






