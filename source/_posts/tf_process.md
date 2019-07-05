title: Tensorflow模型构建通用流程
date: 2019/07/01
categories:
- Program Development
tags:
- Tensorflow
---


## 1. prediction

#### dropout

### [seq2seq]

*encoder + decoder*

#### teacher forcing

*training model + inference model*

- training model用于训练，获取encoder、decoder的weights、biases，其中decoder的input使用**ground truth**
- inference model用于预测，保留training model的weights、biases，改变decoder的input使用**前一时刻输出（sampling）**


#### attention


## 2. loss


## 3. optimizer

- `tf.train.AdamOptimizer()`


## 4. train_step


## 5. batch and epoch


