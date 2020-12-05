---
title: 算法随记
mathjax: true
---

# 矩阵的转置、翻转和旋转

对于如下方阵 M

$$
M = 
\begin{bmatrix}
x_{0,0} & x_{0,1} & x_{0,2} \\
x_{1,0} & x_{1,1} & x_{1,2} \\
x_{2,0} & x_{2,1} & x_{2,2} 
\end{bmatrix}
$$

**转置 (Transpose)** 很好理解，将 \\( x_{i,j} \rightleftharpoons x_{j,i} \\) 两两交换即可，
如果矩阵非方阵，则在上面基础上，对于没有对应可交换项的元素，直接移过去就好。

**翻转 (Flip)** 分为水平翻转和垂直翻转。以垂直翻转为例，实际上就是将矩阵的行交换，
即 \\( x_{i,\*} \rightleftharpoons x_{j,\*} \\)，其中 \\( j=RowCount(M)-1-i \\)。示例代码如下：

```Javascript
// 矩阵的垂直翻转
// M 矩阵，rows() 为行数，columns() 为列数，swap() 为交换矩阵 M 的元素
for (let i = 0; i < rows(M); i++) {
    for (let j = 0; j < columns(M); j++) {
        swap([i, j], [rows(M)-1-i, j]);
    }
}
```

**旋转 (Rotate)** 则是结合了以上两种操作，逆时针旋转和顺时针旋转仅在操作顺序上有所不同。

* 逆时针旋转：先转置，再垂直翻转 \\( flipv(M^T) \\)。
* 顺时针旋转：先垂直翻转，再转置 \\( (flipv(M))^T \\)。