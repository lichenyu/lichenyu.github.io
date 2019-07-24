title: Weight of Evidence (WOE)
date: 2019/07/24
categories:
- Data Analysis
tags:
- Algorithm
---

## weight of evidence (WOE)

woe编码可对标于dummy变量，对一个**类别型**特征进行**编码**，转换成一个数值型变量。
例如，在credit scoring的case中，label是一个binary的值（Good，Bad）。则，对于某一个类别型特征（或数值型特征分箱后），其各类别的woe可定义为：

$$
WOE_{i} = \ln{\frac{\frac{\#G_{i}}{\#G}}{\frac{\#B_{i}}{\#B}}} = \ln{\frac{\frac{\#G_{i}}{\#B_{i}}}{\frac{\#G}{\#B}}}
$$

其中，$\\#G_{i}$为该类别Good数量，$\\#G$为总Good数量，$\\#B_{i}$为该类别Bad数量，$\\#B$为总Bad数量。
可以看出，$\ln$中的值在描述“本类别的G、B分布比例，与总体的G、B分布比例，是否相似”。
若本类别的分布极为特殊，则woe值较大，表明本类别具有较强的区分性；若本类别的分布与总体基本一致，则woe近似为0，表明本类别区分性较弱。

**分箱与woe：**
- For continuous independent variables: First, create bins (categories / groups) for a continuous independent variable and then combine categories with similar WOE values and replace categories with WOE values. Use WOE values rather than input values in your model.
- For categorical independent variables: Combine categories with similar WOE and then create new categories of an independent variable with continuous WOE values. In other words, use WOE values rather than raw categories in your model. The transformed variable will be a continuous variable with WOE values. It is same as any continuous variable.

**Rules related to 分箱与woe：**
- Each category (bin) should have at least 5% of the observations.
- Each category (bin) should be non-zero for both non-events and events.
- The WOE should be distinct for each category. Similar groups should be aggregated.
- For logistic regression, the WOE should be monotonic, i.e. either growing or decreasing with the groupings. (It is because logistic regression assumes there must be a linear relationship between logit function and independent variable.)
- Missing values are binned separately.

Why combine categories with similar WOE?
It is because the categories with similar WOE have almost same proportion of events and non-events. In other words, the behavior of both the categories is same.


## information value (IV)

WOE only considers the relative risk of each bin, without considering the proportion of accounts in each bin. （各个bin占总体的比例，或者说各个bin中的绝对数量）
The information value can be utilised instead to assess the relative contribution of each bin.
$$
IV = \sum_{i}(\frac{\#G_{i}}{\#G} - \frac{\#B_{i}}{\#B}) \times WOE_{}i
$$

Information value is one of the most useful technique to **select important variables** in a predictive model. It helps to rank variables on the basis of their importance.

**特征选择**

If the IV statistic is:
- Less than 0.02, then the predictor is not useful for modeling (separating the Goods from the Bads)
- 0.02 to 0.1, then the predictor has only a weak relationship to the Goods/Bads odds ratio
- 0.1 to 0.3, then the predictor has a medium strength relationship to the Goods/Bads odds ratio
- 0.3 to 0.5, then the predictor has a strong relationship to the Goods/Bads odds ratio.
- 0.5, suspicious relationship (Check once)

Information value is not an optimal feature (variable) selection method when you are building a classification model other than binary logistic regression (for eg. random forest or SVM) as conditional log odds (which we predict in a logistic regression model) is highly related to the calculation of weight of evidence. In other words, it's designed mainly for binary logistic regression model. Also think this way - Random forest can detect non-linear relationship very well so selecting variables via Information Value and using them in random forest model might not produce the most accurate and robust predictive model.

