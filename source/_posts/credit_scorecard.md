title: 信用评分卡
date: 2019/07/25
categories:
- Data Analysis
tags:
- Credit Scoring
---

## 评分卡构建

### 通用流程

1. 数据预处理
  - 缺失值处理（填充、丢弃字段）
  - 异常值处理（丢弃记录）
  - *探索性数据分析EDA（各字段统计信息）*
2. 特征选择
  - 分箱
  - 特征选择
  - 相关性分析
3. 模型训练
  - Logistic Regression
  - *系数符号筛选特征：模型中各个特征的系数，如果出现了负数，说明一些特征的线性相关性较强*
  - 性能评估（AUC）
4. 计算评分

### 分箱

- 等距分箱
- 等频分箱
- 基于WOE分箱
  - {% post_link weight_of_evidence WOE与IV %}

### 特征选择

- 基于IV特征选择
  - {% post_link weight_of_evidence WOE与IV %}
- 基于RF特征选择
  获取随机森林中，各特征importance

### 计算评分

首先来看一下评分卡的形式：

|特征名称|特征范围|得分|
|---|---|---|
|基准分||xxx|
|特征1|分箱1|aaa|
||分箱2|bbb|
||...||
||分箱p|ppp|
|特征2|分箱1|mmm|
||分箱2|nnn|
||...||
||分箱q|qqq|
|...|...|...|

由于得分最终是由基础分、各“特征-某分箱”得分的累加，可知特征值与得分是线性关系。
又由逻辑回归，可知
$$
\ln(\frac{p}{1-p}) = \mathbf{\theta}^{T} \mathbf{x} = w_{0} + w_{1}x_{1} + ... + w_{0}x_{n}
$$
其中$p$为预测好用户概率，$\frac{p}{1-p}$为odds，$x_{i}$为第$i$维特征。
利用此线性关系，不妨定义信用评分$Score$为：
$$
\begin{align*}
Score &= A + B \times \ln(\frac{p}{1-p})\\
&= A + B \times (w_{0} + w_{1}x_{1} + ... + w_{0}x_{n})\\
&=(A + Bw_{0}) + Bw_{1}x_{1} + Bw_{2}x_{2} + ... + Bw_{n}x_{n}
\end{align*}
$$
$x_{1}, ..., x_{n}$是各维特征的WOE编码。为了容易看出评分卡的求算，不妨将各维特征的分箱展开，则上式为：
$$
\begin{align*}
Score = (A + Bw_{0}) &+ Bw_{1}WOE_{1,1}\sigma_{1,1} + Bw_{1}WOE_{1,2}\sigma_{1,2} + ... + Bw_{1}WOE_{1,p}\sigma_{1,p}\\
&+ Bw_{2}WOE_{2,1}\sigma_{2,1} + Bw_{2}WOE_{2,2}\sigma_{2,2} + ... + Bw_{2}WOE_{2,q}\sigma_{2,q}\\
&+ ...
\end{align*}
$$
其中，$WOE_{1,2}$为第1个特征第2个分箱的WOE值，$\sigma_{1,2}$是一个取值为0、1的指示变量，$\sigma_{1,1}, \sigma_{1,2}, ..., \sigma_{1,p}$只能有一个为1。
由此，填充评分卡。注意，评分卡的输入变量，只能是$\sigma$指示变量，即WOE要提出去。

|特征名称|特征范围|得分|
|---|---|---|
|基准分||$A + Bw_{0}$|
|特征1|分箱1|$Bw_{1}WOE_{1,1}$|
||分箱2|$Bw_{1}WOE_{1,2}$|
||...||
||分箱p|$Bw_{1}WOE_{1,p}$|
|特征2|分箱1|$Bw_{2}WOE_{2,1}$|
||分箱2|$Bw_{2}WOE_{2,2}$|
||...||
||分箱q|$Bw_{2}WOE_{2,q}$|
|...|...|...|

上面卡片中，$w$由逻辑回归模型得出，WOE由特征分箱求算，剩下的$A, B$可以看做超参数，预先定义。
由于
$$
Score = A + B \times \ln(odds)
$$
可以进行设置，给出两个场景条件：当$odds = \theta_{0}$时，$Score = P_{0}$；当$odds = 2\theta_{0}$翻倍时，$Score = P_{0} + PDO$，以此来求得$A, B$。
$$
\begin{align*}
&P_{0} = A + B \times \ln(\theta_{0})\\
&P_{0} + PDO = A + B \times \ln(2\theta_{0})
\end{align*}
$$
可得:
$$
\begin{align*}
&B = \frac{PDO}{\ln2}\\
&A = P_{0} - B\ln(\theta_{0})
\end{align*}
$$

一般行业规则：当$odds = \theta_{0} = 50$时，$Score = 600$；当$odds$翻倍时，$Score = P_{0} + 20$。







