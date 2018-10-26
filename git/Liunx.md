# 一.创建新的虚拟机步骤
- 记得添加CD/DVD的文件路径(即CentOS-7-x86 64-DVD的路径)或者在创建的时候直接添加也可以
- 新建时,记得网络连接选择桥接网络.
- 开启新建虚拟机,进行设置.
	- 1.语言选择中文简体
	- 2.在安装信息摘要里,点击安装位置进去,然后不做任何修改,直接完成
	- 3.点击软件选择,只执行命令测试可以选择最小安装,右侧全选;若是想选择友好的图形交互界面,可以选择GNOME桌面,右侧全选,点击完成.
	- 4.选择网络和主机名选项,进入,打开以太网开关,点击完成
	- 5.然后点击右下角的安装,在安装进行界面,点击ROOT密码,进行root密码设置,设置完成后,等待安装成功
	
# 二.Linux命令
- Linux命令的通用命令格式
	- 命令字  [选项]  [参数]
		- 选项: 用于调节命令的具体功能
		- 参数: 命令操作的对象，如文件、目录名等

- 目录操作命令
	- pwd,cd,ls,mkdir,du
- 文件操作命令
	- touch,file,cp,rm,mv,which,find

- 文件内容操作命令
	- cat,more,less,head,tail,wc,grep

- 压缩命令
	- gzip,bzip2,tar

- ls: 显示文件和目录列表(list)
	- 常用参数：
		- -l (long)	详细信息显示
 		- -a(all)         显示所有子目录和文件的信息，包括隐藏文件  
		- -t	(time)
		- -A	类似于"-a",但不显示"."和".."目录的信息
		- -R	递归显示内容


- du命令: 统计目录及文件的空间占用情况（estimate file space  usage）
	- -a: 统计时包括所有的文件,而不仅仅只统计目录
	- -h: 以更易读的字节单位（K、M等）显示信息
	- -s: 统计每个参数所占用空间总的大小
	
- file: 查看文件类型
	- file 文件名

- cp命令: 复制（copy）文件或目录
	- -r: 递归复制整个目录树
	- -p: 保持源文件的属性不变
	- -f: 强制覆盖目标同名文件或目录
	- -i: 需要覆盖文件或目录时进行提醒

- rm命令：删除（Remove）文件或目录
	- 格式: rm   [选项]   文件或目录
	- 常用命令选项
		- -f: 强行删除文件，不进行提醒
		- -i: 删除文件时提醒用户确认
		- -r: 递归删除整个目录树

-  mv命令: 移动（Move）文件或目录—— 如果目标位置与源位置相同,则相当于改名
	- 格式: mv   [选项]...   源文件或目录…   目标文件或目录

- which命令: 显示系统命令所在目录
	- which <选项> 命令名称
	- 常用命令选项
	- -a	将所有由PATH路径中可以找到的指令均列出,而不止第一个被找到的指令名称

- 内部命令：属于Shell解析器的一部分
	- cd 切换目录（change directory）
	- pwd 显示当前工作目录（print working directory）
	- help 帮助

- 外部命令: 独立于Shell解析器之外的文件程序
	- ls 显示文件和目录列表（list）
	- mkdir 创建目录（make directoriy）
	- cp 复制文件或目录（copy）
	
- 查看帮助文档
	- 内部命令: help + 命令（help cd）
	- 外部命令: man + 命令（man ls）



- 重启命令(root用户才可用)
	- shutdown -r now (立即重启)
	- shutdown -r 时间 (多长分钟后自动启动)
	- shutdown -r 13:30 在时间为13:30时重启
	- 如果是通过shutdown命令设置重启的话,可以用shutdown -c命令取消重启

- 关机命令
 
	- halt 	立刻关机
	- poweroff 	立刻关机
	- shutdown -h now 	立刻关机(root用户使用)
	- shutdown -h 10 	10分钟后自动关机

- Linux运行级别:
![Linux运行级别](https://upload-images.jianshu.io/upload_images/14467627-90fd4261dab3737d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	- runlevel  切换前的运行级别,当前的运行级别(第一个数字代表未切换过的运行级别)
	- init N 	N代表临时要切换到的运行级别
- 查看主机名
	- hostname

- 修改主机名
	- hostname 主机名 (一次性修改，重启后变回原来的)
	- vi /etc/hostname	(永久修改,修改的时配置文件)

- 测试网络连接
	- ping 主机名 (按Ctrl+c停止)


- 查看网络接口信息
	- 执行ifconfig命令
	- 如果执行不了,需要安装,执行yum install net-tools 命令进行安装
			- 安装途中,要选择y继续安装,如果不想进行选择,可以直接执行yum install net-tools -y命令,就不会让你进行选择了
- 重启network网络服务

	- service network restart
- 设置防火墙

	- service firewalld status	查看防火墙状态
	- systemctl stop firewalld	临时关闭
	- systemctl disable firewalld	禁止开机启动

- /etc/hosts 文件	保存主机名与IP地址的映射记录


- su 切换到root用户

- exit 退出到登录时的用户
- :q 退出
- :q! 强制退出(修改不保存)
- :wq 保存修改退出
- 进入修改时,按i就会出现inset选项,按esc退出修改
