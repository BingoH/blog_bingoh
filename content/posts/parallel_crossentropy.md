---
title: "About Cross Entropy"
date: 2020-09-02T19:19:30+08:00
draft: false
---

Cross entropy loss(交互熵)是多分类问题中最常用的损失函数。它可以由最大似然得到，也可以从KL散度(Kullback-Leibler divergence)导出。本文记录cross entropy loss的推导和梯度计算，以及在实现时遇到的问题。

## 推导
### 最大似然
对于样本点$(x_i, y_i)$,我们最大化其在目标类别上的概率$P(y_i|x_i, \theta)$,$\theta$表示模型参数。若样本$x_i$提取特征$f_i \in \mathbb{R}^d$，通常利用softmax分类器得到概率分布$P(y=j|f_i) = \frac{\exp(w_j^Tf_i)}{\sum_{k=1}^C \exp(w_k^T f_i)}$，其中$w_k,k=1, \ldots, C$为分类器参数。因此利用最大似然法的到目标函数
$$
\max \mathcal{L} = \max \Pi_{i=1}^N P(y_i|f_i) = \max \Pi_{i=1}^N \frac{\exp(w_i^Tf_i)}{\sum_{k=1}^C \exp(w_k^T f_i)},
$$其等价于
$$
\max \log \mathcal{L} = \max \sum_{i=1}^N \log P(y_i|f_i)
$$
$z^i = [w_1^Tf_i, \ldots, w_C^T f_i]$称为logits。
### KL散度
利用KL散度衡量概率分布的距离：预测分布$P(y=j|f_i)= \frac{\exp(z^i_j)}{\sum_k \exp(z^i_k)} $以及样本真实分布$Q(y=j|f_i)= \begin{cases} 1, j=y_i \\ 0, \text{otherwise} \end{cases}$，得到
$$
\min \text{KL}(P, Q) = \sum_{j=1}^C Q(y=j|f_i) \log \frac{Q(y=j|f_i)}{P(y=j|f_i)} = -\log P(y_i|f_i)
$$
### 梯度计算
softmax分类器以及cross entropy loss得以广泛使用的原因之一为其梯度计算的方便。对于logits$z^i$和损失函数$\mathcal{L} = -\frac{1}{N}\sum_{i=1}^N \log \frac{\exp(z^i_{y_i})}{\sum_k \exp(z^i_k)}$可以得到
$$
\frac{\partial{\mathcal{L}}}{\partial{z^i_j}} = \frac{1}{N} [P(y_i|f_i)- \delta_{j, y_i}]
$$
## 计算cross entropy
### 数值稳定算法
因指数和对数函数的性质，在实际计算中并不直接根据定义式子计算。例如如果logits有分量数值过大，则$\exp$容易overflow，或者某个分量数值过小，则$-\log$也可能趋向$inf$。即采用如下等价形式
$$
\mathcal{L_i} =  - \log \frac{\exp(z^i_{y_i})}{\sum_k \exp(z^i_k)} = -\log \frac{\exp(z^i_{y_i}-m)}{\sum_k \exp(z^i_k-m)} = - (z^i_{y_i} -m ) + \log \sum_k \exp(z^i_k-m)
$$
### 并行版本
当分类类别数量$C$取极大值时，参数$w_i$以及logits将占用大量存储空间，对于深度CNN模型可能会导致GPU显存不够。此时可以将logits划分到多个GPU上实现cross entropy loss计算。
