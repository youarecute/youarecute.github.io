### OutputFormat
在outputFormat中接收的文件是,我们要写出去的文件,所以泛型要和Mapper里的写出文件类型对应.
事例如下:
Test类
```
package com.lanou.test;

import java.io.IOException;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.util.MyOutputFormat;
import com.lanou.util.MyoutputMapper;

public class MyOutputTest {
	
	public static void main(String[] args) throws IOException, URISyntaxException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();

		conf.set("fs.defaultFS", "hdfs://172.18.24.28:9000");
		System.setProperty("HADOOP_USER_NAME", "hadoop");
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(MyOutputTest.class);
		job.setMapperClass(MyoutputMapper.class);
		job.setOutputFormatClass(MyOutputFormat.class);
		job.setNumReduceTasks(0);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		}
		MyOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

Mapper类
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MyoutputMapper extends Mapper<Object, Text, Text, NullWritable> {
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		String[] vals = value.toString().split("\\s+");
		if (vals.length != 3) {
			StringBuffer sb = new StringBuffer(value.toString());
			sb.append("\t");
			sb.append("error");
			value.set(sb.toString());
			context.getCounter("lanou", "error").increment(1);
		}
		context.getCounter("lanou", "all").increment(1);
		context.write(value, NullWritable.get());
	}
}
```

OutputFormat类继承FileOutputFormat类
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MyOutputFormat extends FileOutputFormat<Text, NullWritable> {
	
	
	@Override
	public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// 在outputFormat中接收到我们要往哪儿些文件写
		
		Configuration conf = job.getConfiguration();
		// 获取文件系统
		FileSystem fileSystem = FileSystem.get(conf);
		// 获取路径
		Path outputPath = this.getOutputPath(job);
		System.out.println("name是:" + outputPath.getName());
		System.out.println("self是:" + outputPath);
		System.out.println("parent是:" + outputPath.getParent());
		// 定义结果输出路径,返回的是流
		FSDataOutputStream success = fileSystem.create(new Path(outputPath.toString() + "/success.log"));
		FSDataOutputStream error = fileSystem.create(new Path(outputPath.toString() + "/error.log"));
	
		return new MyRecordWriter(success, error);
	}
}
```

RecordWriter类
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

public class MyRecordWriter extends RecordWriter<Text, NullWritable> {
	// 两个输出流是为了在format中的输出流用
	private FSDataOutputStream success;
	private FSDataOutputStream error;

	public MyRecordWriter(FSDataOutputStream success, FSDataOutputStream error) {
		super();
		this.success = success;
		this.error = error;
	}

	@Override
	public void close(TaskAttemptContext arg0) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		if (success != null) {
			success.close();
		}else if (error != null) {
			error.close();
		}
	}

	@Override
	public void write(Text key, NullWritable value) throws IOException, InterruptedException {
		// TODO Auto-generated method stub  
		if (key.toString().contains("error")) {
			error.write(key.toString().getBytes());
		}else {
			success.write(key.toString().getBytes());
		}
		System.out.println("key:" + key);
		System.out.println("value:" + value);
	}

}
```

### InputFormat
Test类
```
package com.lanou.test;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.util.MyInputFormat;
import com.lanou.util.MyInputMapper;

public class MyInputFormatTest {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();

		conf.set("fs.defaultFS", "hdfs://172.18.24.28:9000");
		System.setProperty("HADOOP_USER_NAME", "hadoop");
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(MyInputFormatTest.class);
		job.setMapperClass(MyInputMapper.class);
		job.setInputFormatClass(MyInputFormat.class);
		job.setNumReduceTasks(0);
		
		// job.setReducerClass(JoinReduce.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
			
		}
		 FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		// yarn
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

Mapper类
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MyInputMapper extends Mapper<Text, NullWritable, Text, NullWritable> {
	
	@Override
	protected void map(Text key, NullWritable value, Mapper<Text, NullWritable, Text, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		System.out.println("map is run");
		System.out.println("key:" + key);
		System.out.println("value:" + value);
		context.write(key, value);
	}

}
```

InputFormat类继承FileInputFormat类
```
package com.lanou.util;

import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

public class MyInputFormat extends FileInputFormat<Text, NullWritable> {
	
	// 判断是否需要切片
	@Override
	protected boolean isSplitable(JobContext context, Path filename) {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public RecordReader<Text, NullWritable> createRecordReader(InputSplit split, TaskAttemptContext context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		MyRecordReader dsMyRecordReader = new MyRecordReader();
		dsMyRecordReader.initialize(split, context);
		return dsMyRecordReader;
	}

}
```

需要实现RecordReader类
```
package com.lanou.util;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class MyRecordReader extends RecordReader<Text, NullWritable> {
	private FileSplit fileSplit;
	private Configuration conf;
	private boolean result = false;
	private Text resultText = new Text();
	
	
	@Override
	public void close() throws IOException {
		// TODO Auto-generated method stub
		
	}

	@Override
	public Text getCurrentKey() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return resultText;
	}

	@Override
	public NullWritable getCurrentValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return null;
	}

	// 进度
	@Override
	public float getProgress() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return (float) (result ? 1.0 : 0.0);
	}

	// 获取信息
	@Override
	public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// 为了知道要读取哪儿个文件
		fileSplit = (FileSplit)split;
		// hdfs的配置信息
		conf = context.getConfiguration();
	}

	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// 一次性读取出来小文件的所有内容,一次性传给map.
//		只有第一次调用的时候,返回true,之后都返回false.
		if (!result) {
			// 获取 文件系统
			FileSystem fs = FileSystem.get(conf);
			// 根据文件系统获取输入流
			FSDataInputStream open = fs.open(fileSplit.getPath());
			// 进行读.
			// 创建一个能存下整个文件内容的buffer
			byte[] buffer = new byte[(int)fileSplit.getLength()];
			// 将整个文件的内容写入buffer中
			IOUtils.readFully(open, buffer, 0, buffer.length);
//			BufferedReader br = new BufferedReader(new InputStreamReader(open));
//			br.readLine();
			resultText.set(buffer, 0, buffer.length);
			// 作为一个开关,决定了map循环是否要继续
			result = true;
			return true;
		}
		return false;
	}

}
```

### InputFormat和OutputFormat一起使用
Test类
```
package com.lanou.homework;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class InputOutPutFormatTest {
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();

		conf.set("fs.defaultFS", "hdfs://172.18.24.28:9000");
		System.setProperty("HADOOP_USER_NAME", "hadoop");
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(InputOutPutFormatTest.class);
		job.setMapperClass(SelfMapper.class);
		job.setInputFormatClass(SelfInputFormat.class);
		job.setOutputFormatClass(SelfOutputFormat.class);
		job.setNumReduceTasks(0);
		// job.setReducerClass(JoinReduce.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);

		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		}
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		// yarn
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

Mapper类
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
// 因为我们使用了自定义的InputFormat,所以keyin,valuein不再是object,text
public class SelfMapper extends Mapper<NullWritable, Text, Text, Text> {
	private Text fileName = new Text();
	private Text valueText = new Text();
	
	// 利用setup方法在map循环中只执行的特性,来获取文件名.
	@Override
	protected void setup(Mapper<NullWritable, Text, Text, Text>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		FileSplit fileSplit = (FileSplit)context.getInputSplit();
		fileName.set(fileSplit.getPath().getName());
		
	}
	
	@Override
	protected void map(NullWritable key, Text value, Mapper<NullWritable, Text, Text, Text>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		System.out.println("map is run");
		System.out.println("value:" + value);
		String[] vals = value.toString().split("\n");
		for (String string : vals) {
			System.out.println("result:" + string);
			String[] values = string.split("\\s+");
			// 替换掉之后,才会让加的error不换行显示.
			string = string.replace("\r", "");
			string = string.replace("\n", "");
			if (values.length != 3) {
				StringBuffer sb = new StringBuffer(string);
				sb.append("\t");
				sb.append("error");
				valueText.set(sb.toString());
			}else {
				valueText.set(string);
			}
			context.write(fileName, valueText);

		}
		
	}

}
```

RecordWriter类
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

public class SelfRecordWriter extends RecordWriter<Text, Text> {
	private FSDataOutputStream success;
	private FSDataOutputStream error;

	@Override
	public void write(Text paramK, Text paramV) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		String result = paramK.toString() + "\t" + paramV.toString() + "\n";
		if (paramV.toString().contains("error")) {
			this.error.writeUTF(result);
		}else {
			this.success.writeUTF(result);
		}
	}
// 有参构造
	public SelfRecordWriter(FSDataOutputStream success, FSDataOutputStream error) {
		super();
		this.success = success;
		this.error = error;
	}

	@Override
	public void close(TaskAttemptContext paramTaskAttemptContext) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		if (success != null) {
			success.close();
		}else if (error != null) {
			
			error.close();
		}
		
	}

}
```

InputFormat类
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

public class SelfInputFormat extends FileInputFormat<NullWritable, Text> {
	
	// 判断是否需要切片
	@Override
	protected boolean isSplitable(JobContext context, Path filename) {
		// TODO Auto-generated method stub
		// 不切片
		return false;
	}

	@Override
	public RecordReader<NullWritable, Text> createRecordReader(InputSplit arg0, TaskAttemptContext arg1)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// 要使用自定义的InputFormat实际上就是要使用自定义的
		// recordReader来进行数据的读取,所以我们需要创建自定义的recordWriter来返回给InputFormat进行使用
		SelfRecordReader selfRecordReader = new SelfRecordReader();
		// 只创建不够,因为recordReader是真正读取数据的;所以我们需要通过initialize()方法将切片信息和job信息给到
		// recordReader,这样recordReader才知道该去哪儿个文件系统读取哪儿个文件
		selfRecordReader.initialize(arg0, arg1);
		return selfRecordReader;
	}
}
```

RecordReader类
```
package com.lanou.homework;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class SelfRecordReader extends RecordReader<NullWritable, Text> {
	private FileSplit fileSplit;
	private Configuration conf ;
	private boolean res = true;// 是否要执行的开关
	private Text valueText = new Text();

	@Override
	public void close() throws IOException {
		// TODO Auto-generated method stub
		
	}

	@Override
	public NullWritable getCurrentKey() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return NullWritable.get();
	}

	@Override
	public Text getCurrentValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return valueText;
	}

	@Override
	public float getProgress() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		return (float)(res ? 0.0 : 1.0);
	}

	@Override
	public void initialize(InputSplit arg0, TaskAttemptContext arg1) throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// 因为我们通过initialize获取到的内容在netKeyValue这个方法中进行使用,所以需要定义对应的属性进行接收
		this.fileSplit = (FileSplit)arg0;
		this.conf = arg1.getConfiguration();
	}

	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// 先想好开关怎么设置
		// 因为我们要一次性将整个文件读出来
		// 所以nexKeyValue()只需要在第一次调用的时候,返回true就好了.
		if (this.res) {
			// 因为我们的conf属性已经接收到了job中的configuration
			// 所以通过conf属性就能得到真正要读取的文件系统,接下来只需要通过调用文件系统的api获取文件就可以了.
			FileSystem fileSystem = FileSystem.get(conf);
			// 获取当前文件路径
			Path path = fileSplit.getPath();
			// 通过open获取当前要读文件的输入流
			// 根据我们的fileSplit通过切片获取到,我们要读的文件的输入流.
			FSDataInputStream open = fileSystem.open(path);
			// 接收整个文件的内容
			byte[] buff = new byte[(int)fileSplit.getLength()];
			// 真正将输入流的内容写到字节数组中,执行完这一句,buff中就是我们想要的结果了
			IOUtils.readFully(open, buff, 0, buff.length);
			// 将我们的结果写入到我们对应的属性中,
			// 供对应的方法进行调用
			valueText.set(buff, 0, buff.length);
			
			this.res = false;
			return true;
		}
		return false;
	}

}
```
