title: Spark - DataFrame UDF返回多个col
date: 2018/11/12
categories:
- Program Development
tags:
- Spark
---


```scala
def addClusteringCols(featureVecSepDf: DataFrame, model: KMeansModel): DataFrame = {
    val clusteringUdf = udf((featureVec: Vector) => {
        //(label, distance)
        val rtv = KpiClusteringThreshold.calDist2Center(featureVec, model.clusterCenters)
        Array(rtv._1, rtv._2)
    })
    featureVecSepDf.withColumn("udfResult", clusteringUdf(col($(featureVecCol)))).cache
            .withColumn("label", col("udfResult")(0))
            .withColumn("distance", col("udfResult")(1))
            .drop("udfResult")
}
```
