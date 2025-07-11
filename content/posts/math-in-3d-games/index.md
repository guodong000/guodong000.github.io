+++
date = '2024-06-05T16:37:58+08:00'
draft = false
title = '3D游戏中的数学'
summary = '坐标系、向量、矩阵和四元数'
showtoc = true
math = true
+++

## 坐标系

在 3D 游戏中，最常用的是笛卡尔坐标系（Cartesian Coordinates），即：三条相交于原点不共面且相互垂直的数轴构成的坐标系。

{{< figure
    src="./images/coordinate-system-left-right.png"
    align="center"
    caption="Coordinate System Chirality"
>}}

四指从 x 轴握向 y 轴，此时拇指指向为 z 轴。由此分为左手坐标系和右手坐标系。坐标系手性的不同一般不影响其空间内的数学运算（向量的叉积需要考虑坐标系手性，因为向量的叉积是一个伪向量 pseudovector）。

手性变换：反转任意一个轴。

## 向量

向量是具有大小（magnitude）和方向的量，可以理解为相对某个点的偏移。

向量大小（模）：\(|\boldsymbol{a}| = \sqrt{a_x^2 + a_y^2 + a_z^2}\)（由于开方计算开销大，一般使用 \(|\boldsymbol{a}|^2\) 替代）。

单位向量（**unit vector**）：大小为 1 的向量。

将一个向量 \(v\) 转变为单位向量 \(u\) 的操作被称为正规化（**normalization**）

$$\boldsymbol{u} = \frac{\boldsymbol{v}}{|\boldsymbol{v}|}$$

### 向量运算

#### 标量乘法

有向量 \(\boldsymbol{a} = (a_x, a_y, a_z)\)，标量 \(s\)

$$s\boldsymbol{a} = (sa_x, sa_y, sa_z)$$

#### 元素乘法

有向量 \(\boldsymbol{a} = (a_x, a_y, a_z)\) 和 \(\boldsymbol{s} = (s_x, s_y, s_z)\)

$$\boldsymbol{s}\otimes\boldsymbol{a} = (s_xa_x, s_ya_y, s_za_z)$$

#### 加（减）法

有向量 \(\boldsymbol{a} = (a_x, a_y, a_z)\) 和 \(\boldsymbol{b} = (b_x, b_y, b_z)\)

$$\boldsymbol{a}+\boldsymbol{b} = (a_x+b_x, a_y+b_y, a_z+b_z)$$

![Vector Add](./images/vector_add.png#center)

* 向量 + 向量 = 向量
* 点 + 向量 = 点
* 点 - 点 = 向量
* 点 + 点【没有意义】

#### 点积 Dot Product

点积又称为标量积、内积，其结果为标量。

有向量 \(\boldsymbol{a} = (a_x, a_y, a_z)\) 和 \(\boldsymbol{b} = (b_x, b_y, b_z)\)，\(\boldsymbol{a}\) \(\boldsymbol{b}\) 之间的夹角为 \(\theta\)。

$$\boldsymbol{a}\bullet\boldsymbol{b} = a_xb_x + a_yb_y + a_zb_z$$

$$\boldsymbol{a}\bullet\boldsymbol{b} = |\boldsymbol{a}||\boldsymbol{b}|\cos \theta$$

$$\boldsymbol{a}\bullet\boldsymbol{b} = \boldsymbol{b}\bullet\boldsymbol{a}$$

$$\boldsymbol{a}\bullet(\boldsymbol{b}+\boldsymbol{c}) = \boldsymbol{a}\bullet\boldsymbol{b} + \boldsymbol{a}\bullet\boldsymbol{c}$$

点积的几何意义：**a** 向量在 **b** 向量上投影的大小与 **b** 的大小的积。当 **b** 为单位向量时，**a** 与 **b** 的点积为 **a** 在 **b** 上投影的大小（因为单位向量的大小为1）。

点积可用于检测向量的方向关系：

* 同向共线：\(\boldsymbol{a}\bullet\boldsymbol{b} = |\boldsymbol{a}||\boldsymbol{b}|\)
* 反向共线：\(\boldsymbol{a}\bullet\boldsymbol{b} = -|\boldsymbol{a}||\boldsymbol{b}|\)
* 垂直：\(\boldsymbol{a}\bullet\boldsymbol{b} = 0\)
* 同向：\(\boldsymbol{a}\bullet\boldsymbol{b} > 0\)
* 反向：\(\boldsymbol{a}\bullet\boldsymbol{b} < 0\)

#### 叉积 Corss Product

三维向量 **a** **b** 的叉积的结果为垂直于两向量的向量（两向量所在平面的法向量 normal vector），其方向取决于当前坐标系的手性，比如在右手坐标系中，将右手四指从 **a** 握向 **b** ，拇指的指向即为其方向（反之亦然）。
叉积仅定义在三维空间中。

![Corss Product](./images/cross_product-1.png#center)

$$
\begin{align}
\boldsymbol{a}\times\boldsymbol{b} & = (a_yb_z-a_zb_y, a_zb_x-a_xb_z, a_xb_y-a_yb_x) \\
& = \begin{bmatrix}
0 & -a_z & a_y \\
a_z & 0 & -a_x \\
-a_y & a_x & 0
\end{bmatrix}
\begin{bmatrix}b_x \\ b_y \\ b_z \end{bmatrix}
\end{align}
$$

$$|\boldsymbol{a}\times\boldsymbol{b}| = |\boldsymbol{a}||\boldsymbol{b}|\sin \theta \qquad (\theta \text{为向量夹角})$$

\(|\boldsymbol{a}\times\boldsymbol{b}|\) 可以理解为向量 a、b 围成的平行四边形的面积。

$$\boldsymbol{a}\times\boldsymbol{b} \ne \boldsymbol{b}\times\boldsymbol{a}$$

$$\boldsymbol{a}\times\boldsymbol{b} = -(\boldsymbol{b}\times\boldsymbol{a})$$

$$\boldsymbol{a}\times(\boldsymbol{b} + \boldsymbol{c}) = \boldsymbol{a}\times\boldsymbol{b} + \boldsymbol{a}\times\boldsymbol{c}$$

$$(s\boldsymbol{a})\times\boldsymbol{b} = \boldsymbol{a}\times(s\boldsymbol{b}) = s(\boldsymbol{a}\times\boldsymbol{b}) \qquad (s \text{为标量})$$

叉积的结果是一个伪向量（**pseudovector**），向量与伪向量的区别仅体现在进行坐标系变换（手性变换）时，伪向量的方向会反转。

#### 线性插值

线性插值 linear interpolation 简称 LERP，通常用于动画中，在两个状态之间平滑过渡。

比如从点A到点B的线性插值，\(\beta\) 为百分比：

$$LERP(A, B, \beta) = (1-\beta)A + \beta B$$

## 矩阵相关知识

$$\boldsymbol{A}\boldsymbol{B} \ne \boldsymbol{B}\boldsymbol{A}$$

### 单位矩阵（identity matrix）

对角线为1，其余为0的矩阵，表示为 \(\boldsymbol{I}\)。

$$I_{3x3} = \begin{bmatrix}1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

矩阵 \(\boldsymbol{A}\) 的逆表示为 \(\boldsymbol{A}^{-1}\)，且 \(\boldsymbol{A}\boldsymbol{A}^{-1} = \boldsymbol{I}\)。

### 逆矩阵

变换矩阵的逆矩阵可以理解为其对应的反向变换。

不是所有的矩阵都有逆矩阵，但所有的仿射矩阵都可逆。

$$(\boldsymbol{A}\boldsymbol{B}\boldsymbol{C})^{-1} = \boldsymbol{C}^{-1}\boldsymbol{B}^{-1}\boldsymbol{A}^{-1}$$

### 矩阵转置 Transposition

矩阵 \(\boldsymbol{M}\) 的转置矩阵记为 \(\boldsymbol{M}^{T}\)

$$
\begin{bmatrix}
a & b \\ c & d
\end{bmatrix}^{T}=
\begin{bmatrix}
a & c \\ b & d
\end{bmatrix}
$$

$$(\boldsymbol{A}\boldsymbol{B}\boldsymbol{C})^{T} = \boldsymbol{C}^{T}\boldsymbol{B}^{T}\boldsymbol{A}^{T}$$

## 变换矩阵

一个4x4的矩阵可以表示任意一种3D变换（平移、旋转、缩放、剪切），这种矩阵被称为变换矩阵（**transformation matrix**）。

将任意不同的变换矩阵组合一起（相乘）得到的矩阵被称为仿射矩阵（**affine matrix**），用于仿射变换。

$$
\boldsymbol{M}_{affine} = 
\begin{bmatrix}
\boldsymbol{U}_{3\times3} & 0_{3\times1} \\
\boldsymbol{t}_{1\times3} & 1
\end{bmatrix}
$$

其中 \(\boldsymbol{U}_{3\times3}\) 表示缩放与旋转，\(\boldsymbol{t}_{1\times3}\) 表示平移。

为了将三维向量与 4x4 仿射矩阵进行计算，需要将三维向量 \((x, y, z)\) 转化为 \((x, y, z, 1)\)。而对于三维方向向量，由于对其进行平移没有意义，其可表示为 \((x, y, z, 0)\)。

### 平移变换 Translation

$$
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
t_x & t_y & t_z & 1
\end{bmatrix}=
\begin{bmatrix}
\boldsymbol{I} & 0 \\
\boldsymbol{t} & 1
\end{bmatrix}
$$

其逆矩阵为 \(\begin{bmatrix}\boldsymbol{I} & 0 \\ -\boldsymbol{t} & 1\end{bmatrix}\)

### 旋转变换 Rotation

$$\begin{bmatrix}\boldsymbol{R} & 0 \\ 0 & 1\end{bmatrix}$$

$$
rotate_x(\boldsymbol{r}, \phi) = 
\boldsymbol{r}
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & \cos \phi & \sin \phi & 0 \\
0 & -\sin \phi & \cos \phi & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\qquad (绕\ x\ 轴旋转)
$$

$$
rotate_y(\boldsymbol{r}, \theta) = 
\boldsymbol{r}
\begin{bmatrix}
\cos \theta & 0 & -\sin \theta & 0 \\
0 & 1 & 0 & 0 \\
\sin \theta & 0 & \cos \theta & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\qquad (绕\ y\ 轴旋转)
$$

$$
rotate_z(\boldsymbol{r}, \gamma) = 
\boldsymbol{r}
\begin{bmatrix}
\cos \gamma & \sin \gamma & 0 & 0 \\
-\sin \gamma & \cos \gamma & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\qquad (绕\ z\ 轴旋转)
$$

旋转变换矩阵的逆等于其转置矩阵。这是因为旋转 \(\theta\) 角度的变换矩阵的逆为旋转 \(-\theta\) 角度的变换矩阵，而 \(\sin(-\theta) = -\sin\theta, \cos(-\theta) = \cos\theta\)。

### 缩放 Scale

$$
\boldsymbol{r}\boldsymbol{S} = 
\boldsymbol{r}
\begin{bmatrix}
S_x & 0 & 0 & 0 \\
0 & S_y & 0 & 0 \\
0 & 0 & S_z & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

其逆矩阵为： 

$$
\begin{bmatrix}
\frac{1}{S_x} & 0 & 0 & 0 \\
0 & \frac{1}{S_y} & 0 & 0 \\
0 & 0 & \frac{1}{S_z} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

当 \(S_x = S_y = S_z\) 时称之为统一缩放（**uniform scale**）。

> 由于 4x4 仿射矩阵的最后一列始终为 \([0\ 0\ 0\ 1]^T\)，所以在实际应用中为了节约空间往往会将其省略。

## 坐标空间

在父坐标空间中的物体可以拥有自己的子坐标空间，从而形成坐标空间的继承关系。将一个坐标在相关坐标空间之间的转换操作被称为 change of basis。

全局空间（world space）没有父坐标空间，其为坐标空间继承树的根。

将点和方向从子坐标空间 C 转换到父坐标空间 P 的矩阵表示为 \(\boldsymbol{M}_{C\to P}\)。

对于任意子坐标空间 C 中的位置向量 \(\boldsymbol{P}_C\)，且其在父坐标空间 P 中的表示为 \(\boldsymbol{P}_P\)，存在以下公式：

$$\boldsymbol{P}_P = \boldsymbol{P}_C \boldsymbol{M}_{C\to P}$$

$$
\boldsymbol{M}_{C\to P} = 
\begin{bmatrix}
\boldsymbol{i}_C & 0 \\
\boldsymbol{j}_C & 0 \\
\boldsymbol{k}_C & 0 \\
\boldsymbol{t}_C & 0
\end{bmatrix}
$$

其中 \(\boldsymbol{i}_C\ \boldsymbol{j}_C\ \boldsymbol{k}_C\) 为子坐标空间坐标轴上的单位基向量在父坐标空间内的表示。\(\boldsymbol{t}_C\) 为子坐标空间坐标原点相对于父坐标空间坐标原点的平移。

对子坐标系统的缩放可直接表示为对 \(\boldsymbol{i}_C\ \boldsymbol{j}_C\ \boldsymbol{k}_C\) 的缩放。

> 将子坐标空间中的法向量转换到父坐标空间中，需要使用 \((\boldsymbol{M}_{C\to P}^{-1})^T\)。这是因为法向量是一个伪向量。

## 四元数

四元数（**quaternion**）在3D游戏中主要用于表示旋转。

四元数 \(q = [q_x, q_y, q_z, w]\) 看起来像是一个四维向量（但并非向量）。

单位长度（unit-length）四元数（\(|q| = 1\)）用于表示3D旋转，本文只考虑单位四元数。

单位四元数可以看作一个三维向量 \(\boldsymbol{q}_V\) 外加一个标量 \(q_S\)。

以单位向量 \(\boldsymbol{a}\) 为旋转轴，\(\theta\) 为旋转角度，单位四元组 \(\boldsymbol{q}\) 表示为：

$$
\begin{align}
q
& = [\boldsymbol{q}_V \quad q_S] \\
& = [\boldsymbol{a}\sin\frac{\theta}{2} \quad \cos\frac{\theta}{2}]
\end{align}
$$

### 四元数运算

#### Grassman 乘法

> 实际上四元数有多种乘法，这里只讨论 Grassman 乘法。

多个四元数相乘表示组合。

$$
p q = [
(p_S\boldsymbol{q}_V + 
q_S\boldsymbol{p}_V +
\boldsymbol{p}_V\times\boldsymbol{q}_V) \quad
(p_S q_S -
\boldsymbol{p}_V\bullet\boldsymbol{q}_V)
]
$$

#### 逆四元数

四元数 \(q\) 的逆记为 \(q^{-1}\)。可以将逆四元数理解为反向操作。

$$q q^{-1} = [0 \quad 0 \quad 0 \quad 1]$$

共轭四元数（conjugate）记为 \(q^{\ast}\)。

$$q^\ast = [-\boldsymbol{q}_V \quad q_S]$$

$$q^{-1} = \frac{q^{\ast}}{|q|^2}$$

由于我们只考虑单位四元数（\(|q| = 1\)），所以：

$$q^{-1} = q ^\ast = [-\boldsymbol{q}_V \quad q_S] \qquad (当\ |q| = 1)$$

这意味着针对单位四元数可以快速求逆。

$$(p q)^\ast = q^\ast p^\ast$$

$$(p q)^{-1} = q^{-1} p^{-1}$$

### 四元数应用于向量旋转

首先将三维向量 \(\boldsymbol{v}\) 转化为四元数形式 \(v\)：

$$v = [\boldsymbol{v} \quad 0]$$

通过单位四元数 \(q\) 对向量 \(\boldsymbol{v}\) 旋转得到 \(v'\)：

$$v' = rotate(q, \boldsymbol{v}) = q v q^{-1} = q v q^\ast$$

最后从结果四元数 \(v\) 中提取结果向量 \(\boldsymbol{v}\)。

多个四元数组合：

$$q_{net} = q_3 q_2 q_1$$

$$
\begin{align}
v' 
& = q_3 q_2 q_1\ v\ q_1^{-1} q_2^{-1} q_3^{-1} \\
& = q_{net}\ v\ q_{net}^{-1}
\end{align}
$$

### 四元数与变换矩阵

四元数可以与旋转变换矩阵（3x3形式）相互转化。

这里的变换矩阵为仅包含3x3旋转部分的矩阵。

设四元数 \(q = [x\ y\ z\ w]\) ，其对应的旋转矩阵为 \(\boldsymbol{R}\)

$$
\boldsymbol{R} =
\begin{bmatrix}
1-2y^2-2z^2 & 2xy+2zw & 2xz-2yw \\
2xy-2zw & 1-2x^2-2z^2 & 2yz+2xw \\
2xz+2yw & 2yz-2xw & 1-2x^2-2y^2
\end{bmatrix}
$$

同样的，给定旋转矩阵 \(\boldsymbol{R}\)，可以找到对应的四元数 \(q\)。
代码引用自 [Gamasutra [Nick Bobic]](https://web.archive.org/web/20170318235841/http://www.gamasutra.com/view/feature/3278/rotating_objects_using_quaternions.php)。

```c {linenos=inline}
void matrixToQuaternion(const float R[3][3], float q[/*4*/]) {

    float trace = R[0][0] + R[1][1] + R[2][2];

    // check the diagonal
    if (trace > 0.0f) {
        float s = sqrt(trace + 1.0f);
        q[3] = s * 0.5f;
        float t = 0.5f / s;
        q[0] = (R[2][1] - R[1][2]) * t;
        q[1] = (R[0][2] - R[2][0]) * t;
        q[2] = (R[1][0] - R[0][1]) * t;
    } else {
        // diagonal is negative
        int i = 0;
        if (R[1][1] > R[0][0]) i = 1;
        if (R[2][2] > R[i][i]) i = 2;
        
        static const int NEXT[3] = {1, 2, 0};
        int j = NEXT[i];
        int k = NEXT[j];
        
        float s = sqrt((R[i][j] - (R[j][j] + R[k][k])) + 1.0f);
        
        q[i] = s * 0.5f;
        
        float t;
        if (s!=0.0) t = 0.5f / s; 
        else t = s;
        
        q[3] = (R[k][j] - R[j][k]) * t;
        q[j] = (R[j][i] + R[i][j]) * t;
        q[k] = (R[k][i] + R[i][k]) * t;
    }
}
```

### 线性插值

常用于动画系统、动作系统、Camera系统。

在 \(q_A\) \(q_B\) 之间进行插值，\(\beta\) 为过程百分比：

$$
\begin{align}
q_{LERP} 
& = LERP(q_A, q_B, \beta) \\
& = \frac{(1-\beta)q_A+\beta q_B}{|(1-\beta)q_A+\beta q_B|}
\end{align}
$$

四元数可以看作是四维超球体上的点，LERP 实际上是沿着超球体的弦进行插值，而非超球体的表面。这导致使用 LERP 时四元数的变化速度（角速度）不固定，而是两端快中间慢。

![Lerp vs Slerp](./images/lerp_vs_slerp.png#center)

相比 LERP，SLERP 是沿着超球体的表面进行插值。

$$SLERP(p, q, \beta) = w_p p + w_q q$$

$$
w_p = \frac{\sin(1-\beta)\theta}{\sin \theta}
\qquad
w_q = \frac{\sin \beta\theta}{\sin \theta}
$$

$$
\begin{align}
\cos\theta & = p\bullet q = p_x q_x + p_y q_y + p_z q_z + p_w q_w \\
\theta &= \cos^{-1}(p \bullet q)
\end{align}
$$

一般来说，SLERP 相对开销较大，但存在可降低开销的优化算法。实际应用中，可以通过对实际代码进行性能分析来确定使用 SLERP 或 LERP。