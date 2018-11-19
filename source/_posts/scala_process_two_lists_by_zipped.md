title: Scala - 使用zipped方法处理2个List
date: 2018/11/19
categories:
- Program Development
tags:
- Spark
---


```scala
// 2个List生成Map
val baselineMap = (featureList, baseline).zipped.toMap

// 2个List的值依次配对计算，生成List
val Si = (baseline, curline).zipped.map(_ + _)
val chisA = (baseline, EAi).zipped.map((d, e) => 1.0 * (d - e) * (d - e) / e)
```

`zipped`将两`List`的元素，依次生成一个`Tuple`，交给后续处理。
