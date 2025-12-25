# Incidence Rate Ratios(发生率比)

发生率比（IRR）是流行病学和医学研究中用于比较两组事件发生率的相对效应量指标，适用于基于人时（Person-Time）数据的研究设计（如队列研究）。其核心是衡量暴露组与非暴露组事件发生率的比值。

* 人年（person year）

基本概念：1人年 = 1个人被观察满1年。

示例：若10人每人被观察1年，总人年=10；若1人被观察10年，总人年=10。

作用：作为分母计算发生率（如每100人年的发病数），更精准地反映动态人群的风险。

$$
IRR = \frac{暴露组发生率(Rate_1)}{非暴露组发生率(Rate_2)} = \frac{\frac{E_{treat}}{T_{treat}}}{\frac{E_{control}}{T_{control}}}
$$

* 取对数
  
$$
logIRR = log_e(IRR)
$$

* 标准误差

$$
SE_{logIRR} =  \sqrt{\frac{1}{E_{treat}}+\frac{1}{E_{control}}}
$$

* 代码翻译

```R
####
# irr
#### 

# Define Data

# Number of events in the treatment group
e_treat <- 28
# Number of events in the control group
e_contr <- 28
# Person-time in the treatment group
t_treat <- 3025
# Person-time in the control group
t_contr <- 2380

# Calculate IRR
irr <- (e_treat/t_treat)/(e_contr/t_contr)
irr
# Calculate log-IRR
log_irr <- log(irr)
# Calculate standard error
se_log_irr <- sqrt((1/e_treat)+(1/e_contr))
```