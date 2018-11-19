#### 1.用流copy获取第二个block块的内容
```

	@Test
	public void downloadBySeek() throws IllegalArgumentException, IOException {
		// 获取hadoop的压缩包文件
		RemoteIterator<LocatedFileStatus> listFiles = 
				fileSystem.listFiles(new Path("/list/hadoop-2.7.3.tar.gz"), false);
		// 创建读入流,读入hadoop压缩包.
		FSDataInputStream fsDataInputStream =
				fileSystem.open(new Path("/list/hadoop-2.7.3.tar.gz"));
		// 获取该文件的所有块.
		while (listFiles.hasNext()) {
			// 
			LocatedFileStatus next = listFiles.next();
			// 获取 改文件的所有block块信息
			BlockLocation[] blockLocations = next.getBlockLocations();
			// 循环块
			for (int i = 0; i < blockLocations.length; i++) {
				// 判断是第二个块,进行读取到流中.
				if (i == 1) {
					fsDataInputStream.seek(blockLocations[i].getOffset());
					break;
				}
			}
			
		}
		// 创建输出流,指定位置.
		FileOutputStream foStream = 
				new FileOutputStream("C:/Users/Administrator/Desktop/ll/oo.tar.gz");
		// 进行输出.
		org.apache.commons.compress.utils.IOUtils.copy(fsDataInputStream, foStream);
		
	}
```


#### 2.获取所有block内容(通过设置偏移量实现)
```

	@Test
	// 将所有block块内容下载下来
	public void getBlocks() throws FileNotFoundException, IllegalArgumentException, IOException {
		// 通过fileSystem的listFiles方法可以自动实现递归(自带递归)列出文件类型,返回的是一个远程可迭代对象,需要传入两个参数,第一个参数是服务器路径,第二个参数是否递归
		RemoteIterator<LocatedFileStatus> listFiles = fileSystem.listFiles(new Path("/list/had.zip"), false);
		// 用流打开要读取的文件
		FSDataInputStream open = 
				fileSystem.open(new Path("/list/had.zip"));
		while (listFiles.hasNext()) {
			LocatedFileStatus next = listFiles.next();
			BlockLocation[] locations = 
					next.getBlockLocations();
			for (int i = 0; i < locations.length; i++) {
				// 为了不用每次读取一个输入流,直接设置他的偏移量.
				// seek是对输入流的文件指针做操作的
				open.seek(0);
				FileOutputStream fos = 
						new FileOutputStream("C:/Users/Administrator/Desktop/ll/had1" + i + ".zip");
				// 语法糖
				org.apache.commons.io.IOUtils.copyLarge(open, fos, locations[i].getOffset(), locations[i].getLength());
			}
			
		}
	}
```

#### 3.获取所有block内容(用org.apache.commons.io.IOUtils.copyLarge()的四参方法)
```
// 将所有block块内容下载下来
	public void getBlocksSec() throws FileNotFoundException, IllegalArgumentException, IOException {
		RemoteIterator<LocatedFileStatus> listFiles = fileSystem.listFiles(new Path("/list/had.zip"), false);
		FSDataInputStream open = 
				fileSystem.open(new Path("/list/had.zip"));
		while (listFiles.hasNext()) {
			LocatedFileStatus next = listFiles.next();
			BlockLocation[] locations = 
					next.getBlockLocations();
			Long sum = 0L;
			for (int i = 0; i < locations.length; i++) {
				
				FileOutputStream fos = 
						new FileOutputStream("C:/Users/Administrator/Desktop/ll/had1" + i + ".zip");
				long length = locations[i].getLength();
				long offset = locations[i].getOffset();
				// 每次的读取是一个新的block块
				// 参数一:读入流
				// 参数二:写出流
				// 参数三:读入的偏移量
				// 参数四:正在读取的block块的长度.
				org.apache.commons.io.IOUtils.copyLarge
				(open, fos, sum, locations[i].getLength());
				// 因为开始的偏移量为0,所以sum要在最后进行加.
				sum = length + offset;
			}
			
		}
	}
```

#### 4.获取文件内容,通过实例化实现
```
public void testSeek() throws IllegalArgumentException, IOException {
		BlockInfo blockInfo1 = new BlockInfo(0, 6);
		BlockInfo blockInfo2 = new BlockInfo(6, 6);
		BlockInfo blockInfo3 = new BlockInfo(12, 3);
		List<BlockInfo> blockInfos = new ArrayList<>();
		blockInfos.add(blockInfo1);
		blockInfos.add(blockInfo2);
		blockInfos.add(blockInfo3);
		FSDataInputStream inputStream = fileSystem.open(new Path("/list/in.md"));
		int sum = 0;
		for (int i = 0; i < blockInfos.size(); i++) {
			int offset = blockInfos.get(i).getOffset();
			int length = blockInfos.get(i).getLength();
			FileOutputStream fileOutputStream = new FileOutputStream("C:/Users/Administrator/Desktop/hdfs/output/seek"+i+".txt");
			// 偏移量设置为零,指针移到偏移量为0的位置.
			org.apache.commons.io.IOUtils.copyLarge(inputStream, fileOutputStream, 0, length);
			sum = offset+length;
			
		}
	}
```
