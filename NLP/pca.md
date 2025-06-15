

2025/4/20

[参考视频](https://www.bilibili.com/video/BV1E5411E71z/?spm_id_from=333.337.search-card.all.click&vd_source=841d763fe9132a9c849a39245e699de4)

[latex](https://www.latexlive.com/)

## PCA是什么
就是降维，降维的同时尽可能防止数据失真。
## 降维好处
简化特征的复杂程度，减少训练模型计算量；
## PCA降维缺点
离群点的影响较大。
## 降维的衡量指标
降维后，在各保留维度中的**方差**要最大，因为方差越大数据越散，防止了数据重叠导致信息失真。
## 如何找到方差最大的维度
设降维后的数据叫做“白数据”，我们以“白数据”为目标。
手头数据矩阵D' -> 白数据矩阵D，通过拉伸和旋转；

问题转化为求拉伸的比例、旋转的角度；
设拉伸矩阵S、旋转矩阵R，问题转化为求矩阵S和R；



拉伸矩阵,将x轴拉伸a倍,y轴拉伸b倍
$$S=\begin{pmatrix}
  a&0 \\
  0&b
\end{pmatrix}$$

旋转矩阵,将数据逆时针旋转角度 $\theta$
$$R=\begin{pmatrix}
  cos(\theta )&-sin(\theta)  \\
  sin(\theta)&cos(\theta )
\end{pmatrix}$$

原白噪声数据 D ，经过拉伸，旋转之后得到 $D'=RSD$
$$S^{-1} =\begin{pmatrix}
  1/a&0 \\
  0&1/b
\end{pmatrix}$$

$$R^{-1}=\begin{pmatrix}
  cos(-\theta )&-sin(-\theta)  \\
  sin(-\theta)&cos(-\theta )
\end{pmatrix}=\begin{pmatrix}
  cos(\theta )&sin(\theta)  \\
  -sin(\theta)&cos(\theta )
\end{pmatrix}=R^{T}$$

$D=S^{-1}R^{-1}D'=S^{-1}R^{T}D'$

现在的问题是，已知D'矩阵，要求一个的旋转R，使得RD’的方差最大，（因为拉伸不影响方差，可以不考虑）

## 如何求R

### 结论
协方差矩阵的特征向量就是R

协方差 $cov(x, y)=\frac{ {\textstyle \sum_{i=1}^{n}}(x_i-\bar{x})(y_i-\bar{y})   }{n-1}$，用来衡量两个变量的同向程度，当x=y时，就是方差。x，y不相关时，协方差为0。

协方差矩阵：
$$C=\begin{pmatrix}
  cov(x,x)&cov(x,y)  \\
  cov(x,y)&cov(y,y)
\end{pmatrix}$$

对于白噪声数据，协方差矩阵是对角阵
按cov的定义，可以推出

$$C=\frac{1}{n-1}\begin{pmatrix}
  x_1&x_2&x_3&x_4  \\
  y_1&y_2&y_3&y_4
\end{pmatrix}\begin{pmatrix}
  x_1&y_1  \\
  x_2&y_2  \\
  x_3&y_3  \\
  x_4&y_4  \\
\end{pmatrix}=\frac{1}{n-1}DD^{T}$$

对于白数据D，C为单位矩阵，S是对角阵$S=S^{T}$

$$C'=\frac{1}{n-1}D'D'^{T}=\frac{1}{n-1}RSD(RSD)^{T}=RS(\frac{1}{n-1}DD^{T})S^{T}R^{T}=RSCS^{T}R^{T}=RSS^{T}R^{T}=RLR^{-1}$$

特征向量$\vec{v}$ 和特征值 $\lambda$，具有关系
$C'V=V\Lambda$, 其中$V$是特征向量组成的矩阵，$\Lambda$是所以特征值放在对角线位置的矩阵

$$C'=V\Lambda V^{-1}$$
由此可知，R矩阵就是V，即每个特征向量就是矩阵之后的每个坐标方向，$\lambda$矩阵就是S矩阵的平方，即特征值就是各个方法的拉伸值。

结论证毕。

在实际应用中，通过协方差矩阵计算特征向量R，有时计算量会比较大，因此可能使用奇异值分解（SVD），其中奇异值就是特征值的平方根，

$$D^{T}=U\Sigma V^{T},   其中\Sigma 是奇异值矩阵，V是特征矩阵R$$


