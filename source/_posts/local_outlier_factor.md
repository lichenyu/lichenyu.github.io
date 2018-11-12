title: Local Outlier Factor
date: 2018/11/12
categories:
- Data Analysis
tags:
- KPI Check
---


- K邻近距离（k-distance）：在距离数据点$p$最近的几个点中，第$k$个最近的点跟点$p$之间的距离称为点$p$的K邻近距离，记为$\text{k-distance}(p)$。
- 可达距离（rechability distance）：可达距离的定义跟K邻近距离是相关的，给定参数$k$时，数据点$p$到数据点$o$的可达距离为“数据点$o$的K邻近距离”和“数据点$p$与点$o$之间的直接距离”的最大值。即，
{% math %}
\begin{aligned}
\text{reach_dist}_{k}(p,o) = \max\{\text{k-distance}(o),d(p,o)\}
\end{aligned}
{% endmath %}
- 局部可达密度（local rechability density）：局部可达密度的定义是基于可达距离的。对于数据点$p$，与其距离小于等于$\text{k-distance}(p)$的数据点称为它的k近邻（k-nearest-neighbor），记为$N_k(p)$。数据点$p$的局部可达密度，为它与k近邻的平均可达距离的倒数，即，
{% math %}
\begin{aligned}
\text{lrd}_{k}(p) = 1 / \frac{\sum_{o\in{N_{k}(p)}}\text{reach_dist}_{k}(p,o)}{|N_k(p)|}
\end{aligned}
{% endmath %}
- 局部异常因子（local outlier factor）：根据局部可达密度的定义，如果一个数据点跟其他点比较疏远的话，那么显然它的局部可达密度就小。但LOF算法衡量一个数据点的异常程度，并不是看它的绝对局部密度，而是看它跟周围邻近的数据点的相对密度。这样做的好处是可以允许数据分布不均匀、密度不同的情况。局部异常因子即是用局部相对密度来定义的。数据点$p$的局部相对密度（局部异常因子）为点$p$的k近邻的平均局部可达密度，跟数据点$p$的局部可达密度的比值，即，
{% math %}
\begin{aligned}
\text{LOF}_{k}(p) = \frac{\sum_{o\in{N_k(p)}}\frac{\text{lrd}_k(o)}{\text{lrd}_k(p)}}{|N_{k}(p)|} = \frac{\sum_{o\in{N_{k}(p)}}\text{lrd}_k(o)}{|N_{k}(p)|}/\text{lrd}_k(p)
\end{aligned}
{% endmath %}


根据局部异常因子的定义，如果数据点$p$的LOF得分在1附近，表明数据点p的局部密度跟它的邻居们差不多；如果数据点$p$的LOF得分小于1，表明数据点$p$处在一个相对密集的区域，不像是一个异常点；如果数据点$p$的LOF得分远大于1，表明数据点$p$跟其他点比较疏远，很有可能是一个异常点。
