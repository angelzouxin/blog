title: machine learing 学习笔记
date: 2016-08-12 20:21:30
categories:
  - machine learing
tags:
  - 笔记
---

自从开了machine learing的坑以后，很久没有学了，总算暑假有空，开始正式学习orz

<!--more-->

在**course**中选择了stanford的机器学习一课[传送门][link1],从基础开始系统的学习ML的知识。该篇作为笔记，记录学到的知识。

##Week 1
主要介绍了ML的分类：**Supervised Learing(监督学习)**和**Unsupervised Learing(无监督学习)**
区别在于,输入的数据是否有label(标签)，有则为监督学习，无则为无监督学习
具体可以看这[什么是无监督学习？(知乎)][link2]。

线性回归和聚类，分别属于监督学习和无监督学习。比如Google News就使用了聚类算法。

###代价函数$J(\theta)$
在Week 1中，主要讲解的是Linear Regression（线性回归）。并定义了一些术语。
在每个线性回归中，我们都有一个训练集，其中：
m代表了训练样本的数量，$h_\theta(x^{(i)})$即我们的拟合函数在第i个样本时的结果，$y^{(i)}$即第i组样本的实际结果。
接下来，就可以定义J(θ)(cost function)的概念了。
{% blockquote %}
$J(\theta) = \frac 1 {2m} {\sum\_{i=1}^m (h\_\theta(x^{(i)})-y^{(i)})^2}$
{% endblockquote %}
我们可以通过$J(\theta)$这个函数，直观的反应出现在的拟合函数与训练样本的拟合度。


###梯度下降算法

我们可以通过选择合适的算法，让$J(\theta)$达到局部最优，从而让数据更好的拟合。
其中的一个方法为Gradient Descent(梯度下降算法)。
{% blockquote 单一特征变量时,其中$\alpha$为学习效率%}
${\theta_j := \alpha\frac \partial {\partial\theta_j}J(\theta_0,\theta_1)}$ $for$ ( $j = 0$ $and$ $j = 1$)
{% endblockquote %}

**记住，要同时更新$\theta_0$和$\theta_1$**

###线性代数知识

网上有很多，不再做赘述

##Week 2

###安装Octave/MATLAB
安装Octave/MATLAB，具体注意事项看[http://wiki.octave.org/GNU_Octave_Wiki](http://wiki.octave.org/GNU_Octave_Wiki)
与[https://www.coursera.org/learn/machine-learning/supplement/ks2m0/setting-up-your-programming-assignment-environment](https://www.coursera.org/learn/machine-learning/supplement/ks2m0/setting-up-your-programming-assignment-environment)

###多种特征量的线性回归
当有多种特征量时，我们可以用n代表特征的数量，$x^{(i)}$代表第$i$个训练样本集合，$x\_j^{(i)}$代表第$j$个特征向量的第$i$个训练数据。
因此，$h\_\theta(x)$变成了
{% blockquote %}
$h\_\theta(x) = \theta\_0 + \sum\_{i=1}^n(\theta\_ix\_i)$
{% endblockquote %}
同时，我们可以把$x$和$\theta$写成向量的形式，那么$h_\theta(x)$可以表示为
{% blockquote 记得$\theta_0$，总共为n+1项%}
$h\_\theta(x) = \theta^Tx$
{% endblockquote %}
那么我们就可以把多个特征值的梯度下降写成
{% blockquote update $\theta_j$ for $j=0...n$ %}
$\theta\_j := \theta\_j - \alpha\frac 1 m {\sum\_{i=1}^m (h\_\theta(x^{(i)})-y^{(i)})}x\_j^{(i)}$
{% endblockquote %}
在梯度下降算法中，若特征值的范围之间相差很大，那么就会使下降的过程变得很缓慢
因此，我们可以通过 **特征缩放 (feature scaling)** 来加快下降的速度。
{% blockquote 用该值代替$x_i-u_i$ 中的$x_i$ %}
$x\_i\leftarrow \frac {x\_i-\overline{x_i}} {Range(x\_i)}$
{% endblockquote %}
在梯度下降中，我们可以通过不断的调整学习效率$\alpha$的值，来得出最有效的学习效率，比如倍增等。
另外，选择合适的特征向量后，可以使用线性回归拟合非常复杂的函数(详细算法暂时不表)

###正规方程
正规方程(Normal Equation是一种区别于迭代方法的直接解法，可以一次得出最优的$\theta$值，
{% blockquote %}
$\theta = (X^TX)^{-1}X^Ty$
Octave：pinv(x'\*x)\*x'\*y (pinv为逆函数)
{% endblockquote %}
X矩阵的每一行即每一组特征向量的转制，**记得常数值1**
所以一共有n+1列，y即训练样本结果向量（m维）
我们比较正规方程和梯度下降两个算法，可以看出，正规方程有如下优点
{% blockquote %}
不需要设置学习效率$\alpha$
一步求解，不需要多次迭代
{% endblockquote %}
按此来看，正规方程完全优于梯度下降，那么，为何还有梯度下降呢？因为正规方程有一个缺点
{% blockquote %}
矩阵乘法的效率为O($n^3$),当特征值过多时，会导致效率低下
{% endblockquote %}
####对于算法的选择
比较而言，梯度下降算法更稳定，适应性更强
所以，一般若特征量个数在10,000以内,考虑使用正规方程,若超过，则使用梯度下降

###Octave命令笔记
####Octave的一些命令
命令详细内容可以通过help检索(感觉大致和MATLAB差不多)
{% blockquote 文件保存和加载%}
 save load
{% endblockquote %}

{% blockquote 矩阵的操作%}
 ones zeros eye(单位矩阵) magic(行列与斜对角和相同) randn(符合正态分布)
{% endblockquote %}

{% blockquote 图形绘制%}
 plot()(绘制二维函数) plot3(绘制三维函数) hold on(plot直接绘制在上次图中)
        xlabel(备注行) ylabel(备注列)
        legend(在右上角标注函数名称与线条颜色) title(标注表名) figure(为绘制图表标号)
        close(关闭图表) subplot(分割grid,显示不同图像)
        axis(设置坐标范围和间距) clf(清除) imagesc(绘制图像并且用不同颜色表示不同的数字)
        imagesc(),colorbar.colormap gary(生成灰度图)
{% endblockquote %}
记得熟练运用 **,** 操作符；
另外
函数的调用由于和MATLAB几乎一样，直接跳过

####向量化
这节讲到了一个简化码量的trick: 由于各个语言中，一般都有内置的线性函数库，所以在实现一些公式
比如$\sum\_{i=1}^m {\theta\_iX\_i}$时，我们可以通过向量化大大简化实现所需代码量
```cpp
double prediction
 = theta.transpose() * x
```
其中，theta和x为各自数据集的向量
同理，我们也可以用这种方法，来简洁的实现对$\theta$的更新

[link1]:https://www.coursera.org/learn/machine-learning/home/welcome
[link2]:http://www.zhihu.com/question/23194489
