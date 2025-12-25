# (Standardized) Mean Differences

## 1. Between-Group Mean Differences 

组间均值差（MD）是两组独立样本均值之间的原始差值，用于量化实验组与对照组的绝对效应大小。​​定义​​：两组均值之间的​​原始差异​​（未标准化），即实验组均值减去对照组均值（或其他比较组）。

$$
MD_{between} = \overline{x_1} - \overline{x_2}
$$
$\overline{x_1}代表第一组的均值、\overline{x_2}代表第二组的均值$

* 对应的标准差公式

$$
SE_{MD_{between}} = S_{pooled}\sqrt{\frac{1}{n_1}+\frac{1}{n_2}}
$$
$n_1和n_2分别$代表第一二组的样本数

* $s_{pooled}$

$$
S_{pooled} = \sqrt{\frac{(n_1-1)s_1^2+(n_2-1)s_2^2}{(n_1-1)+(n_2-1)}}
$$
$s_1 s_2$分别代表各自的标准差

* 代码实现

```R
# Generate two random variables with different population means
set.seed(123)
x1 <- rnorm(n = 20, mean = 10, sd = 3)
x2 <- rnorm(n = 20, mean = 15, sd = 3)

# Calculate values we need for the formulas
s1 <- sd(x1)
s2 <- sd(x2)
n1 <- 20
n2 <- 20

# Calculate the mean difference
MD <- mean(x1) - mean(x2)
MD

# Calculate s_pooled
s_pooled <- sqrt(
  (((n1-1)*s1^2) + ((n2-1)*s2^2))/
    ((n1-1)+(n2-1))
)

# Calculate the standard error
se <- s_pooled*sqrt((1/n1)+(1/n2))
se
```

## 2. Between-Group Standardized Mean Difference

两组均值差异经过​​标准化​​（除以 pooled standard deviation），消除单位影响，使结果可跨研究比较

$$
SMD_{between} = \frac{\overline{x_1}-\overline{x_2}}{S_{pooled}}
$$

* 合并过后的标准差
  
$$
S_{pooled} = \sqrt{\frac{(n_1-1)s_1^2+(n_2-1)s_2^2}{(n_1-1)+(n_2-1)}}
$$

* 对应的标准误差

$$
SE_{SMD_{between}} = \sqrt{\frac{n_1+n_2}{n_1n_2} + \frac{SMD^2_{betwenn}}{2(n_1+n_2)}}
$$

```R
# Load esc package
library(esc)
# Define the data we need to calculate SMD/d
grp1m <- 50  # mean of group 1
grp2m <- 60  # mean of group 2
grp1sd <- 10 # sd of group 1
grp2sd <- 10 # sd of group 2
grp1n <- 100 # n of group1
grp2n <- 100 # n of group2
# Calculate effect size
esc_mean_sd(grp1m = grp1m, grp2m = grp2m,grp1sd = grp1sd, grp2sd = grp2sd,grp1n = grp1n, grp2n = grp2n)
```

## 3. Within-Group (Standardized) Mean Difference(组内标准化均只差)

组内标准化均值差（通常称为 $ Cohen's d_{within}$）用于衡量同一组个体在干预前后（或不同时间点）的标准化变化幅度，是纵向研究或前后对照试验中的核心效应量指标。

* 非标准化unstandardized版本
  
$$
MD_{within} = \overline{x_{t_2}} - \overline{x_{t_1}}
$$

* 对应的标准误差

$$
  SE_{MD_{within}} = \sqrt{\frac{s^2_{t1}+s^2_{t2}-(2r_{t_1t_2}s_{t_1}s_{t_2})}{n}}
$$


* 标准化standardized版本
   
$$
SMD_{within} = \frac{\overline{x_{t_2}} - \overline{x_{t_1}}}{\frac{\sqrt{(s^2_{t_1}+s^2_{t_2})/2}}{\sqrt{2(1-r_{t_1t_2})}}}
$$

* 对应的标准误差

$$
SE_{SMD_{within}} = \sqrt{\frac{2(1-r_{t_1t_2})}{n}+\frac{SMD^2_{within}}{2n}}
$$

```R
####
# Within-Group SMD
####


# Define data needed for effect size calculation
x1 <- 20
# mean at t1
x2 <- 30
# mean at t2
sd1 <- 13
# sd at t1
sd2 <- 16
# sd at t2
n <- 80
# sample size
r <- 0.5
# correlation between t1 and t2

# raw mean difference
md_within <- x2-x1
# 计算smd
smd_within <- md_within/
  ((sqrt((sd1^2+sd2^2)/2))/
     (sqrt(2*(1-r))))
# Calculate standard error
se_within <- sqrt(
  ((2*(1-r))/n) +
    (smd_within^2/(2*n))
)
se_within
```