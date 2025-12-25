# Effect-Size-Pooling-in-R

## 4.2.1 Pre-Calculated Effect Size Data

* 代码实现

```R
####
# Pre-Calculated Effect Size Data The “ThirdWave” Data Set
####

####
# Pre-Calculated Effect Size Data The “ThirdWave” Data Set
####
library(tidyverse) # needed for 'glimpse'
library(dmetar)
library(meta)
data("ThirdWave")
glimpse(ThirdWave)
m.gen <- metagen(TE = TE,
                 seTE = seTE,
                 studlab = Author,
                 data = ThirdWave,
                 sm = "SMD",
                 comb.fixed = FALSE,
                 comb.random = TRUE,
                 method.tau = "REML", # 极大似然估计
                 hakn = TRUE,
                 title = "Third Wave Psychotherapies")
m.gen

# 该函数计算的固定效应，即使fixed设置为False，但是还是计算了
m.gen$TE.fixed
#0.4805045 可以看出和Random effects model (HK) 0.5771还是有差距的
dir.create(tempdir())
# Paule-Mandel instead of the restricted maximum likelihood estimator
m.gen_update <- update(m.gen,method.tau = "PM")


# Get pooled effect
m.gen_update$TE.random # 0.5873544

# Get tau^2 estimate
m.gen_update$tau2 # [1] 0.1104957
>>> m.gen
Review:     Third Wave Psychotherapies

Number of studies: k = 18

                             SMD           95%-CI    t  p-value
Random effects model (HK) 0.5771 [0.3782; 0.7760] 6.12 < 0.0001

Quantifying heterogeneity (with 95%-CIs):
 tau^2 = 0.0820 [0.0295; 0.3533]; tau = 0.2863 [0.1717; 0.5944]
 I^2 = 62.6% [37.9%; 77.5%]; H = 1.64 [1.27; 2.11]

Test of heterogeneity:
     Q d.f. p-value
 45.50   17  0.0002

Details of meta-analysis methods:
- Inverse variance method
- Restricted maximum-likelihood estimator for tau^2
- Q-Profile method for confidence interval of tau^2 and tau
- Calculation of I^2 based on Q
- Hartung-Knapp adjustment for random effects model (df = 17)
```

• The next section provides us with the core result: the pooled effect size. We see
that the estimate is g ≈ 0.58 and that the 95% confidence interval ranges from
g ≈ 0.38 to 0.78. We are also presented with the results of a test determining if
the effect size is significant. This is the case (p < 0.001). Importantly, we also see
the associated test statistic, which is denoted with t. This is because we applied the
Knapp-Hartung adjustment, which is based on a t-distribution.

• Underneath, we see results concerning the between-study heterogeneity. We will
learn more about some of the results displayed here in later chapters, so let us only
focus on τ2 . Next to tauˆ2, we see an estimate of the variance in true effects: τ2 =
0.08. We see that the confidence interval of tauˆ
2 does not include zero (0.03–0.35),
meaning that τ2 is significantly greater than zero. All of this indicates that between-
study heterogeneity exists in our data and that the random-effects model was a
good choice.

• The last section provides us with details about the meta-analysis. We see that ef-
fects were pooled using the inverse variance method, that the restricted maximum-
likelihood estimator was used, and that the Knapp-Hartung adjustment was ap-
plied.

## 4.2.2 (Standardized) Mean Differences

```R
####
# (Standardized) Mean Differences 4.2.2
####



library(meta)
library(dmetar)

data("suicideprevention")

# Use metcont to pool results.
m.cont <- metacont(n.e = n.e,           # 实验组样本量
                   mean.e = mean.e,      # 实验组均值
                   sd.e = sd.e,          # 实验组标准差
                   n.c = n.c,            # 对照组样本量
                   mean.c = mean.c,      # 对照组均值
                   sd.c = sd.c,          # 对照组标准差
                   studlab = author,     # 研究标签
                   data = SuicidePrevention,  # 数据集
                   sm = "SMD",           # 效应量指标：标准化均值差
                   method.smd = "Hedges", # SMD的计算方法（Hedges' g，校正小样本偏倚）
                   comb.fixed = FALSE,    # 同上
                   comb.random = TRUE,    # 同上
                   method.tau = "REML",   # 同上
                   hakn = TRUE,           # 同上
                   title = "Suicide Prevention")  # 标题

m.cont  


>>> m.cont
Review:     Suicide Prevention

Number of studies: k = 9
Number of observations: o = 1147 (o.e = 571, o.c = 576)

                         SMD             95%-CI     t p-value
Random effects model -0.2304 [-0.3734; -0.0874] -3.71  0.0059

Quantifying heterogeneity (with 95%-CIs):
 tau^2 = 0.0044 [0.0000; 0.0924]; tau = 0.0661 [0.0000; 0.3040]
 I^2 = 7.4% [0.0%; 67.4%]; H = 1.04 [1.00; 1.75]

Test of heterogeneity:
    Q d.f. p-value
 8.64    8  0.3738

Details of meta-analysis methods:
- Inverse variance method
- Restricted maximum-likelihood estimator for tau^2
- Q-Profile method for confidence interval of tau^2 and tau
- Calculation of I^2 based on Q
- Hartung-Knapp adjustment for random effects model (df = 8)
- Hedges' g (bias corrected standardised mean difference; using exact formulae)
```

## 4.2.3 Binary Outcomes 二分类结果（固定效应权重）

### 4.2.3.1 Risk&Odds Ratios

#### 4.2.3.1.1 The Mantel-Haenszel Method

* 权重计算

1. **Risk Ratio:**

$$
w_k = \frac{(a_k+b_k)c_k}{n_k}
$$

2. **Odds Ratio:**

$$
w_k = \frac{b_kc_k}{n_k}
$$

​​a​​：治疗组发生事件人数

​​b​​：治疗组未发生事件人数

​​c​​：对照组发生事件人数

​​d​​：对照组未发生事件人数

#### 4.2.3.1.2 The Peto Method

* 观察到的治疗组事件数

$$
O_k= a_k
$$

* 理论期望事件数（治疗组样本量x总事件比例）

$$
E_k = \frac{(a_k+b_k)(a_k+c_k)}{a_k+b_k+a_k+c_k}
$$

* 方差

$$
V_k = \frac{(a_k+b_k)(c_k+d_k)(a_k+c_k)(d_k+b_k)}{(a_k+b_k+a_k+c_k)^2(a_k+b_k+a_k+c_k-1)}
$$

* 合并

$$
log\hat{\psi_k} = \frac{O_k-E_k}{V_k}
$$

#### 4.2.3.1.3 The Bakbergenuly-Sample Size Method

$$
w_k = \frac{n_{treat_k}n_{control_k}}{n_{treat_k}+n_{control_k}}
$$

#### 4.2.3.1.4  Pooling Binary Effect Sizes in R

```R

####
# 4.2.3.1.4 Pooling Binary Effect Sizes in R
####



library(dmetar)
library(tidyverse)
library(meta)
data(DepressionMortality)
glimpse(DepressionMortality)

# 效应量指标为RR不再是SMD
m.bin <- metabin(
  event.e = event.e,     # 实验组事件数（如死亡人数）
  n.e = n.e,             # 实验组总样本量
  event.c = event.c,     # 对照组事件数
  n.c = n.c,             # 对照组总样本量
  studlab = author,      # 研究标签（通常为作者或研究ID）
  data = DepressionMortality,  # 数据集
  sm = "RR",             # 效应量指标：风险比（Risk Ratio）
  method = "MH",         # 使用Mantel-Haenszel法 （pooling method使用的方法）
  MH.exact = TRUE,       # 启用精确MH计算（避免零单元格问题）
  comb.fixed = FALSE,    # 禁用固定效应模型
  comb.random = TRUE,    # 启用随机效应模型
  method.tau = "PM",     # 异质性方差估计方法：Paule-Mandel
  hakn = TRUE,           # 启用Hartung-Knapp校正
  title = "Depression and Mortality"  # 分析标题
)

m.bin
# 更改异质性方差估计方法
m.bin_update <- update(m.bin,method.tau = "REML")
exp(m.bin_update$TE.random)
m.bin_update$tau2

# 更改效应量指标为OR
m.bin_or <- update(m.bin,sm='OR')
m.bin_or

```

#### 4.2.3.1.5 Pooling Pre-Calculated Binary Effect Sizes

```R
# 当数据集没有详细的数据，而是只有与处理过的数据怎么办呢？

DepressionMortality$TE <- m.bin$TE
DepressionMortality$seTE <- m.bin$seTE

# Set seTE if study 7 to NA
DepressionMortality$seTE[7] <- NA
# Create empty columns 'lower' and 'upper'
DepressionMortality[,"lower"] <- NA
DepressionMortality[,"upper"] <- NA

# Fill in values for 'lower' and 'upper' in study 7
# As always, binary effect sizes need to be log-transformed
DepressionMortality$lower[7] <- log(1.26)
DepressionMortality$upper[7] <- log(2.46)

# 看看我们刚刚设置的数据
DepressionMortality[,c("author", "TE", "seTE", "lower", "upper")]

>>> output
                  author         TE       seTE     lower     upper
1    Aaroma et al., 1994  0.7418532 0.20217061        NA        NA
2     Black et al., 1998  0.5603039 0.14659887        NA        NA
3     Bruce et al., 1989  0.9235782 0.43266994        NA        NA
4     Bruce et al., 1994  0.1488720 0.15526121        NA        NA
5    Enzell et al., 1984  0.6035076 0.17986279        NA        NA
6   Fredman et al., 1989 -0.9236321 0.99403676        NA        NA
7    Murphy et al., 1987  0.5675840         NA 0.2311117 0.9001613
8   Penninx et al., 1999  0.3816630 0.22842369        NA        NA
9    Pulska et al., 1998  0.6645639 0.18819368        NA        NA
10  Roberts et al., 1990  0.8333374 0.09218909        NA        NA
11      Saz et al., 1999  0.7810180 0.17380951        NA        NA
12   Sharma et al., 1998  0.7178398 0.32962024        NA        NA
13  Takeida et al., 1997  1.9428138 0.26758609        NA        NA
14  Takeida et al., 1999  1.7599912 0.20599112        NA        NA
15   Thomas et al., 1992  0.2853747 0.27366749        NA        NA
16   Thomas et al., 1992  0.5721946 0.23995571        NA        NA
17 Weissman et al., 1986  0.2231436 0.31986687        NA        NA
18    Zheng et al., 1997  0.6832705 0.17690650        NA        NA

# 事实上在现实生活中很难看到这样的数据，但是metagen函数也能计算这样的效应值
m.gen_bin <- metagen(TE = TE,
                     seTE = seTE,
                     lower = lower,
                     upper = upper,
                     studlab = author,
                     data = DepressionMortality,
                     sm = "RR",
                     method.tau = "PM",
                     comb.fixed = FALSE,
                     comb.random = TRUE,
                     title = "Depression Mortality (Pre-calculated)")
m.gen_bin
```

### 4.2.3.2 Incidence Rate Ratios

```R
####
# 4.2.3.2 Incidence Rate Ratios
####

library(dmetar)
library(tidyverse)
library(meta)
data(EatingDisorderPrevention)
glimpse(EatingDisorderPrevention)


m.inc <- metainc(event.e = event.e,
                 time.e = time.e,
                 event.c = event.c,
                 time.c = time.c,
                 studlab = Author,
                 data = EatingDisorderPrevention,
                 sm = "IRR",
                 method = "MH",
                 comb.fixed = FALSE,
                 comb.random = TRUE,
                 method.tau = "PM",
                 hakn = TRUE,
                 title = "Eating Disorder Prevention")
m.inc
```

## 4.2.4 Correlations

* 怎么合并相关系数呢？

```R
####
# 4.2.4 Correlations
####

data(HealthWellbeing)
glimpse(HealthWellbeing)
# 如何合并相关系数呢？
m.cor <- metacor(cor = cor,
                 n = n,
                 studlab = author,
                 data = HealthWellbeing,
                 comb.fixed = FALSE,
                 comb.random = TRUE,
                 method.tau = "REML",
                 hakn = TRUE,
                 title = "Health and Wellbeing")
m.cor
# 绘制森林图
forest(m.cor, col.square = "blue", col.diamond = "red")

# 漏斗图检查发表偏倚
funnel(m.cor, studlab = TRUE)

>>>OUTPUT

Review:     Health and Wellbeing

Number of studies: k = 29
Number of observations: o = 853794

                        COR           95%-CI     t  p-value
Random effects model 0.3632 [0.3092; 0.4148] 12.81 < 0.0001

Quantifying heterogeneity (with 95%-CIs):
 tau^2 = 0.0241 [0.0141; 0.0436]; tau = 0.1554 [0.1186; 0.2088]
 I^2 = 99.8%; H = 24.14

Test of heterogeneity:
        Q d.f. p-value
 16320.87   28       0

Details of meta-analysis methods:
- Inverse variance method
- Restricted maximum-likelihood estimator for tau^2
- Q-Profile method for confidence interval of tau^2 and tau
- Calculation of I^2 based on Q
- Hartung-Knapp adjustment for random effects model (df = 28)
- Fisher's z transformation of correlations
```

​​即使只有 r 和 n，也能计算方差和异质性​​，因为 Var(Z) = 1/(n-3)。
​​Fisher's Z 变换​​ 使 Meta 分析更准确。
​​hakn = TRUE​​ 会使用 Hartung-Knapp 校正，使小样本研究的置信区间更保守。

## 4.2.5 Means

```R
####
# 4.2.5 Means
####


data(BdiScores)
# We only need the first four columns
glimpse(BdiScores[,1:4])

# # Our goal is to calculate the overall mean depression score based on this collection of
# studies. We will use a random-effects model and the restricted maximum-likelihood
# estimator to pool the raw means in our data set. We save the results in an object
# called m.mean.


m.mean <- metamean(n = n,
                   mean = mean,
                   sd = sd,
                   studlab = author,
                   data = BdiScores,
                   sm = "MRAW",
                   comb.fixed = FALSE,
                   comb.random = TRUE,
                   method.tau = "REML",
                   hakn = TRUE,
                   title = "BDI-II Scores")

m.mean

>>>output

> m.mean
Review:     BDI-II Scores

Number of studies: k = 6
Number of observations: o = 920

                        mean             95%-CI
Random effects model 31.1221 [29.6656; 32.5786]

Quantifying heterogeneity (with 95%-CIs):
 tau^2 = 1.0937 [0.0603; 12.9913]; tau = 1.0458 [0.2456; 3.6043]
 I^2 = 64.3% [13.8%; 85.2%]; H = 1.67 [1.08; 2.60]

Test of heterogeneity:
     Q d.f. p-value
 14.00    5  0.0156

Details of meta-analysis methods:
- Inverse variance method
- Restricted maximum-likelihood estimator for tau^2
- Q-Profile method for confidence interval of tau^2 and tau
- Calculation of I^2 based on Q
- Hartung-Knapp adjustment for random effects model (df = 5)
- Untransformed (raw) means
```

## 4.2.6 Proportions