# CFD Solver Scheme

## SIMPLE 算法

SIMPLE（Semi-Implicit Method for Pressure-Linked Equations）算法是一种用于求解不可压缩 Navier-Stokes 方程的压力-速度耦合的迭代方法，主要用于稳态流动计算，也可用于瞬态问题。

### 1.算法初始化

在第$n$时间步，使用上一时间步或初始猜测值 $u_i^n$、$p^n$ 作为当前迭代的初始估计 $u_i^{n+1}$、$p^{n+1}$。

### 2.动量方程预测

在第$m$迭代步，使用上一迭代步的压力 $p^{m-1}$ 求解线性化后的动量方程，得到预测速度 $u_i^*$：

$$A^{m-1} u_i^* = Q_i^{m-1} - G_i(p^{m-1})$$

其中：

- $A$ 是系数矩阵（包含对流、扩散、非稳态项）
- $Q$ 是源项（包含显式处理的项、体力、非稳态项中已知部分等）
- $G_i$ 是压力梯度算子

### 3.压力修正

压力修正方程来源于连续性方程，形式为：

$$D\left( \rho (A_D)^{-1} G(p') \right) = D(\rho u_i^*)$$

其中：

- $D$ 是散度算子
- $A_D$ 是动量方程中对角线系数矩阵

修正压力 $p^m$：

$$p^m = p^{m-1} + \alpha_p p'$$

其中 $\alpha_p$ 是压力修正的松弛因子（$\alpha_p < 1$）。

### 4.速度修正

使用压力修正 $p'$ 修正速度：

$$u_i^{**} = u_i^* - (A_D)^{-1} G_i(p')$$

修正后的速度 $u_i^{**}$ 满足连续性方程（在离散意义上）。

### 5.其他输运方程求解

如能量、湍流、组分等方程，使用更新后的速度和压力。

### 6.物性更新

如为变物性流动，使用当前迭代的变量值更新流体物性（如密度、粘度等）。

### 7.收敛性检查

若未收敛，返回外层迭代循环继续迭代；若收敛，则进入下一时间步或结束计算。

### 算法关键特点

- **忽略非对角项影响（SIMPLE 算法的关键）**：在推导速度修正关系时，忽略了非对角项的影响，这是 SIMPLE 算法的主要简化，也是其收敛较慢的原因
- **压力修正的松弛**：为防止发散，压力修正通常需要松弛（$\alpha_p \approx 0.1 \sim 0.3$）
- **适用于稳态和瞬态**：通过引入非稳态项，也可用于瞬态问题
- **适用于不可压缩流动**：为不可压缩 Navier-Stokes 方程设计
- **可扩展性强**：易于耦合其他输运方程（如能量、湍流等）
- **计算效率**：相较于其他压力-速度耦合方法（如 PISO），SIMPLE 算法计算效率较低，但实现简单，适用于多种流动问题

## 时间步与迭代步的关系

时间步和迭代步的关系是**多层次、嵌套且因问题类型而异**的。以下是关于这一关系的核心论述整理。

### 基本关系：外层与内层

在求解过程中，尤其是在处理**非线性和变量耦合**时，会形成一个**嵌套的循环结构**：

- **时间步循环**：最外层的循环，用于推进物理时间（瞬态问题）或作为一种收敛手段（稳态问题）
- **外层迭代**：在每个时间步内部，用于更新**非线性和变量耦合**（如速度-压力耦合、湍流模型等）的循环
- **内层迭代**：在外层迭代的每一步中，用于求解**单个线性方程组**（如一个动量方程或压力修正方程）的循环

> **文件原文（第 212 页）**：
> "The iterations within one time step, in which the non-linear and coupling terms are updated, are called _outer iterations_ to distinguish them from the _inner iterations_ performed on linear systems with fixed coefficients."
> 在一个时间步内，更新非线性和耦合项的迭代被称为 _外层迭代_，以区别于在系数固定的线性系统上执行的 _内层迭代_。

### 瞬态问题（时间精确模拟）

对于瞬态问题，目标是获得精确的时间演化历程。

- **时间步长 $\Delta t$**：根据**精度要求**选择，需要足够小以捕捉流动的物理变化
- **外层迭代**：在每个时间步内，必须进行多次外层迭代，直到该时间步下的**整个非线性方程组系统**满足一个较窄的容差

**目的**：确保在进入下一个时间步之前，当前时间步 $t_{n+1}$ 的解完全满足离散化的 Navier-Stokes 方程。

> **文件原文（第 212 页）**：
> "If we are computing an unsteady flow and time accuracy is required, iteration must be continued within each time step until the entire system of non-linear equations is satisfied to within a narrow tolerance."

### 稳态问题（寻求稳态解）

对于稳态问题，最终的解与时间历史无关，时间推进只是一种达到稳态的计算策略。

- **时间步长 $\Delta t$**：可以视为一种**收敛加速参数或松弛因子**。通常使用**大时间步长**或**无限时间步长**（即忽略时间导数项）来快速逼近稳态解
- **外层迭代**：
  - **方法一（无限时间步）**：完全去掉非稳态项，直接迭代求解稳态非线性方程。此时没有时间步的概念，只有外层迭代
  - **方法二（时间推进法）**：保留非稳态项，但每个时间步**只进行一次外层迭代**。这种方法对收敛容差的要求可以宽松很多

> **文件原文（第 212 页）**：
> "For steady flows, the tolerance can be much more generous; one can then either take an infinite time step and iterate until the steady non-linear equations are satisfied, or march in time without requiring full satisfaction of the non-linear equations at each time step (in that case, one usually performs only one iteration per time step)."

### 时间步长与松弛因子的等价性

在求解稳态问题时，**时间步长 $\Delta t$** 和**速度的松弛因子 $\alpha_u$** 在代数方程中的作用是等价的。两者都会对离散方程的对角线系数 $A_P$ 做出贡献。

> **文件原文（第 218-219 页）**：
> "There is a strong similarity between the algebraic equations resulting from the use of under-relaxation when solving steady problems and those resulting from applying the implicit Euler scheme to unsteady equations."

两者的等价关系式：

$$\Delta t = \frac{\rho \alpha_u}{- (1 - \alpha_u) \sum A_k}$$

或

$$\alpha_u = \frac{ - \Delta t \sum A_k }{ - \Delta t \sum A_k + \rho }$$

**关键区别**：

- 使用固定的 $\Delta t$ 进行时间推进，等价于在每个网格点上使用了**不同的、变化的** $\alpha_u$
- 使用固定的 $\alpha_u$ 进行松弛迭代，等价于在每个网格点上使用了**不同的、变化的** $\Delta t$

### 总结

这一章将时间步和迭代步的关系描绘成一个灵活的计算框架：

| 问题类型     | 时间步 $\Delta t$ 的角色       | 外层迭代的作用                                                   | 典型策略                                     |
| :----------- | :----------------------------- | :--------------------------------------------------------------- | :------------------------------------------- |
| **瞬态问题** | **物理时间增量**，由精度决定   | 在**每个时间步内**进行多次迭代，以精确满足该时刻的方程           | 小 $\Delta t$ + 多次外层迭代                 |
| **稳态问题** | **数值松弛因子**，用于加速收敛 | 可以**没有时间步**（纯迭代），或**每个时间步一次迭代**以逼近稳态 | 大 $\Delta t$（或无限） + 一次或多次外层迭代 |

这个框架允许根据具体问题的需求（是追求时间精确性，还是快速获得稳态解）来灵活地配置计算流程。
