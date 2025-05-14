* 皮尔逊相关系数

$$
    r_{xy} = \frac{\sigma^2_{xy}}{\sigma_x\sigma_y}
= \frac{\sum(x_i-\overline{x})(y_i-\overline{y})}{\sqrt{\sum(x_i-\overline{x})^2}\sqrt{\sum(y_i-\overline{y})^2}}
$$

分子：协方差，反应两变量的协同变化

分母：两变量标准差的乘积，用于标准化，消除量纲影响

* 使用皮尔逊相关系数需满足以下假设：

1. 线性关系：变量间的关联可通过直线近似。

2. 正态性：两变量应近似服从正态分布（可通过直方图或Q-Q图检验）。

3. 同方差性：数据点的离散程度在变量范围内大致相同。

4. 无显著异常值：极端值可能扭曲相关系数。

* 皮尔逊相关系数的标准误差

$$
SE_{r_{xy}} = \frac{1-r^2_{xy}}{\sqrt{n-2}}
$$

* 解除r相关系数-1--1的限制

correlations are restricted in their
range, and it can introduce bias when we estimate the standard error for studies
with a small sample size (Alexander et al., 1989). 

> $Fisher's$  $z^4$

$$
z = \frac{1}{2} log_e\frac{1+r}{1-r}
$$

> Fisher'z $z^4$ 标准误差

$$
SE_z = \frac{1}{\sqrt{n-3}}
$$

代码实现：

```R
set.seed(12345)
x <- rnorm(20,50,10)
y <- rnorm(20,10,3)

r <- cor(x,y)
z <- 0.5*log((1+r)/(1-r))
```