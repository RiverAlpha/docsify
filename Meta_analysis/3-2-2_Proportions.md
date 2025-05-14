> A proportion is another type of one-variable measure. It specifies how many units of a
sample fall into a certain subgroup. Proportions can take values between zero and
one, which can be transformed into percentages by multiplying with 100. Proportions
may, for example, be used as an outcome measure when we want to examine the
prevalence of a disease at a given point in time. To calculate a proportion p, we have
to divide the number of individuals k falling into a specific subgroup by the total
sample size n.

* 比例公式(0-1)：

$$
    p = \frac{k}{n}
$$

* 概率标准误差

$$
        SE_p = \sqrt{\frac{p(1-p)}{n}}
$$


* logit

对比例p的算式 进行logit运算

logP
$$
p_{logit} = log_e(\frac{p}{1-p})
$$
映射效果：

$p->1 时，logit -> -\infin ;
 p->1时，logit -> +\infin;
  p = 0.5 时, logit = 0$

转换后的logit值可自由分布在实数轴上，避免CI越界

log标准差
$$
SE_{p_{logit}} \approx \sqrt{\frac{1}{np}+\frac{1}{n(1-p)}}
$$
映射效果：

1. 即使p接近0或者1方差仍由样本量n主导，而非p的大小；
2. 确保meta分析中权重分配更合理，减少极端值的过度影响

* 题外话:为什么要用log

1. 边界限制导致置信区间溢出
问题：当原始比例接近0或1时（如罕见事件p=0.02或高发事件p=0.98），直接计算的置信区间（CI）可能超出10,1范围，失去实际意义。
示例：
若某研究感染率p=0.02，标准差SE=0.01，则正态近似法计算的95%CI为：0.02±1.96×0.01=[0.0004,0.0396]
虽然未越界，但当p更接近0时（如p=0.005），CI下限可能变为负数。

2. 方差不稳定（Variance Instability）
问题：比例的方差
$Var(p)= \frac{p(1-p)}{n}​$在p=0.5时最大，而在p接近0或1时趋近于0。这种非恒定方差会导致：
   1. 小样本或极端比例的研究在Meta分析中被错误地赋予高权重；
   2. 异质性检验（如Q统计量）失真。

1. 非对称分布（Asymmetric Distribution）
问题：比例的抽样分布在p接近0或1时呈现高度偏态（如右偏或左偏），违反正态性假设，使得传统的加权平均方法失效。
示例：
p=0.05的分布集中在左侧，而p=0.95集中在右侧，两者合并时无法对称处理。
