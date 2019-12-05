title: Deep CXR
date: 2019/12/05
categories:
- Machine Learning
tags:
- Model
---


## Deep Models for CTR, CVR Prediction

### 0. Naive Method

- categorical fields -> one-hot encoding
- 上面直接堆叠deep

### 1. 对高维稀疏特征进行embedding

one-hotted features高维稀疏，这对LR可以勉强忍受，但对DNN计算开销太大，拖低了模型性能effectiveness & efficiency。
对此，将各个field进行embedding，即将一个高维稀疏向量（one-hot），映射为一个低维稠密向量（real values）。

#### [Deep Crossing](Deep Crossing Web-Scale Modeling without Manually Crafted Combinatorial Features.pdf)
  - Microsoft 2016
  - embedding：通过dense，将各个稀疏域转化为低维稠密特征向量并concat在一起
  - deep：5层残差网络（何恺明ResNet），避免梯度爆炸、消失

### 2. 交叉特征与低维特征

DNN的优势在于能够自动学得高抽象层次（high-level）的特征，这些特征在而CV、NLP中往往是非常有用的，因为CV、NLP往往解决的是human替代任务，即模型代替人工进行工作。
但CXR预估并不是一个human替代任务，因而除了DNN能够隐式学得的高抽象层次特征，还应该尝试向模型中引入一些其他的特征来提升模型性能。
引入点包括：1)交叉特征；2)低抽象层次特征。

对于交叉特征，由于DNN的节点计算方式（对输入加权求和，再套一个非线性，不存在特征间相乘），对feature interactions并不能很好的描述。
为此，可以主动做特征交叉，补充到模型中，以增加模型对于特征间interactions的表达能力。
模型结构调整方式：生成的交叉特征，可以作为deep的input的补充，输入到deep中（串行结构）；也可以concat在deep旁边，与deep结果一起进入最终预测节点（并行结构）。

对于低抽象层次特征，由于我们的CXR预估任务并不是只关注于高抽象层次，故模型不仅使用DNN的高抽象层次特征对应结果，还应该考虑引入低抽象层次特征，综合两种层次的特征来生成结果。
模型结构调整方式：低抽象层次特征，concat在deep的低抽象层次特征旁边，与一起进入最终预测节点（并行结构）。

#### 2.1. 引入FM对特征做embedding [描述feature interactions]

- FM的特异之处：
  - FM基于隐向量，可以学得数据集中出现次数较少、或根本未出现的特征组合
  - 使用FM进行embedding，尤其对于高维稀疏的one-hotted features，能通过2阶交叉特征的引入，增强模型的预测能力
  - 当然，FM的shortcomings也很明显：1）linear model；2）at most 2-order features

#### [FNN](%28FNN%29 Deep Learning over Multi-field Categorical Data – A Case Study on User Response Prediction.pdf)
  - 2016
  - pretrain一个基于Dense的FM，做embedding层，上面叠加deep
    - embedding层包含FM一阶、二阶部分（将u_units看做$$1+k$$，即1个一阶参数，u_units - 1个二阶参数）

#### 2.2. 在embedding和deep之间插入“特征交互描述”层 [描述feature interactions]

我们知道基于dense做embedding、然后进行concat、然后上面堆叠deep，并不能描述feature interactions。
对此，在embedding和deep之间插入“特征交互描述”层，来专门描述feature interactions。

#### [PNN](%28PNN%29 Product-based Neural Networks for User Response Prediction.pdf)
  - 2016
  - 在embedding和deep之间加入了product layer进行不同field之间特征交叉（同时concat了一阶特征）
  - 使用inner product、outer product
    - inner product相当于引入了FM的内积部分
    - outer product会得到一个矩阵，而非像inner product得到一个数。向后进入deep在一个unit中加权求和时，与一个数操作类似，只不过是对矩阵各个元素加权求和
  - 不是被动通过deep学习特征之间关系，而是强制通过product层引入field之间特征交叉

#### [NFM](Neural Factorization Machines for Sparse Predictive Analytics.pdf)
  - 2017
  - NFM中使用：$$y(\mathbf{x}) = w_0 + \sum_{i=1}^{n}w_ix_i + f(\mathbf{x})$$
  - $$f(\mathbf{x})$$，即文中所谓Bi-Interaction，本质上是一个pooling，对得到的embeddings进行了encode而非concat：
    - 1）embedding层，隐向量$$\times$$feat_val（因为不只one-hot输入，还有real-value输入），获取embeddings$$ = [\mathbf{e_1}, \mathbf{e_2}, \mathbf{e_3}, ...]$$；
    - 2）$$\text{Bi-Interaction Sum Pooling} = \sum_{i=1}\sum_{j=i+1}\mathbf{e_i} \text{  element-wise product  } \mathbf{e_j} $$，即“各种$$\mathbf{e_i}, \mathbf{e_j}$$组合的元素乘”的向量和，此结果shape与$$\mathbf{e_i}$$的shape相同，都是embedding_size
    - 这个结果encode了embedding space的2-order feature interactions，作为“将feature embeddings进行concat”的替代
  - 相较于FNN，NFM的网络结构中多了一个Bi-Interaction - sum pooling，形式上更像FM（但element-wise product后未将向量各元素累加，仍保留向量形式，所以还不是内积）
  - 相较于PNN，PNN也是不想直接将embeddings concat后输入到deep中，觉得会欠缺feature interactions，对此PNN的做法是各两个embeddings内积得到一个数之后，再将各个数concat在一起，再concat上一阶特征，输入到DNN中；而NFM则不是对各两个embeddings做内积，而是做元素乘，然后sum pooling
  - 相较于Wide & Deep（或DeepFM）,NFM不通过joint一个Wide来引入cross features，而是在embedding与deep之间加一个Bi-Interaction pooling来描述特征交叉（in the low level）
  - NFM更像FNN、PNN，但也可看成是下文Wide & Deep的一种改进形式（因为一阶特征未进入deep，而是进入了最终结果层）

#### [AFM](Attentional Factorization Machines Learning the Weight of Feature Interactions via Attention Networks.pdf)
  - 2017
  - AFM是在NFM基础上改的，作者中包含NFM的作者
  - 加入了attention机制，即，Pire-wise Interaction之后，根据$ij$组合不同，后续操作时分配不同的权重
  - 即在Interaction Layer中，sum pooling时，为每个“$$\mathbf{e_i}, \mathbf{e_j}$$组合的元素乘”分配一个权重，再做向量和。（此结果shape与$$\mathbf{e_i}$$的shape相同）
  - 这个attention weight是不好学习的：$$n^2$$数量级、有可能组合未出现过。对此参数的学习，单独使用一个所谓的attention network
    - attention network输入为各种“$$\mathbf{e_i}, \mathbf{e_j}$$组合的元素乘，输出维度为各组合数量的softmax。这样对于某个$$i,j$$，我们将该网络的结果$ij$项作为attention weight
  - Interaction Layer之后，原文的AFM没有再堆叠deep了

#### 2.3. 为deep joint一个wide结构 [同时考虑低抽象层次特征]

#### [Wide & Deep](Wide & Deep Learning for Recommender Systems.pdf)
  - Google 2016
  - Wide部分：直接对原始特征、交叉特征（人工选择）做linear
  - Deep部分：先用Dense做embedding，然后上面叠加deep
    - Deep部分的参数使用随机初始化，未pretrain
  - joint train Wide部分和Deep部分，即，两部分做linear后，套一个sigmoid，作为结果，使用一个loss做优化
    - Wide部分本身就是linear，所以两部分做linear时不需要再乘一个权重，只给Deep部分结果乘一个权重即可

#### 2.3.1. 在wide中考虑特征交叉 [同时考虑低抽象层次特征、描述feature interactions]

#### [DeepFM](DeepFM A Factorization-Machine based Neural Network for CTR Prediction.pdf)
  - Huawei 2017
  - 对Wide & Deep引入FM进行优化
  - 最底层使用Dense
    - Wide部分：使用FM（包含一阶、二阶、常数项，Dense作为FM的二阶部分）的结果，作为低抽象层次特征
    - Deep部分：使用Dense（FM的二阶部分）作为embedding层
    - Dense、FM的参数使用随机初始化，未pretrain（若做pretrain，label其实与overall network的label相同，所以无需做pretrain，直接end-to-end训练）

#### [DCN](Deep & Cross Network for Ad Click Predictions.pdf)
  - Google 2017
  - Wide部分 -> Cross部分：N层可显式的表示到N阶的特征交叉
    - 每层为一个变换节点，为$$x^{[l+1]} = w^{[l]}({x^{[0]}}^T x^{[l]}) + b^{[l]} + x^{[l]}$$
    - $$x^{[l]}$$.shape = $$(1, d)$$
    - $${x^{[0]}}^T x^{[l]}$$是个矩阵，使得交叉特征阶数+1，$$+ x^{[l]}$$则保存了之前所有低阶交叉特征
    - 最后层的输出，相当于各阶交叉特征加权求和
















### 3. 加入attention机制



#### [DIN]
  - Ali 2018

#### [DIEN]
  - Ali 2018

#### [DSIN]
  - Ali 2019

### TODO

xDeepFM

AutoInt
FiBiNet

CNN系CCPM、FGCNN



