
Spark - 是个大数据集的集群计算框架；不使用MapReduce执行引擎，而是使用自己的分布式运行环境去执行集群工作；Spark也与MapReduce就API和运行有许多相似点；能够与Hadoop紧密集成：能够运行在YARN和同Hadoop的存储文件一同工作；  

Spark-最广为人知的是它将工作中Job之间的大数据集保存在内存中；最适用的场景有:遍历和交互分析；它的DAG引擎和用户体验也是选择Spark的理由，DAG能够处理任意的管道的操作,然后转换为单个Job。

## Installing Spark

1. `% tar xzf spark-x.y.z-bin-distro.tgz`
2. `% export SPARK_HOME=~/sw/spark-x.y.z-bin-distro`  
`% export PATH=$PATH:$SPARK_HOME/bin`

## An Example

1. 启动shell交互  
``` shell
% spark-shell
Spark context available as sc.
scala >
```
2. 加载文件  
```shell
scala > val line = sc.textFile("input/ncdc/micro-tab/sample.txt")
lines:org.apache.spark.rdd.RDD[String] = MappedRDD[1] at textFile at <console>:12
```
**Spark的加载和转换数据不会触发数据访问，这些只是创建了一个计算的执行计划；当比如:`foreach()` 执行时才会触发数据访问**
3. 拆分行`Stringg`  
```
scala > val records = lines.map(_.split("\t"))
records:org.apache.spark.rdd.RDD[Array[String]] = MappedRDD[2] at map at <console>:14
```
4. 删除错误的数据记录  
```
scala > val filtered = records.filter(rec =>(rec(1) != "9999" && rec(2).matches("[01459]")))
filtered:org.apache.spark.rdd.RDD[Array[String]] = FilteredRDD[3] at filter at <console>:16
```
5. 将数据格式转为 `key-value` ,便于`reduceByKey()`  
```
scala > val tuples = filterd.map(rec =>(rec(0).toInt,rec(1).toInt))
tuples:org.apache.spark.rdd.RDD[(Int,Int)] = MappedRDD[4] at map at <console>:18
```
6. 求每年的最大值  
```
scala> val maxTemps = tuples.reduceByKey((a,b)=> Math.max(a,b))
maxTemps:org.apache.spark.rdd.RDD[(Int,Int)] = MapPartitionsRDD[7] at reduceByKey at <console>:21
```
7. 触发计算动作  
```
scala > maxTemps.foreach(println(_))
(1950,22)
(1949,111)
--或者
scala>maxTemps.saveAsTextFile("output") #也会触发一个SparkJob
```

### Spark Applications,Jobs,Stages, and Tasks

Stages被SparkRuntime分解为`task`,以并行的方式运行在集群的RDD的分片中-如同MapReduce一样。

Job总数运行在Application( `SaprkContext` 实例代表)，它服务于RDDs的团和共享变量


### A Scala Standalone Application
### A Java Example
### A Python Example

## Resilient Distributed Datasets
RDD是每个Spark程序的核心。

### Creation
创建RDD的三种方式：
- 内存中的集合对象(parallelizing) 适用小数据量的CPU密集
- 利用外部的存储系统(HDFS)
- 转换一个现有的RDD

### Transformations and Actions
Spark提供2种RDD的操作：
1. 转换-由现有的RDD生成一个新的RDD
2. 动作-触发RDD的运算和结果的操作(返回结果给用户，保持结果到外部存储器)

区分一个操作是何种类型：看操作的返回类型；返回类型为RDD就是转换，其他的是动作了。

转换类型操作的结果可以保存在内存中，以便后续的操作更加效率。


### Persistence
```Scala
scala> tuples.cache()

#以上并不是直接将结果缓存，在这里只是给RDD作了标记-当SparkJob运行时，这个应该缓存
```


缓存的RDD只能被同一个应用访问；如果需要不同的应用访问共享访问此缓存，需要将结果缓存到外部存储中(`saveAs*()` 等方法缓存结果，在其他应用中适用`SparkContext`的(`textFile()` `hadoopFile()`等方法访问缓存))；当应用结束时，此应用的所有缓存都会被清理，不能被再次访问；

Kryo是Spark序列化的更好的选择，就大小和速度而言。

### Serialization
序列化数据和序列化函数

### 数据序列化
`spark.serializer` 设置为`org.apache.spark.serializer.KyroSerializer`

Kyro 不需要对象实现特定的接口就可以进行序列化

### 函数序列化
通常序列化函数能够工作，函数所属的类必须是实现了java同样的序列化接口

## Shared Variables

### Broadcast Variables
### Accumulators

## Anatomy of a Spark Job Run

driver-持有应用和job的任务
excutor-应用独有的，在应用期间运行，执行应用的任务
### Job Submission
### DAG Construction
### Task Scheduling
### Task Execution

## Excutors and Cluster Managers
### Spark on YARN

## Further Reading


















---
