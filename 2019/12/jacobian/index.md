# Jacobian Matrix


# Jacobian Matrix
向量分析中,雅可比矩阵是一阶偏导数以一定方式排列成的矩阵。在代数几何中, 代数曲线的雅可比量表示雅可比簇：伴随该曲线的一个代数群, 曲线可以嵌入其中。

雅可比矩阵体现了一个可微方程与给出点的最优线性逼近，雅可比矩阵类似于多元函数的导数。

## 定义
设 $f:\mathbb{R}_n\rightarrow\mathbb{R}_m$ 是一个函数，它的输入是向量 $x\in\mathbb{R}_n$ ，输出是向量 $y=f(x)\in\mathbb{R}_m$ :

$$\\left \\{\begin{matrix}
y_1=f_1(x_1,x_2,...,x_n) \\\\\\
y_2=f_2(x_1,x_2,...,x_n) \\\\\\
... \\\\\\
y_m=f_m(x_1,x_2,...,x_n)
\end{matrix} \\right.$$

那么雅可比矩阵是一个m×n矩阵:

$$\\mathbf{J}=
\begin{bmatrix}
\frac{\partial{f_1}}{\partial{x_1}} & \cdots & \frac{\partial{f_1}}{\partial{x_n}} \\\\\\
\vdots & \ddots & \vdots \\\\\\
\frac{\partial{f_m}}{\partial{x_1}} & \cdots & \frac{\partial{f_m}}{\partial{x_n}} \\\\\\
\end{bmatrix}$$

由于矩阵描述了向量空间中的运动——变换，而雅可比矩阵看作是将点 $(x_1,x_2,\cdots,x_n)$ 转化到点 $(y_1,y_2,\cdots,y_m)$ ，或者说是从一个n维的欧式空间转换到m维的欧氏空间。

如果 $m = n$， 可以定义雅可比矩阵 $\mathbf{J}$ 的行列式，也就是雅可比行列式（Jacobian determinant）。

## 理解
单变量情况，如果$f:R\rightarrow R$,则有：
$\bf f^\prime(x) = \lim_{\nabla x \rightarrow 0} \frac{f(x + \nabla x) - f(x)}{\nabla x}$

以一种非常有用的方式考虑$f^\prime(x)$就是：
$\bf f(x + \nabla x) \approx  f(x) + f^\prime(x)\nabla x$

上述方程在n维欧式空间到m维欧式空间的映射$F:R^n\rightarrow R^m$同样非常有意义：
$\bf F(x + \nabla x) \approx  F(x) + F^\prime(x)\nabla x$

* 其中$F^\prime(x)$就是函数在一点xx的导数也就是Jacobian矩阵JF(x)JF(x)
* $x \in R^n,F(x) \in R^m,J_F(x)$为 $m\times n$：可见上式完成了不同维度空间的**线性映射**
* 其重要意义在于它表现了一个多变数向量函数的最佳线性逼近
* 雅可比矩阵类似于单变数函数的导数
* Jacobian矩阵$J_F(x)$允许我们通过线性函数（affine function）局部估计一个函数$F$