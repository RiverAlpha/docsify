# Unet

[Unet 论文原文 https://arxiv.org/pdf/1505.04597](https://arxiv.org/pdf/1505.04597)

## 1. 前置知识

> 1. 卷积输出长宽计算

输入尺寸（input）： $i$

卷积核大小（kernel size）： $k$

步幅（stride）： $s$

边界扩充（padding）： $p$

输出尺寸（output）： $o$

$$
o = \lfloor \frac{i+2p-k}{s} \rfloor +1
$$

## 2. 2D张量推演过程

为了方便计算并符合目前工程上最常用的 U-Net 变体，采用 Same Padding（即卷积后特征图尺寸不缩水，padding=1, stride=1, kernel_size=3）。

假设输入图像张量维度为 $C \times H \times W = 1 \times 256 \times 256$。

> Encoder Block1(编码器第一层)【下采样】

1. 第一次卷积

特征提取与第一次下采样，第一次卷积 (Conv2d):

输入: $1 \times 256 \times 256$

卷积核权重张量 ($W$): $C_{out} \times C_{in} \times K \times K = 64 \times 1 \times 3 \times 3$

计算: 输出尺寸 $H_{out} = \lfloor\frac{256 - 3 + 2(1)}{1}\rfloor + 1 = 256$

输出张量: $64 \times 256 \times 256$

2. 第二次卷积

输入: $64 \times 256 \times 256$

卷积核权重张量 ($W$): $64 \times 64 \times 3 \times 3$

-具体卷积实现，每个通道一个3X3卷积核，一共64个卷积核，卷积之后每个元素对应相加得到(1,256,256),这样的卷积核64套-

输出张量: $64 \times 256 \times 256$

关键动作： 复制并保留这个 $64 \times 256 \times 256$ 的特征图，用于后续的跳跃连接 (Skip Connection)。

3. 最大池化（2X2）

输入: $64 \times 256 \times 256$

计算: 空间维度减半 ($stride=2$)。

输出张量: $64 \times 128 \times 128$（传递给下一层）

> 将Encoder Block1一共重复四次，每次卷积核都乘二，第二层Ecoder 128个核，第三层256个核...，池化比例不变，都是2X2

$第一层(1X256X256) => 卷积(64X256X256)=> 2X2池化(64X128X128)$

$第二层(64X128X128) => 卷积(128X128X128)=> 2X2池化(128X64X64)$

$第三层(128X64X64) => 卷积(256X128X128)=> 2X2池化(256X32X32)$

$第四层(256X32X32) => 卷积(512X32X32)=> 2X2池化(512X16X16)$

> 特征提取卷积（不进行上采样或下采样）

到达 U 型网络的最底部假设经过 4 次池化后，空间维度变为原来的 $1/16$，通道数翻倍到 512。

输入张量: $512 \times 16 \times 16$ （来自编码器第 4 层的池化输出）

卷积核权重张量 ($W$): 两个连续的 $1024 \times 512 \times 3 \times 3$ 和 $1024 \times 1024 \times 3 \times 3$。

瓶颈层输出张量: $1024 \times 16 \times 16$

> Decoder Block 1(解码器第 1 层) 【上采样】

上采样与跳跃连接拼接

1. 上采样转置卷积 (ConvTranspose2d):

输入: $1024 \times 16 \times 16$

权重张量 ($W$): 转置卷积的权重形状为 $C_{in} \times C_{out} \times K \times K = 1024 \times 512 \times 2 \times 2$。

计算: 空间维度翻倍 ($stride=2$)，通道数减半。

在 PyTorch 中，两种卷积层的权重维度定义如下：

- 标准卷积 (Conv2d) 权重形状：(输出通道数, 输入通道数, 高, 宽)

- 转置卷积 (ConvTranspose2d) 权重形状：(输入通道数, 输出通道数, 高, 宽)

输出张量: $512 \times 32 \times 32$

2. 跳跃连接 (Skip Connection - Concat):

取出编码器对应层（第 4 层）之前保存的特征图，维度刚好是 $512 \times 32 \times 32$。

计算: 在通道维度（Channel）上进行拼接（Concatenation）。

输出张量: $(512 + 512) \times 32 \times 32 = 1024 \times 32 \times 32$。

3. 融合卷积 (Conv2d):

输入: $1024 \times 32 \times 32$（注意这里因为拼接，通道数变厚了）。

卷积核权重张量 ($W$): $512 \times 1024 \times 3 \times 3$（这一步负责将通道数重新降维压缩）。

输出张量: $512 \times 32 \times 32$。

Decoder (解码器) 张量变化表

| 网络层级      | 输入特征图尺寸 (C×H×W) |卷积操作 (上采样 → 拼接 →组)|输出特征图尺寸 (C×H×W)|
| ----------- | ----------- |----------- |----------- |
|  Decoder 1      | $1024 \times 16 \times 16$(来自瓶颈层)       | Up-Conv: 变为 $512 \times 32 \times 32$ Concat: 拼入 Encoder 4 ($512$) $\rightarrow$ $1024$ 通道Conv组: 压缩并输出 $512$ 通道 | $512 \times 32 \times 32$ |
| Decoder 2   | $512 \times 32 \times 32$        |Up-Conv: 变为 $256 \times 64 \times 64$Concat: 拼入 Encoder 3 ($256$) $\rightarrow$ $512$ 通道Conv组: 压缩并输出 $256$ 通道 |$256 \times 64 \times 64$ |
|Decoder 3 |$256 \times 64 \times 64$ |$256 \times 64 \times 64$Up-Conv: 变为 $128 \times 128 \times 128$Concat: 拼入 Encoder 2 ($128$) $\rightarrow$ $256$ 通道Conv组: 压缩并输出 $128$ 通道 |$128 \times 128 \times 128$ |
|Decoder 4 |$128 \times 128 \times 128$ | $128 \times 128 \times 128$Up-Conv: 变为 $64 \times 256 \times 256$Concat: 拼入 Encoder 1 ($64$) $\rightarrow$ $128$ 通道Conv组: 压缩并输出 $64$ 通道 | $64 \times 256 \times 256$(恢复原图分辨率) |
| Output Head | $64 \times 256 \times 256$ | $64 \times 256 \times 256$Conv 1x1: 仅改变通道数，映射到目标类别 | $1 \times 256 \times 256$(最终预测图) |


