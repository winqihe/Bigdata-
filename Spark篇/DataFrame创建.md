Spark使用全新的SparkSession接口替代Spark1.6中的SQLContext及HiveContext接口来实现其对数据加载、转换、处理等功能。SparkSession实现了SQLContext及HiveContext所有功能

SparkSession支持从不从的数据源加载数据，并把数据转换成DataFrame，并且支持把DataFrame转换成SQLContext自身的表，然后用SQL语句来操作数据。

使用SparkSession来创建DataFrame

Spark提供了样例数据，“/usr/local/spark/examples/src/main/resources/”这个目录下，又people.json和people.txt

people.json文件的内容如下：

```
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

people.txt文件的内容如下：

```
Michael, 29
Andy, 30
Justin, 19
```

从people.json中读取数据并生成DataFrame并显示数据

打开pyspark

```shell
cd /usr/local/spark
./bin/pyspark
```

执行命令：

```python
spark = SparkSession.builder.getOrCreate()
df = spark.read.json("file:///usr/local/spark/examples/src/main/resources/people.json")
df.show()
```

执行一些常用的DataFrame操作：

```python
//打印信息模式
df.printSchema
//选择多列
df.select(df.name,df.age+1).show()
//条件过滤
df.filter(df.age>20).show()
//分组聚合
df.groupBy("age").count().show()
//排序
df.sort(df.age.desc()).show()
//多列排序
df.sort(df.age.desc(),df.name.asc()).show()
//对列进行重命名
df.select(df.name.alias("username"),df.age).show()
```





