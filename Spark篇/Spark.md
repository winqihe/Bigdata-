# Spark

### RDD创建：

1. 读取外部数据集
2. 调用SparkContext的parallelize方法，在Driver中一个已经存在的集合上创建

### 创建RDD的准备工作：

首先启动hdfs组件 使用start-hdfs.sh来启动

之后启动pyspark命令：pyspark

创建工作目录：在目录下创建RDD mkdir RDD

### 从文件系统中加载数据创建RDD

Spark采用textFile()方法来从本地文件系统中加载数据创建RDD，

textFile()使用文件的url作为参数，这个url既可以是本地文件系统的地址，也可以是分布式文件系统hdfs的地址

```python
lines = sc.textFile("file:///usr/local/spark/mycode/rdd/word.txt")
```

从本地文件系统加载数据，使用file:///作为前缀

从hdfs分布式文件系统加载数据：

```python
lines = sc.textFile("hdfs://localhost:9000/user/hadoop/word.txt")
lines = sc.textFile("/user/hadoop/worf.txt")
lines = sc.textFile("word.txt")
```

注意：

如果使用本地数据创建RDD，那么必须保证worker节点上面的也能通过相同的访问路径访问到文件。

textFile()方法也可以接受第2个输入参数（可选），用来指定分区的数目。

### 通过并行集合创建RDD

```python
nums = [1,2,3,4]
rdd = sc.parallelize(nums)
```

### RDD操作

RDD一般有两种操作

转换 ：基于现有的数据集创建一个新的数据集

行动 ：对现有的数据集进行计算，返回计算值。

### 转换操作

对于RDD而言，每一次转换操作都会产生不同的RDD，供给下一个转换使用。转换得到的RDD是惰性求值的，整个转换过程只记录了转换的轨迹，并不会发生真正的计算，只有遇到行动操作时，才会发生真正的计算，开始从血缘关系源头开始，进行物理的转换操作。

操作的转换操作api：

\* filter(func)：筛选出满足函数func的元素，并返回一个新的数据集
\* map(func)：将每个元素传递到函数func中，并将结果返回为一个新的数据集
\* flatMap(func)：与map()相似，但每个输入元素都可以映射到0或多个输出结果
\* groupByKey()：应用于(K,V)键值对的数据集时，返回一个新的(K, Iterable)形式的数据集
\* reduceByKey(func)：应用于(K,V)键值对的数据集时，返回一个新的(K, V)形式的数据集，其中的每个值是将每个key传递到函数func中进行聚合

### 行动操作

行动操作会触发计算，Spark执行行动操作的时候，才会进行真正的计算，从文件中加载数据，完成一次又一次的转换操作，最后，完成行动操作得到结果。

行动操作的api：

\* count() 返回数据集中的元素个数
\* collect() 以数组的形式返回数据集中的所有元素
\* first() 返回数据集中的第一个元素
\* take(n) 以数组的形式返回数据集中的前n个元素
\* reduce(func) 通过函数func（输入两个参数并返回一个值）聚合数据集中的元素
\* foreach(func) 将数据集中的每个元素传递到函数func中运行*

### 惰性机制

```python
lines = sc.textFile("data.txt")
//textFile方法只是一个转换，并不会执行计算。
lineLengths =line.map(lambda s:len(s))
//计算每行的长度，map方法也是一个转换，不会执行计算
totalLength = lineLengths.reduce(lambda a,b :a+b)
//reduce方法是一个行动操作，会触发计算,Spark将计算分解成多个任务在不同的机器上执行。
```

### 持久化

```python
list = ["hadoop","spark","hive"]
rdd = sc.parallelize(list)
print(rdd.count()) //行动操作，触发一次计算
print(',',join(rdd.collect())) //行动操作，触发一次计算
```

上面代码的执行过程中，总共触发了两次计算。

可以通过持久话（缓存）机制来避免反复计算对内存的开销。

使用persist()方法对一个RDD标记为持久化，叫标记为持久化，因为出现persist()的地方并不会立即计算生成RDD把它持久化，而是要遇到一个行动操作才会对计算结果进行持久化。