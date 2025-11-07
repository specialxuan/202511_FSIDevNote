# 神经算子（Neural Operators）的发展综述、核心原理与复杂系统代理建模分析

## I. 绪论：算子学习在科学计算中的崛起

传统的科学计算方法，特别是基于空间离散化的数值求解技术，如有限元法（FEM）和计算流体力学（CFD），在处理复杂、大规模或参数化的物理系统时，面临巨大的计算瓶颈 1。对于参数化偏微分方程（PDEs），每当输入参数（如边界条件、材料属性或几何形状）发生变化时，都需要重新进行耗时的网格划分、离散化和迭代求解过程。这种固有的计算负担限制了实时预测、大规模设计优化以及不确定性量化等应用的发展。

神经算子（Neural Operators, NOs）的出现，标志着深度学习在科学计算领域的一场重要范式转变 1。传统的神经网络（NNs）主要致力于学习有限维欧几里得空间 $\mathbb{R}^n$ 到 $\mathbb{R}^m$ 之间的映射，而神经算子则被设计用于学习**无限维函数空间**之间的映射 $\mathcal{G}: \mathcal{A} \to \mathcal{U}$ 1。这里的 $\mathcal{A}$ 是输入函数空间（如初始条件或参数函数），$\mathcal{U}$ 是解函数空间。通过直接学习PDE的**解算子**，神经算子能够实现对整个PDE族系的快速、高保真度预测，极大地加速了科学与工程仿真过程 1。

## II. 神经算子的理论基石与核心特性

### 2.1 算子通用逼近定理

神经算子，尤其是深度算子网络（DeepONet），其发展在理论上得到了算子通用逼近定理（Universal Approximation Theorem for Operators）的支持 3。这一重要理论指出，一个具有单隐层的神经网络原则上能够准确地逼近任何非线性连续算子 4。研究人员将这一结果扩展到深度网络结构，并以此为基础设计了 DeepONet 架构，该架构的核心目标是学习连续算子或从分散数据流中学习复杂系统 4。

### 2.2 核心特性：网格无关性与离散化收敛性

神经算子与传统神经网络模型最显著的区别在于其对离散化的依赖性。神经算子的核心要求之一是实现**网格无关性**（Mesh-Independence），即其输出函数不应依赖于输入函数的具体离散化方式，并且能够被评估在任意查询坐标 $y$ 上 2。

为了实现这一目标，许多神经算子架构（如傅里叶神经算子 FNO）被构建成与离散化具有一致性（discretization-consistent），从而更适合求解PDE 6。

#### 2.2.1 架构结构与置换不变性

神经算子的一般架构通常遵循“**Lifting-迭代更新-Projection**”的结构 8。

1. **Lifting (编码器):** 将输入数据映射到高维隐层表示 $H_0$。通常由元素级前馈神经网络（Feed-Forward Neural Networks, FNNs）或特定问题定制的卷积模块实现 8。
    
2. **迭代更新 (处理器):** 执行算子学习的核心操作，例如核积分操作或傅里叶变换，以捕捉输入和输出之间的非局部依赖性。
    
3. **Projection (解码器):** 将隐层表示 $H_L$ 映射回所需的输出函数空间 8。
    

对于 Mesh-Independent Neural Operator (MINO) 这样的先进架构，其采用全注意力机制（Transformer的变体）来处理输入数据 8。在这种设计中，观测数据被视为无序集合，天然满足**置换不变性**（Permutation Invariance）2。这种特性对于强化网格无关性至关重要，因为它确保了模型的输出不随输入采样点序列的改变而变化。相比之下，现有的某些神经算子架构的 Lifting 组件可能不具备置换不变性，这限制了其在处理高度不规则或动态离散化时的泛化能力 8。

对算子的学习，意味着模型能够捕捉到解的内在流形结构。这种结构捕获机制，使得神经算子能够作为一种广义的、可微分的降阶模型（Reduced Order Model, ROM）。DeepONet 采用的分解式结构，实质上是在执行一种非线性的、端到端可微分的基函数发现过程，这与传统 ROM 依赖于固定基函数（如 POD 基）形成了鲜明对比，赋予了 NOs 强大的逼近和泛化能力。

## III. DeepONet 架构的机制与物理意义：主干与分支网络的耦合

DeepONet 是最早提出且应用最广泛的神经算子架构之一 4。它将学习到的解算子 $\mathcal{G}$ 分解为两个独立但耦合的神经网络：分支网络（Branch Net）和主干网络（Trunk Net）4。

### 3.1 结构分解与功能定义

#### 3.1.1 分支网络 (Branch Net)

分支网络是一个深度神经网络（DNN），其输入是输入函数 $u(x)$ 在一组固定传感器点 $\{x_1, \ldots, x_m\}$ 上的离散采样值 $u(x_1), \ldots, u(x_m)$ 4。

分支网络的作用是编码这些离散输入，并将其转化为一组低维特征系数 $b = (b_1, \ldots, b_p)$ 12。

#### 3.1.2 主干网络 (Trunk Net)

主干网络是另一个深度神经网络（DNN），其输入是输出解函数需要评估的空间或时间坐标 $y$ 4。

主干网络的作用是将查询点 $y$ 映射为一组空间上的基函数或特征模态 $t = (t_1(y), \ldots, t_p(y))$ 12。

### 3.2 耦合原理与物理意义

DeepONet 最终的算子逼近是通过分支网络和主干网络的输出进行线性组合实现的：

$$\mathcal{G}_\theta(u)(y) = \sum_{k=1}^p b_k(u) t_k(y)$$

12

这种耦合机制具有深刻的物理和数学意义：

#### 3.2.1 非局部性与解耦重组

PDE 的解算子通常是非局部的，意味着输入函数 $u$ 的微小变化可以在整个域上引起解 $G(u)$ 的变化。DeepONet 通过将输入条件（由 $b_k$ 编码）与空间结构（由 $t_k(y)$ 编码）解耦，然后通过求和重新组合，有效地学习了这种非局部映射。

#### 3.2.2 特征模态与激励强度

- **主干网络 $t_k(y)$ 的物理意义：** $t_k(y)$ 扮演了**解空间的基函数**的角色。它们是网络通过学习数据自动发现的最优空间模式（或特征模态），用于跨参数族系表示所有可能的解场 4。这种数据驱动的基函数发现过程，使得 $t_k(y)$ 能够比传统固定基函数（如傅里叶或多项式）更有效地捕捉复杂解的结构。
    
- **分支网络 $b_k(u)$ 的物理意义：** $b_k$ 编码了输入函数 $u$ 对系统解的**激励强度或权重**。它们决定了在特定输入条件下，每一种空间基函数 $t_k(y)$ 在最终解 $G(u)(y)$ 中所占的相对贡献或幅度。因此，分支网络实质上是在识别并量化输入函数对解场结构的影响。
    

通过这种分解与耦合，DeepONet 能够针对一系列参数化的PDE问题，高效地从函数空间 $\mathcal{A}$ 映射到函数空间 $\mathcal{U}$，避免了对每个新参数进行昂贵的重新求解。

## IV. Fourier 神经算子 (FNO) 及其在复杂几何中的变体

DeepONet 并非唯一的神经算子架构。傅里叶神经算子（Fourier Neural Operator, FNO）是另一种主流架构，它在频谱域操作中实现了极高的计算效率。

### 4.1 FNO 的核心原理与优势

FNO 通过利用快速傅里叶变换（FFT）在频域内执行**全局卷积**操作 7。这种方法允许网络高效地捕捉函数中的所有长程依赖性，避免了传统空间卷积的局部性限制。通过将全局卷积算子与非线性激活函数结合，FNO 能够逼近高度非线性和非局部的解算子 7。

FNO 的主要优势在于其**极快的推理速度**和**分辨率不变性**。在流体力学（如Navier-Stokes方程）和达西流等应用中，FNO 相比传统数值求解器，推理速度可以提高 $100\times$ 以上，同时保持高保真度 13。此外，通过结构设计，FNO 学习到的网络参数与输入和输出空间的具体离散化无关 6。虽然傅里叶层本身可能损失高频模式或仅适用于周期边界，但 FNO 整体的编码器-解码器结构和偏差项有助于恢复高阶傅里叶模式和适应非周期边界 14。

### 4.2 FNO 与 DeepONet 的性能比较及挑战

在相对简单的设置中，DeepONet 和 FNO 的性能相似。然而，对于复杂的几何形状和噪声数据，两者表现出明显的差异 10。

|**架构**|**核心原理**|**几何适应性**|**噪声鲁棒性**|**典型优势/适用场景**|
|---|---|---|---|---|
|DeepONet|算子通用逼近定理；Trunk/Branch分解 4|适用于复杂几何，依赖于采样点 10|强 10|复杂几何，噪声数据，系统辨识|
|FNO|基于傅里叶变换的全局卷积操作 7|原始结构要求规则网格 14|差，对高频噪声高度敏感 10|极高推理速度 ($100\times$+) 13，流体力学，高分辨率仿真|

FNO 对噪声的敏感性是其基于谱方法的直接体现。在处理带有 0.1% 噪声的输入数据时，FNO 的误差可能增加 $10000$ 倍，使其不适用于某些关键应用，而 DeepONet 的性能几乎不受影响 10。这种差异表明 DeepONet 的密集网络结构在输入编码过程中对噪声具有更强的非线性滤波能力，使其在真实世界数据质量不佳的场景中更具结构鲁棒性。

### 4.3 应对复杂几何：图神经算子与几何感知架构

由于 FNO 的原始结构需要规则网格，这限制了其在真实世界复杂工程几何上的应用。为了解决这一局限性，研究人员开发了 Graph Neural Operators (GNOs) 和 Geometry-Informed Neural Operator (GINO) 13。

- **GNOs：** 专注于处理在不规则域上定义的函数空间映射，通过图结构来捕捉非结构化数据之间的远程依赖关系 13。
    
- **GINO：** GINO 是结合 GNO 和 FNO 的混合架构，专门针对具有不规则和非网格匹配几何体的大规模 3D PDE 问题 15。GINO 的关键机制是将不规则的输入几何体（如点云）通过 GNO **投影到潜在的规则网格**上 16。随后，在这一规则的隐层空间中，利用 FNO 进行高效的频谱计算，最后再通过另一个 GNO 层将结果投影回原始域的查询点 16。
    

GINO 的这一几何感知策略至关重要。它保留了 FNO 在规则网格上进行计算的巨大速度优势，同时通过 GNO 解决了几何不规则性问题 16。在求解大规模 3D PDE 时，GINO 的推理速度可以比标准数值求解器快 $10^5$ 倍 15。这种能够处理非网格兼容几何体的能力，对于解决复杂的工程问题（如流固耦合）是必不可少的基础。

## V. 神经算子在复杂 PDE 求解中的应用

神经算子作为 PDE 求解的代理模型，已在科学和工程的多个领域展示了其超越传统方法的潜力。

### 5.1 求解参数化 PDE 的前向问题

神经算子最主要的成功应用是作为参数化 PDE 的高性能代理模型 1。它们被应用于模拟多孔介质流、碳封存、湍流、含参热传导方程以及 PDE 约束控制等复杂物理现象 13。NOs 的推理速度是其关键优势，使实时预测和大规模参数探索成为可能。

### 5.2 物理信息融合的神经算子 (PI-NO)

在许多科学计算场景中，获取大量的参数-解样本对（标记数据）是昂贵且困难的 18。为了解决数据稀缺问题并确保模型满足物理定律，研究人员开发了 Physics-Informed Neural Operators (PI-NOs)，如 PI-DeepONet 12。

PI-NO 的核心机制是将物理约束直接整合到模型的损失函数中 12。总损失函数包含数据保真度损失和基于 PDE 残差的损失 $L_{PDE}$ 19。PDE 残差的计算通过**自动微分**(Automatic Differentiation, AD)来实现，网络输出相对于域坐标的偏导数可以被精确计算 19。这种方法允许模型在训练过程中最小化 PDE 偏差，从而在无标记数据或数据稀疏的情况下，求解 PDE 问题 18。PI-DeepONet 结合了算子学习的强大泛化能力与物理约束的准确性，在处理含参 PDE 和逆问题时，展现出比传统方法更高的精度和数据效率 12。

### 5.3 性能与泛化能力对比

神经算子与传统的数值求解器以及基于神经网络的函数拟合器（如 PINN）在求解范式、效率和泛化能力上存在根本区别。

#### 表格二：神经算子与传统求解器的性能与泛化对比

|**指标**|**传统数值求解器 (CFD/FEM)**|**PINN (Physics-Informed NN)**|**神经算子 (NOs)**|
|---|---|---|---|
|**学习目标**|特定实例的离散解|特定实例的函数 $u(x)$ 19|整个PDE族系的算子 $\mathcal{G}$ 1|
|**计算速度 (推理)**|极慢（需重新迭代求解）|慢/中等（依赖于评估点数）|极快 ($100\times$+加速) 13|
|**训练数据需求**|无（基于微分方程）|无（基于残差最小化）19|高（需要大量参数-解样本对）18|
|**参数泛化能力**|仅适用于当前参数|难以泛化到高维参数空间 17|强（零样本泛化到新参数）20|
|**网格依赖性**|强（依赖于网格离散化）|无（无网格方法）|弱（网格无关/离散化不变）5|

分析显示，PINNs 虽然是无网格方法，但在面对高维参数空间和多实例任务时，往往难以收敛，精度不高 17。相比之下，神经算子学习的是函数到函数的映射，在训练完成后，可以对新的、未曾见过的输入参数实现高效的**零样本（zero-shot）预测**，从而在大规模参数空间中展现出无与伦比的泛化优势 20。

## VI. 专题分析：FSI 动态网格的代理建模

### 6.1 流固耦合（FSI）问题的本质与挑战

流固耦合（Fluid-Structure Interaction, FSI）是典型的多物理场系统，涉及流体流动与可变形或弹性结构的相互作用 22。FSI 在心血管建模、水坝稳定性分析和空气动力学等领域至关重要 22。

FSI 问题的计算难点在于其核心挑战：**移动和变形边界** 23。流固界面随着时间不断演化，要求流体域的网格必须动态更新（动态网格/动网格）。传统的 CFD 方法处理动网格计算成本极高，极大地限制了实时或大规模 FSI 仿真的实现。

### 6.2 神经算子作为 FSI 代理模型的潜力

神经算子为加速 FSI 仿真和实现实时应用提供了新的工具 22。已有研究开始探索 DeepONet 和 FNO 变体用于加速波场模拟和预测囊泡动力学 24。

成功的 FSI 代理建模要求神经算子必须能够处理两个核心要素：流体-结构场的耦合动力学，以及**几何形状作为输入函数**的动态变化。GNOs 和 GINO 等几何感知架构的发展，特别是它们处理不规则域和点云的能力 13，为 FSI 建模奠定了基础。通过将几何特征（如符号距离函数 SDF）集成到编码器中，模型理论上可以学习并参数化流体域的复杂几何结构 16。

### 6.3 动态网格代理模型的核心挑战

尽管神经算子在处理静态复杂几何方面取得了进展，但分析表明，现有工作尚未解决带有**移动和可变形边界**的 FSI 问题的根本挑战 23。

造成这一挑战的深层机制在于：

1. **算子依赖于时变域：** FSI 系统中的计算域 $\Omega(t)$ 是时间 $t$ 的函数，这要求模型必须学习一个更复杂的高阶算子 $\mathcal{G}: (\mathcal{A} \times \mathcal{T}) \to \mathcal{U}(\Omega(t))$，其中 $\mathcal{U}$ 是在随时间变化的域上定义的解函数。
    
2. **几何泛化和耦合动力学：** 在 FSI 中，流体解的压力会驱动结构变形，结构变形又会改变流体域的几何（动网格），从而影响流体解。代理模型必须准确复制这种闭环耦合。传统的网格无关性通常假设域结构是大致固定的或仅发生微小变化。然而，FSI 中的剧烈变形要求神经算子在每次时间步长上，对完全**新的、未曾见过的几何体**上的解进行准确预测，这需要模型内生地学习到**几何变形的动力学算子** $\mathcal{M}$ 并将其与流体解算子 $\mathcal{G}$ 耦合。
    
3. **计算要求：** 在 FSI 场景中，模型不仅要预测流场和结构场，还需要实时预测网格变形本身，这要求 NO 架构必须具备对非网格兼容几何的实时、高维编码能力。未来的研究方向需要利用 GINO 等架构，将几何感知和时间演化算子结合起来，以应对这一挑战 16。
    

## VII. 总结、挑战与未来展望

神经算子代表了科学计算领域的一项重大技术飞跃，其核心价值在于能够学习并高效逼近无限维函数空间之间的非线性算子。DeepONet 和 FNO 是该领域的两大支柱：DeepONet 依赖于算子通用逼近定理，提供了一个鲁棒且易于处理复杂几何的分解框架；而 FNO 则利用频谱操作，在规则网格上实现了无与伦比的推理速度。变体架构如 GINO 正在通过几何感知机制，将 FNO 的效率带入不规则 3D 域，极大地拓宽了 NOs 的应用范围。

### 7.1 当前技术挑战与瓶颈

1. **数据依赖性：** 尽管 PI-NOs 缓解了对标记数据的需求，但 NOs 的训练仍然需要大量的参数-解样本对，这在许多高保真度的 CFD 或实验场景中，数据获取成本仍然很高 18。
    
2. **鲁棒性与泛化：** FNO 对输入噪声的高度敏感性限制了其在真实世界或噪声数据环境中的可靠性 10。提高 NOs 在未见过的物理参数或外部分布（OOD）条件下的泛化能力，仍然是关键的研究领域 20。
    
3. **动态网格和 FSI：** 解决涉及移动和变形边界的 FSI 问题，要求 NO 架构不仅要处理几何不规则性，还必须内生地学习几何变形的时间动力学，这仍然是亟待突破的科学难题 23。
    

### 7.2 展望：下一代神经算子研究方向

未来的研究应集中在以下几个方向，以实现 NOs 在更广泛、更复杂工程问题中的应用：

1. **强化物理先验（PI-NOs）：** 除了利用 AD 计算 PDE 残差外，应探索将更深层的物理不变量（如能量守恒、质量守恒）或哈密顿结构整合到损失函数中，以确保模型输出的物理一致性和稳定性 18。
    
2. **几何动力学学习：** 针对 FSI 和动态网格问题，需要开发能够实时编码和预测几何形状随时间演化的新型算子。这可能涉及将隐式神经表示（INRs）或专门的几何表示网络与 NO 架构结合，从而参数化和驱动动网格的变形。
    
3. **可解释性分析：** 对 DeepONet 的主干网络学习到的基函数 $t_k(y)$ 进行深入的数学和物理分析，以了解它们如何捕获解流形的结构，从而增强模型的可信度和科学解释力。



## 参考文献

1. Neural operators - Wikipedia, 访问时间为 十一月 7, 2025， [https://en.wikipedia.org/wiki/Neural_operators](https://en.wikipedia.org/wiki/Neural_operators)
    
2. Mesh-Independent Operator Learning for Partial Differential Equations | TransferLab, 访问时间为 十一月 7, 2025， [https://transferlab.ai/pills/2024/mesh-independent-operator-learning/](https://transferlab.ai/pills/2024/mesh-independent-operator-learning/)
    
3. lululxvi/deeponet - Learning nonlinear operators - GitHub, 访问时间为 十一月 7, 2025， [https://github.com/lululxvi/deeponet](https://github.com/lululxvi/deeponet)
    
4. Learning nonlinear operators via DeepONet based on the universal approximation theorem of operators (Journal Article) | OSTI.GOV, 访问时间为 十一月 7, 2025， [https://www.osti.gov/biblio/2281727](https://www.osti.gov/biblio/2281727)
    
5. A Mathematical Guide to Operator Learning - arXiv, 访问时间为 十一月 7, 2025， [https://arxiv.org/html/2312.14688v1](https://arxiv.org/html/2312.14688v1)
    
6. Fourier Neural Operator with Learned Deformations for PDEs on General Geometries - arXiv, 访问时间为 十一月 7, 2025， [https://arxiv.org/html/2207.05209v2](https://arxiv.org/html/2207.05209v2)
    
7. Fourier Neural Operator with Learned Deformations for PDEs on General Geometries, 访问时间为 十一月 7, 2025， [https://www.jmlr.org/papers/volume24/23-0064/23-0064.pdf](https://www.jmlr.org/papers/volume24/23-0064/23-0064.pdf)
    
8. Mesh-Independent Operator Learning for Partial Differential Equations - OpenReview, 访问时间为 十一月 7, 2025， [https://openreview.net/pdf?id=JUtZG8-2vGp](https://openreview.net/pdf?id=JUtZG8-2vGp)
    
9. Scattering with neural operators | Phys. Rev. D - Physical Review Link Manager, 访问时间为 十一月 7, 2025， [https://link.aps.org/doi/10.1103/PhysRevD.108.L101701](https://link.aps.org/doi/10.1103/PhysRevD.108.L101701)
    
10. [2111.05512] A comprehensive and fair comparison of two neural operators (with practical extensions) based on FAIR data - arXiv, 访问时间为 十一月 7, 2025， [https://arxiv.org/abs/2111.05512](https://arxiv.org/abs/2111.05512)
    
11. What do physics-informed DeepONets learn? Understanding and improving training for scientific computing applications - arXiv, 访问时间为 十一月 7, 2025， [https://arxiv.org/html/2411.18459v1](https://arxiv.org/html/2411.18459v1)
    
12. PI-DeepONet: Physics-Informed Deep Operator Networks - Emergent Mind, 访问时间为 十一月 7, 2025， [https://www.emergentmind.com/topics/physics-informed-deep-operator-network-pi-deeponet](https://www.emergentmind.com/topics/physics-informed-deep-operator-network-pi-deeponet)
    
13. A comprehensive comparison of neural operators for 3D industry-scale engineering designs, 访问时间为 十一月 7, 2025， [https://arxiv.org/html/2510.05995v1](https://arxiv.org/html/2510.05995v1)
    
14. Fourier Neural Operator - Zongyi Li, 访问时间为 十一月 7, 2025， [https://zongyi-li.github.io/blog/2020/fourier-pde/](https://zongyi-li.github.io/blog/2020/fourier-pde/)
    
15. Geometry-Informed Neural Operator for Large-Scale 3D PDEs | Request PDF, 访问时间为 十一月 7, 2025， [https://www.researchgate.net/publication/373642438_Geometry-Informed_Neural_Operator_for_Large-Scale_3D_PDEs](https://www.researchgate.net/publication/373642438_Geometry-Informed_Neural_Operator_for_Large-Scale_3D_PDEs)
    
16. Geometry-Informed Neural Operators: Transforming 3D Fluid Dynamics Simulations, 访问时间为 十一月 7, 2025， [https://dimensionlab.org/blog/geometry-informed-neural-operators-transforming-3d-fluid-dynamics-simulations](https://dimensionlab.org/blog/geometry-informed-neural-operators-transforming-3d-fluid-dynamics-simulations)
    
17. DeepONet在求解含参热传导方程中的应用, 访问时间为 十一月 7, 2025， [https://pdf.hanspub.org/aam_2624580.pdf](https://pdf.hanspub.org/aam_2624580.pdf)
    
18. Separable Physics-Informed DeepONet: Breaking the Curse of Dimensionality in Physics-Informed Machine Learning - arXiv, 访问时间为 十一月 7, 2025， [https://arxiv.org/html/2407.15887v3](https://arxiv.org/html/2407.15887v3)
    
19. Discrete Residual Loss Functions for Training Physics-Informed Neural Networks - The International Conference on Computational Science, 访问时间为 十一月 7, 2025， [https://www.iccs-meeting.org/archive/iccs2025/papers/159070177.pdf](https://www.iccs-meeting.org/archive/iccs2025/papers/159070177.pdf)
    
20. Towards Generalizable PDE Dynamics Forecasting via Physics-Guided Invariant Learning, 访问时间为 十一月 7, 2025， [https://arxiv.org/html/2509.24332v1](https://arxiv.org/html/2509.24332v1)
    
21. Hybrid acceleration techniques for the physics-informed neural networks: a comparative analysis, 访问时间为 十一月 7, 2025， [https://publications.hse.ru/pubs/share/direct/880992661.pdf](https://publications.hse.ru/pubs/share/direct/880992661.pdf)
    
22. Neural operators for simulation of wave propagation and applications in physics and engineering - Institutt for informatikk - Det matematisk-naturvitenskapelige fakultet, 访问时间为 十一月 7, 2025， [https://www.mn.uio.no/ifi/studier/masteroppgaver/dsb/neural-operators-for-simulation-of-wave-propagatio.html](https://www.mn.uio.no/ifi/studier/masteroppgaver/dsb/neural-operators-for-simulation-of-wave-propagatio.html)
    
23. Learning Fluid-Structure Interaction Dynamics with Physics-Informed Neural Networks and Immersed Boundary Methods - arXiv, 访问时间为 十一月 7, 2025， [https://arxiv.org/html/2505.18565v4](https://arxiv.org/html/2505.18565v4)
    

Fourier neural operator based fluid–structure interaction for predicting the vesicle dynamics | Request PDF - ResearchGate, 访问时间为 十一月 7, 2025， [https://www.researchgate.net/publication/379588667_Fourier_neural_operator_based_fluid-structure_interaction_for_predicting_the_vesicle_dynamics](https://www.researchgate.net/publication/379588667_Fourier_neural_operator_based_fluid-structure_interaction_for_predicting_the_vesicle_dynamics)**