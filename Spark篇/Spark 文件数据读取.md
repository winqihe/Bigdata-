Spark除了本地文件系统进行读写以外，还支持其他常见的文件格式和文件系统和数据库。

### 文件系统的数据读写

本地文件系统和分布式文件系统HDFS的数据读写

#### 本地文件系统的数据读写

```python
cd /usr/local/spark/mycode/wordcount
cat word.txt
```

```python
textFile = sc.textFile("file:///usr/local/spark/mycode/wordcount/word.txt")
textFile.first()
textFile = sc.textFile("file:///usr/local/spark/mycode/wordcount/word123.txt")
textFile.first()
textFile = sc.textFile("file:///usr/local/spark/mycode/wordcount/word.txt")
textFile = saveAsTextFile("file:///usr/local/spark/mycode/wordcount/writeback.txt")
```

```shell
cd /usr/local/spark/mycode/wordcount/writeback.txt/
ls
```

writeback这个目录下面又两个文件：

```
part-00000
_SUCCESS
```

可以通过cat命令来查看文件的数据

```python
textFile = sc.textFile("file:///usr/local/spark/mycode/wordcount/writeback.txt")
```

#### 分布式文件系统HDFS的数据读写

在读取HDFS的文件，首先启动Hadoop的HDFS组件。

```python
cd /usr/local/hadoop
./sbin/start-dfs.sh
```

启动HDFS之后，在文件系统中创建目录。

```python
./bin/hdfs dfs -mkdir -p /user/hadoop
```

查看一下HDFS文件系统中的目录和文件:

```python
./bin/hdfs dfs -ls .
```

我们把本地文件系统中的“/usr/local/spark/mycode/wordcount/word.txt”上传到分布式文件系统HDFS中

```python
./bin/hdfs dfs -put /usr/local/spark/mycode/wordcount/word.txt .
```

查看文件是否上传成功

```python
./bin/hdfs dfs -ls .
```

使用cat命令查看HDFS中的word.txt

```python
./bin/hdfs dfs -cat ./word.txt
```

编写语句从HDFS中加载word.txt文件，并显示第一行文本内容：

```python
var textFile = sc.textFile("hdfs://localhost:9000/user/hadoop/word.txt")
textFile.first()
```

需要注意的是，sc.textFile(“hdfs://localhost:9000/user/hadoop/word.txt”)中，“hdfs://localhost:9000/”是前面介绍Hadoop安装内容时确定下来的端口地址9000。

三条语句等价：

```python
val textFile = sc.textFile("hdfs://localhost:9000/user/hadoop/word.txt")
val textFile = sc.textFile("/user/hadoop/word.txt")
val textFile = sc.textFile("word.txt")
```

再把textFile中的内容写入到HDFS文件系统中(写到hadoop用户目录下)：

```python
var textFile = sc.textFile("word.txt")
textFile.saveAsTextFile("writeback.txt")
```

查看一下HDFS文件目录下是否写入成功

```python
./bin/hdfs dfs -ls .
```

可以看到有个writeback.txt目录，查看一下下面有什么文件：

```python
./bin/hdfs dfs -ls ./writeback.txt
```

执行结果，看到存在的两个文件。可以使用cat命令查看文件的数据，

### 不同文件格式的读写

#### 文本文件

把本地文件系统中的文本文件加载到RDD的语句：

```python
rdd = sc.textFile("file:///usr/local/spark/mycode/wordcount/word.txt")
```

如果我们给textFile()函数传递的不是文件名，而是一个目录，该目录下的所有内容都会被读取到RDD中。

关于把RDD中的数据保存到文件文件的语句：

```python
rdd.saveAsTextFile("file:///usr/local/spark/mycode/wordcount/outputFile")
```

在saveAsTextFile()函数的参数中给出的是目录，不是文件名，RDD中的数据会被保存到给定的目录下。

#### JSON

Spark提供了一个JSON样例数据文件，存放在“/usr/local/spark/examples/src/main/resources/people.json”中。people.json文件的内容如下：

```json
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

进入"/user/local/spark/mycode"目录中，并创建一个json子目录

```shell
cd /usr/local/spark/mycode
mkdir json
cd json
```

首先将本地文件系统中的people.json文件加载到RDD中

```python
jsonStr = sc.textFile("file:///usr/local/spark/examples/src/main/resources/people.json")
jsonStr.foreach(print)
```

Scala中有一个自带的JSON库——scala.util.parsing.json.JSON，可以实现对JSON数据的解析。JSON.parseFull(jsonString:String)函数，以一个JSON字符串作为输入并进行解析，如果解析成功则返回一个Some(map: Map[String, Any])，如果解析失败则返回None。

可以使用模式匹配来处理解析结果

```python
cd /usr/local/spark/mycode/json
vim testjson.py
```

再testjson.py中写代码：

```python
From pyspark import SparkContent
import json
sc = SparkContext('local','JSONAPP')
inputFile = "file:///home/dblab/people.json"
jsonStrs = sc.textFile(inputFile)
result = jsonStrs.map(lambda s : json.loads(s))
result.foreach(print)  
```

运行脚本

```python
python3 testjson.py
```

