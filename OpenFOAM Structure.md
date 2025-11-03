---
aliases:
  - OpenFOAM笔记
tags:
  - 待完善
  - 开源软件
---
## 一、OpenFOAM 核心架构 (面向对象设计)

OpenFOAM的核心是一个庞大的C++库集合，它采用**面向对象 (Object-Oriented)** 的设计，将CFD所需的数学、物理、网格和数值方法都封装在不同的类中。

### 1. **库与命名空间**

- **命名空间：** 所有的类和函数都定义在 `Foam` 或 `OpenFOAM` 命名空间内。
    
- **核心哲学：** **“一切皆是场 (Field)”** 或 **“一切皆是对象 (Object)”**。例如，速度 $U$ 不仅仅是一个数据数组，它是一个 `volVectorField` 对象，知道自己的边界条件、如何离散、如何求解。

### 2. **基础数据结构 (Utility & Data)**

| **概念**   | **核心类/类型**                            | **作用**                                                  |
| -------- | ------------------------------------- | ------------------------------------------------------- |
| **基础数据** | `scalar`, `vector`, `tensor`, `label` | 浮点数、矢量、张量、整数索引等基础数学类型。                                  |
| **容器**   | `List<T>`, `PtrList<T>`               | 动态数组、指针列表等，用于存储网格信息或场数据。                                |
| **文件读写** | `IOobject`, `dictionary`              | 管理数据的输入/输出。**`dictionary`** 是处理 `controlDict` 或物性文件的核心。 |

### 3. **网格与场 (Mesh & Fields)**

CFD的核心数据：

- **`fvMesh` (有限体积网格):** 存储几何信息，包括单元、面、点、边界等。是所有计算的基础。
    
- **`GeometricField` (场):** 存储物理量的抽象基类。
    
    - **`volScalarField` / `volVectorField`:** 定义在**单元中心 (volume/vol)** 上的标量（如 $p$）或矢量（如 $U$）场。
        
    - **`surfaceScalarField` / `surfaceVectorField`:** 定义在**面上 (surface)** 的场，如面通量 $\phi$。
        

### 4. **有限体积方法 (fv)**

实现离散化和方程求解：

- **`fvSchemes` (离散格式):** 封装了所有的空间和时间离散方法（`grad`, `div`, `laplacian` 等）。
    
- **`fvMatrix` (有限体积矩阵):** 存储离散后的线性方程组 $[A]\phi = b$ 的矩阵结构。
    
- **`fvSolution` (线性求解器):** 定义如何求解 `fvMatrix`，选择迭代求解器（如 PCG）和预条件子（如 DIC）。
    

---

## 二、OpenFOAM 源码目录结构映射

源码是进行二次开发的地图。你的源码通常位于 `$WM_PROJECT_DIR` 环境变量指向的目录下。

|**顶级目录**|**作用**|**耦合开发的重要性**|
|---|---|---|
|**`src`**|**核心库源码**。所有的底层数据结构、场、网格和数值方法实现。|**最高** (理解数据结构和方法)|
|`src/OpenFOAM`|最底层基础类、IO和时间步进。||
|`src/finiteVolume`|`fvMesh`, `fvSchemes`, `fvMatrix` 等有限体积实现。||
|`src/turbulenceModels`|湍流模型，如 $k-\epsilon$。||
|**`applications`**|**所有可执行程序**，包括求解器和工具。|**高** (你的二次开发将基于此)|
|`applications/solvers`|所有的CFD求解器，如 `simpleFoam.C`。||
|`applications/utilities`|所有的后处理和网格生成工具。||
|**`tutorials`**|**官方示例**，学习如何设置算例和调用求解器。|**高** (参考算例配置)|

---

## 三、求解器核心流程解析 (SIMPLE 算法)

所有的求解器（如 `simpleFoam`）都是对核心库的调用。它们的基本结构高度一致，代码通常在 `applications/solvers/.../<SolverName>.C` 中。

|**阶段**|**核心包含文件/代码段**|**作用**|
|---|---|---|
|**初始化**|`#include "createMesh.H"`<br><br>  <br><br>`#include "createFields.H"`|读入网格、时间、物性、初始条件等，创建 $U, p, \phi$ 等场对象。|
|**时间循环**|`while (runTime.loop()) { ... }`|控制时间步进。对于稳态求解器（如 `simpleFoam`），`runTime.loop()` 实际是迭代循环。|
|**动量方程**|`fvVectorMatrix UEqn(...)`|**$U$ 求解。** 构造和求解动量方程，得到一个非耦合的 $U^*$。|
|**压力泊松方程 (PPE)**|`fvScalarMatrix pEqn(...)`<br><br>  <br><br>`pEqn.solve();`|**$p$ 求解。** 构造并求解压力泊松方程，得到压力 $p$。|
|**速度修正**|`U += fvc::grad(p) * rAU;`<br><br>  <br><br>`phi = ...`|使用新压力 $p$ 修正速度 $U$ 和面通量 $\phi$，使速度场满足连续性。|
|**边界条件**|`U.correctBoundaryConditions();`|每次迭代后，更新边界上的场值。|

> **提示：** 在OpenFOAM中，微分算子分为两类：
> 
> - `fvm::...` (Finite Volume Matrix): **隐式 (Implicit)** 离散，用于构造左端项（系数矩阵 $A$）。
>     
> - `fvc::...` (Finite Volume Calculus): **显式 (Explicit)** 离散，用于计算右端项（源项 $b$）。
>     

---

## 四、耦合开发关注点（与 CalculiX）

针对你与 CalculiX 的耦合二次开发，你的主要工作将集中在 **数据交换** 和 **时间步进控制** 上。

1. **数据交换接口 (BCs):**
    
    - 你需要研究 OpenFOAM 的 **`boundaryConditions`** 机制。你很可能需要开发一个新的 **自定义边界条件类**（继承自 `fixedValueFvPatchField` 等）来接收和发送 CalculiX 的数据。
        
    - 关注文件：`src/finiteVolume/fvPatchFields/...`。
        
2. **时间步进控制:**
    
    - 你需要一个统一的时间步进/迭代循环来同步 OpenFOAM 和 CalculiX 的计算。这可能涉及修改求解器主循环 (`simpleFoam.C`) 以在特定迭代或时间步结束时调用数据交换函数。
        
3. **IO 通信:**
    
    - 你可能需要使用外部库（如 MPI, sockets）来实现 OpenFOAM 进程和 CalculiX 进程之间的通信。这部分的代码将是你二次开发中新增的核心模块。
        

---

现在你已经拥有了查看源码的环境和一份系统性的架构指南。

**我想问的是：为了准备耦合项目，你希望我接下来重点帮你梳理 OpenFOAM 中**哪个方面**的源码实现细节？**

1. **边界条件（BCs）类的结构和源码实现？** (这是数据交换的关键)
    
2. **时间步进 (Time) 和 I/O (IOobject) 的实现细节？** (这是流程控制的关键)
    
3. **线性求解器 (fvMatrix / fvSolution) 的工作原理？** (这是数值收敛的关键)