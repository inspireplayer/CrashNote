[TOC]

# 十二、三角形

## 1. 内心

三角形内心（三角形内切圆的圆心）：三角形内角平分线的交点

已知三点坐标，求三角形内心坐标，[证明过程](https://www.zybang.com/question/272657890b84080ca669265cd181789c.html)

![](images/incenter.png)

设 $a = |BC|, \space b = |AC|,\space c = |AB|$ 则
$$
(x_I,y_I) = {a(x_A,y_A)+b(x_B,y_B)+c(x_C,y_C) \over a+b+c}
$$



## 2. 求面积

已知三点坐标，求证三角形面积 $S_{\Delta ABC} = {1 \over 2}[(x_2-x_1)(y_3-y_1) - (y_2-y_1)(x_3-x_1)]$

![](images/triangleSquare.png)
$$
\begin{align}
S_{\Delta ABC} 
&= {1 \over 2} |height||L_1| \\
&= {1 \over 2} |sin\theta||L_2||L_1| \\
&= {1 \over 2} \sqrt{1-(cos\theta)^2}|L_2||L_1| \\
&= {1 \over 2} \sqrt{1-({\vec {BA} \cdot \vec {CA} \over |L_2||L_1|})^2}\cdot |L_2||L_1| \\
&= {1 \over 2} \sqrt{(|L_1||L_2|)^2-(\vec {BA} \cdot \vec {CA})^2} \\
&= {1 \over 2} \sqrt{[(x_3 - x_1)^2 + (y_3-y_1)^2][(x_2-x_1)^2 + (y_2-y_1)^2]-[(x_2 - x_1)(x_3-x_1) + (y_2-y1)(y_3-y_1)]^2} \\
\end{align}
$$

设 $a=x_2-x_1, \space b=y_2-y_1, \space c=x_3-x_1, \space d=y_3-y_1$  则 

$$
\begin{align}
S_{\Delta ABC} 
&= {1 \over 2} \sqrt{(c^2 + d^2)(a^2 + b^2)-(ac + bd)^2} \\
&= {1 \over 2}|ad - bd| \\
&= {1 \over 2}|(x_2-x_1)(y_3-y_1) - (y_2-y_1)(x_3-x_1)|
\end{align}
$$

