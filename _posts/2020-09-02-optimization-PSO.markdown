---
layout: post
title:  "智能优化算法——PSO"
date:   2020-09-02 02:35:56
category: optimization
---

粒子群算法(Particle Swarm Optimization, PSO)是一种群智能算法，通过反复迭代一组候选解决方案（这里称为粒子），然后根据粒子位置和速度上的简单数学公式，将这些粒子在搜索空间中移动来解决问题。每个粒子的运动都受其局部的最佳已知位置影响，但也被引导向搜索空间中的最佳已知位置，当其他粒子找到更好的位置时，这些位置将更新。PSO收敛速度快，算法简单易行，应用比较广泛，但是存在容易早熟陷入局部最优等问题，近些年有些算法提出改进PSO这方面的缺点，本文不做过多描述。

## 算法
粒子群算法的原理是先给出一组粒子，每个粒子可视为N维搜索空间中的一个搜索个体，粒子的当前位置即为对应优化问题的一个候选解，粒子的飞行过程即为该个体的搜索过程。这些粒子只拥有两个属性：位置$position$和速度$velocity$，通过每次迭代后全局最优位置$gbest$和每个粒子的个体历史最优位置$pbest$来更新位置和速度，直到满足条件停止。

### 初始化
根据给定的粒子规模$m$，初始化每个粒子的位置和速度值，在不设定给定初始值的情况下，采取的方法是在每个维度的给定的范围下随机生成一个初始值，并将这个随机生成的位置记录下来，作为个体历史最优位置$pbest$，通过计算每个粒子的适应度函数（一般情况下即为优化目标函数），选出最优的那个粒子，将其存储为全局最优位置$gbest$。

### 迭代过程
根据每个粒子的当前位置$location_{i}$，个体历史最优位置$pbest_{i}$，全局最优位置$gbest$以及当前速度$velocity_{i}$，更新每个粒子的速度：

$$
velocity_{i} = \omega \times velocity_{i} + c_{1} \times rand \times (pbest_{i} - position_{i}) + c_{2} \times rand \times (gbest - position_{i})
$$

其中，$\omega$为惯性权重；$c_1，c_2$为定义的学习因子或者叫加速常数，是事先定义确保寻优速度和精度的常数；$rand$为(0,1)之间的随机数;i为当前粒子的序号，$i=1,2,3,\cdots,m$;为了让粒子速度不至于过大以致跑过最优点，一般会设置一个最大速度$V_{max}$，若$velocity_{i}>V_{max}$，则$velocity_{i}=V_{max}$。              

现在一般采用动态惯性权重，实际上就是为了让优化算法在初始阶段能够跨度比较大这样收敛更快，而后续阶段能减小跨度实现更精准的寻找最优值。比较常见的是线性递减权值，即：

$$
\omega_{i} = \frac{(\omega_{ini}-\omega{end})(G_{max}-i)}{G_{max}}+\omega_{end}
$$

其中，$\omega_{i}$表示第i步迭代时的值，$\omega_{ini}$代表设定的初始值，$\omega_{end}$代表设定的终止值，$G_{max}$代表最大迭代数。

更新完速度之后，需要更新粒子位置：

$$
location_{i} = location_{i}+velocity_{i}
$$

更新完成之后，需要对每个粒子所对应的适应度函数进行计算，若当前值是个体历史最佳位置，则更新$pbest$，若当前值是全局最佳位置，则更新$gbest$，若达到设定的迭代次数或者满足其他设定的可以终止的条件则终止迭代，否则进入下一次迭代。

## 代码
PSO的实现代码共享在[github](https://github.com/conceptclear/pso_cpu-gpu)上，分别在cpu和gpu上对粒子群算法进行了实现。


## Reference
[1] https://en.wikipedia.org/wiki/Particle_swarm_optimization                                           
[2] https://blog.csdn.net/daaikuaichuan/article/details/81382794