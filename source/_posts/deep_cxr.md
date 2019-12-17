title: Deep CXR
date: 2019/12/05
categories:
- Machine Learning
tags:
- Model
---


### 0. Naive Method

- categorical fields -> one-hot encoding
- 上面直接堆叠deep，deep一般是MLP，multi-layer perceptron

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
  - 在embedding和deep之间加入了product layer进行不同field之间特征交叉
  - 使用inner product、outer product
    - inner product相当于引入了FM的内积部分
    - outer product会得到一个矩阵，而非像inner product得到一个数。向后进入deep在一个unit中加权求和时，与一个数操作类似，只不过是对矩阵各个元素加权求和
    - product得到的结果（一系列数值），同时concat一阶特征，作为deep输入
  - 不是被动通过deep学习特征之间关系，而是强制通过product层引入field之间特征交叉

#### [NFM](Neural Factorization Machines for Sparse Predictive Analytics.pdf)
  - 2017
  - NFM中使用：$$y(\mathbf{x}) = w_0 + \sum_{i=1}^{n}w_ix_i + f(\mathbf{x})$$
  - $$f(\mathbf{x})$$，即文中所谓Bi-Interaction，本质上是一个pooling，对得到的embeddings进行了encode而非concat：
    - 1）embedding层，隐向量$$\times$$feat_val（因为不只one-hot输入，还有real-value输入），获取embeddings$$ = [\mathbf{e_1}, \mathbf{e_2}, \mathbf{e_3}, ...]$$；
    - 2）$$\text{Bi-Interaction Sum Pooling} = \sum_{i=1}\sum_{j=i+1}\mathbf{e_i} \text{  element-wise product  } \mathbf{e_j} $$，即“各种$$\mathbf{e_i}, \mathbf{e_j}$$组合的元素乘”的向量和，此结果shape与$$\mathbf{e_i}$$的shape相同，都是embedding_size
    - 这个结果encode了embedding space的2-order feature interactions，作为“将feature embeddings进行concat”的替代
    - 3）上面再堆叠deep
  - 相较于FNN，NFM的网络结构中多了一个Bi-Interaction - sum pooling，形式上更像FM（但element-wise product后未将向量各元素累加，仍保留向量形式，所以还不是内积）
  - 相较于PNN，PNN也是不想直接将embeddings concat后输入到deep中，觉得会欠缺feature interactions，对此PNN的做法是各两个embeddings内积得到一个数之后，再将各个数concat在一起，再concat上一阶特征，输入到DNN中；而NFM则不是对各两个embeddings做内积，而是做元素乘，然后sum pooling
  - 相较于Wide & Deep（或DeepFM）,NFM不通过joint一个Wide来引入cross features，而是在embedding与deep之间加一个Bi-Interaction pooling来描述特征交叉（in the low level）
  - NFM更像FNN、PNN，但也可看成是下文Wide & Deep的一种改进形式（因为一阶特征未进入deep，而是进入了最终结果层）

#### [AFM](Attentional Factorization Machines Learning the Weight of Feature Interactions via Attention Networks.pdf)
  - 2017
  - AFM是在NFM基础上改的，作者中包含NFM的作者
  - 加入了attention机制，即，Pire-wise Interaction之后，根据$$ij$$组合不同，后续操作时分配不同的权重
  - 即在Interaction Layer中，sum pooling时，为每个“$$\mathbf{e_i}, \mathbf{e_j}$$组合的元素乘”分配一个权重，再做向量和。（此结果shape与$$\mathbf{e_i}$$的shape相同）
  - 这个attention weight是不好学习的：$$n^2$$数量级、有可能组合未出现过。对此参数的学习，单独使用一个所谓的attention network
    - attention network输入为各种“$$\mathbf{e_i}, \mathbf{e_j}$$组合的元素乘，输出维度为各组合数量的softmax。这样对于某个$$i,j$$，我们将该网络的结果$$ij$$项作为attention weight
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
    - $$x^{[l]}\text{.shape} = (1, d)$$
    - $${x^{[0]}}^T x^{[l]}$$是个矩阵，使得交叉特征阶数+1，$$+ x^{[l]}$$则保存了之前所有低阶交叉特征
    - 最后层的输出，相当于各阶交叉特征加权求和

### 3. 加入attention机制

#### [DIN](Deep Interest Network for Click-Through Rate Prediction.pdf)
  - Ali 2018
  - 对于“用户的历史商品列表”这个multi-hot特征，在embedding后得到不定个数的embeddings向量。通常做法是将这项向量sum pooling或avg pooling，然后concat到其他特征embeddings，输入给deep。进一步，在做sum pooling时引入weights，表现不同的历史兴趣item，重要性不同
    - 在遇见sum pooling、avg pooling这种东西时，都可以考虑进一步引入weights
    - 如果只是加一个general weights，其描述的“不同item的重要程度”是固定的，不会区分场景。再进一步，对于不同candidate ad，可以让weights不同，即，对于不同的ad，商品embeddings的weights是不同的，然后再weighted sum pooling，获取基于历史信息的、用户对该商品的兴趣表达，即引入了attention机制
    - 理解一下，attention不仅仅是加weights，还需要对于不同的query，使用的是一组不同的weights（一个query对应一组weights）
    - recall一下AFM，其实是在其设计的NFM的interaction layer中出现了sum pooling，就向其中进一步引入了weights，从而声称使用了attention机制（但其实并没有query的概念，也就没有为每个query生成不同的weights组的概念，仅是一个general的weightes进行sum pooling）
  - attention weights的计算：local activation unit
    - 某item embedding（待weighted项）与ad embedding（query）进行element-wise product，然后concat两个embeddings，输入一层deep（输出未进行softmax，想描述weights的绝对差异）（u_units对齐embedding_size，保证weight能element-wise product上去）（end-to-end training）

#### [DIEN](Deep Interest Evolution Network for Click-Through Rate Prediction.pdf)
  - Ali 2018
  - 本深度模型关注点，不同域之间的特征交互描述 -> 用户兴趣表示
  - 对于“行为列表”这类特征，考虑时序，使用seq模型
    - 不同于DIN中，用户行为列表是没有顺序的，故使用multi-hot
    - 对用户interest的representation，往往直接使用用户行为（embedding & pooling）；DIEN尝试通过显示的用户行为，提取隐含的用户当前兴趣表示、及兴趣发展趋势
  - 时序兴趣特征提取：
    - GRU，使用下一动作作为监督信号
    - 描述特征之间的影响
  - 兴趣发展进程：
    - Attention Update GRU
    - 描述与target AD相关的interest

#### [DSIN]
  - Ali 2019
  - 以session为粒度

### TODO

xDeepFM
FiBiNet

Behavior Sequence Transformer
Deep Spatio-Temporal Neural Networks for Click-Through Rate Prediction
ATRank: An Attention-Based User Behavior Modeling Framework for Recommendation

AutoInt


CNN系CCPM、FGCNN




---

### 4. 多任务学习

通常来说，如果你发现你需要的不止一个损失函数，你就是在做多任务学习。

- MTL可以有效增加用于模型训练的样本量
- 学到更好的特征表示、关系
- 约束参数适应于各个子任务，降低过拟合风险

#### 4.1. 解决“训练数据空间”与“预测数据空间”不一致的问题

#### [ESMM](Entire Space Multi-Task Model An Effective Approach for Estimating Post-Click Conversion Rate.pdf)
  - Ali 2018
  - pCVR的问题：**Sample Selection Bias**，即pCVR训练基于点击数据，推理却基于曝光数据
    - 现在想对pCVR使用全特征域（曝光数据）学习
  - 分析：$$\text{pCTCVR} = \text{pCTR} \times \text{pCVR}$$
    - 其中，$$\text{pCTCVR}$$与$$\text{pCTR}$$是可以通过全特征域学习的
    - 故，可以引入**多任务学习**MTL，使用全域特征显示学习$$\text{pCTCVR}$$与$$\text{pCTR}$$，同时隐式学习$$\text{pCVR}$$
  - 模型关键：
    - 一个$$\text{pCVR}$$网络、一个$$\text{pCTR}$$网络（网络结构为简单的embedding+MLP），结果相乘得到$$\text{pCTCVR}$$
    - loss精巧设计，显示的、joint训练$$\text{pCTR}$$与$$\text{pCTCVR}$$，这两部分都可以通过全域特征进行训练，一定程度的消除了Sample Selection Bias
$$
\begin{aligned}
loss &= L_{pCTR} + L_{pCTCVR} \\
&= L(\theta_{pCTR}, \theta_{pCVR}) \\
&= \sum_{i=1}^{N}l(y_i,f(\mathbf{x}_i;\theta_{pCTR})) + \sum_{i=1}^{N}l(y_i\&z_i),f(\mathbf{x}_i;\theta_{pCTR})*f(\mathbf{x}_i;\theta_{pCVR}))
\end{aligned}
$$
    - loss由两部分组成，即显示学习、joint训练的pCTR部分和pCTCVR部分。而pCVR只能隐式学习，因为1）loss中各任务应该是基于全域特征学习的；2）其负样本标注是有些问题的。在loss中，要保证label都是准的。所以，pCVR不适合基于全域特征，在loss中直接进行监督
      - pCVR使用全域特征，隐含了一个负样本标注的问题：“**CVR预估到底要预估什么**”，论文虽未明确提及，但理解这个问题才能真正理解CVR预估困境的本质。想象一个场景，一个item，由于某些原因，例如在feeds中的展示头图很丑，它被某个user点击的概率很低，但这个item内容本身完美符合这个user的偏好，若user点击进去，那么此item被user转化的概率极高。CVR预估模型，预估的正是这个转化概率，**它与CTR没有绝对的关系，很多人有一个先入为主的认知，即若user对某item的点击概率很低，则user对这个item的转化概率也肯定低，这是不成立的。更准确的说，CVR预估模型的本质，不是预测“item被点击，然后被转化”的概率（CTCVR），而是“假设item被点击，那么它被转化”的概率（CVR）。**这就是不能直接使用全部样本训练CVR模型的原因，因为咱们压根不知道这个信息：那些unclicked的item，假设他们被user点击了，它们是否会被转化。**如果直接使用0作为它们的label，会很大程度上误导CVR模型的学习。**
    - 注意，loss虽然是对pCTR部分和pCTCVR部分的描述，但其参数还是来自于pCTR网络和pCVR网络的，即$$\theta_{pCTR}, \theta_{pCVR}$$
    - $$y$$是点击label，$$z$$是转化label，$$l$$是单独一个网络的loss函数（交叉熵），$$f$$是模型函数
    - 此外，两个网络共享embedding，joint学习，这也解决了转化正样本稀疏的问题，**Data Sparsity**
      - 当然，pCVR网络的训练、推理，都使用全域特征了哦。即，隐式学得了一个输入全域特征，预测$$pCVR$$的子网络
  - 为什么使用乘法形式，而不使用除法形式，即$$\text{pCVR} = \frac{\text{pCTCVR}}{\text{pCTR}}$$？
    - 因为$$pCTR$$数值往往较小，容易造成数值溢出



#### ESM2

#### DUPN

#### Modeling Task Relationships in Multi-task Learning with Multi-gate Mixture-of-Experts



















