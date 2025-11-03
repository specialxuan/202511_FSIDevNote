# Navier-Stokes Equations

## 不可压缩流体

### 连续性方程

对于不可压缩流动，连续性方程的微分形式为：

$$\nabla \cdot \bold{U} = 0$$

该方程表示速度场的散度为零，即质量守恒。

### 动量方程

动量方程的微分守恒形式为：

$$\rho\frac{\partial \bold{U}}{\partial t} + \rho\nabla \cdot (\bold{U} \otimes \bold{U}) = -\nabla p + \mu \nabla^2 \bold{U} + \rho\bold{g}$$

其中：

- $\bold{U}$ 是速度矢量
- $t$ 是时间
- $\rho$ 是恒定密度
- $p$ 是压力
- $\mu$ 是恒定粘度
- $\bold{g}$ 是单位质量体积力矢量
- $\otimes$ 表示张量积，流体力学中常省略

方程中各项的物理意义如下：

- $\rho\frac{\partial \bold{U}}{\partial t}$ 是瞬态项
- $\rho\nabla \cdot (\bold{U} \otimes \bold{U})$ 是对流项，表示动量的输运
- $-\nabla p$ 是压力梯度项
- $\mu \nabla^2 \bold{U}$ 是扩散项
- $\rho\bold{g}$ 是体积力项

## 两个展开式

### 对流项的散度恒等式

$$\nabla \cdot (\bold{U} \otimes \bold{U}) = (\nabla \cdot \bold{U}) \bold{U} + (\bold{U} \cdot \nabla) \bold{U}$$

在不可压缩流动的特定情况下，由于连续性方程 $\nabla \cdot \bold{U} = 0$，该式简化为：

$$\nabla \cdot (\bold{U} \otimes \bold{U}) = (\bold{U} \cdot \nabla) \bold{U}$$

这个恒等式在推导和理解纳维-斯托克斯方程时非常重要：

- 它将对流项 $\nabla \cdot (\bold{U} \otimes \bold{U})$（守恒形式）转换为 $(\bold{U} \cdot \nabla) \bold{U}$（非守恒形式）
- 揭示了动量输运中局部变化与空间输运之间的关系
- 在计算流体动力学中，不同形式的数值稳定性和精度特性有所不同

### 矢量拉普拉斯算子展开为梯度和旋度

$$\nabla^2 \bold{U} = \nabla (\nabla \cdot \bold{U}) - \nabla \times (\nabla \times \bold{U})$$

这个重要的恒等式将矢量拉普拉斯算子分解为两项：

- $\nabla (\nabla \cdot \bold{U})$：散度的梯度（一个无旋场）
- $\nabla \times (\nabla \times \bold{U})$：旋度的旋度（一个无散场）

这个展开在流体力学中尤其有用，因为它将粘性项 $\nu \nabla^2 \bold{U}$ 与流体的旋转（旋度）和压缩（散度）特性联系起来。对于不可压缩流动 $\nabla \cdot \bold{U} = 0$，该式简化为：

$$\nabla^2 \bold{U} = - \nabla \times (\nabla \times \bold{U})$$
