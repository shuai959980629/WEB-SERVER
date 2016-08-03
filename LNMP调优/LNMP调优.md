>#LNMP调优归纳总结
>###一、Nginx编译前的优化
>
>
>
>
>###二、Nginx编译安装完成的优化
>#####1.Nginx运行进程个数[worker_processes](https://www.nginx.com/resources/wiki/start/topics/examples/fullexample2/?highlight=worker_processes)
>	 [root@zhoushuai zhoushuai]# vim /usr/local/nginx/conf/nginx.conf
>	 worker_processes  2; #一般我们设置CPU的核心或者核心数x2,如果你不了解，top命令之后按1也可以看出来
>	 [root@zhoushuai zhoushuai]# /etc/init.d/nginx  reload
>	 [root@zhoushuai zhoushuai]# ps -aux | grep nginx
>	 root       1564  0.0  0.0  22904  1756 ?        Ss   09:45   0:00 nginx: master process /usr/local/nginx/sbinnginx
>	 nginx      3252  0.0  0.0  24988  1704 ?        S    11:25   0:00 nginx: worker process
>	 nginx      3253  0.0  0.0  24988  1704 ?        S    11:25   0:00 nginx: worker process
>	 root       3256  0.0  0.0 112664   968 pts/1    S+   11:25   0:00 grep --color=auto nginx
>#####2.Nginx运行CPU亲和力[worker_cpu_affinity](https://www.nginx.com/resources/wiki/start/topics/examples/SSL-Offloader/?highlight=worker_cpu_affinity)
>	 这个要根据你的CPU线程数配置
>	 比如4核4线程配置
>	 [root@zhoushuai ~]# vim /usr/local/nginx/conf/nginx.conf
>	 worker_processes  4;
>	 worker_cpu_affinity 0001 0010 0100 1000;			
>	 比如8核8线程配置
>	 worker_processes  8;
>	 worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;			
>	 那么如果我是4线程的CPU，我只想跑两个进程呢？
>	 worker_processes  2;
>	 worker_cpu_affinity 0101 1010;			
>	 意思就似乎我开启了第一个和第三个内核，第二个和第四个内核，两个进程分别在这两个组合上轮询！worker_processes最多开启8个，8个以上性能提升不会再提升了，而且稳定性变得更低，所以8个进程够用了。
>#####3.Nginx最多可以打开文件数[worker_rlimit_nofile](https://www.nginx.com/resources/wiki/start/topics/examples/full/?highlight=worker_rlimit_nofile)
>	 worker_rlimit_nofile 102400;
>	 这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n的值保持一致。
>#####4.Nginx事件处理模型
>	 events {
>	     use epoll;
>	     worker_connections  1024;
>	 }
>	 知道在linux下nginx采用epoll事件模型，处理效率高，关于epoll的时间处理其他只是，可以自行百度，了解即可！
>	 Work_connections是单个进程允许客户端最大连接数，这个数值一般根据服务器性能和内存来制定，也就是单个进程最大连接数，实际最大值就是work进程数乘以这个数
>	 如何设置，可以根据设置一个进程启动所占内存，top -u nginx，但是实际我们填入一个102400，足够了，这些都算并发值，一个网站的并发达到这么大的数量，也算一个大站了！




