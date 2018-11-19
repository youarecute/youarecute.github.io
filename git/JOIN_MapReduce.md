### Reduce端join
```
/*
* reduce 端join
* 1. 文件格式要注意,否则有乱码,可以自己处理,String string = new String(value.getBytes(),"GBK");
* 2.创建一个接受完整的bean(技能存放订单信息,又能存放商品信息)
* 3.在map端接受不同文件的数据,根据是哪儿个文件,像完整的bean当中设置上对应的信息.没有的也不要是null,要设置默认值,否则
* 会造成空指针异常.这时候map的工作结束了<thingsid,bean>要注意序列化.
* 4.这时候因为map端输出的key是tingsId,所以传入reduce的数据已经帮我们进行合并了
* 例如以下形式:<thingsId,[orderBean1,orderBean2,thingsBean]>
* 要将迭代器中的数据进行拆分成如下形式:thingsBan与[orderBean1,orderBean2],我们只需要对orderBean的集合进行循环,将orderBean
* 缺少的商品信息通过thingsBean不全,这就得到了我们想要的完整的join之后的数据了,然后就可以输出了.因为我们要使用的数据
* 来自迭代器,所以我们拿出来使用,要进行深拷贝.
 */
```

Bean类
```
package com.lanou.model;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.Writable;

public class JoinBean implements Writable {
	private int orderId;// 订单ID
	private int count;// 订单购买数量
	private int stockCount;// 商品库存
	private String thingsId;// 商品ID
	private String thingsName;// 商品名称
	private int flag;//表示当前对象是通过哪儿个文件得到的
	// 给bean赋订单的文件的内容
	public void setOrder(int orderId, String thingsId, int count) {
		this.thingsId = thingsId;
		this.orderId = orderId;
		this.count = count;
		// 没获取到的数据也不要以null 的形式写入,
		// 否则进入reduce的时候就会空指针异常.
		this.thingsName = "";
		this.stockCount = -1;
		this.flag = 0;
	}
	// 给bean赋商品的文件的内容
	public void setThings(String thingsId, String thingsName, int stockCount) {
		this.thingsId = thingsId;
		this.thingsName = thingsName;
		this.stockCount = stockCount;
		// 没获取到的数据也不要以null 的形式写入,
		// 否则进入reduce的时候就会空指针异常.
		this.orderId = 0;
		this.stockCount = 0;
		this.flag = 1;
	}

	public void setThingsByBean(JoinBean thingsBean) {
		this.thingsName = thingsBean.getThingsName();
		this.stockCount = thingsBean.getStockCount();
	}
	
	public int getFlag() {
		return flag;
	}
	public void setFlag(int flag) {
		this.flag = flag;
	}
	public JoinBean() {
		super();
	}
	
	public JoinBean(int orderId, int count, int stockCount, String thingsId, String thingsName) {
		super();
		this.orderId = orderId;
		this.count = count;
		this.stockCount = stockCount;
		this.thingsId = thingsId;
		this.thingsName = thingsName;
	}

	public int getOrderId() {
		return orderId;
	}

	public void setOrderId(int orderId) {
		this.orderId = orderId;
	}

	public int getCount() {
		return count;
	}

	public void setCount(int count) {
		this.count = count;
	}

	public int getStockCount() {
		return stockCount;
	}

	public void setStockCount(int stockCount) {
		this.stockCount = stockCount;
	}

	public String getThingsId() {
		return thingsId;
	}

	public void setThingsId(String thingsId) {
		this.thingsId = thingsId;
	}

	public String getThingsName() {
		return thingsName;
	}

	public void setThingsName(String thingsName) {
		this.thingsName = thingsName;
	}

	@Override
	public void readFields(DataInput input) throws IOException {
		this.orderId = input.readInt();
		this.stockCount = input.readInt();
		this.count = input.readInt();
		this.thingsId = input.readUTF();
		this.thingsName = input.readUTF();
		this.flag = input.readInt();
	}
	@Override
	public void write(DataOutput output) throws IOException {
		output.writeInt(this.orderId);
		output.writeInt(this.stockCount);
		output.writeInt(this.count);
		output.writeUTF(this.thingsId); 
		output.writeUTF(this.thingsName);  
		output.writeInt(this.flag);
	}

	@Override
	public String toString() {
		return orderId + "\t" + count + "\t" + stockCount + "\t"
				+ thingsId + "\t" + thingsName + "\t" + flag;
	}
}
```

map类
```
package com.lanou.until;

import java.io.IOException;
import java.util.Arrays;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import com.lanou.model.JoinBean;

public class ReduceJoinMapper extends Mapper<Object, Text, Text, JoinBean> {
	private JoinBean joinBean = new JoinBean();;
	private Text thingsText = new Text();
	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, JoinBean>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// 获取切片(用子类接收,可以调用其方法)
		
		FileSplit fileSplit = (FileSplit)context.getInputSplit();
		String fileName = fileSplit.getPath().getName();
		System.out.println(fileName);
		//String string = new String(value.getBytes(), "GBK");
		String[] vals = value.toString().split("\\s+");
		System.out.println(Arrays.toString(vals));
		//JoinBean joinBean = new JoinBean();
		if ("orders.txt".equals(fileName)) {
			joinBean.setOrder(Integer.parseInt(vals[0]), vals[2], Integer.parseInt(vals[3]));
		}else {
			joinBean.setThings(vals[0], vals[1], Integer.parseInt(vals[2]));
		}
		thingsText.set(joinBean.getThingsId());
		context.write(thingsText, joinBean);
	}
}
```

reduce类
```
package com.lanou.until;

import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.util.ArrayList;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.JoinBean;

public class ReduceJoinReduce extends Reducer<Text, JoinBean, Text, JoinBean> {
	private JoinBean thingBean = new JoinBean();
	//private JoinBean orderBean = new JoinBean();
	
	@Override
	protected void reduce(Text key, Iterable<JoinBean> value, Reducer<Text, JoinBean, Text, JoinBean>.Context context)
			throws IOException, InterruptedException {
		//
		// key 商品id
		// 商品 value [订单1,订单2]
		ArrayList<JoinBean> ordersList = new ArrayList<>();
		for (JoinBean joinBean : value) {
			System.out.println("i am run");
			if (joinBean.getFlag() == 1) {
				thingBean.setCount(joinBean.getCount());
				thingBean.setThingsName(joinBean.getThingsName());
				thingBean.setThingsId(joinBean.getThingsId());
				thingBean .setCount(joinBean.getCount());
				thingBean.setStockCount(joinBean.getStockCount());
			}else {
				JoinBean joinBean2 = new JoinBean();
				try {
					BeanUtils.copyProperties(joinBean2, joinBean);
				} catch (IllegalAccessException | InvocationTargetException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				ordersList.add(joinBean2);
			}
		}
		// 商品1      订单1,订单2
		// 给迭代器中,所有的订单设置商品信息
		for (JoinBean joinBean : ordersList) {
			joinBean.setThingsByBean(thingBean);
			context.write(key, joinBean);
		}
	}

}
```

main类
```
package com.lanou.test;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.model.JoinBean;
import com.lanou.until.ReduceJoinMapper;
import com.lanou.until.ReduceJoinReduce;


public class ReduceJoinTest {
	
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		// 
		/*
		* reduce 端join
		* 1. 文件格式要注意,否则有乱码,可以自己处理,String string = new String(value.getBytes(), "GBK");
		* 2.创建一个接受完整的bean(技能存放订单信息,又能存放商品信息)
		* 3.在map端接受不同文件的数据,根据是哪儿个文件,像完整的bean当中设置上对应的信息.没有的也不要是null,要设置默认值,否则
		* 会造成空指针异常.这时候map的工作结束了<thingsid,bean>要注意序列化.
		* 4.这时候因为map端输出的key是tingsId,所以传入reduce的数据已经帮我们进行合并了
		* 例如以下形式:<thingsId,[orderBean1,orderBean2,thingsBean]>
		* 要将迭代器中的数据进行拆分成如下形式:thingsBan与[orderBean1,orderBean2],我们只需要对orderBean的集合进行循环,将orderBean
		* 缺少的商品信息通过thingsBean不全,这就得到了我们想要的完整的join之后的数据了,然后就可以输出了.因为我们要使用的数据
		* 来自迭代器,所以我们拿出来使用,要进行深拷贝.
		 */
		  Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(ReduceJoinTest.class);
		    job.setMapperClass(ReduceJoinMapper.class);
		    job.setReducerClass(ReduceJoinReduce.class);
		    job.setOutputKeyClass(Text.class);
		    job.setOutputValueClass(JoinBean.class);
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

###  map端join
利用的mapper类中setup方法只执行一次的特性.

```
/*
* map端join
* 主表:orders.txt 从表:things.txt
* 主表正常map处理,从表以缓存文件的形式读入,加入到job中
* 主表数据在map中处理之后,就是缺少从表数据的完整性.
* 就需要对存放了主表数据的完整对象,设置上缺失的从表数据
* 从表数据从--->setup()这个会在循环调用map前只执行一次的方法
* 来对我们的缓存文件进行加载.将解析后的一个一个储存了商品信息的对象以<商品ID,商品对象>形式加入map中,方便我们使用.
* 这样在map中,我们通过map.get(key)就当能取到当前这个订单所对应的商品了,进赋值和写入就可以了.
* 
*/
```

Bean类
```
package com.lanou.model;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.WritableComparable;
public class OrderInfo implements WritableComparable<OrderInfo> {
	private String orderId;
	private String goodsId;
	private double price;
	public void setOrder(String orderId, String goodsId, double price) {
		this.goodsId = goodsId;
		this.orderId = orderId;
		this.price = price;
	}
	@Override
	public String toString() {
		return "OrderInfo [orderId=" + orderId + ", goodsId=" + goodsId + ", price=" + price + "]";
	}
	public String getOrderId() {
		return orderId;
	}
	public void setOrderId(String orderId) {
		this.orderId = orderId;
	}
	public String getGoodsId() {
		return goodsId;
	}
	public void setGoodsId(String goodsId) {
		this.goodsId = goodsId;
	}
	public double getPrice() {
		return price;
	}
	public void setPrice(double price) {
		this.price = price;
	}
	public OrderInfo() {
		super();
	}
	@Override
	public void write(DataOutput output) throws IOException {
		output.writeUTF(this.orderId);
		output.writeUTF(this.goodsId);
		output.writeDouble(this.price);
	}
	@Override
	public void readFields(DataInput input) throws IOException {
		this.orderId = input.readUTF();
		this.goodsId = input.readUTF();
		this.price = input.readDouble();
		
	}
	@Override
	public int compareTo(OrderInfo arg0) {
		int relP = (int) (this.price - arg0.getPrice());
		int relI = arg0.getGoodsId().compareTo(this.orderId);
		return relI == 0 ? relP : -relI;
	}
}
```

map类
```
package com.lanou.until;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.MapJoinBean;

public class MapJoinMapper extends Mapper<Object, Text, Text, MapJoinBean> {
	private Text keyText = new Text();
	private MapJoinBean mapJoinBean = new MapJoinBean();
	private Map<String, MapJoinBean> containMap = new HashMap<>();
	
	
	// map会循环执行
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, Text, MapJoinBean>.Context context)
			throws IOException, InterruptedException {
	//FileInputStream fileInputStream	= new FileInputStream("things.txt");
		String[] vals = value.toString().split("\\s+");
		// 只有订单信息
		mapJoinBean.setOrder(vals[0], vals[2], vals[3]);
		// 根据订单中的商品信息,来找到容器中对应的商品对象,进行数据填充.
		MapJoinBean thingsBean = containMap.get(mapJoinBean.getThingsId());
		
		// 将我们订单对象缺少的商品属性进行赋值
		mapJoinBean.setThings(thingsBean.getThingsName(), thingsBean.getStockCount());
		
		keyText.set(mapJoinBean.getThingsId());
		context.write(keyText, mapJoinBean);
	}
	
	// 在map之前只运行一次
	@Override
	protected void setup(Mapper<Object, Text, Text, MapJoinBean>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		//FileReader fileReader = new FileReader("things.txt");
		FileInputStream fileInputStream	= new FileInputStream("things.txt");
		BufferedReader br = new BufferedReader(new InputStreamReader(fileInputStream, "UTF-8"));
		String line = "";
		// 定义一个容器,来存储所有解析出来的商品 对象 --->map
		while (!StringUtils.isEmpty(line = br.readLine())) {
			// 获取的商品信息
			System.out.println("line:" + line);
			String[] things = line.split("\\s+");
			// 创建商品对象
			MapJoinBean thingsBean = new MapJoinBean();
			// 给对象赋值
			thingsBean.setThings(things[0], things[1], Integer.parseInt(things[2]));
			// 商品id,商品对象的形式存在map中
			containMap.put(thingsBean.getThingsId(), thingsBean);

		}
	}

}
```

reduce类
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.OrderInfo;

public class MaxOrderReduce extends Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable> {

	@Override
	protected void reduce(OrderInfo orderInfo, Iterable<NullWritable> value,
			Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable>.Context arg2)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		for (NullWritable nullWritable : value) {
			System.out.println(nullWritable);
		}
		arg2.write(orderInfo, NullWritable.get());
	}
}
```

main函数
```
package com.lanou.test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.until.FileWordReduce;
import com.lanou.until.MapJoinMapper;

public class MapJoinTest {
	/*
	 * map端join
	 * 主表:orders.txt 从表:things.txt
	 * 主表正常map处理,从表以缓存文件的形式读入,加入到job中
	 * 主表数据在map中处理之后,就是缺少从表数据的完整性.
	 * 就需要对存放了主表数据的完整对象,设置上缺失的从表数据
	 * 从表数据从--->setup()这个会在循环调用map前只执行一次的方法
	 * 来对我们的缓存文件进行加载.将解析后的一个一个储存了商品信息的对象以<商品ID,商品对象>形式加入map中,方便我们使用.
	 * 这样在map中,我们通过map.get(key)就当能取到当前这个订单所对应的商品了,进赋值和写入就可以了.
	 * 
	 */
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException, URISyntaxException {
		Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(MapJoinTest.class);
		    job.setMapperClass(MapJoinMapper.class);
		    job.setNumReduceTasks(0);
		    job.setReducerClass(FileWordReduce.class);
		    job.setOutputKeyClass(Text.class);
		    job.setOutputValueClass(Text.class);
		    
		    // 添加缓存文件
		    job.addCacheFile(new URI("hdfs://172.18.24.28:9000/list/things.txt"));
		    
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```
### 通过设置GroupingComparator获取相同订单号中的最大金额订单
Bean类和上一个一样
```
/*
 * 分组 - GroupingComparator
 * 1.要创建好自定义的分组逻辑类
 * 1.1创建一个构造函数在内部调用super(要进行分组比较的类)
 * 1.2注意这里要进行比较的类必须去实现Comparable接口
 * 因为我们分组比较在sort排序之后,而且只能对相邻的对象进行比较.所以要在实体类的compareTo方法中,
 * 将要进行分组的条件作为我们排序的第一要素.只要这样才能保证相同的属性相邻.如果想要排序,从第二要素开始排
 * 1.3根据我们要进行分组的逻辑,实现抽象方法
 * public int compare(WritableComparable a, WritableComparable b)
 * 对相邻的两个对象进行比较,如果返回0,就会认为是该进一个reduce
 * 但是不会;立即调用reduce,会继续进行下一轮比较.例如:1.getOrderId = 2.getOrderId?,2.getOrderId = 3.getOrderId
 * 直到比较结果部位0了,知道不是一个reduce了,这时将所有相同的放入一个reduce中,如果想在reduce当中获取到这些被认为
 * 该进入一组的bean,只要循环迭代器,在迭代器中的key就是每一个bean.
 * 当一个reduce执行完,会继续执行我们的分组比较,遇到不同的,在执行reduce,从而形成一个循环,一直到我们reduce运行结束.
 * 
 */
```

mapper类
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.OrderInfo;

public class MaxOrderMapper extends Mapper<Object, Text, OrderInfo, NullWritable> {
	private OrderInfo orderinfo = new OrderInfo();
	
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, OrderInfo, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		String[] vals = value.toString().split("\\s+");
		orderinfo.setOrder(vals[0], vals[1], Double.parseDouble(vals[2]));
		context.write(orderinfo, NullWritable.get());
	}

}
```

Reduce类
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.OrderInfo;

public class MaxOrderReduce extends Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable> {

	@Override
	protected void reduce(OrderInfo orderInfo, Iterable<NullWritable> value,
			Reducer<OrderInfo, NullWritable, OrderInfo, NullWritable>.Context arg2)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		for (NullWritable nullWritable : value) {
			System.out.println(nullWritable);
		}
		arg2.write(orderInfo, NullWritable.get());
	}
}
```

Test类
```
package com.lanou.test;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.model.OrderInfo;
import com.lanou.until.GroupByOrderCompartor;
import com.lanou.until.MaxOrderMapper;
import com.lanou.until.MaxOrderReduce;
/*
  * 分组 - GroupingComparator
 * 1.要创建好自定义的分组逻辑类
 * 1.1创建一个构造函数在内部调用super(要进行分组比较的类)
 * 1.2注意这里要进行比较的类必须去实现Comparable接口
 * 因为我们分组比较在sort排序之后,而且只能对相邻的对象进行比较.所以要在实体类的compareTo方法中,
 * 将要进行分组的条件作为我们排序的第一要素.只要这样才能保证相同的属性相邻.如果想要排序,从第二要素开始排
 * 1.3根据我们要进行分组的逻辑,实现抽象方法
 * public int compare(WritableComparable a, WritableComparable b)
 * 对相邻的两个对象进行比较,如果返回0,就会认为是该进一个reduce
 * 但是不会;立即调用reduce,会继续进行下一轮比较.例如:1.getOrderId = 2.getOrderId?,2.getOrderId = 3.getOrderId
 * 直到比较结果部位0了,知道不是一个reduce了,这时将所有相同的放入一个reduce中,如果想在reduce当中获取到这些被认为
 * 该进入一组的bean,只要循环迭代器,在迭代器中的key就是每一个bean.
 * 当一个reduce执行完,会继续执行我们的分组比较,遇到不同的,在执行reduce,从而形成一个循环,一直到我们reduce运行结束.
 */
public class MaxOrderTest {
	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(MaxOrderTest.class);
		    job.setMapperClass(MaxOrderMapper.class);
		    //job.setNumReduceTasks(0);
		    job.setReducerClass(MaxOrderReduce.class);
		    job.setOutputKeyClass(OrderInfo.class);
		    job.setOutputValueClass(NullWritable.class);
		    
		    job.setGroupingComparatorClass(GroupByOrderCompartor.class);
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```

### 利用缓冲文件实现Reduce_Join
实体类
```
package com.lanou.model;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.WritableComparable;

public class MapJoinBean implements WritableComparable<MapJoinBean> {
	private int orderId;// 订单ID
	private int count;// 订单购买数量
	private int stockCount;// 商品库存
	private String thingsId;// 商品ID
	private String thingsName;// 商品名称
	
	public void setOrder(String orderId, String thingsId, String count) {
		this.thingsId = thingsId;
		this.orderId = Integer.parseInt(orderId);
		this.count = Integer.parseInt(count);
		
	}
	
	public void setThings(String thingsName, int stockCount) {
		this.thingsName = thingsName;
		this.stockCount = stockCount;
	}
	public void setThings(String thingsId, String thingsName, int stockCount) {
		this.thingsId = thingsId;
		this.thingsName = thingsName;
		this.stockCount = stockCount;
		// 没获取到的数据也不要以null 的形式写入,
//		// 否则进入reduce的时候就会空指针异常.
//		this.orderId = 0;
//		this.stockCount = 0;
	}
	public MapJoinBean() {
		super();
		// TODO Auto-generated constructor stub
	}
	
	public MapJoinBean(int orderId, int count, int stockCount, String thingsId, String thingsName) {
		super();
		this.orderId = orderId;
		this.count = count;
		this.stockCount = stockCount;
		this.thingsId = thingsId;
		this.thingsName = thingsName;
	}
	public int getOrderId() {
		return orderId;
	}
	public void setOrderId(int orderId) {
		this.orderId = orderId;
	}
	public int getCount() {
		return count;
	}
	public void setCount(int count) {
		this.count = count;
	}
	public int getStockCount() {
		return stockCount;
	}
	public void setStockCount(int stockCount) {
		this.stockCount = stockCount;
	}
	public String getThingsId() {
		return thingsId;
	}
	public void setThingsId(String thingsId) {
		this.thingsId = thingsId;
	}
	public String getThingsName() {
		return thingsName;
	}
	public void setThingsName(String thingsName) {
		this.thingsName = thingsName;
	}
	@Override
	public String toString() {
		return "MapJoinBean [orderId=" + orderId + ", count=" + count + ", stockCount=" + stockCount + ", thingsId="
				+ thingsId + ", thingsName=" + thingsName + "]";
	}
	@Override
	public void readFields(DataInput input) throws IOException {
		// TODO Auto-generated method stub
		this.orderId = input.readInt();
		this.stockCount = input.readInt();
		this.count = input.readInt();
		this.thingsId = input.readUTF();
		this.thingsName = input.readUTF();
	}
	@Override
	public void write(DataOutput output) throws IOException {
		// TODO Auto-generated method stub
		output.writeInt(this.orderId);
		output.writeInt(this.stockCount);
		output.writeInt(this.count);
		output.writeUTF(this.thingsId); 
		output.writeUTF(this.thingsName);
	}

	@Override
	public int compareTo(MapJoinBean o) {
		// TODO Auto-generated method stub
		int id = this.orderId - o.getOrderId();
		int countrel = this.thingsId.compareTo(o.getThingsId());
		
		return id != 0 ? (- id) : (countrel == 0 ? -1 : countrel);
	}
	
	

}
```

mapper类
```
package com.lanou.until;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import com.lanou.model.MapJoinBean;

public class JoinMapper extends Mapper<Object, Text, MapJoinBean, NullWritable> {
	private Text keyText = new Text();
	private MapJoinBean mapJoinBean = new MapJoinBean();
	private Map<String, MapJoinBean> containMap = new HashMap<>();

	// map会循环执行
	@Override
	protected void map(Object key, Text value, Mapper<Object, Text, MapJoinBean, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// FileInputStream fileInputStream = new FileInputStream("things.txt");
		String[] vals = value.toString().split("\\s+");
		// 只有订单信息
		mapJoinBean.setOrder(vals[0], vals[2], vals[3]);
		// 根据订单中的商品信息,来找到容器中对应的商品对象,进行数据填充.
		MapJoinBean thingsBean = containMap.get(mapJoinBean.getThingsId());

		// 将我们订单对象缺少的商品属性进行赋值
		mapJoinBean.setThings(thingsBean.getThingsName(), thingsBean.getStockCount());

		keyText.set(mapJoinBean.getThingsId());
		context.write(mapJoinBean, NullWritable.get());
	}

	// 在map之前只运行一次
	@Override
	protected void setup(Mapper<Object, Text, MapJoinBean, NullWritable>.Context context)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		// FileReader fileReader = new FileReader("things.txt");
		FileInputStream fileInputStream = new FileInputStream("things.txt");
		BufferedReader br = new BufferedReader(new InputStreamReader(fileInputStream, "UTF-8"));
		String line = "";
		// 定义一个容器,来存储所有解析出来的商品 对象 --->map
		while (!StringUtils.isEmpty(line = br.readLine())) {
			// 获取的商品信息
			System.out.println("line:" + line);
			String[] things = line.split("\\s+");
			// 创建商品对象
			MapJoinBean thingsBean = new MapJoinBean();
			// 给对象赋值
			thingsBean.setThings(things[0], things[1], Integer.parseInt(things[2]));
			// 商品id,商品对象的形式存在map中
			containMap.put(thingsBean.getThingsId(), thingsBean);

		}
	}

}
```

Reduce类
```
package com.lanou.until;

import java.io.IOException;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import com.lanou.model.MapJoinBean;

public class JoinReduce extends Reducer<MapJoinBean, NullWritable, MapJoinBean, NullWritable> {
	
	
	@Override
	protected void reduce(MapJoinBean arg0, Iterable<NullWritable> arg1,
			Reducer<MapJoinBean, NullWritable, MapJoinBean, NullWritable>.Context arg2)
			throws IOException, InterruptedException {
		// TODO Auto-generated method stub
		arg2.write(arg0, NullWritable.get());
	}

}
```

Test类
```
package com.lanou.test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

import com.lanou.model.MapJoinBean;
import com.lanou.until.JoinMapper;
import com.lanou.until.JoinReduce;

public class JinTest {
	public static void main(String[] args) throws IOException, URISyntaxException, ClassNotFoundException, InterruptedException {
		Configuration conf = new Configuration();
		  String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		    if (otherArgs.length < 2) {
		      System.err.println("Usage: wordcount <in> [<in>...] <out>");
		      System.exit(2);
		    }
		    Job job = Job.getInstance(conf, "word count");
		    job.setJarByClass(MapJoinTest.class);
		    job.setMapperClass(JoinMapper.class);
		    //job.setNumReduceTasks(0);
		    job.setReducerClass(JoinReduce.class);
		    job.setOutputKeyClass(MapJoinBean.class);
		    job.setOutputValueClass(NullWritable.class);
		    
		    // 添加缓存文件
		    job.addCacheFile(new URI("hdfs://172.18.24.28:9000/list/things.txt"));
		    
		    for (int i = 0; i < otherArgs.length - 1; i++) {
		      FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
		    }
		    
		    FileOutputFormat.setOutputPath(job, new Path(otherArgs[(otherArgs.length - 1)]));
//			yarn
			System.exit(job.waitForCompletion(true) ? 0 : 1);
	}

}
```
