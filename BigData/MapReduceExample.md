## 0. 介绍

* 开发MR任务，要求功能如下：
  * 文件1为工商信息，文件2为税务信息，两张表均包含一个公司id，可以做为关联查询字段。
  * 使用MR任务进行两个表的数据关联查询。
* 思路：
  * MapRduce任务join分为两种情况：
    * MapJoin，在Map端初始化的时候，加载join表数据。适用于join表数据表较少的情况下。
    * ReduceJoin，在Reduce端收集相同id的两张表数据，在Reduce端合并。适用于大表join大表的情况。
  * 本场景两个文件均可能是一个大文件。这里选择适用ReduceJoin

## 1. 搭建Hadoop环境

```sh
git clone https://github.com/big-data-europe/docker-hadoop.git
cd docker-hadoop
# 默认的Yarn ResourceManager节点没有开放8088端口，无法访问到yarn，需要额外暴露端口
docker-compose up
```

## 2. 创建文件

business.txt

公司id,公司名称,公司地址,公司类型

```
1,阿里巴巴,杭州,电商
2,比亚迪,深圳,汽车
3,华为,深圳,手机
4,小米,北京,手机
5,特斯拉,上海,汽车
6,拼多多,杭州,电商
```

tax.txt

公司id，税号，开户人，开户时间

```
1,0001,马云,2000-01-01
3,0003,任正非,1990-01-01
4,0004,雷军,2010-08-16
5,0005,马斯克,2019-12-23
7,0007,王兴,2008-01-01
```

## 3. 上传txt到Docker容器，然后上传到HDFS

```sh
# 拷贝txt文件
docker cp /Users/jack/Desktop/MR/business.txt 2b0c8e808057dceb75a11796c2740ae6d7da6ad10929f3328d6894fa87c24a8b:/data
docker cp /Users/jack/Desktop/MR/tax.txt 2b0c8e808057dceb75a11796c2740ae6d7da6ad10929f3328d6894fa87c24a8b:/data 

docker exec -it 2b0c8e808057dceb75a11796c2740ae6d7da6ad10929f3328d6894fa87c24a8b /bin/bash
cd /data
hdfs dfs -mkdir /data
hdfs dfs -put * /data
```

## 4. 创建MR任务

### 4.1 pom.xml

* 导入Hadoop相关pom依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>MapReduce</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <properties>
        <hadoop.version>3.2.1</hadoop.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>commons-cli</groupId>
            <artifactId>commons-cli</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.1.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-app</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-hs</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
    </dependencies>
    
</project>
```

### 4.1 CompanyInfo.java

* 公司基本信息

```java
package com.hurenjieee.hadoop;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import static com.hurenjieee.hadoop.GlobalConstant.BUSINESS;
import static com.hurenjieee.hadoop.GlobalConstant.TAX;

/**
 * 公司基本信息
 */
public class CompanyInfo implements WritableComparable<CompanyInfo> {

    /**
     * 公司id
     */
    private String companyId;

    /***** 公司信息 *****/

    /**
     * 公司名称
     */
    private String name;

    /**
     * 公司地址
     */
    private String address;

    /**
     * 企业类型
     */
    private String type;


    /***** 税务信息 *****/

    /**
     * 开户账号
     */
    private String accountId;

    /**
     * 户名
     */
    private String accountName;

    /**
     * 开户时间
     */
    private String createDate;

    private String table;

    public CompanyInfo() {
    }


    @Override
    public String toString() {
        return "CompanyInfo{" +
                "companyId='" + companyId + '\'' +
                ", name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", type='" + type + '\'' +
                ", accountId='" + accountId + '\'' +
                ", accountName='" + accountName + '\'' +
                ", createDate='" + createDate + '\'' +
                '}';
    }


    @Override
    public int compareTo(CompanyInfo o) {
        return 0;
    }


    /**
     * 输出数据
     *
     * @param dataOutput
     * @throws IOException
     */
    @Override
    public void write(DataOutput dataOutput) throws IOException {
        dataOutput.writeUTF(companyId);
        dataOutput.writeUTF(name);
        dataOutput.writeUTF(address);
        dataOutput.writeUTF(type);
        dataOutput.writeUTF(accountId);
        dataOutput.writeUTF(accountName);
        dataOutput.writeUTF(createDate);
        dataOutput.writeUTF(table);
    }

    /**
     * 从输入中读取数据
     *
     * @param dataInput
     * @throws IOException
     */
    @Override
    public void readFields(DataInput dataInput) throws IOException {
        companyId = dataInput.readUTF();
        name = dataInput.readUTF();
        address = dataInput.readUTF();
        type = dataInput.readUTF();
        accountId = dataInput.readUTF();
        accountName = dataInput.readUTF();
        createDate = dataInput.readUTF();
        table = dataInput.readUTF();
    }

    /**
     * 初始化工商信息
     *
     * @param companyId
     * @param name
     * @param address
     * @param type
     * @param isMap     是否是Map阶段
     */
    public void initBusinessInfo(String companyId, String name, String address, String type, Boolean isMap) {
        this.companyId = companyId;
        this.name = name;
        this.address = address;
        this.type = type;
        if (isMap) {
            // 设置空值，不然会NPE
            this.accountId = "";
            this.accountName = "";
            this.createDate = "";
            // 表类型为工商
            this.table = BUSINESS;
        }
    }

    /**
     * 初始化税务信息
     *
     * @param companyId
     * @param accountId
     * @param accountName
     * @param createDate
     * @param isMap       是否是Map阶段
     */
    public void initTax(String companyId, String accountId, String accountName, String createDate, Boolean isMap) {
        this.companyId = companyId;
        this.accountId = accountId;
        this.accountName = accountName;
        this.createDate = createDate;
        if (isMap) {
            // 设置空值，不然会NPE
            this.name = "";
            this.address = "";
            this.type = "";
            // 表类型为税务
            this.table = TAX;
        }
    }


    /**
     * 合并数据
     *
     * @param value
     */
    public void merge(CompanyInfo value) {
        if (StringUtils.equals(value.getTable(), BUSINESS)) {
            this.initBusinessInfo(value.getCompanyId(), value.getName(), value.getAddress(), value.getType(), Boolean.FALSE);
        } else {
            this.initTax(value.getCompanyId(), value.getAccountId(), value.getAccountName(), value.getCreateDate(), Boolean.FALSE);
        }
    }

    public String getCompanyId() {
        return companyId;
    }

    public void setCompanyId(String companyId) {
        this.companyId = companyId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getAccountId() {
        return accountId;
    }

    public void setAccountId(String accountId) {
        this.accountId = accountId;
    }

    public String getAccountName() {
        return accountName;
    }

    public void setAccountName(String accountName) {
        this.accountName = accountName;
    }

    public String getCreateDate() {
        return createDate;
    }

    public void setCreateDate(String createDate) {
        this.createDate = createDate;
    }

    public String getTable() {
        return table;
    }

    public void setTable(String table) {
        this.table = table;
    }

    /**
     * 转换文本输出
     *
     * @return
     */
    public String toText() {
        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append(StringUtils.defaultString(companyId, ""));
        stringBuffer.append(',');
        stringBuffer.append(StringUtils.defaultString(name, ""));
        stringBuffer.append(',');
        stringBuffer.append(StringUtils.defaultString(address, ""));
        stringBuffer.append(',');
        stringBuffer.append(StringUtils.defaultString(type, ""));
        stringBuffer.append(',');
        stringBuffer.append(StringUtils.defaultString(accountId, ""));
        stringBuffer.append(',');
        stringBuffer.append(StringUtils.defaultString(accountName, ""));
        stringBuffer.append(',');
        stringBuffer.append(StringUtils.defaultString(createDate, ""));
        return stringBuffer.toString();
    }
}

```

### 4.2 MyMap.java

```java
package com.hurenjieee.hadoop;

import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import java.io.IOException;

/**
 * Map
 * 收集工商、税务信息，相同到都发送到同一个Reduce中
 */
public class MyMap extends Mapper<LongWritable, Text, Text, CompanyInfo> {

    String businessFile;

    String taxFile;

    public MyMap() {
    }

    private String filename;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        // 获取文件路径（配置中加载）
        businessFile = context.getConfiguration().get(GlobalConstant.BUSINESS_FILE);
        taxFile = context.getConfiguration().get(GlobalConstant.TAX_FILE);
        // 获取所属的文件名称
        FileSplit inputSplit = (FileSplit) context.getInputSplit();
        filename = inputSplit.getPath().getName();
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] split = StringUtils.split(value.toString(), ',');
        CompanyInfo companyInfo = new CompanyInfo();
        // 如果是工商信息文件
        if (StringUtils.endsWith(businessFile, filename)) {
            companyInfo.initBusinessInfo(split[0], split[1], split[2], split[3], Boolean.TRUE);
        } else {
            companyInfo.initTax(split[0], split[1], split[2], split[3], Boolean.TRUE);
        }
        // 工商、税务根据company发送到同一个Reduce中。
        context.write(new Text(companyInfo.getCompanyId()), companyInfo);

    }
}

```

### 4.3 MyReduce.java

```java
package com.hurenjieee.hadoop;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Reduce
 * 合并数据
 */
public class MyReduce extends Reducer<Text, CompanyInfo, Text, NullWritable> {

    @Override
    protected void reduce(Text key, Iterable<CompanyInfo> values, Context context) throws IOException, InterruptedException {
        // 同一个key都数据都会在这里计算
        CompanyInfo companyInfo = new CompanyInfo();
        for (CompanyInfo value : values) {
            companyInfo.merge(value);
        }
        context.write(new Text(companyInfo.toText()), NullWritable.get());
    }
}

```

### 4.4 GlobalConstant.java

* 

```java
package com.hurenjieee.hadoop;

public class GlobalConstant {
    /** 工商 */
    public static final String BUSINESS = "BUSINESS";
    /** 税务 */
    public static final String TAX = "TAX";

    /** 工商文件 */
    public static final String BUSINESS_FILE = "BUSINESS_FILE";
    /** 税务文件 */
    public static final String TAX_FILE = "TAX_FILE";

}
```

### 4.5 MapReduce.java

```java
package com.hurenjieee.hadoop;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;


public class MapReduce {

    public static void main(String[] args) {
        try {
            // 输入文件
            String inputPath1 = args[0];
            String inputPath2 = args[1];
            // 输出文件
            String outputPath = args[2];
            // 读取加载配置
            Configuration configuration = new Configuration();
            configuration.set(GlobalConstant.BUSINESS_FILE, args[0]);
            configuration.set(GlobalConstant.TAX_FILE, args[1]);
            // 创建Job实例
            Job job = Job.getInstance(configuration, MapReduce.class.getSimpleName());
            // 确定运行Jar
            job.setJarByClass(MapReduce.class);
            // 设置输入/输出路径
            FileInputFormat.setInputPaths(job, new Path(inputPath1), new Path(inputPath2));
            FileOutputFormat.setOutputPath(job, new Path(outputPath));

            // 通过job设置输入/输出格式，输出为文本格式数据
            job.setInputFormatClass(TextInputFormat.class);
            job.setOutputFormatClass(TextOutputFormat.class);

            job.setMapperClass(MyMap.class);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(CompanyInfo.class);

            job.setReducerClass(MyReduce.class);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(NullWritable.class);

            // 设置Reduce个数
            job.setNumReduceTasks(2);

            // 提交作业
            boolean b = job.waitForCompletion(true);
            System.out.println(b ? "SUCCESS" : "ERROR");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 5. 编译、上传、执行任务

```sh
docker cp /Users/jack/Documents/Github/MapReduce/target/MapReduce-1.0-SNAPSHOT.jar 2b0c8e808057dceb75a11796c2740ae6d7da6ad10929f3328d6894fa87c24a8b:/data

# 执行任务
docker exec -it 2b0c8e808057dceb75a11796c2740ae6d7da6ad10929f3328d6894fa87c24a8b /bin/bash
hadoop jar MapReduce-1.0-SNAPSHOT.jar com.hurenjieee.hadoop.MapReduce /data/business.txt /data/tax.txt /data/result
```

![image-20200821204815278](http://img.hurenjieee.com/uPic/image-20200821204815278.png)

![image-20200821204854178](http://img.hurenjieee.com/uPic/image-20200821204854178.png)

```sh
# hdfs dfs -cat /data/result/part-r-00000
1,阿里巴巴,杭州,电商,0001,马云,2000-01-01
3,华为,深圳,手机,0003,任正非,1990-01-01
5,特斯拉,上海,汽车,0005,马斯克,2019-12-23
7,,,,0007,王兴,2008-01-01
# hdfs dfs -cat /data/result/part-r-00001
2,比亚迪,深圳,汽车,,,
4,小米,北京,手机,0004,雷军,2010-08-16
6,拼多多,杭州,电商,,,
```

