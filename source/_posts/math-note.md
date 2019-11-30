---
mathjax: true
abbrlink: '0'
title: 数学知识补习
tags:
 - 矩阵
categories:
 - 数学
---
数理基础太差，被关了起来
<img src="https://raw.githubusercontent.com/Archaeoraptor/image_resources/ImageofBlog/found.png" alt="Picture" style="zoom:40%;" />
<!-- more -->
### 矩阵

## 雅可比矩阵

将梯度记为沿各坐标轴偏导的一组坐标,可以表示成向量形式

$\frac{\partial y}{\partial x}=\left[\begin{array}{cccc}
    \frac{\partial y_1}{\partial x} \\ \frac{\partial y_2}{\partial x} \\ ...  \\ \frac{\partial y_n}{\partial x}
\end{array}\right]$

## 奇异值分解（singular value decomposition）

$n \times n$方阵可以分解为$\pmb{A}=\pmb{X}\Lambda\pmb{X}^{-1}$ ， 类似的$n*m$矩阵也能分解$\pmb{A}=\pmb{U}\pmb{D}\pmb{V}^{-1}$

D为$m*n$对角矩阵，对角线元素为矩阵 A 的奇异值。U、V都是正交矩阵，因此可以用SVD可以用于求解矩阵的伪逆：

$$A^+=VD^+U^T$$

$A^+$为伪逆，伪逆$D^+$通过对矩阵 D 的非零元素求倒数后求转
置生成

奇异值分解还能用来降维

## 距离计算

### 欧式距离

略

### 余弦距离

计算向量（矩阵）两两之间的相对距离，再除以绝对值

$$ cos\theta=\frac{|\vec{a}\cdot \vec{b}|}{|\vec{a}|*|\vec{b}|}$$

余弦相似度：
$sim(\theta)=1-cos(\theta)$
