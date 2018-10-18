# vi文本编辑器
- vim是vi的增强版
- 插入命令
	
![插入命令](https://upload-images.jianshu.io/upload_images/14467627-014d81926a08de40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 定位命令
	
	![定位命令](https://upload-images.jianshu.io/upload_images/14467627-d50f1b5723512cf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	
- 删除命令
	- dd:删除当前行

	- ndd:删除光标所在当前行向下数n行
	- D:删除当前行光标所在的位置后面的字符
	- x:向后删除光标所在位置的字符
	- X:向前删除光标前面的字符
	- nX:删除前面的n个字符,光标所在的字符将不会被删
- 复制和粘贴命令
	- yy或Y:复制当前行
	- nyy或nY:复制以下n行
	- p:在光标后面插入buffer中的内容
	- P:在光标前面插入buffer中的内容
- 替换和撤销命令
	- r:取代光标所在处的字符
	- R:从光标所在处开始替换字符，按esc结束
	- u:撤销上一步操作
	- 定位命令
	
![定位命令](https://upload-images.jianshu.io/upload_images/14467627-1e2a9e7325700860.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 查找操作
	- :set nu	设置行号
	- :set nonu	取消行号
	- :n	移到第n行
	- :/查找的关键字
	
- 替换操作
	- :s /old/new	将当前行中查找到的第一个字符“old” 串替换为“new”
	- :s /old/new/g	将当前行中查找到的所有字符串“old” 替换为“new”
	- :#,# s/old/new/g 	在行号“#,#”范围内替换所有的字符串“old”为“new”
	- :% s/old/new/g	在整个文件范围内替换所有的字符串“old”为“new”
	- :%s/old/new	查找文件中所有行第一次出现的old，替换为new
- 其他命令
	- :W[文件路径]保存当前文件
	- :q 如果未对文件做改动则退出
	- :q! 放弃存储名退出
	- :wq或:x 保存退出

- 可视模式
	- v:可视模式
	- V:可视行模式
	- Ctrl+v:可视块模式
	- 注意:
		- 在所有可视模式中,d和x键可以用删除选定的内容
		- 在可视块模式中,选中所需行,按I键输入内容,之后按两次esc键,可在所有选定行光标处添加同样的内容.
# 用户管理命令
- 用户管理命令

![用户管理命令](https://upload-images.jianshu.io/upload_images/14467627-5e5b44bf245d4f55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例子含义:创建用户id为888,所属组为users,且还指定了组sys,root的用户zhangsan,其描述为"hr zhang"用户,设置密码
	- useradd -u 888 -g users -G sys,root -c "hr zhang" zhangsan
	- passwd zhangsan
		- 命令格式:passwd   [选项]  <用户名>
		
	- 常用命令选项
	
	![常用命令选项](https://upload-images.jianshu.io/upload_images/14467627-6c1868cb543a156b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	
- 修改已有用户属性
	- 修改用户命令:usermod (user modify)
		- -l 修改用户名(login); 例子:usermod -l a b（b改为a）
		- -g 添加组 ;例子:usermod -g sys tom
		- -G 添加多个组 ;例子:usermod -G sys,root tom
		- –L 锁定用户账号密码(Lock);
		- –U 解锁用户账号(Unlock)
	- 删除用户命令:userdel(user delete)
		- -r 删除账号时同时删除目录(remove)[把用户的宿主目录一并删除]

# 组命令管理
- 添加组:groupadd
	- -g 指定gid
- 修改组:groupmod
	- -n 更改组名(new group)
- 删除组: groupdel
- groups 显示用户所属组
- 修改用户组的属性
	- 命令格式: groupmod [选项]  <用户名>
	- 常用命令选项
		- -g: 设置想要使用的GID
		- -o: 使用已经存在的GID
		- -n: 设置想要使用的群组名称
- 设置组帐号密码(极少用),添加/删除组成员
	- 命令格式:gpasswd  [选项]  组帐号名
	- 常用命令选项
		- -a: 向组内添加一个用户
		- -d: 从组内删除一个用户成员
		- -M: 定义组成员列表,以逗号分隔

- 删除组账号
	- 命令格式：groupdel   <组账号名>
	- 注意：只能删除那些没有被任何用户指定为主组的组.
- 显示用户所属组
	- 命令格式: groups	[用户名]
# 文件目录权限管理
- 权限
	- 读(r): 读取文件的内容;列出目录里的对象
	- 写(w):允许修改文件；在目录里面新建或者删除文件
	- 执行(x):允许执行文件;允许进入目录里
- 数字权限

	- 除了用字母rwx来表示权限,还可以使用3位数字来表 达文件或目录的权限
		- 读: 4
		- 写:2
		- 执行:1
- chmod命令(change mode)
	- 格式
		- chmod [ugoa] [+-=] [rwx] file/dir 或 chmod nnn file/dir
	- 参数解释
		- u:属主  g:属组  o:其他用户  a:所有用户
		- +:添加权限  -:删除权限  =:赋予权限
		- nnn:三位八进制的权限
	- 常用命令选项
		- -R 递归修改指定目录下的所有子文件及文件夹的权限
		- -f 强制改变文件访问特权;如果是文件的拥有者,则得不到任何错误信息
	- 例子: chmod g+w ll.txt
	- 用数字表示权限(r=4，w=2，x=1，-=0)
		- 例如: chmod  750  b.txt;rwx用二进制表示是111,十进制4+2+1=7
; r-x用二进制表示是101,十进制4+0+1=5
- chown命令
	- 格式:
	- chown 属主 file/dir 
	- chown :属组 file/dir
	- chown 属主:属组 file/dir
	- 常用命令选项
		- -R:递归的修改指定目录下所有文件,子目录的归属
