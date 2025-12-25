# Effect Size Correction

In Chapter 3.1, we covered that the effect size θu� ̂ we calculate for a study k is an
estimate of the study’s true effect size θu� , and that θu� ̂ deviates from θu� due to the
sampling error εu� . Unfortunately, in many cases, this is an oversimplification. In the
equation we discussed before, the only thing that separates the estimated effect size
from the true effect is the sampling error. Following the formula, as the sampling
error decreases, the effect size estimate “naturally” converges with the true effect
size in the population. This is not the case, however, when our effect size estimate is
additionally burdened by systematic error, or bias. Such biases can have different
reasons. Some are caused by the mathematical properties of an effect size metric
itself, while other biases are created by the way a study was conducted.
We can deal with biases arising from the way a study was conducted by evaluating
its risk of bias (see Chapter 1.4.5 for an introduction to risk of bias assessment tools
and Chapter 15 for ways to visualize the risk of bias). This judgment can then also
be used to determine if the risk of bias is associated with differences in the pooled
effect, for example, in subgroup analyses (Chapter 7). To deal with biases arising
from the statistical properties of an effect size metric, we can use specific effect size
correction methods to adjust our data before we begin with the meta-analysis.
In this chapter, we will cover three commonly used effect size correction procedures,
and how we can implement them in R.

## 3.4.1 small sample Bias(小样本偏差)

Hedges' g 是 标准化均值差（Standardized Mean Difference, SMD） 的一种校正形式，专门针对小样本研究（通常 n<20）中 Cohen's d 的偏差问题。它通过调整分母的自由度，提供更准确的效应量估计，广泛应用于心理学、教育学和医学等领域的元分析。

* Hedges' g 的定义与公式

$$
g = SMD * (1-\frac{3}{4n-9})
$$
In this formula, n represents the total sample size of the study.

* 代码翻译

```R
# Load esc package
library(esc)
# Define uncorrected SMD and sample size n
SMD <- 0.5
n <- 30
# Convert to Hedges g
g <- hedges_g(SMD, n)
g
```

## 3.4.2 Unreliability(测量误差和效应量低估)

问题根源：当测量工具存在误差（如问卷信度低）时，观测到的效应量（如相关系数 $r_{xy}$或标准化均值差）会系统性低估真实效应。

* 衰减公式

$$
r_{xy}^{observe} = \sqrt{r_{xx}r_{yy}}r_{xy}^{true}
$$

我们得出的相关系数会受到及测量工具的影响
比如测量x的时候工具的问题出现偏差，则对x方法进行矫正

> $r_{xx},r_{yy}：变量x和y的测量信度(0-1)$

$$
r_{xy_{correct}} = \frac{r_{xy}}{\sqrt{r_{xx}}}
$$

但是如果x数据被分为了两组，或者我们计算的就是两组之间的工具，对x的矫正SMD可以直接表示为

$$
SMD_{correct} = \frac{SMD}{\sqrt{r_{xx}}}
$$

当数据中出现两个变量x,y时，需要对这两个变量进行矫正

$$
r_{xy_{correct}} = \frac{r_{xy}}{\sqrt{r_{xx}}\sqrt{r_{yy}}}
$$

* 如果我们想要矫正标准误差中的变量x，用这个方法

$$
SE_{correct} = \frac{SE}{\sqrt{r_{xx}}}
$$

* 如果x,y都要矫正

$$
SE_{correct} = \frac{SE}{\sqrt{r_{xx}}\sqrt{r_{yy}}}
$$

* 代码翻译

```R
####
# correction
#### 

#define uncorrected correlation and SMD with their standard error
r_xy <- 0.34
se_r_xy <- 0.09
smd <- 0.65
se_smd <- 0.18

# Define reliabilities of x and y
r_xx <- 0.8
r_yy <- 0.7

# Correct SMD for unreliability in x
smd_c <- smd/sqrt(r_xx)
smd_c

se_c <- se_smd/sqrt(r_xx)
se_c

# Correct correlation for unreliability in x and y
r_xy_c <- r_xy/(sqrt(r_xx)*sqrt(r_yy))
r_xy_c

se_c <- se_r_xy/(sqrt(r_xx)*sqrt(r_yy))
se_c

>>>output

> ####
> # correction
> #### 
> 
> #define uncorrected correlation and SMD with their standard error
> r_xy <- 0.34
> se_r_xy <- 0.09
> smd <- 0.65
> se_smd <- 0.18
> 
> # Define reliabilities of x and y
> r_xx <- 0.8
> r_yy <- 0.7
> 
> # Correct SMD for unreliability in x
> smd_c <- smd/sqrt(r_xx)
> smd_c
[1] 0.7267221
> 
> se_c <- se_smd/sqrt(r_xx)
> se_c
[1] 0.2012461
> 
> # Correct correlation for unreliability in x and y
> r_xy_c <- r_xy/(sqrt(r_xx)*sqrt(r_yy))
> r_xy_c
[1] 0.4543441
> 
> se_c <- se_r_xy/(sqrt(r_xx)*sqrt(r_yy))
> se_c
[1] 0.1202676
```

可以发现校正后的的标准化的标准差变大了

> It is common in some fields, for example, in organizational psychology, to apply
attenuation corrections. However, in other disciplines, including the biomedical field,
this procedure is rarely used. In meta-analyses, we can only perform a correction for
unreliability if the reliability coefficient ru�u� (and ru�u� ) is reported in each study. Very
often, this is not the case. In this scenario, we may assume a value for the reliability
of the instrument based on previous research. However, given that the correction
has a large impact on the value of the effect size, taking an inappropriate estimate
of ru�u� can distort the results considerably. Also, it is not possible to only correct
some effect sizes in our meta-analysis, while leaving others uncorrected. Due to these
reasons, the applicability of the reliability correction is unfortunately often limited
in practice.

不常用，除非有现成的研究证明某种方法的实际测量系数

## 3.4.3 Range Restriction（范围限制）

如果我们想要分析年龄和认知能力之间的关系，但是样本都是65-69岁的群体，这样分析出的数据可能很难发现这两个因素的关系（当研究样本中变量的变异度（如标准差）小于目标总体时，观测到的效应量（如相关系数 r或回归系数）会被系统性低估。）

* 但是这也是可以矫正的
  
1. ratio U

$$
U = \frac{S_{unrestricted}}{S_{restricted}}
$$

2. 校正后的 $r_{xy}$

$$
r_{xy} = \frac{U*r_{xy}}{\sqrt{(U^2-1)r^2_{xy}+1}}
$$

3. 校正后的 $SMD$

$$
SMD_{corrected} = \frac{U*SMD}{\sqrt{(U^2-1)SMD^2+1}}
$$

4. 校正后的标准差

$$
SE_{r_{xy_c}} = \frac{r_{xy_c}}{r_{xy}}SE_{r_{xy}}
$$
$$
SE_{r_{SMD_c}} = \frac{SMD_{c}}{SMD}SE_{SMD}
$$

* 代码翻译

```R
####
# Range Restriction
#### 


# Define correlation to correct
r_xy <- 0.34
se_r_xy <- 0.09
# Define restricted and unrestricted SD
sd_restricted <- 11
sd_unrestricted <- 18
# Calculate U
U <- sd_unrestricted/sd_restricted
# Correct the correlation
r_xy_c <- (U*r_xy)/sqrt((U^2-1)*r_xy^2+1)
r_xy_c
# Correct the standard error
se_r_xy_c <- (r_xy_c/r_xy)*se_r_xy
se_r_xy_c

```