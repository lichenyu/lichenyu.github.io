title: 数据预处理、特征工程
date: 2019/10/31
categories:
- Data Analysis
tags:
- Algorithm
---


在不引起重要信息丢失的前提下去除掉无关特征 (irrelevant feature) 和冗余特征 (redundant feature)


## 0. NaN值处理

### 0.1. 是否保留NaN值

如果NaN值过多，或者有明确的含义（比如用户拒绝选择），可以选择保留NaN值，作为一个类别型值

### 0.2. 删除NaN值过多的字段

`df1.dropna()`
检查各字段，如果某字段的NaN值过多，则去除该字段

### 0.3. 对某字段进行NaN值替换

`df1.fillna()`
对某字段进行处理，将NaN值替换为某值


## 1. 去除冗余

### 1.1. 低方差特征

`X_new = VarianceThreshold(threshold).fit_transform(X)`
阈值选取可以考虑通过均值标准化

### 1.2. 高相关特征

**变量两两相关性分析**
计算特征之间的皮尔森相关系数，当两特征的相关系数大于阈值时（一般阈值设为0.7或0.4），剔除其中一个特征（比如不均衡的）

```python
import seaborn as sns
import matplotlib.pyplot as plt

# pd.DataFrame
def plot_corr_heatmap(df):
    dfData = df.corr()
    plt.subplots(figsize=(9, 9)) # 设置画面大小
    # annot=True显示热力图上的数值大小
    # vmax是显示最大值
    # square=True将图变成一个正方形，默认是一个矩形
    # cmap="Blues"指定一种颜色配置方案
    sns.heatmap(dfData, annot=True, vmax=1, square=True, cmap="Blues")
    plt.savefig('./BluesStateRelation.png')
    plt.show()
```

### 1.3. 多重共线性特征

某特征可由其他一系列特征线性组合计算出

多元共线性带来的一些问题：
 - 特征不显著
 - 对参数估计值的正负号产生影响
 - 对准确性能影响较小，但会影响模型（求算）的复杂程度

举个简单的例子：比如对一个二元线性回归模型，自变量是x1和x2，如果我们画图大家可以很自然的想象出一个三维（三轴）坐标系。假如x1和x2之间没有多重共线性，那么这个模型就是一个确定了的超平面。但假如x1和x2有很强的多重共线性，那么这个模型就近似是一个直线向量，而以这个直线所拟合出来的平面是无数个的（穿过一条直线的平面是不固定的）。这也就造成了回归系数的不确定性，以及模型无法稳定。

`variance_inflation_factor`
通常用VIF值来衡量一个变量和其他变量的多重共线性，当某个变量的VIF大于阈值时（一般阈值设为10 或 7），需要剔除特征

此外，使用降维、模型加正则项的方式，也可以解决

```python
b=np.array(df1)
VIF=[]
for i in range(df1.shape[1]):
    vifi=variance_inflation_factor(exog=b,exog_idx=i)
    VIF.append(vifi)
print(VIF)
```


## 2. 特征选择-基于相关性检验

### 2.1. 单特征与label之间的相关性

`SelectKBest`、`SelectPercentile`

**评价指标**
对于分类: `chi2`（卡方检验）、`f_classif`（F检验）、`mutual_info_classif`（互信息）
对于回归: `f_regression`（F检验）、`mutual_info_regression`（互信息）

例如，**chi2检验**：
chi2检验用于**独立性**检验，即关注两变量的**相关**程度（注意区别于，同分布检验-ks检验）
{% post_link chi2_test chi2检验具体介绍 %}
chi2越大，x与y越相关（取前k个）
`X_new = SelectKBest(chi2, k=5).fit_transform(X, y)`
`X_new = SelectPercentile(chi2, percentile=10).fit_transform(X, y)`


## 3. 特征选择-基于预训练模型-有效性

`RFE`、`SelectFromModel`

最常用的包装法是递归消除特征法（recursive feature elimination，RFE）。递归消除特征法使用一个机器学习模型来进行多轮训练，每轮训练后，消除若干权值系数的对应的特征，再基于新的特征集进行下一轮训练。

嵌入法也是用机器学习的方法来选择特征，但是它和RFE的区别是它不是通过不停的筛掉特征来进行训练，而是使用的都是特征全集。

### 3.1. 线性模型权重

对于包装法，以经典的SVM-RFE算法来讨论这个特征选择的思路。这个算法以支持向量机来做RFE的机器学习模型选择特征。它在第一轮训练的时候，会选择所有的特征来训练，得到了分类的超平面$wx+b=0$后，如果有n个特征，那么RFE-SVM会选择出$w$中分量的平方值$w^2_i$最小的那个序号$i$对应的特征，将其排除。在第二次分类的时候，特征数就剩下$n-1$个了，我们继续用这$n-1$个特征和输出值来训练SVM，同样的，去掉$w^2_i$最小的那个序号$i$对应的特征。以此类推，直到剩下的特征数满足我们的需求为止。
`X_new = RFE(estimator=SVC(kernel="linear", C=1), n_features_to_select=5, step=1).fit_transform(X, y)`

对于嵌入法，最常用的是使用L1正则化和L2正则化来选择特征。正则化惩罚项越大，那么模型的系数就会越小。当正则化惩罚项大到一定的程度的时候，部分特征系数会变成0，当正则化惩罚项继续增大到一定程度时，所有的特征系数都会趋于0. 但是我们会发现一部分特征系数会更容易先变成0，这部分系数就是可以筛掉的。也就是说，我们选择特征系数较大的特征。常用的L1正则化和L2正则化来选择特征的基学习器是逻辑回归。
`X_new = SelectFromModel(LogisticRegression(penalty="l1", C=0.1)).fit_transform(X, y)`

### 3.2. 树形模型feature importance

此外对于嵌入法，也可以使用决策树或者GBDT。
一般来说，可以得到特征系数coef或者可以得到特征重要度(feature importances)的算法才可以做为嵌入法的基学习器。
`SelectFromModel`可以用来处理任何带有`coef_`或者`feature_importances_`属性的训练之后的评估器。
`X_new = SelectFromModel(GradientBoostingClassifier()).fit_transform(X, y)`


## 4. 特征选择-面向仅有类别型（分箱）变量的场景

对于“评分卡”这种模型形式，所有特征都是分箱的（并使用WOE编码）。
计算各个特征的**IV值**，取IV值大的特征。
某特征的IV值计算如下：

$$
IV = \sum_{i}(\frac{\#G_{i}}{\#G} - \frac{\#B_{i}}{\#B}) \times WOE_{}i  = \sum_{i}(\frac{\#G_{i}}{\#G} - \frac{\#B_{i}}{\#B}) \times \ln{\frac{\frac{\#G_{i}}{\#G}}{\frac{\#B_{i}}{\#B}}}
$$

其中$i$为该特征的某个分箱。


## 5. 特征选择-特征在时间上稳定性

### 5.1. PSI

Population Stability Index
关注特征的取值，会不会随着时间的推移发生较大的变动

对某特征，获取base集与target集（分别对应不同时期的数据）。在base集进行分箱，分出$i$个等距区间，在target集使用同样标准划分。$#_{i, base}$表示该特征在base集中$i$分箱的记录数量，则该特征PSI定义为：
$$
PSI = \sum_i [(\#_{i, target} - \#_{i, base}) \ln (\frac {\#_{i, target}} {\#_{i, base}})]
$$
PSI衡量了某特征在两个数据集中各个分箱中记录数量的分布，是否发生了变化；也就反应了该特征的总体分布，是否发生了变化
可以计算base集与一些列不同后推时间的target集的PSI值，检查PSI变化情况




### 5.2. KS检验

同分布检验


## 6. 降维

常见的降维方法有主成分分析法（PCA）和线性判别分析（LDA），线性判别分析本身也是一个分类模型。
PCA和LDA有很多的相似点，其本质是要将原始的样本映射到维度更低的样本空间中，但是PCA和LDA的映射目标不一样：

- PCA是为了让映射后的样本具有最大的发散性（各点相距尽可能远）
- LDA是为了让映射后的样本有最好的分类性能（同类别的点相距尽可能近，不同类别的点相距尽可能远）

所以说PCA是一种无监督的降维方法，而LDA是一种有监督的降维方法。

`X_new = PCA(n_components=5).fit_transform(X)`
`X_new = LDA(n_components=5).fit_transform(X, y)`






