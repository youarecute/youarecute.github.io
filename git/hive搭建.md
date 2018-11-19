### hive环境的搭建
1.先把hive解压,然后在虚拟机的/etc/profile文件中配置hive环境变量.配置好后,记得运行此文件.

```
export HIVE_HOME=hive解压包路径
在path变量后面加入 :$HIVE_HOME/bin
```

2.在hive文件夹下的lib文件夹里传入mysql-connector-java-5.1.34.jar包
3.在hive解压包下的conf文件夹下创建hive-site.xml(如果没有的话创建)
4.在改文件里添加如下内容
```
<configuration>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://namenode的ip地址:3306/hivedb?createDatabaseIfNotExist=true</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>root</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>数据库的密码</value>
	</property>
</configuration>
```

6.输入hive查看是否安装搭建完成.

### hive与mysql连接的操作步骤.
###### sql的下载命令如下(按顺序执行):
```
// 安装wget的命令
yum -y install wget
// 下载网址
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
sudo yum -y install mysql
sudo yum -y install mysql-server
```

查看sql安装完成的命令如下:
```
// 启动命令
systemctl start mysqld
// 查看状态,是running才算成功
systemctl status mysqld
// 关闭命令
systemctl stop mysqld
```

###### mysql的密码修改命令
```
// 修改密码的命令(需要在user用户下修改,所以用 mysql -u root -p登录mysql)
UPDATE mysql.user SET authentication_string=PASSWORD('密码要和hive-site.xml文件内的一直') where USER='root';
// 修改完后需要刷新一下
flush privileges;
// 退出后,要重启mysql
systemctl restart mysqld
```

- 为了保证能在hive上使用mysql,需要在这些做完之后,输入以下命令,进行连接
```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '数据库密码' WITH GRANT OPTION;
刷新命令(不需要封号)
FLUSH PRIVILEGES
```

### mysql忘记密码的操作
1.在/etc/my.cnf文件里加入
skip-grant-tables
2.用然后用mysql这个语句进入mysql
3.输入UPDATE mysql.user SET authentication_string=PASSWORD('6757DUgu') where USER='root'修改密码为6757DUgu
4.flush privileges刷新
5.重启mysql.
