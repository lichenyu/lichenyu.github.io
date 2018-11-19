title: Spark - 从DataFrame中选取一个List的Columns
date: 2018/11/19
categories:
- Program Development
tags:
- Spark
---


```scala
var featureDf = dataset.select(featureList.head, featureList.tail: _*)
```

`tail`方法返回一个`Array`，使用`: _*`将其作为函数的多个入参。
