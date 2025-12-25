# 5.1 Measures of Heterogeneity

## 5.1.1 Cochran's Q

传统上，meta分析使用Cochran's Q来区别抽样误差和研究间误差

$$
Q = \sum_{k=1}^Kw_k(\hat{\theta_k}-\hat{\theta})^2
$$
$\hat{\theta_k} :k$研究下 观测效应值

$\hat{\theta} :$ 固定效应模型下的合并效应量

$w_k :$k研究下方差的倒数（作为权重）

因为式子中有某个研究的方差，所以Q值不仅取决于某个研究的计算出的效应值，而且考虑到了此研究的精确度（精确度越高方差的倒数越大）

* 有研究间和仅有抽样误差下Q的表现

```R
####
# 5.1.1 Cochran’s Q
####
set.seed(123)
rnorm(n=40,mean=0,sd=1)

set.seed(123)
# 模拟10000次，每次生成40个标准正态分布值，代表​​仅含抽样误差​​的研究间差异。
# rnorm(40)​​：生成一个包含40个来自标准正态分布（均值=0，标准差=1）的随机数的向量。
# ​​replicate(n=10000, rnorm(40))​​：将 rnorm(40) 的操作重复10,000次。每次生成一个40个随机数的向量，最终将这些结果组合成一个矩阵或数组。
error_fixed <- replicate(n=10000,rnorm(40))


set.seed(123)
# 在抽样误差基础上，额外添加一个独立的正态分布项（rnorm(40)），代表​​异质性误差（τ²）​​。
# rnorm(40) + rnorm(40)​​：
# 生成​​两个独立的标准正态随机向量​​（各40个值），然后逐元素相加。
# 结果相当于从均值为0、方差为2（因为方差可加：Var(X+Y)=Var(X)+Var(Y)=1+1=2）的正态分布中抽样。
error_random <- replicate(n = 10000, rnorm(40) + rnorm(40))


set.seed(123)
# 在零假设（τ²=0）下，Q服从自由度为39（k-1=40-1）的​​卡方分布​​。
# rnorm(40)^2 对生成的40个符合正态分布的值进行平方处理即 符合卡方分布
Q_fixed <- replicate(10000, sum(rnorm(40)^2))
# 当存在异质性时，Q的分布向右偏移（均值增大），反映额外变异。
Q_random <- replicate(10000, sum((rnorm(40) + rnorm(40))^2))




# Histogram of the residuals (theta_k - theta)
# - We produce a histogram for both the simulated values in
#error_fixed and error_random
# - `lines` is used to add a normal distribution in blue.
hist(error_fixed,
     xlab = expression(hat(theta[k])~-~hat(theta)), prob = TRUE,
     breaks = 100, ylim = c(0, .45), xlim = c(-4,4),
     main = "No Heterogeneity")
lines(seq(-4, 4, 0.01), dnorm(seq(-4, 4, 0.01)),
      col = "blue", lwd = 2)
hist(error_random,
     xlab = expression(hat(theta[k])~-~hat(theta)), prob = TRUE,
     breaks = 100,ylim = c(0, .45), xlim = c(-4,4),
     main = "Heterogeneity")
lines(seq(-4, 4, 0.01), dnorm(seq(-4, 4, 0.01)),
      col = "blue", lwd = 2)
# Histogram of simulated Q-values
# - We produce a histogram for both the simulated values in
#Q_fixed and Q_random
# - `lines` is used to add a chi-squared distribution in blue.
# First, we calculate the degrees of freedom (k-1)
# remember: k=40 studies were used for each simulation
df <- 40-1 # 自由度39
hist(Q_fixed, xlab = expression(italic("Q")), prob = TRUE,
     breaks = 100, ylim = c(0, .06),xlim = c(0,160),
     main = "No Heterogeneity")
lines(seq(0, 100, 0.01), dchisq(seq(0, 100, 0.01), df = df),
      col = "blue", lwd = 2)
hist(Q_random,
     xlab = expression(italic("Q")), prob = TRUE,
     breaks = 100, ylim = c(0, .06), xlim = c(0,160),
     main = "Heterogeneity")
lines(seq(0, 100, 0.01), dchisq(seq(0, 100, 0.01), df = df),
      col = "blue", lwd = 2)
```

* Q统计量和Q检验的问题

尽管Q统计量在meta分析中经常提及和使用，但是也有几个缺点。一个焦点问题是Q会随着研究数K和某个研究k的样本数的增加而增加。

## 5.1.2 Higgins & Thompson’s $I^2$ Statistic

$I^2$统计量是直接建立在Q统计量上的另一个量化研究间异质性的方法。
其统计量假设Q服从自由度为K-1的卡方分布$\chi^2，并且在研究间没有异质性的假设下$


$$
I^2 = \frac{Q-(k-1)}{Q}
$$