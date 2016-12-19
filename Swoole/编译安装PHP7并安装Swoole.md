>#编译安装PHP7并安装Swoole
>###方法一.下载[swoole-1.8.8-stable](http://pecl.php.net/get/swoole-1.8.8.tgz)
>	 [root@zhoushuai ~]# cd /usr/local/src/
>	 [root@zhoushuai src]# wget -c http://pecl.php.net/get/swoole-1.8.8.tgz
>	 [root@zhoushuai src]# tar zxvf swoole-1.8.8.tgz && cd swoole-1.8.8
>	 [root@zhoushuai swoole-1.8.8]# /server/php/bin/phpize
>	 [root@zhoushuai swoole-1.8.8]# ./configure --with-php-config=/server/php/bin/php-config
>	 [root@zhoushuai swoole-1.8.8]# make -j 4 && make install
>	 [root@zhoushuai swoole-1.8.8]# ls /server/php/lib/php/extensions/no-debug-zts-20151012/
>	  memcached.so  memcache.so  opcache.a  opcache.so  redis.so  swoole.so
>###2、修改php.ini文件，加载swoole组件。
>	 [root@zhoushuai src]# vim /server/php/php.ini
>	 [swoole]
>	 extension_dir = "/server/php/lib/php/extensions/no-debug-zts-20151012/"
>	 extension=swoole.so
>###3、重启。
>	 [root@zhoushuai src]# /etc/init.d/php-fpm restart
>	 [root@zhoushuai src]# php -m | grep swoole
>	 swoole







