title: Spark - 将DataFrame中的Vector col切分
date: 2018/11/12
categories:
- Program Development
tags:
- Spark
---


```scala
def splitVectorCol(df: DataFrame, append: Boolean = false): DataFrame = {
    lazy val first = df.first()(0)
    val numAttrs = first.asInstanceOf[Vector].size
    val attrs = Array.tabulate(numAttrs)(n => "feature_" + n)

    val vecToArray = udf((featureVec: Vector) => featureVec.toArray)
    val dfArr = df.withColumn("featureArr", vecToArray(col(df.schema.head.name)))

    // Create a SQL-like expression using the array
    val sqlExpr = attrs.zipWithIndex.map { case (alias, idx) => col("featureArr").getItem(idx).as(alias) }

    // Extract Elements from dfArr
    if (append)
        dfArr.select(col("*") +: sqlExpr : _*)
    else
        dfArr.select(sqlExpr: _*)
}
```
