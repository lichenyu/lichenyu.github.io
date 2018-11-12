title: Spark - 动态获取UDF
date: 2018/11/12
categories:
- Program Development
tags:
- Spark
---


```scala
def getPreprocessFunc(meanList: List[Double], stdList: List[Double]): Vector => Vector = (featureVec: Vector) => {
    var featureListBuffer = new ListBuffer[Double]()
    for (i <- 0 until featureVec.size){
        if (stdList(i) > 0)
            featureListBuffer += (featureVec(i) - meanList(i)) / stdList(i)
        else
            featureListBuffer += featureVec(i)
    }
    Vectors.dense(featureListBuffer.toList.toArray)
}

val preprocessUdf = udf(getPreprocessFunc(meanList, stdList))
df.withColumn("featureVecCol", preprocessUdf(col("featureVecCol")))
```
