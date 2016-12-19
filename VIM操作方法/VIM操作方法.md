>#VIM使用方法
>##配置：（[root@zhoushuai /]# vim /etc/vimrc ）
>	 11 set history=50      " keep 50 lines of command line history
>	 12 set ruler       " show the cursor position all the time
>	 13 set number
>	 14 filetype on
>	 15 set autoindent
>	 16 set smartindent
>	 17 set tabstop=4
>	 18 set shiftwidth=4
>	 19 set showmatch

>
>##命令模式：
    字符操作
        i 当前字符之前插入
        I 行首插入
        a 当前字符之后插入
        A 行尾插入
        esc 退出当前模式
        o 下一行插入
        O 上一行插入
        x 向后删除一个字符		del
    	X 向前删除一个字符
    	u 撤销一步
>##行操作
        home键或^ 行首
        $行尾 	end键
        dd 删除一行 Ndd
        yy 复制一行 Nyy 复制N行
        p  将复制行粘贴 P上粘
        扩展：剪切
        先删除，再粘贴
        删除到行首    d + HOME  或^
        删除到行尾	d + END    或$
>##词操作
        dw 删除一个词，删除时要将光标移动到这个词的行首。 另外，如果光标不在行首，则删除光标之后的字母。
        yw 复制一个词
        w 切换单词
>##块操作
        大D 或d+$删至行尾 d+^ 删至行首
        y+$ 复制至尾 y+^ 复制至首
>##v 模式
        进入v模式 移动光标选择区域、
        编程的时候需要进行多行注释：
        1、注释：ctrl+v 进入列编辑模式
        2向下或向上移动光标
        3把需要注释的行的开头标记起来
        4然后按大写的I
        5再插入注释符,比如"#"。
        6再按Esc,就会全部注释了。
        删除多行注释：
        删除：再按ctrl+v 进入列编辑模式；向下或向上移动光标 ；选中注释部分,然后按d, 就会删除注释符号。
>##VIM命令行模式操作
        :w 保存 save
        :q 没有进行任何修改，退出 quit
        :q! 修改了，不保存，强制退出
        :wq 保存并退出 
        ：wq! 强制保存并退出。
        保存：
        ZZ 
        改一个字符： r   再对应文字
        替换
        :% s/this/that 每一行的第一个this被替换成that   
        :% s/this/that/g 将文本中所有的this替换成that
        :5,10 s/sbin/mk/g			#替换5到10行 的sbin
        :set nu/nonu   			#显示行号
        / 正向查找  ：/target     n 往下查找，N 往上查找
        去消高亮显示：  noh  或 随便查找一组没有的字符
        :!ifconfig	调用系统命令
        编辑文目录：
        如果不小心打开目录，直接退出就可以了。
        vim中定位到某行：
        gg  定位到行首
        G  定位到最后一行，行首
        #G 定位到某一行
        :#	定位到某一行
        #gg	定位到某一行			
        #代表行号

读取其他文件
:r /etc/ssh/sshd_config.bak

vim打开多个文件：
[root@xuegod60 ~]# vim -o /etc/passwd /etc/hosts
[root@xuegod60 ~]# vim -O /etc/passwd /etc/hosts
ctrl+WW  在文件之间进行切换

大写O左右分屏，小写的o上下分屏
自定义vim
#vim 	~/.vimrc
输入
set nu
set history=10

[root@xuegod60 ~]# gedit

实战：
在windows中编辑好的汉字文本文档，上传到Linux下打开乱码。
[root@xuegod60 ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-36.el7.x86_64.rpm
rz   上传
sz   下载

[root@xuegod60 ~]# rpm -qf `which iconv`
glibc-common-2.17-105.el7.x86_64
[root@xuegod60 ~]# rpm -ihv /mnt/Packages/glibc-common-2.17-105.el7.x86_64.rpm
通过iconv命令转码
输入/输出格式规范：
-f, --from-code=名称 原始文本编码
-o, --output=FILE 输出文件
-l, --list 列举所有已知的字符集
[root@xuegod60 ~]# iconv -f gb2312 c.txt -o c2.txt
[root@xuegod60 ~]# cat c2.txt
#!/bin/bash
echo "学神IT"
echo "学神IT"
echo "学神IT"
echo "学神IT"
echo "学神IT"

解决将公司服务器上脚本导到windows上打开串行的问题
这是因为windows和linux处理回车不同。
Linux系统中处理回车”\n”   windows系统中处理回车采用的是“\r\n”
 

方法一：通过windows 写字板打开 或者用word文档打开
方法二：
[root@xuegod60 ~]# rpm -ivh /mnt/Packages/dos2unix-6.0.3-4.el7.x86_64.rpm
warning: /mnt/Packages/dos2unix-6.0.3-4.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:dos2unix-6.0.3-4.el7             ################################# [100%]
[root@xuegod60 ~]# unix2dos b.sh
unix2dos: converting file b.sh to DOS format ...
=


