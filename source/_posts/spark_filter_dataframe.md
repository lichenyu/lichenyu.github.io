title: Spark - 过滤DataFrame
date: 2018/11/12
categories:
- Program Development
tags:
- Spark
---


## 使用when ##

```scala
def filterDataframe(df: DataFrame, featureCols: Array[String],
                            diskrwInfo: (String, String, Double),
                            timestampInfo: (String, Int)): DataFrame = {
    var newDf = df.withColumn("invalidFlag", lit(true))

    // 特征不能都接近0
    for (c <- featureCols) {
        newDf = newDf.withColumn("invalidFlag", when(col(c) > 0.001, false)
                                                .otherwise(col("invalidFlag")))
    }

    // 磁盘读写之和不能小于阈值
    newDf = newDf.withColumn("invalidFlag", when(col(diskrwInfo._1) + col(diskrwInfo._2) < diskrwInfo._3, true)
                                            .otherwise(col("invalidFlag")))

    // 时间戳不能早于阈值
    newDf = newDf.withColumn("invalidFlag", when(col(timestampInfo._1) % (24 * 60 * 60 * 1000) < timestampInfo._2 * 60 * 60 * 1000, true)
                                            .otherwise(col("invalidFlag")))

    newDf = newDf.filter("invalidFlag = false").drop(col("invalidFlag"))
    newDf
}
```


## 使用UDF ##

```scala
def filterOutliers(featureVecDf: DataFrame, quantileThrArr: Array[Double]): DataFrame = {
    val filterUdf = udf((featureVec: Vector) => {
        var flag = false
        for (i <- quantileThrArr.indices) {
            if (featureVec(i) > quantileThrArr(i))
                flag = true
        }
        flag
    })
    val newDf = featureVecDf.withColumn("flag", filterUdf(col($(featureVecCol))))
            .filter("flag = false")
            .drop(col("flag"))

    newDf
}
```


## 使用filter，多条件 ##

```scala
featureDf = featureDf.filter(featureList.map(f => col(f) >= 0.001).reduce(_ or _))
```