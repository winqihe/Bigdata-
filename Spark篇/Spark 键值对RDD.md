# 键值对RDD

键值对RDD是一种常见的RDD元素类型，分组和聚合操作中经常会用到。

Spark中经常会用到键值对RDD，用于完成聚合计算。

### 创建RDD的准备工作

启动hdfs组件：``` start-dfs.sh```

启动pyspark  ```pyspark```

创建目录：```mkdir pairrdd```

之后vim word.txt 输入一些数据即可。

### 键值对RDD的创建

1. 从文件中加载

主要的创建方式是使用map()函数来实现。

```python
lines = sc.textFile("file:///usr/local/spark/mycode/pairrdd/word.txt")
pairrdd = lines.flatMap(lambda line : line.split(" ")).map(lambda word :(word,1))
pairrdd.foreach(print)
```

>1. (i,1)
>2. (love,1)
>3. (hadoop,1)
>4. (i,1)
>5. (love,1)
>6. (Spark,1)
>7. (Spark,1)
>8. (is,1)
>9. (fast,1)
>10. (than,1)
>11. (hadoop,1)

2. 通过并行集合（列表）创建RDD

```python
list = ["hadoop","hive","spark","spark"]
rdd = sc.parallelize(list)
pairrdd = rdd.map(lambda word : (word,1))
pairrdd.foreach(print)
```

>1. (Hadoop,1)
>2. (Spark,1)
>3. (Hive,1)
>4. (Spark,1)

### 常用的键值对操作转换

常用的键值对转换操作包括reduceByKey()、groupByKey()、sortByKey()、join()、cogroup()等

#### reduceByKey(func)

功能：使用func函数合并具有相同键的值，比如，reduceByKey((a,b) => a+b)，有四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)，对具有相同key的键值对进行合并后的结果就是：(“spark”,3)、(“hadoop”,8)。

```python
pairrdd.reduceByKey(lambda a,b : a+b).foreach(print)
```

>1. (Spark,2)
>2. (Hive,1)
>3. (Hadoop,1)

#### groupByKey()

功能：对具有相同键的值进行分组。对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)，采用groupByKey()后得到的结果是：(“spark”,(1,2))和(“hadoop”,(3,5))。

```python
pairRDD.groupByKey()
# PythonRDD[11] at RDD at PythonRDD.scala:48
pairRDD.groupByKey().foreach(print)
#('spark', <pyspark.resultiterable.ResultIterable object at 0x7f1869f81f60>)
#('hadoop', <pyspark.resultiterable.ResultIterable object at 0x7f1869f81f60>)
#('hive', <pyspark.resultiterable.ResultIterable object at 0x7f1869f81f60>)
```

#### keys()

会将键值对RDD中的key返回形成一个新的RDD。比如，对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)构成的RDD，采用keys()后得到的结果是一个RDD[Int]，内容{“spark”,”spark”,”hadoop”,”hadoop”}。

```python
pairRDD.keys()
pairRDD.keys().foreach(print)
```

#### values()

会将键值对RDD中的value返回形成一个新的RDD。比如，对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)构成的RDD，采用values()后得到的结果是一个RDD[Int]，内容是{1,2,3,5}。

```python
pairRDD.values()
pairRDD.values().foreach(print)
```

#### sortByKey()

返回一个根据键排序的RDD。

```python
pairRDD.sortByKey()
pairRDD.sortByKey().foreach(print)
```

#### mapValues(func)	

经常会遇到一种情形，我们只想对键值对RDD的value部分进行处理，而不是同时对key和value进行处理。对于这种情形，Spark提供了mapValues(func)，对键值对RDD中的每个value都应用一个函数，但是，key不会发生变化。比如，对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)构成的pairRDD，如果执行pairRDD.mapValues(lambda x : x+1)，就会得到一个新的键值对RDD，它包含下面四个键值对(“spark”,2)、(“spark”,3)、(“hadoop”,4)和(“hadoop”,6)。

```python
pairRDD.mapValues(lambda x : x+1)
pairRDD.mapValues(lambda x : x+1).foreach(print)
```

#### join

join表示内连接。

对于内连接，对于给定的两个输入数据集(K,V1)和(K,V2)，只有在两个数据集中都存在的key才会被输出，最终得到一个(K,(V1,V2))类型的数据集。pairRDD1是一个键值对集合{(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)}，pairRDD2是一个键值对集合{(“spark”,”fast”)}，那么，pairRDD1.join(pairRDD2)的结果就是一个新的RDD，这个新的RDD是键值对集合{(“spark”,1,”fast”),(“spark”,2,”fast”)}。

```python
pairRDD1 = sc.parallelize([('spark',1),('spark',2),('hadoop',3),('hadoop',5)])
pairRDD2 = sc.parallelize([('spark',fast)])
pairRDD1.join(pairRDD2)
pairRDD1.join(pairRDD2).foreach(print)
```

实战：

> 题目：给定一组键值对(“spark”,2),(“hadoop”,6),(“hadoop”,4),(“spark”,6)，键值对的key表示图书名称，value表示某天图书销量，请计算每个键对应的平均值，也就是计算每种图书的每天平均销量。

通过观察题目可以很明显的看出结果：("spark",4),("hadoop",5)

通过代码实现：

```python
rdd = sc.parallelize([(“spark”,2),(“hadoop”,6),(“hadoop”,4),(“spark”,6)])
//首先构建了一个RDD
rdd.mapValues(lambda x : (x,1)).reduceByKey(lambda x,y :(x[0]+y[0],x[1]+y[1])).mapValues(lambda x:(x[0]/x[1])).collect()
```

首先上面代码中变量x，y在不同的表达式中代表的含义不同。

针对构建得到的rdd，我们调用mapValues()函数，把rdd中的每个每个键值对(key,value)的value部分进行修改，把value转换成键值对(value,1)，其中，数值1表示这个key在rdd中出现了1次，为什么要记录出现次数呢？因为，我们最终要计算每个key对应的平均值，所以，必须记住这个key出现了几次，最后用value的总和除以key的出现次数，就是这个key对应的平均值。比如，键值对(“spark”,2)经过mapValues()函数处理后，就变成了(“spark”,(2,1))，其中，数值1表示“spark”这个键的1次出现。

reduceByKey(func)的功能是使用func函数合并具有相同键的值。这里的func函数就是Lamda表达式 x,y : (x[0]+y[0],x[1] + y[1])，这个表达式中，x和y都是value，而且是具有相同key的两个键值对所对应的value，比如，在这个例子中， (“hadoop”,(6,1))和(“hadoop”,(4,1))这两个键值对具有相同的key，所以，对于函数中的输入参数(x,y)而言，x就是(6,1)，序列从0开始计算，x[0]表示这个键值对中的第1个元素6，x[1]表示这个键值对中的第二个元素1，y就是(4,1)，y[0]表示这个键值对中的第1个元素4，y[1]表示这个键值对中的第二个元素1，所以，函数体(x[0]+y[0],x[1] + y[2])，相当于生成一个新的键值对(key,value)，其中，key是x[0]+y[0]，也就是6+4=10，value是x[1] + y[1]，也就是1+1=2，因此，函数体(x[0]+y[0],x[1] + y[1])执行后得到的value是(10,2)，但是，要注意，这个(10,2)是reduceByKey()函数执行后，”hadoop”这个key对应的value，也就是，实际上reduceByKey()函数执行后，会生成一个键值对(“hadoop”,(10,2))，其中，10表示hadoop书籍的总销量，2表示两天。同理，reduceByKey()函数执行后会生成另外一个键值对(“spark”,(8,2))。

对上面得到的两个键值对(“hadoop”,(10,2))和(“spark”,(8,2))所构成的RDD执行mapValues()操作，得到每种书的每天平均销量。当第一个键值对(“hadoop”,(10,2))输入给mapValues(x => (x[0] / x[1]))操作时，key是”hadoop”，保持不变，value是(10,2)，会被赋值给Lamda表达式x => (x[0] / x[1]中的x，因此，x的值就是(10,2)，x[0]就是10，表示hadoop书总销量是10，x[1]就是2，表示2天，因此，hadoop书籍的每天平均销量就是x[0] / x[1]，也就是5。mapValues()输出的一个键值对就是(“hadoop”,5)。同理，当把(“spark”,(8,2))输入给mapValues()时，会计算得到另外一个键值对(“spark”,4)。