title: Spark Cheatsheet
date: 2018/10/22
categories:
- Program Development
tags:
- Spark
---


## 1.1. RDD ##

### 1.1.1. 创建 ###

RDD可以通过两种方式创建：
1. 读取一个外部数据集。比如，从本地文件加载数据集，或者从HDFS文件系统、HBase、Cassandra、Amazon S3等外部数据源中加载数据集。Spark可以支持文本文件、SequenceFile文件（Hadoop提供的 SequenceFile是一个由二进制序列化过的key/value的字节流组成的文本存储文件）和其他符合Hadoop InputFormat格式的文件。
2. 调用SparkContext的parallelize方法，在Driver中一个已经存在的集合（数组）上创建。

```scala
val lines1 = sc.textFile("file:///usr/local/spark/mycode/rdd/word.txt")

val lines2 = sc.textFile("hdfs://localhost:9000/user/hadoop/word.txt")
```

textFile()方法的输入参数，可以是文件名，也可以是目录，也可以是压缩文件等。

```scala
val array = Array(1,2,3,4,5)
val rdd = sc.parallelize(array)
```


### 1.1.2. 操作 ###

**窄依赖/宽依赖**：窄依赖表现为一个父RDD的分区对应于一个子RDD的分区（一对一）；宽依赖则表现为存在一个父RDD的一个分区对应一个子RDD的多个分区（一对多）。

**转换**：每一次转换操作都会产生不同的RDD，供给下一个“转换”使用。转换得到的RDD是惰性求值的，也就是说，整个转换过程只是记录了转换的轨迹，并不会发生真正的计算，只有遇到行动操作时，才会发生真正的计算，开始从血缘关系源头开始，进行物理的转换操作。

常见的转换：

|||
|---|---|
|filter(func)|筛选出满足函数func的元素，并返回一个新的数据集|
|map(func)|将每个元素传递到函数func中，并将结果返回为一个新的数据集|
|flatMap(func)|与map()相似，但每个输入元素都可以映射到0或多个输出结果|
|groupByKey()|应用于(K,V)键值对的数据集时，返回一个新的(K, Iterable)形式的数据集|
|reduceByKey(func)|应用于(K,V)键值对的数据集时，返回一个新的(K, V)形式的数据集，其中的每个值是将每个key传递到函数func中进行聚合|

**行动**：行动操作是真正触发计算的地方。

常见的行动：

|||
|---|---|
|count()|返回数据集中的元素个数|
|collect()|以数组的形式返回数据集中的所有元素|
|first()|返回数据集中的第一个元素|
|take(n)|以数组的形式返回数据集中的前n个元素|
|reduce(func)|通过函数func（输入两个参数并返回一个值）聚合数据集中的元素|
|foreach(func)|将数据集中的每个元素传递到函数func中运行|

**持久化**：

在Spark中，RDD采用惰性求值的机制，每次遇到行动操作，都会从头开始执行计算。
在一些情形下，我们需要多次调用不同的行动操作，这就意味着，每次调用行动操作，都会触发一次从头开始的计算。
可以通过持久化（缓存）机制避免这种重复计算的开销。

使用`persist()`方法对一个RDD标记为持久化。
（之所以说“标记为持久化”，是因为出现persist()语句的地方，并不会马上计算生成RDD并把它持久化，而是要等到遇到第一个行动操作触发真正计算以后，才会把计算结果进行持久化，持久化后的RDD将会被保留在计算节点的内存中被后面的行动操作重复使用。）
`persist()`的圆括号中包含的是持久化级别参数。
比如，`persist(MEMORY_ONLY)`表示将RDD作为反序列化的对象存储于JVM中，如果内存不足，就要按照LRU原则替换缓存中的内容；`persist(MEMORY_AND_DISK)`表示将RDD作为反序列化的对象存储在JVM中，如果内存不足，超出的分区将会被存放在硬盘上。
一般而言，使用`cache()`方法时，会调用`persist(MEMORY_ONLY)`。

```scala
val list = List("Hadoop","Spark","Hive")
val rdd = sc.parallelize(list)
rdd.cache()   //会调用persist(MEMORY_ONLY)，但是，语句执行到这里，并不会缓存rdd，这是rdd还没有被计算生成
println(rdd.count())   //第一次行动操作，触发一次真正从头到尾的计算，这时才会执行上面的rdd.cache()，把这个rdd放到缓存中
println(rdd.collect().mkString(","))   //第二次行动操作，不需要触发从头到尾的计算，只需要重复使用上面缓存中的rdd
```

使用`unpersist()`方法手动地把持久化的RDD从缓存中移除。

**常见键值对RDD操作**：

|||
|---|---|
|reduceByKey(func)|pairRDD.reduceByKey((a,b) => a+b)对具有相同key的键值对进行合并、计算。比如输入四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)；输出(“spark”,3)、(“hadoop”,8)。|
|groupByKey()|pairRDD.groupByKey()对具有相同键的值进行分组。比如输入四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)；输出(“spark”,(1,2))和(“hadoop”,(3,5))。|
|sortByKey()|pairRDD.sortByKey()根据key排序一个RDD。|
|mapValues(func)|pairRDD.mapValues(x => x+1)只会对键值对RDD中的每个value都应用一个函数，而key不会发生变化。比如输入四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)；输出(“spark”,2)、(“spark”,3)、(“hadoop”,4)和(“hadoop”,6)|
|join|pairRDD1.join(pairRDD2)表示内连接，即对于给定的两个输入数据集(K,V1)和(K,V2)，只有在两个数据集中都存在的key才会被输出，最终得到一个(K,(V1,V2))类型的数据集。|


## 1.2. 共享变量 ##

### 1.2.1. 广播变量 ###

广播变量允许程序开发人员在每个机器上缓存一个只读的变量（类似分布式缓存）。

```scala
val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar.value  //Array(1, 2, 3)
```


### 1.2.2. 累加器 ###

累加器是仅仅被相关操作累加的变量，通常可以被用来实现计数器（counter）和求和（sum）。可以通过调用SparkContext.longAccumulator()或者SparkContext.doubleAccumulator()来创建。
运行在集群中的任务，就可以使用add方法来把数值累加到累加器上，但是，这些任务只能做累加操作，不能读取累加器的值，只有任务控制节点（Driver Program）可以使用value方法来读取累加器的值。

```scala
scala> val accum = sc.longAccumulator("My Accumulator")
accum: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 0, name: Some(My Accumulator), value: 0)
scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))
scala> accum.value
res1: Long = 10
```



