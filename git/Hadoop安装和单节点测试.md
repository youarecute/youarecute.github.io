#### hadoop的安装(前提必须安装了JDK,文件路径都使用的绝对路径,可以根据自己所在位置适当调整)
- 1.把hadoop安装包放在虚拟机内,解压.我的是在/home/hadoop/tars文件夹下放着,以下会以这个为例.
- 2.在/etc/profile或者  ~/.bash_profile里面添加hadoop的路径
	- 配置之后要执行命令: . ~/.bash_profile或者. /etc/profile,使之生效,配置的哪儿个文件,就用哪儿个命令.
	- 测试生效的命令是:hadoop
	
```
export HADOOP_HOME=/home/hadoop/tars/hadoop-2.7.3 #hadoop解压包所在路径
#在PATH变量后面添加如下命令
:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

#### 单节点测试
- 3.创建一个文件夹,命令是:mkdir kllinput.
- 4.在kllinput文件夹下创建一个kll.txt文件,命令是:touch kll.txt
- 5.使用命令:vi /home/hadoop/kllinput/kll.txt,进入文本编辑界面,随意输入内容.

- 6.执行命令:hadoop jar /home/hadoop/tars/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /home/hadoop/kll/kll.txt /home/hadoop/output
	- 这个命令是利用jar包hadoop-mapreduce-examples-2.7.3.jar里的wordcount命令统计kll文件夹下的单词的出现次数.
	- 记得关闭防火墙:systemctl disable firewalld 永久关闭防火墙;systemctl stop firewalld 暂时关闭防火墙,下次启动,需要再次关闭.
- 7.执行命令:cat output/*
	- 是指查看6的命令的运行结果.
