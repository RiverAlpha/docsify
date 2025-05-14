## 3.3.2.1 Risk Ratio(风险比)

​​Meta分析中的Risk Ratio（风险比，RR）​​
​​Risk Ratio（RR）​​，也称为 ​​Relative Risk（相对风险）​​，是Meta分析中常用的效应量（effect size）之一，用于比较两组（通常是实验组 vs. 对照组）发生某事件的​​相对风险​​。
它适用于​​二分数据（binary/dichotomous outcomes）​​，如疾病发生/未发生、死亡/存活等。

$$
RR = \frac{实验组事件发生率}{对照组事件发生率} = \frac{a/(a+b)}{c/(c+d)}
$$
其中：

a = 实验组发生事件的人数

b = 实验组未发生事件的人数

c = 对照组发生事件的人数

d = 对照组未发生事件的人数

* 对RR Risk Ratio取对数

> ​​关键点​​：RR的效果大小不是对称的！

​​RR=0.5​​（风险减半）的“相反效果”不是 RR=1.5，而是 ​​RR=2​​（风险加倍）。

这种非线性关系导致RR的分布右偏（如下图），直接合并会引入偏差。

> 解决问题：

​​对称化​​：
log(0.5) = -0.693，log(2) = +0.693 → 效果量对称围绕0（无效应）。

​​线性化​​：
风险减半（RR=0.5）和加倍（RR=2）在log尺度上是等距的（±0.693）。

​​统计兼容性​​：
对数转换后可用逆方差加权法合并效应量，计算置信区间更准确。

$$
logRR = log_e(RR)
$$

* 对应的标准误差
  
$$
SE_{log_e(RR)} = \sqrt{\frac{1}{a}+\frac{1}{c}-\frac{1}{a+b}-\frac{1}{c+d}}
$$

```R
####
# Risk Ratio
#### 
# Define data
a <- 46
# events in the treatment group
c <- 77
# events in the control group
n_treat <- 248
# sample size treatment group
n_contr <- 251
# sample size control group
# Calculate the risks
p_treat <- a/n_treat
p_contr <- c/n_contr
# Calculate the risk ratio
rr <- p_treat/p_contr
rr
log_rr <- log(rr)
log_rr
se_log_rr <- sqrt((1/a) + (1/c) - (1/n_treat) - (1/n_contr))
se_log_rr
```

## 3.3.2.2 Odds Ratio（比值比）

比值比（OR）​​ 是Meta分析中处理二分类数据（如患病/未患病、死亡/存活）的核心效应量，反映​​两组事件发生可能性的相对比​​。其定义为：

$$
RR = \frac{实验组事件发生的比值}{对照组事件发生的比值} = \frac{a/b}{c/d} = \frac{ad}{bc}
$$

​​a​​：治疗组发生事件人数

​​b​​：治疗组未发生事件人数

​​c​​：对照组发生事件人数

​​d​​：对照组未发生事件人数

* 转化为log

$$
logOR = log_e(OR)
$$

* 对应的标准误差

$$
SE_{log_e(OR)} = \sqrt{\frac{1}{a}+\frac{1}{b}+\frac{1}{c}+\frac{1}{d}}
$$