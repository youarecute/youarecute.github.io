## 连接
```
Configuration conf = new Configuration();
// 设置用户,告诉要用的用户是谁
//System.setProperty("HADOOP_USER_NAME", "hadoop");
// 设置要使用的文件系统是hdfs ->地址是 172.18.24.28:9000
//conf.set("fs.defaultFS",  "hdfs://172.18.24.28:9000");
//fileSystem = FileSystem.get(conf);
fileSystem = FileSystem.get(new URI("hdfs://172.18.24.28:9000"), conf, "hadoop");
```

## 操作
#### 1.上传操作
```
// 1.本地文件路径记得更改斜杠
// 2.hdfs上的文件路径,注意文件夹是否存在,例如/kll,如果文件夹存在,就会上传到该文件夹下;如果不存在,就会以该文件替代原文件变成新的存在.
fileSystem.copyFromLocalFile(new Path("G:/MapReduce/input/things.txt"), new Path("/list"));
```

#### 2.删除操作
```
// 删除方法的一参形式已经启用
// 两参中的第二个参数表示是否要使用递归操作,对我们删除一个文件没什么影响,但是如果要删除的是文件夹,只要不是空文件夹,就必须设置成true,允许递归操作才能真正删除.
fileSystem.delete(new Path("/ccc/ll.md"), true);
```

#### 3.用流上传
```
// input 把数据读入
FileInputStream fis = new FileInputStream("C:/Users/Administrator/Desktop/kk/stream.txt");
// 把数据读入
FSDataOutputStream fsDataOutputStream = fileSystem.create(new Path("/list/in.txt"));
IOUtils.copyBytes(fis, fsDataOutputStream, 1024);
```

#### 4.用流下载
```
FSDataInputStream fsDataInputStream = fileSystem.open(new Path("/list/in.txt"));
FileOutputStream foStream = new FileOutputStream("C:/Users/Administrator/Desktop/ll/ll.txt");
org.apache.commons.compress.utils.IOUtils.copy(fsDataInputStream, foStream);
```
