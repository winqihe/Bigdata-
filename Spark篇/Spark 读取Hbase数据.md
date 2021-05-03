Spark读取Hbase数据

### 创建一个Hbase表

进入Hbase，创建一个student表。首先启动hadoop

```shell
cd /usr/local/hadoop
./sbin/start-all.sh
```

启动Hbase

```python
cd /usr/local/hbasse
./bin/start-hbase.sh
./bin/hbase shell
```

在Hbase中直接可以创建表，而不用创建数据库。

```python
list
//使用list命令可以查看Hbase数据库中有哪些已经创建好的表
```

```python
//如果库中存在student表，则先执行
disable 'student'
drop 'student
```

在表中录入以下数据：

```
id   | name     | gender | age  |
+------+----------+--------+------+
|    1 | Xueqian  | F      |   23 |
|    2 | Weiliang | M      |   24 |
+------+----------+--------+------+
```

```shell
create 'student','info'
```

在创建create语句后，跟上表名student，然后跟上列族名info

这个列族info包含三个列name，gender，age。Hbase中会有一个系统默认的属性作为行键，无需自行创建。

创建完表后，可以通过describe命令查看表的基本信息：

```shell
describe 'student'
```

Hbase中使用put命令添加数据。注意：一次只能为一个表的一行数据的一个列（也就是一个单元格，单元格是HBase中的概念）添加一个数据，所以直接用shell命令插入数据效率很低，在实际应用中，一般都是利用编程操作数据。

```shell
put	'student','1','info:name','Xueqian'
put 'student','1','info:gender','F'
put 'student','1','info:age','23'
put 'student','2','info:name','Weiliang'
put 'student','2','info:gender','M'
put 'student','2','info:age','24'
```

查看插入的数据

```
get 'student','1'
```

查看所有数据

```python
scan 'student'
```

### 配置Spark

把Hbase的lib目录下的一个jar文件拷贝到Spark中，所有hbase开头的jar文件、guava-12.0.1.jar、htrace-core-3.1.0-incubating.jar和protobuf-java-2.5.0.jar，可以打开一个终端按照以下命令来操作：

```python
cd  /usr/local/spark/jars
mkdir  hbase
cd  hbase
cp  /usr/local/hbase/lib/hbase*.jar  ./
cp  /usr/local/hbase/lib/guava-12.0.1.jar  ./
cp  /usr/local/hbase/lib/htrace-core-3.1.0-incubating.jar  ./
cp  /usr/local/hbase/lib/protobuf-java-2.5.0.jar  ./
```

在Spark 2.0版本上缺少相关把hbase的数据转换python可读取的jar包，需要我们另行下载。

打开[spark-example-1.6.0.jar](https://mvnrepository.com/artifact/org.apache.spark/spark-examples_2.11/1.6.0-typesafe-001)下载jar包,执行:

```shell
mkdir -p /usr/local/spark/jars/hbase
mv ~/下载/spark-examples* /usr/local/spark/jars/hbase/
```

然后打开spark-env.sh文件，设置Spark的spark-env.sh文件，告诉spark可以在那个路径下找到hbase相关的jar文件。

```shell
cd /usr/local/spark/conf
vim spark-env.sh
```

在spark-env.sh最前面加一行内容：

```shell
export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath):$(/usr/local/hbase/bin/hbase classpath):/usr/local/spark/jars/hbase/*
```

### 编写程序读取Hbase数据

如果想要Spark读取Hbase，需要使用SparkContext提供的newAPIHadoopRDD API将表的内容以RDD的形式加载到Spark中。

```
pyspark
```

```python
host = 'localhost'
table = 'student'
conf = {"hbase.zookeeper.quorum": host, "hbase.mapreduce.inputtable": table}
keyConv = "org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter"
valueConv = "org.apache.spark.examples.pythonconverters.HBaseResultToStringConverter"
hbase_rdd = sc.newAPIHadoopRDD("org.apache.hadoop.hbase.mapreduce.TableInputFormat","org.apache.hadoop.hbase.io.ImmutableBytesWritable","org.apache.hadoop.hbase.client.Result",keyConverter=keyConv,valueConverter=valueConv,conf=conf)
count = hbase_rdd.count()
hbase_rdd.cache()
output = hbase_rdd.collect()
for (k, v) in output:
        print (k, v)
```

### 编写程序向Hbase写入数据

````python
host = 'localhost'
table = 'student'
keyConv = "org.apache.spark.examples.pythonconverters.StringToImmutableBytesWritableConverter"
valueConv = "org.apache.spark.examples.pythonconverters.StringListToPutConverter"
conf = {"hbase.zookeeper.quorum": host,"hbase.mapred.outputtable": table,"mapreduce.outputformat.class": "org.apache.hadoop.hbase.mapreduce.TableOutputFormat","mapreduce.job.output.key.class": "org.apache.hadoop.hbase.io.ImmutableBytesWritable","mapreduce.job.output.value.class": "org.apache.hadoop.io.Writable"}
 
rawData = ['3,info,name,Rongcheng','4,info,name,Guanhua']
sc.parallelize(rawData).map(lambda x: (x[0],x.split(','))).saveAsNewAPIHadoopDataset(conf=conf,keyConverter=keyConv,valueConverter=valueConv)
````

查看数据：

```shell
scan 'student'
```





