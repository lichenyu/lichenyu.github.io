title: Spark - DataFrame UDF返回多个col
date: 2018/11/12
categories:
- Program Development
tags:
- Spark
---

- 使用Array
```scala
def addClusteringCols(featureVecSepDf: DataFrame, model: KMeansModel): DataFrame = {
    val clusteringUdf = udf((featureVec: Vector) => {
        //(label, distance)
        val rtv = KpiClusteringThreshold.calDist2Center(featureVec, model.clusterCenters)
        Array(rtv._1, rtv._2)   // same type
    })
    featureVecSepDf.withColumn("udfResult", clusteringUdf(col($(featureVecCol)))).cache
            .withColumn("label", col("udfResult")(0))
            .withColumn("distance", col("udfResult")(1))
            .drop("udfResult")
}
```

- 使用Tuple
```scala
val calUdf = udf((r: Row) => {
    val chi2 = KpiChiSquareThreshold.calChiSquareValue(baseline, r.toSeq.toList.asInstanceOf[List[Long]])
    if (chi2 > kpiChiSquareThr)
        (1, Math.abs(chi2 - kpiChiSquareThr), if (Math.abs(chi2 - kpiChiSquareThr) / kpiChiSquareThr > 1) 1.0 else Math.abs(chi2 - kpiChiSquareThr) / kpiChiSquareThr)
    else
        (0, 0.0, 0.0)
})
df = df.withColumn("detect", calUdf(struct(featureList.map(f => col(f)): _*)))
        .withColumn($(outputCol), col("detect._1"))
        .withColumn($(absDevCol), col("detect._2"))
        .withColumn($(relDevCol), col("detect._3"))
        .drop("detect")
```
