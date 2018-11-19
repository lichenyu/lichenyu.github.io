title: Spark - DataFrame UDF使用多个col
date: 2018/11/19
categories:
- Program Development
tags:
- Spark
---

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
