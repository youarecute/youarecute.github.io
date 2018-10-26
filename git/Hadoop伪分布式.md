# Hadoop伪分布式
#### 1.创建用户
#### 2.使用新创建的用户连接到虚拟机
#### 3.防火墙关闭 
- (1)systemctl disable firewalld 
- (2)systemctl status firewalld
#### 4.JDK
- (1)下载
- (2)上传
- (3)解压、改名
- (4)配置环境变量
#### 5.hadoop配置
- (1)下载hadoop包
- (2)上传
- (3)解压,改名
- (4)配置环境变量(bin\sbin)
#### 6.配置hadoop中几个文件
- (1)hadoop-env.sh 
	- ①($JAVA_HOME路径设置成JDK解压包的路径)
- (2)core-site.xml
	- ①fs.defaultFS：
		- 1)默认是本地文件系统file:///
		- 2)使用URL统一资源定位符的格式设置成hdfs://本机的ip地址(实质是namenode的ip):9000
			- 类比：jdbc:mysql://localhost:3306
	- ②hadoop.tmp.dir:存放hadoop临时文件的文件目录
	- ③代码如下(是放在<configuration>标签内的):

```
	<property>  
        <name>hadoop.tmp.dir</name> //存放临时文件 
        <value>/home/hadoop/tmp</value>// 存放路径
    </property>  
    <property>  
        <name>fs.defaultFS</name>  
        <value>hdfs://172.18.24.28:9000</value>  
    </property>
```

- (3)hdfs-site.xml:
	- ①dfs.namenode.name.dir：namenode所在路径(该路径需要允许当前登录的用户具有写的权限)
	- ②dfs.datanode.data.dir：datanode所在路径(该路径需要允许当前登录的用户具有写的权限)
	- ③代码如下:
	
```
	<property>    
        <name>dfs.replication</name>//设置备份(可暂时删除)    
        <value>1</value>    
    </property>    
    <property>    
        <name>hdfs.namenode.name.dir</name>    
        <value>file:/home/hadoop/dfs/name</value>    
    </property>    
    <property>    
        <name>dfs.datanode.data.dir</name>    
        <value>file:/home/hadoop/dfs/data</value>    
    </property>
```

- (4)yarn-site.xml
	- ①yarn.nodemanager.aux-services：使用mapreduce_shuffle机制
	- ②代码如下:

```
	<property>  
		<name>mapreduce.framework.name</name>  
		<value>yarn</value>  
	</property>  
	<property>  
		<name>yarn.nodemanager.aux-services</name>  
		<value>mapreduce_shuffle</value>  
	</property>
```

- (5)mapred-site.xml（cp mapred-site.xml.templete mapred-site.xml[备份一份,在备份里面写,这样模板被保存,以后好修改]）
	- ①Mapreduce.framework.name：yarn
	- ②代码如下:
	
```
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
```
 
#### 7.格式化HDFS
- (1)Hadoop  namenode  -format
	- 格式化成功的标志:Storage:.....successfully 
#### 8.启动HADOOP服务
- (1)start-all.sh或者start-dfs.sh and start-yarn.sh
	- 成功的标志是:运行jps命令出现的如下图:

#### 9.关闭hadoop服务
- (1)Stop-all.sh或者 stop-yarn.sh and stop-dfs.sh

#### 端口进程查询和结束
- netstat -tunlp|grep (端口号|java) 查看占用端口的进程
- kill -9 进程号 通过杀死进程来释放端口,解决端口占用问题.