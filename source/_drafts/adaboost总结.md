---
title: adaboost总结
tags: ML algo
category: 机器学习
---

在公司里面用adaboost做了一个量化选股的模型，虽然算法部分是调用sklearn包里面的函数完成的，算法的核心思想还是需要弄个清楚。在这里梳理一下。
# 集成学习
AdaBoost算法是属于集成学习的分类算法。这里有必要介绍一下集成学习。
集成学习（Ensemble Learning）的思想类似于“三个臭皮匠赛过诸葛亮”。单个决策者做的决策准确率如果大于50%，那么多个决策者一起参与决策肯定会提高最终决策的准确度。
用二项分布和Hoeffding不等式可以简单地证明，随着集成中个体分类器的数目T增大，集成的错误了呈指数下降，最终趋向于0.

## 如何选择/生成单个决策者(弱分类器)？
首先，单个弱分类器的分类准确率一定要大于0.5，不然就像民粹主义的结果一样，一群愚蠢的民众会做出愚蠢的决定。  
第二， 每个弱分类器一定要有“多样性”，要达到“好而不同”。不然相似的分类器做出相似的决定，对最终分类的表现不会有很大提升。  
生成“好而不同”的弱分类器的思想有两个：Boosting 和 Bagging(*abbr.* Bootstrap aggregating)
- Boosting 的角度是，从一个初始的弱分类器开始，对数据集进行分类以后，增加对那些分错的样本的权重。代表算法是**AdaBoost**。
- Bagging 的角度是，对数据集随机分成很多份，每一份数据集生成一个弱分类器。代表算法是**Random Forest**。
# Notations

 表达式                    |意义
 -------------------------|--
  $D = \{(x,y)|y=f(x)\}$  |  数据集，f(x)表示真实函数
 $h(x)$                   |  基学习器/ 弱分类器
 $w_t$                    |  样本权重
 $L_{exp}(H_t, w_t|D)$    |  指数损失
 $\epsilon$               |  分类错误率
# AdaBoost
##　工作机制
1. 从数据集中训练出一个基分类器，根据基学习器的表现对训练样本的分布进行调整（对样本赋权），使得基学习器分错的样本在下一轮训练中得到更多的关注
2. 基于调整后的训练样本重新训练一个基学习器，如此反复，直到基学习器的个数达到预设的T个。
3. 将这T个基学习器加权结合得到最终的集成学习器

##　推导
**在对任何模型进行推导的时候都要时刻记住：你的模型是什么，你的目标函数是什么，要用到哪些trick**。
在进行机器学习算法推导的时候也要时刻记住这个准则，这样推导的难度会大大降低。
这里的推导参考了周志华的《机器学习》这本书。也是基于“加性模型”，对指数损失函数最小化。  
加性模型中集成学习器表示为基学习器的线性组合：
$$ H_T(x) = \sum_{t=1}^T \alpha_t h_t(x)$$
指数损失函数:
$$ L_{exp}(H_t, w_t|D) = \frac{1}{N} \sum_{i=1}^N e^{-f(x_i)H_t(x_i)} = E_{x\in D}[e^{-f(x)H_t(x)}]$$

我们的目的是最小化指数损失函数。从指数损失函数的定义我们可以看到，在训练第t个弱学习器的时候，影响$L_{exp}$ 的两个参数就是 $\alpha_t$ 和 $h_t(x)$ 。
我们先对固定$h_t(x)$ 分析$L_{exp}$ 与  $\alpha_t$ 之间的关系。
### 找到使$L_{exp}$ 最小化的 $\alpha_t$
$$
\begin{align}
 L_{exp}(H_t,w_t| D) & = E_{x\in D}[e^{-f(x)H_t(x)}]\\
                     & = E_{x \in D}[e^{-f(x)\alpha_t h_t(x)-f(x)H_{t-1}(x)}]\\
                     & = E_{x \in D}[e^{-f(x)\alpha_t h_t(x)}\cdot e^{-f(x)H_{t-1}(x)}]\\
    \end{align}
    \tag{1}
$$
上式中，只有$e^{-f(x)\alpha_t h_t(x)}$ 部分与 $\alpha_t$ 有关。由于期望的线性性，我们只需要关注$E_{x \in D}[e^{-f(x)\alpha_t h_t(x)}]$ 这一部分。
$$
\begin{align}
E_{x \in D}[e^{-f(x)\alpha_t h_t(x)}] & = E_{x \in D}[e^{-\alpha_t} I(f(x)= h_t(x))+e^{\alpha_t} I(f(x) \neq h_t(x))]\\
                                      & = e^{-\alpha_t}\mathbb{P}(f(x)= h_t(x)) + e^{\alpha_t}\mathbb{P}(f(x)\neq h_t(x))\\
                                      & = e^{-\alpha_t}(1-\epsilon_t)+e^{\alpha_t}\epsilon_t
\end{align}
$$
where $I(\cdot)$ is a indicator funtion.
$$
        \mathbb{I}(f(x)=h(x)) =
        \begin{cases}
        1, & \text{if $f(x)=h(x)$}\\
        0, &\text{if $f(x) \neq h(x)$}
        \end{cases}
$$
$E_{x \in D}[e^{-f(x)\alpha_t h_t(x)}]$ 对 $\alpha_t$ 求导，并令导数为$0$可以求得极小值点：
$$
\alpha_t = \frac{1}{2}\ln(\frac{1-\epsilon_t}{\epsilon_t})
$$


### 找到使$L_{exp}(H_{t+1},w_{t+1}| D)$ 最小的 $w_{t+1}$
Adaboost算法在训练出 $t-1$ 个弱学习器之后要根据$H_{t-1}(x)$ 的表现调整样本的分布(给样本重新分配权重)。我们要找到这个权重使得在$t$阶段训练出来的弱学习器$h_t(x)$ 能使分类误差最小。按理来说应该最小化损失函数，但是直接最小化损失函数推导难度较大，改成分类误差。
周志华是从样本分布变化的角度来更新训练样本集而PRML是从样本权重变化的角度来更新样本集。
两本书都不是用分析的方法得到的结果，而是直接令目标函数式里的一部分为权重更新值，最后证明这个值可以使分类误差最小化。我觉得PRML里面的推导方式更好理解一点在这里引用他的方式。

PRML书上的推导直接把 式(1) $\ref{1}$ 中的 $e^{-f(x)H_{t-1}(x)}$ 部分当做 $w_t$，即
$$
w_t = e^{-f(X)H_{t-1}(X)}
$$
从直观的角度来看，这样的赋值是合理的。如果 $f(x)=H_{t-1}(x)$ 则样本x的权重 $w_t^{(x)}$ 会变小，如果 $f(x)\neq H_{t-1}(x)$ 则样本x的权重 $w_t^{(x)}$ 会变大。

式(1)等价表示：
$$
\begin{align}
 L_{exp}(H_t,w_t| D) & = E_{x \in D}[e^{-f(x)\alpha_t h_t(x)}\cdot w_t^{(x)}]\\
                     & = {1\over N}\sum_{i=1}^{N}e^{-f(x_i)\alpha_t h_t(x_i)}\cdot w_t^{(i)}\\
                     & = {1\over N}e^{-\alpha_t}\sum_{i\in T_t}w_t^{(i)} +　{1\over N}e^{\alpha_t}\sum_{i\in M_t}w_t^{(i)}\\
                     & = {1\over N}(e^{\alpha_t}-e^{-\alpha_t})\sum_{i=1}^{N}w_t^{(i)} \mathbb{I}(f(x_i)\neq h_t(x_i))
                          + {1\over N}e^{-\alpha_t}\sum_{i=1}^{N}w_t^{(i)}
    \end{align}
    \tag{2}
$$
等式右边最后一项跟$h_t(x)$ 没有关系，所以只需要最小化 $\sum_{i=1}^{N}w_t^{(i)} \mathbb{I}(f(x_i)\neq h_t(x_i))$ 就可以了。
$$
\begin{align}
w_{t+1} & = e^{-f(X)H_{t}(X)}\\
        & = e^{-f(X)\alpha_t h_{t}(X)}\cdot e^{-f(X)H_{t-1}(X)}\\
        & = e^{-\alpha_t (1-2\mathbb{I}(f(X)\neq h_{t}(X)))} \cdot w_t\\
        & = w_t e^{-\alpha_t}e^{2\alpha_t \mathbb{I}(f(X)\neq h_{t}(X))}
        \end{align}
$$
其中，$e^{-\alpha_t}$ 作为一个常数项可以忽略。$w_t$ 作为权重在计算的时候可以做归一化也可以不做。
# 算法流程

1.  初始化权重 $w_1 = {1 \over N}$
2.  $for \ t = 1:T$
    > - $h_t = argmin(\sum_{i=1}^{N}w_t^{(i)} \mathbb{I}(f(x_i)\neq h_t(x_i)))$  
  compute error rate: $\epsilon_t = {\sum_{i=1}^{N}w_t^{(i)} \mathbb{I}(f(x_i)\neq h_t(x_i)) \over \sum_{i=1}^{N}w_t^{(i)} }$  
     >- if $\epsilon_t > 0.5$, restart the training
     >- else:  
       update $\alpha_t$: $\alpha_t = {1\over 2}\ln {1- \epsilon_t \over \epsilon_t}$
update $w_t$: $w_{t+1} = w_t\cdot e^{2\alpha_t \mathbb{I}(f(X)\neq h_t(X))}$

3. $H(x) = sign(\sum_{t=1}^T \alpha_th_t(x))$

# 总结
Boosting方法的intuition还是很好理解的。关键是要把intuition和数学表示之间的关系建立起来。把直觉数学化。$w_t = e^{-f(X)H_{t-1}(X)}$ 是个很关键的点，它表示了“给那些分错的样本跟多的关注”。周志华和Bishop对样本更新的两种不同的理解角度值得玩味学习。虽然本质上都是对样本权重进行重赋值，周志华理解的是对样本分布的改变，Bishop理解的就是单纯的样本权重。如果是作为一个分布，那么必须满足归一性，而作为权重的话只需要关注相对大小，不必要求归一性了。

抛开AdaBoost推导的细节，我们还应该关注算法流程，因为算法流程为我们的算法推导提供了指导。首先我们有initial weight $w_1$, 在得到第$t$个弱分类器的时候，我们需要找到分类器权重 $\alpha_t$ 来使指数损失函数最小。为了使$t+1$个弱分类器训练出来的时候也能让指数损失最小，我们要对样本reweight。如何找到reweight迭代公式，在推导细节中就体现出来。$w_{t+1}$
的计算既可以用定义式直接计算，也可以用迭代式。但是显然用迭代式的计算开销要小很多。

周志华的西瓜书对指数损失函数和分类错误率之间的一致性(consistent)还做出了解释。令指数损失函数对$H(x)$的偏导值为0，求得极小值点。验证这个极小值点也是使分类错误率最小的点。
