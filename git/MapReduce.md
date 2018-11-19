##### 1.单词个数统计
map类
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class CountMapper extends Mapper<Object, Text, Text, IntWritable>{	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		// 获取到的一行的内容进行分割
		String[] val = value.toString().split(" ");
		// 遍历输出到context
		for (int i = 0; i < val.length; i++) {
			// 为了把字符转成text类型
			// 创建text对象
			Text text = new Text();
			// 用text的set方法,把字符转成text类型
			text.set(val[i]);
			// 把int类型转成IntWritable类型
			context.write(text, new IntWritable(1));
		}
	}
}
```

reduce类
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordReduce extends Reducer<Text, IntWritable, Text, IntWritable> {
	
	
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
		int sum = 0;
		for (IntWritable val : values) {
			sum += val.get();
		}
		context.write(key, new IntWritable(sum));
	}

}
```

main函数
```
package com.lanou.test;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.examples.WordCount;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.Util.CountMapper;
import com.lanou.Util.WordReduce;

public class WordCountTest {
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if (otherArgs.length < 2) {
			System.err.println("Usage: wordcount <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(WordCountTest.class);
		// s设置mapper
		job.setMapperClass(CountMapper.class);
		// 设置combiner
		job.setCombinerClass(WordReduce.class);
		// 设置Reducer
		job.setReducerClass(WordReduce.class);
		// 设置reduce的key和value值的类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		// 把最后一个输出减掉了,只是输入
		for (int i = 0; i < otherArgs.length - 1; i++) {
			FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		}
		// 设置写入到输出位置.
		FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));

		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}
```
##### 2.Partitioner_______从log日志中获取手机号,上行流量,下行流量,总流量(以实体类对象形式)
- 作用:根据返回值,把不同结果放在不同的文件中

```
/*
 * 使用分区的步骤:
 * 1.设置job.setPartitionerClass(分区类.class);
 * 和job.setNumReduceTask(4),如果数量不对,会出现异常;
 * 2.创建分区类,继承Partitioner累,实现getPartitioner方法
 * 3.根据自己的业务逻辑,通过key,value,count的操作,利用返回值,得到当前map需要进入的分区.
 */
/*
 * 从log中获取手机号,上行流量,下行流量,总流量
 * reduce之后,输出结果要手机号,流量对象(上行流量,下行流量,流量总和)
 * 要使用自定义的类型作为输出
 * 1.根据数据我们的数据进行建模(创建实体类)
 * 2.mapper和reducer中的形参怎么写,map()方法当中怎么对数据进行切割筛选,将我们的数据放入自定义的bean(注意javaBean规范)当中,想好bean是以key还是value输出
 * 3.map的结果最终要写入磁盘中,所以要注意我们自定义的类一定要支持序列化WritableComparable
 *    Serializable是java提供的重量级序列化,在hadoop中建议使用hadoop自身提供的WritableComparable接口来实现序列化和反序列化.
 *    要注意重写write()和readFields()方法 否则reducer阶段获取不到数据
 *    注意:readFields()中调用的read方法一定要按照写入的顺序进行读取,否则会属性顺序错乱
 * 4.main中的job.setJarByClass(BeanStreamTest.class);
        job.setMapperClass(BeanStreamMapper.class);
        job.setCombinerClass(BeanStreamReducer.class);
        job.setReducerClass(BeanStreamReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(StreamBean.class);
                    注意设置
                    注意好输出的key,value的类型
 * 5.注意重写toString方法
 */
```

Partitioner类
```
package com.lanou.Util;

import java.util.HashMap;
import java.util.Map;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

import com.lanou.model.StreamBean;
/*
 * 使用分区的步骤:
 * 1.设置job.setPartitionerClass(分区类.class);
 * 和job.setNumReduceTask(4),如果数量不对,会出现异常;
 * 2.创建分区类,继承Partitioner累,实现getPartitioner方法
 * 3.根据自己的业务逻辑,通过key,value,count的操作,利用返回值,得到当前map需要进入的分区.
 */
public class MobilePartitioner extends Partitioner<Text, StreamBean>{

	private static Map map;
	// 1.第一个参数是map方法中的context.write()当中的key
	// 2.第二个参数是map方法中的context.write()当中的value
	// 3.第三个参数是reduceTask的数量
	// 如果我们要在getPartitioner方法中去创建假数据map,会创建太对个对象.
	// 不太符合我们的期望,我们只希望创建一次就好.
	static {
		map = new HashMap<>();
		map.put("134", 0);
		map.put("135", 1);
		map.put("136", 2);
	}
	@Override
	public int getPartition(Text paramKEY, StreamBean paramVALUE, int paramInt) {
		// TODO Auto-generated method stub
		// 输出的
		System.out.println("get 134 Partitioner:" + map.get("134"));
		System.out.println("get 135 Partitioner:" + map.get("135"));
		System.out.println("get 136 Partitioner:" + map.get("136"));
		System.out.println(paramVALUE);
		System.out.println(paramInt);
		String key = paramKEY.toString().substring(0, 3);
		// 返回值,设置这条数据真正的交给哪儿个reduceTask
		Integer result = (Integer) map.get(key);
		if (result == null) {
			result = 3;
		}
		return result;
	}

}

```

实体类
```
package com.lanou.model;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.WritableComparable;

public class StreamBean implements WritableComparable<Object> {
	private int uploadStream;
	private int downloadStream;
	private int sumStream;
	public StreamBean() {
		super();
	}
	public StreamBean(String uploadStream, String downloadStream) {
		super();
		this.uploadStream = Integer.parseInt(uploadStream);
		this.downloadStream = Integer.parseInt(downloadStream);
		this.sumStream = this.downloadStream + this.uploadStream;
	}
	
	public StreamBean(int uploadStream, int downloadStream) {
		super();
		this.uploadStream = uploadStream;
		this.downloadStream = downloadStream;
		this.sumStream = this.downloadStream + this.uploadStream;
	}
	
	public int getUploadStream() {
		return uploadStream;
	}
	public void setUploadStream(int uploadStream) {
		this.uploadStream = uploadStream;
	}
	public int getDownloadStream() {
		return downloadStream;
	}
	public void setDownloadStream(int downloadStream) {
		this.downloadStream = downloadStream;
	}
	public int getSumStream() {
		return sumStream;
	}
	public void setSumStream(int sumStream) {
		this.sumStream = sumStream;
	}
	@Override
	public String toString() {
		return  uploadStream + "\t" + downloadStream + "\t" + sumStream;
	}
	@Override
	public void write(DataOutput paramDataOutput) throws IOException {
		paramDataOutput.writeInt(this.uploadStream);
		paramDataOutput.writeInt(this.downloadStream);
		paramDataOutput.writeInt(this.sumStream);
		
	}
	@Override
	public void readFields(DataInput paramDataInput) throws IOException {
		// TODO Auto-generated method stub
		// 取值要和上边的写入顺序一致
		this.uploadStream = paramDataInput.readInt();
		this.downloadStream = paramDataInput.readInt();
		this.sumStream = paramDataInput.readInt();
	}
	@Override
	public int compareTo(Object o) {
		// TODO Auto-generated method stub
		return 0;
	}
		
}

```

map类
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.StreamBean;

public class BeanStreamMapper 
extends Mapper<Object, Text, Text, StreamBean>{
	private Text phone = new Text();
	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, StreamBean>.Context context)
			throws IOException, InterruptedException {
		// 对结果进行处理的规则
		String[] vals = value.toString().split("\\s+");
		String phone = vals[1];
		String upload = vals[vals.length - 3];
		String download = vals[vals.length - 2];
		StreamBean streamBean = new StreamBean(upload, download);
		this.phone.set(phone);
		// 
		context.write(this.phone, streamBean);
	}

}
```

reduce类
```
package com.lanou.Util;

import java.io.IOException;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.StreamBean;

public class BeanStreamReduce 
extends Reducer<Text, StreamBean, Text, StreamBean> {
	
	@Override
	protected void reduce(Text key, Iterable<StreamBean> lists,
			Reducer<Text, StreamBean, Text, StreamBean>.Context context) throws IOException, InterruptedException {
		// 每个对象的上行流量加一起,下行流量加一起,总流量加一起
		int uploadSum = 0;
		int downloadSum = 0;
		for (StreamBean streamBean : lists) {
			uploadSum += streamBean.getUploadStream();
			downloadSum += streamBean.getDownloadStream();
		}
		StreamBean streamBean = new StreamBean(uploadSum, downloadSum);
		context.write(key, streamBean);
	}

}
```

main函数
```
package com.lanou.test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.Util.BeanStreamMapper;
import com.lanou.Util.BeanStreamReduce;
import com.lanou.Util.MobilePartitioner;
import com.lanou.model.StreamBean;
/*
 * 从log中获取手机号,上行流量,下行流量,总流量
 * reduce之后,输出结果要手机号,流量对象(上行流量,下行流量,流量总和)
 * 要使用自定义的类型作为输出
 * 1.根据数据我们的数据进行建模(创建实体类)
 * 2.mapper和reducer中的形参怎么写,map()方法当中怎么对数据进行切割筛选,将我们的数据放入自定义的bean(注意javaBean规范)当中,想好bean是以key还是value输出
 * 3.map的结果最终要写入磁盘中,所以要注意我们自定义的类一定要支持序列化WritableComparable
 *    Serializable是java提供的重量级序列化,在hadoop中建议使用hadoop自身提供的WritableComparable接口来实现序列化和反序列化.
 *    要注意重写write()和readFields()方法 否则reducer阶段获取不到数据
 *    注意:readFields()中调用的read方法一定要按照写入的顺序进行读取,否则会属性顺序错乱
 * 4.main中的job.setJarByClass(BeanStreamTest.class);
        job.setMapperClass(BeanStreamMapper.class);
        job.setCombinerClass(BeanStreamReducer.class);
        job.setReducerClass(BeanStreamReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(StreamBean.class);
                    注意设置
                    注意好输出的key,value的类型
 * 5.注意重写toString方法
 */
public class PartitionerTest {
	/*
	 * 根据分区将结果输入到不同位置
	 */
	public static void main(String[] args)
		    throws Exception
		  {
		// 配置文件
		    Configuration conf = new Configuration();
		    System.setProperty("HADOOP_USER_NAME", "hadoop");
		    conf.set("fs.defaultFS",  "hdfs://172.18.24.28:9000");
		 // 参数处理
		    String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2){
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		 // 工作 -> mapreduce
		    Job job = Job.getInstance(conf, "word count");
		 // 打包使用的
		    job.setJarByClass(PartitionerTest.class);
		 // 设置执行map方法的类
		    job.setMapperClass(BeanStreamMapper.class);
		 // 合并
		    job.setCombinerClass(BeanStreamReduce.class);
		 // 设置执行reduce方法的类
		    job.setReducerClass(BeanStreamReduce.class);
		    // 设置分区,使用哪儿个Patitioner
		    job.setPartitionerClass(MobilePartitioner.class);
		    // 设置reduce任务数,否则分区不会生效.
		    // 数量不能小于分组的个数,只能大于等于.否则会抛异常
		    // 如果不设置自定义分区,会使用系统默认的HashPartitioner
		    job.setNumReduceTasks(3);
		    // 设置reduce输出结果的类型
		    job.setOutputKeyClass(Text.class);
		    job.setOutputValueClass(StreamBean.class);
		    // 告诉job要读取的文件是哪儿些
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    // 告诉job设置的结果输出位置.
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
		    // job.waitForCompletion(true) 将我们设置好的job真正的提交给yarn来帮我们分配资源.
		    System.exit(job.waitForCompletion(true) ? 0 : 1);
		  }

}
```



