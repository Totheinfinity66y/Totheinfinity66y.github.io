---
title: For Markdown trial
date: 2026-02-12 23:27:54
tags:
---
方案设计的原则以及排除的方案
除了题目要求的Fair，下面几个规则的性质也是一个好的指标必须满足的：
1. 易懂性与易记忆性。DWTS是娱乐节目，因此我们不希望引入过于复杂的公式，即便其指标更好，观众难以理解和记忆规则会不利于节目热度。
2. 鼓励粉丝投票。规则应让粉丝感受到，他们的投票可以让他们喜欢的明星更不容易被淘汰（或排名更靠前），这有利于提高观众参与感与对节目的粘性。
3. 平等。指的是粉丝和评委之间的平衡。不平等的规则，比如调整粉丝投票和评委打分的权重，会让一方感到不公平，即便结果的效果可能更好，因此保持1：1有利于节目规则让人们感到技术与人气的平等。
4. 相对固定。特定赛季每一周的情况都不一样，根据比赛进度动态设计规则看似是一个好办法，但是破坏了规则的固定性，这不利于观众记忆和信服结果。
5. 尊重变化与成长。明星的粉丝投票和评委打分都是变化的，而变化与落差正是节目的核心看点之一，因此不应该引入历史成绩对当前的影响。
为什么选择“赋分制”
历史方法的痛点
之前讨论的规则中，RANK(+SAVE)是对原始数据的极端压缩，极度降低了方差，PERCENT(+SAVE)则是另一个极端，完全压缩数据，放任方差主导结果。二者都不能完全符合这个数据集的特征。因此一个自然的想法就是找一个介于二者之间的分数压缩规则，既能一定程度压缩方差，又不过度损失原来的差距信息。于是我们决定采用赋分制度。
赋分制度的具体内容
- 为了易懂性，排名映射成的分数应为整数，因为100分满分是人类的一个常见直觉，也符合本体的灵敏度，所以我们将总分设为100。
- 为了平等，我们将评委打分和粉丝投票分别映射到1~50分，并且映射规则一致
- 注意到34个赛季中参赛明星最多的有16人，因此将排名的1~16映射为1~50之间的整数。
- 随着赛季的进行，剩余明星数也会下降，如果剩余k人，这时我们取映射规则的前k个。
其合理性在于
  - 后几周被淘汰的明星分数会普遍高于前几周被淘汰的，这能够赋予分数一定程度反映人气和能力绝对值的功能。
  - 保证第一名上限仍为100，最后一名成绩也不会太差，满足了粉丝的荣誉心理从而更加积极参与投票。
- 由于先前验证发现评委拯救的容错效果更好，因此同样引入相同规则的评委拯救制
基于自适应遗传算法AGA的具体映射规则优化
AGA的基本原理
To determine the optimal non-linear mapping from Rank to Score, we employed the Adaptive Genetic Algorithm (AGA). Unlike traditional GAs with fixed parameters, AGA introduces a dynamic adjustment mechanism that alters crossover (Pc) and probabilities based on population fitness. By maintaining high variance for global exploration in early stages and reducing it for local exploitation in later stages, AGA effectively prevents premature convergence, ensuring the identification of the most robust scoring curve that balances both fairness and drama.
关键参数定义
染色体
固定第1名对应为g_1=50，16名为g_16=1，定义染色体$Phen=[g_2,g_3,...,g_15]$（g_i为第i名映射到的数字）
满足：
1<g_i<50
g_i属于N
g_{i-1}>g_i
适应度
按照7.3中定义的指标和权重，定义适应度函数为
$$
Fitness = 0.1820 \cdot PDI + 0.1798 \cdot VPD + 0.1560 \cdot FDI + 0.1561 \cdot VFD + 0.1467 \cdot USSI + 0.1793 \cdot POCI 
$$
其它参数
1. 初始种群：为了扩大全局搜索能力，在解空间内按照均匀分布随机生成初始化种群
2. 种群大小：通常采用染色体长度的10~20倍，本算法采用500作为种群大小
3. 迭代次数：
求解过程
流程如图（流程图）

结果如图
[图片]
求解结果分析
[图片]
"The optimization yielded a unique 'Tri-Phasic Incentive Structure'
1. 整体呈现缓-陡-缓的趋势
2. 第1名和第2名赋分差距很小，目标是让第一名是谁产生巨大悬念
3. 1~7名变化较为缓慢，目的是让不容易被淘汰的明星们名次更容易变化，从而产生戏剧性
4. 8~11名急速下跌，目的是在大部分的比赛周，放大淘汰区边缘的选手分数差距，保证淘汰规则的公平性，避免“冤案”
5. 12~16名分数较低，目的是尽快淘汰人气和实力的低水平者，保证节目效果。
和原来方案的对比
[图片]
按照7.3求得的6个指标及其对应CRITIC权重，将该赋分机制和7.3的四个方案同时带入TOPSIS，可以发现其得分为0.5384，远高于原来的四个模型的任何一个，这说明该赋分机制各个方面的总和表现十分优秀，值得制作组采纳。
灵敏度分析(Fair vs Better)
考虑到制作组追求的目标不只有是Fair，也可以是Drama等其他目标。刚才的CRITIC赋权是对Fair和Drama的权衡，而"Better"的规则通常能够在这两种指标权重不同时表现更优秀。
[图片]
因此如图，我们在CRITIC得到基准权重的基础上对部分指标进行了加权
在基准权重基础上，向左为对公平性指标PDI和VPD的权重分别乘以2,3,4,5，向右为戏剧性指标USSI和POCI的权重分别乘以2,3,4,5。
（参考：）
"As shown in the Sensitivity Analysis, our proposed model maintains the dominant position (Rank 1) across all weight configurations.
This indicates that the model is not just a specialized solution for 'Fairness', but a Pareto Improvement over the existing Rank and Percent systems. It strictly dominates historical methods regardless of whether the producer prioritizes 'Fair Competition' or 'TV Drama'."
带入TOPSIS得到规则偏向公平性或戏剧性时的模型，我们惊奇地发现在任何权重组合下，我们的赋分方案都是表现最优秀的。它兼顾了percent制度的戏剧性和rank+save制度的公平性，且在不同规则下表现出极佳的鲁棒性，是一个非常值得制作组采纳的方案。