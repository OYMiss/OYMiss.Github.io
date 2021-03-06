---
mathjax: true
title: 常见层结构
date: 2020-02-11 18:31:10
tags: [cs231n, 神经网络]
categories: 深度学习
---

所有向量默认指的是列向量，并且记 $A_i \in \mathbb R^m$ 表示矩阵 $A\in \mathbb R^{m \times n}$ 的第 $i$ 个列向量，第 $j$ 个行元素组成的向量（不是行向量，所以维度并不是 $\mathbb R^{1\times n}$）表示为 $(A^T)_j \in \mathbb R^n$。

### Affine 层

#### 公式

$$
\begin{aligned}
    Y &= XW + B \\
    Y_{i,l} &= \sum_k W_{k,l} X_{i,k} + b_l \\
\end{aligned}
$$

其中 $X \in \mathbb R^{N \times D}$，$W \in \mathbb R^{D \times M}$，$B \in \mathbb R^{N \times M}$，$b \in \mathbb R^{M}$，$B =\begin{bmatrix}b^T \\\vdots \\b^T\end{bmatrix}$，$Y \in \mathbb R^{N \times M}$。

<!-- more -->

#### 导数

求对 $b$ 的导数：

$$
\frac{\partial Y_{i,l}}{\partial b_l} = 1
$$

$$
\begin{aligned}
\frac{\partial \ell}{\partial b_l} &=\sum_{i} \frac{\partial \ell}{\partial Y_{i,l}}\frac{\partial Y_{i,l}}{\partial b_l} \\
&= \sum_{i} dY_{i,l} \\
&= {dY}_{l} \cdot \begin{bmatrix} 1 \\ \vdots\\ 1 \end{bmatrix}
\end{aligned}
$$

$$
\frac{\partial \ell}{\partial b} = dY^T \begin{bmatrix}
1 \\
\vdots\\
1
\end{bmatrix} = (\begin{bmatrix}
1 & \cdots & 1
\end{bmatrix} dY)^T
$$


求对 $W$ 的导数：

$$
\frac{\partial Y_{i,l}}{\partial W_{k, l}} = X_{i,k}
$$

$$
\begin{aligned}
\frac{\partial \ell}{\partial W_{k,l}}
&= \sum_{i}\frac{\partial \ell}{\partial Y_{i,l}} \frac{\partial Y_{i,l}}{\partial W_{k, l}} \\
&= \sum_{i}\frac{\partial \ell}{\partial Y_{i,l}} X_{i,k} \\
&= X_{k} \cdot dY_{l}
\end{aligned}
$$

$$
\frac{\partial \ell}{\partial W} = X^TdY
$$

求对 $X$ 的导数：

$$
\frac{\partial Y_{i,l}}{\partial X_{i,k}} = W_{k,l}
$$

$$
\begin{aligned}
\frac{\partial \ell}{\partial X_{i, k} } &= \sum_l \frac{\partial \ell}{\partial Y_{i,l}} \frac{\partial Y_{i,l}}{\partial X_{i,k}} \\
&= \sum_l \frac{\partial \ell}{\partial Y_{i,l}} W_{k, l} \\
&= (dY^T)_i \cdot (W^T)_k
\end{aligned}
$$

$$
\frac{\partial \ell}{\partial X } = dY W^T
$$

### ReLU 层

#### 公式

$$
\begin{aligned}
Y &= \max(0, X)\\
Y_{i, j} &= \max(0, X_{i, j})
\end{aligned}
$$

#### 导数

$$
\frac{\partial Y_{i,j}} {\partial X_{i, j}} = 1\{ X_{i, j} > 0 \}
$$

$$
\begin{aligned}
\frac{\partial \ell}{\partial X_{i, j}} &= \frac{\partial \ell}{\partial Y_{i,j}} \frac{\partial Y_{i,j}} {\partial X_{i, j}} \\
&= \frac{\partial \ell}{\partial Y_{i,j}}1\{ X_{i, j} > 0 \}\\
\end{aligned}
$$

### SVM 损失

#### 公式

$$
\begin{aligned}
L &= \sum_i L_i \\
L_i &= \sum_{j\neq y_i} \max(0, x_j - x_{y_i} + \Delta) \\
\text{margin}_{i, j} &=
\begin{cases} 0 & \text{when $j = y_i$}\\
x_j - x_{y_i} + \Delta & \text{when $x_j - x_{y_i} + \Delta > 0$} \\
0 & \text{when $x_j - x_{y_i} + \Delta \le 0$}
\end{cases} \\
L &= \sum_{i, j} \text{margin}_{i, j}
\end{aligned}
$$

其中 $(X^T)_i = x$，$X\in \mathbb R^{N \times D}$。

#### 导数

先写出 $\text{margin}$ 的导数：

$$
\frac{\partial \text{margin}_{i,j}}{\partial x_k} =
\begin{cases}
1 & k = j \\
-1 & k = y_i \\
\end{cases} \text{ when margin}_{i,j} > 0 \\
$$

对于 $k = y_i$ 的情况下：

$$
\begin{aligned}
\frac{\partial L_i}{\partial X_{i,y_{i}}} &= \frac{\partial L_i}{\partial x_{y_i}} \\
&= \sum_j\frac{\partial \text{margin}_{i,j}}{\partial x_{y_i}} \\
&= -\sum_j 1\{\text{margin}_{i,j} > 0\}\\
\end{aligned} \\
$$

对于 $k \ne y_i$ 的情况下：

$$
\begin{aligned}
\frac{\partial L_i}{\partial X_{i,k}} &= \frac{\partial L_i}{\partial x_k} \\
&= \sum_j\frac{\partial \text{margin}_{i,j}}{\partial x_{k}} \\
&=  1\{\text{margin}_{i,k} > 0\} \\
\end{aligned} \\
$$


### Softmax 损失

#### 公式

$$
\begin{aligned}
    L &= \sum_i L_i \\
    L_i &= -\log\left(\frac{e^{x_{y_i}}}{ \sum_j e^{x_j} }\right) \\
    L_i &= -x_{y_i} + \log\sum_j e^{x_j}
\end{aligned}
$$

其中 $(X^T)_i = x$，$X\in \mathbb R^{N \times D}$。

#### 导数

$$
\frac{\partial L_i}{\partial X_{i,k}} = \frac{\partial L_i}{\partial x_k} = -1\{k = y_i\} + \frac{e^{x_k}}{\sum_j e^{x_j}}
$$

> 计算出 $e^{x_k} / \sum_j e^{x_j}$ 之后，就可以求出 $\ell$ 和 $dX$。

### Batch Normalization 层

#### 公式

$$
\begin{aligned}
    \mu &= \frac{1}{m} \sum_{i=1}^m x_i \\
    \sigma^2 &= \frac{1}{m} \sum_{i=1}^m (x_i - \mu)^2 \\
    \hat x_i &= \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}} \\
    y_i &= \gamma \hat x_i + \beta = \text{BN}_{\gamma, \beta}(x_i)
\end{aligned}
$$

需要注意，这里的要训练的参数为 $\gamma$ 和 $\beta$。

#### 导数

$$
\begin{aligned}
\frac{\partial \ell}{\partial \hat x_i}
&= \frac{\partial \ell}{\partial y_i} \gamma \\
\frac{\partial \ell}{\partial \sigma^2} &= \sum_{i=1}^{m} \frac{\partial \ell}{\partial \hat x_i} (x_i - \mu)(-\frac{1}{2})(\sigma^2 + \epsilon)^{-\frac{3}{2}} \\
\frac{\partial \ell}{\partial \mu} &= \frac{\partial \ell}{\partial \sigma^2}(-\frac{2}{m} )\sum_{i=1}^m (x_i - \mu) + \sum_{i=1}^{m} \frac{\partial \ell}{\partial \hat x_i} \frac{-1}{\sqrt{\sigma^2 + \epsilon}} \\
\frac{\partial \ell}{\partial x_i} &= \frac{\partial \ell}{\partial \hat x_i} \frac{1}{\sqrt{\sigma^2 + \epsilon}} + \frac{\partial \ell}{\partial \sigma^2} \frac{2}{m} (x_i - \mu) + \frac{\partial \ell}{\partial \mu} \frac{1}{m} \\
\frac{\partial \ell}{\partial \gamma} &= \sum_{i=1}^m \frac{\partial \ell}{\partial y_i}\hat x_i \\
\frac{\partial \ell}{\partial \beta} &= \sum_{i=1}^m \frac{\partial \ell}{\partial y_i}1 \\
\end{aligned}
$$

如果把 $\text{BN}$ 层分为多层，然后计算每层的导数：

$$
\begin{aligned}
\mu &= \frac{1}{m} \sum_{i=1}^m x_i \\
v &= \frac{1}{m} \sum_{i=1}^m (x_i - \mu)^2 \\
\sigma &= \sqrt{v + \epsilon} \\
\hat x_i &= \frac{x_i - \mu}{\sigma} \\
\end{aligned}
$$

$\hat x_i \to \sigma, \mu$ 过程：

$$
\begin{aligned}
\frac{\partial \ell}{\partial \sigma} &= \sum_i \frac{\partial \ell}{\partial \hat x_i} \frac{ x_i - \mu }{-\sigma^2} \\
\frac{\partial \ell}{\partial \mu} &= \sum_i \frac{\partial \ell}{\partial \hat x_i} \frac{ -1 }{\sigma} \\
\frac{\partial \ell}{\partial x_i} &= \frac{\partial \ell}{\partial \hat x_i} \frac{ 1 }{\sigma} \\
\end{aligned}
$$

$\sigma \to v$ 过程：

$$
\frac{\partial \ell }{\partial v} = \frac{\partial \ell}{\partial \sigma} \frac{1}{2\sqrt{v + \epsilon}} = \frac{\partial \ell}{\partial \sigma} \frac{1}{2\sigma} \\
$$

$v \to x, \mu$ 过程：

$$
\begin{aligned}
\frac{\partial \ell}{\partial x_i} &= \frac{\partial \ell}{\partial v} \frac{2}{m} (x_i - \mu) \\
\frac{\partial \ell}{\partial \mu} &= \frac{\partial \ell}{\partial v} (-\frac{2}{m}) \sum_i (x_i - \mu) \\
\end{aligned}
$$

$\mu \to x$ 过程：

$$
\frac{\partial \ell }{\partial x_i} = \frac{\partial \ell }{\partial \mu} \frac{1}{m}
$$

### Dropout 层

#### 公式

$$
y = x \frac{1\{a \ge p\}}{p}
$$

其中 $a$ 是 $(0, 1)$ 的随机数。如果不除以 $p$ 会导致输出的均值与输入均值相差 $p$ 倍。

#### 导数

$$
\begin{aligned}
\frac{\partial \ell}{\partial x}
&= \frac{\partial \ell}{\partial y} \frac{\partial y}{\partial x} \\
&= \frac{\partial \ell }{\partial y} \frac{1\{a \ge p\}}{p}
\end{aligned}
$$

### 卷积层

#### 公式

$$
Y_{i,j, k, l} = \sum_c \sum_{a, b} X_{i,c,ks + a,ls + b} W_{j,c,a,b} + b_j
$$

$s$ 为 $W$ 移动的步长，其中 $X \in \mathbb R^{N \times C \times H \times W}$，$W \in \mathbb R^{F \times C \times H_w \times W_w}$，$Y \in \mathbb R^{N \times C \times H' \times W'}$，并有 $H' = 1 + (H - H_w) / s, W' = 1 + (W - W_w) / s$。

利用 `numpy` 的函数：

$$
Y_{i,j,k,l} = \sum_c \text{np.sum}(X^{(k, l)}_{i,c} * W_{j,c}) + b_j
$$

> 这里 $*$ 是 element-wise multiplication。

为了方便计算，令 $X^{(k, l)}_{i,c,a,b} = X_{i,c,ks + a,ls + b}$，这里 $X^{(k, l)}_{i,c}$ 表示在矩阵 $X_{i,c}$ 中左上角坐标为 $(ks, ls)$，右下角坐标为 $(ks + H_w - 1, ls+W_w - 1)$ 的子矩阵。

因为输入数据为维度为 $\mathbb R^{N \times C \times H \times W}$，卷积层的 Normalization 也与通常的不太一样。

下图展示了四种 Normalization 对卷积层的处理方式：

<img src="/asset/common-layers/all-norms.svg" width = "100%" height = "100%" alt="all-norms"/>

> 计算的技巧是将维度转化为二维，过程与 Batch Normalization 几乎一样。

对于 CNN 来说，Group Normalization 既拥有 Layer Normalization 与 batch size 无关的性质又与  Batch Normalization 效果差不多。

#### 导数

首先求各自偏导数：

$$
\begin{aligned}
\frac{\partial Y_{i,j,k,l}}{\partial X^{(k, l)}_{i,c}} &= W_{j,c} \\
\frac{\partial Y_{i,j,k,l}}{\partial W_{j,c}} &= X^{(k, l)}_{i,c} \\
\frac{\partial Y_{i,j,k,l}}{\partial b_j} &= 1\\
\end{aligned}
$$

然后求损失函数的导数：

$$
\begin{aligned}
\frac{\partial \ell}{\partial X^{(k, l)}_{i,c}}
&= \sum_j \frac{\partial \ell}{\partial Y_{i, j, k, l}} \frac{\partial Y_{i, j, k, l}}{\partial X^{(k, l)}_{i,c}} \\
&= \sum_j \frac{\partial \ell}{\partial Y_{i, j, k, l}} W_{j,c} \\
\frac{\partial \ell}{\partial W_{j,c}}
&= \sum_i \sum_{k,l} \frac{\partial \ell}{\partial Y_{i, j, k, l}} \frac{\partial Y_{i, j, k, l}}{\partial W_{j,c}} \\
&= \sum_i \sum_{k,l} \frac{\partial \ell}{\partial Y_{i, j, k, l}} X^{(k, l)}_{i,c} \\
\frac{\partial \ell}{\partial b_j}
&= \sum_i \sum_{k,l} \frac{\partial \ell}{\partial Y_{i, j, k, l}} \frac{\partial Y_{i, j, k, l}}{\partial b_{j}} \\
&= \sum_i \sum_{k,l} \frac{\partial \ell}{\partial Y_{i, j, k, l}}
\end{aligned}
$$

### Max-Pooling 层

#### 公式

$$
Y_{i,c,k,l} =  \max_{a,b} X_{i,c,ks + a,ls + b}
$$

为了方便计算，令 $X^{(k, l)}_{i,c,a,b} = X_{i,c,ks + a,ls + b}$，这里 $X^{(k, l)}_{i,c}$ 表示在矩阵 $X_{i,c}$ 中左上角坐标为 $(ks, ls)$，右下角坐标为 $(ks + H_p - 1, ls+W_p - 1)$ 的子矩阵。

#### 导数

$$
\begin{aligned}
\frac{\partial \ell}{\partial X^{(k, l)}_{i,c,a,b}}
&= \frac{\partial \ell}{\partial Y_{i, c, k, l}} \frac{\partial Y_{i, c, k, l}}{\partial X^{(k, l)}_{i,c,a,b}} \\
&= \frac{\partial \ell}{\partial Y_{i, j, k, l}} 1\{ (a,b) = \arg \max(X^{(k, l)}_{i,c})\}
\end{aligned}
$$

[1] [CS231n: Convolutional Neural Networks for Visual Recognition](http://cs231n.stanford.edu/)

[2] [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167)

[3] [Layer Normalization](https://arxiv.org/abs/1607.06450)

[4] [Group Normalization](https://arxiv.org/abs/1803.08494)

