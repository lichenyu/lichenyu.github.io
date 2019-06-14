title: NLP
date: 2019/06/14
categories:
- Machine Learning
tags:
- Model
---


## 词汇表征

- vocabulary + one-hot **representation**
  - 词汇表：V = [a, an, ..., zulu, <UNK>]
  - 表征：“an” = [0, 1, ..., 0, 0]
- featureized **representation**: word embedding
  - 表征：[feature1, feature2, ...]对应一个词（好处是可以计算各个词之间的距离，识别相似的词）（如何将词映射为一个特征向量？：是学来的）
  - word embedding含义是，将各个词embed嵌入到n维特征空间中去
  - 有点类似人类识别中，将图像encoding
  - cosine similarity $sim(u,v) = \frac{u^{T}v}{\| u \|_{2} \| v \|_{2}}$

## word embedding 学习

**word2vec: skip gram**

构造一个预测（或者映射）模型，输入一个词context（one-hot），预测对应到另一个词target（one-hot）
- 输入context、输出target的配对关系（间隔），是在一个句子中，在一定正负范围内，随机取的
- 而context的选择，则需要特别设计，避免频繁sample无意义的词

$$
o_{context} \overset{E}{\rightarrow} e_{context} = Eo_{context} \rightarrow softmax \rightarrow \hat{o}_{target}
$$

目标是通过训练这个映射关系，学习到embedding matrix $E = [e_{1}, e_{2}, e_{3}, ...]$
- $o_{context}$是context的one-hot表示
- $E$的各列，为词汇表中各个词的embedding；各行则为embedding的各个特征维度
- $Eo_{content}$相当于从$E$中提取content的embedding

词汇表较大时，skip gram中的softmax，分母求和，会有计算的大的问题

**negative sampling**

构造一个预测（或者映射）模型，输入一个词context（one-hot），预测对于一个词target是否存在对应（每个target一个二分类问题）
- 输入context、判断目标target的配对关系（间隔），是在一个句子中，在一定正负范围内，随机取的
- 而context、target的选择，则需要特别设计，避免频繁sample无意义的词

$$
o_{context} \overset{E}{\rightarrow} e_{context} = Eo_{context} \rightarrow \text{vocabulary_size} * \text{logistic_regression} \rightarrow lable
$$

- 使用vocabulary_size个二分类器，代替softmax，提升计算性能
- 训练数据每次构造，从句子中抽取一对context、target（positive sampling），在从词汇表中随机为context抽取k个target（negative sampling）
- 训练时，上述构造的k+1对context、target，训练k+1个分类器

