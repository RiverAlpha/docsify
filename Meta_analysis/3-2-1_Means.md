# 3.1 What is an effect Size

* 总体标准差

$$
\begin{equation}
  s = \sqrt{\frac{1}{N}\sum^{n}_{i=1}(x_i-\overline{x})^2}
\end{equation}
$$

> 计算步骤

1. 计算平均值

$$
    \mu = \frac{1}{N} \sum_{i=1}^Nx_i
$$

2. 求平方差的平均值（方差）

$$
\sigma^2 = \frac{1}{N} \sum_{i=1}^{N}(x_i-\mu)^2
$$

* 样本标准差（在仅有的样本中）

> 适用于从总体中抽取出的样本，计算时需用n-1修正（无偏估计）

$$
\begin{equation}
  s = \sqrt{\frac{1}{n-1}\sum^{n}_{i=1}(x_i-\overline{x})^2}
\end{equation}
$$

1. 计算平均值

$$
    \mu = \frac{1}{n} \sum_{i=1}^Nx_i
$$

2. 求平方差的平均值（方差）

$$
\sigma^2 = \frac{1}{n-1} \sum_{i=1}^{N}(x_i-\mu)^2
$$

> 标准差和标准误差不同

* 标准误差(均值的标准误差 SEM)

$$
 SE = \frac{S}{\sqrt{n}}
$$

```R
set.seed(123)
sample <- rnorm(n=50, mean = 10, sd = 2)
mean(sample)

SE = sd(sample)/sqrt(50)
```

> 对于整个流程或者各个指标的定义，我们可能希望看到的就是误差能够越小越好。
> 如果我们都认可这些公式定义的指标，能减小误差的一个方法就是增大样本量了，也就是增加n,这样SE标准误差就会减小。

> 题外话，之前都不能理解这个均值有什么实质性的作用，如果假定样本数据就是按照正态分布的，则均值就很有用了，或者说不是因为均值才有了正态分布，而是这个世界上很多事物就是正态分布的才显得均值多么重要，很多时候均值不准不是因为均值的定义没有用，而是样本量不够。