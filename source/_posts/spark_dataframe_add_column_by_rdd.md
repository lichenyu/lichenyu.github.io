title: Spark - 使用RDD为DataFrame增加Column
date: 2018/11/19
categories:
- Program Development
tags:
- Spark
---


```scala
val newRdd = featureDf.rdd.map(r => Row.fromSeq(r.toSeq :+ KpiChiSquareThreshold.calChiSquareValue(baseline, r.toSeq.toList.asInstanceOf[List[Long]])))
val newSchema = StructType(featureDf.schema.fields :+ StructField("chi2", DoubleType, true))
val spark = SparkSession.builder().appName("any").getOrCreate()
val sqlContext = spark.sqlContext
featureDf = sqlContext.createDataFrame(newRdd, newSchema)
```
