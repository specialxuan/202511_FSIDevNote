# 深度学习代理模型在流固耦合动网格中的应用综述：聚焦组件化替换

## 第一章. 引言：流固耦合动网格的挑战与DL代理的定位

### 1.1 FSI问题的计算框架与动网格的瓶颈

流固耦合（Fluid-Structure Interaction, FSI）模拟是计算力学中的核心难题之一，广泛应用于航空航天、生物力学和土木工程等领域。在FSI模拟中，计算流体力学（CFD）求解器常采用体贴合（body-conforming）的方法，例如任意拉格朗日-欧拉（Arbitrary Lagrangian-Eulerian, ALE）公式 1。这种方法要求流体网格必须随着结构边界的运动和变形而同步更新。

传统FSI求解通常采用分区（Partitioned）策略，即流体和结构方程分开求解，通过迭代或时间步进行耦合。在这一过程中，流体网格的更新和变形是必不可少的环节。然而，每一次网格更新都需要求解一个或一组网格运动方程，这通常涉及复杂的高维插值或偏微分方程（PDE）的求解 2。对于大规模、瞬态FSI模拟而言，**网格运动技术（Mesh Deformation Technique, MDT）**往往成为主要的计算瓶颈，显著增加了总计算时间 3。

此外，MDT还面临着严峻的网格质量挑战。当结构发生大位移或剧烈变形时，MDT必须保证网格单元的有效性（例如避免负体积）和质量（例如控制纵横比和畸变），否则会导致CFD求解器发散或精度严重下降 3。传统的网格运动方法，如求解拉普拉斯或双调和方程，虽然能提供平滑的位移场，但选择往往依赖于启发式方法，且计算成本随网格规模急剧增加 2。

### 1.2 深度学习代理模型在FSI中的精确角色定位（非端到端）

针对传统网格运动技术的效率和鲁棒性问题，深度学习（DL）代理模型被引入以替代或加速该组件。根据对用户需求的分析，本报告专注于**非端到端预测**的DL代理模型，即DL模型仅替代FSI求解流程中的网格运动模块，而非直接预测整个流场或结构响应。

这种组件替换策略具有战略意义：DL模型作为**网格运动操作算子（Mesh Motion Operator）**的高效替代品，其任务被精确地限定为高维插值或场延拓 4。具体来说，模型学习将结构求解器提供的精确边界位移作为输入（Dirichlet边界条件），映射到流体体积网格中所有节点的位移或新坐标（输出） 5。

将DL模型集成到FSI求解器中作为模块化组件，可以实现显著的局部加速，同时保持整体FSI系统的物理鲁棒性和可信度 6。由于网格边界的位移在物理上是精确已知的，DL模型的任务被简化为几何映射而非复杂的非线性物理预测（如流场或传热），这极大地降低了训练难度、对海量数据的依赖，并提升了模型在不同工况下的可推广性 4。这种方法将DL的快速推理能力与传统CFD/CSD（计算结构动力学）求解器的物理鲁棒性有效结合。

## 第二章. 分类 I: 基于偏微分方程（PDE）的动网格DL代理模型

传统的动网格技术中，求解PDE（如弹性方程或扩散方程）是确保体积网格位移场平滑和连续性的主流方法。DL代理模型主要通过物理信息神经网络（PINN）或操作算子学习（FNO/DeepONet）来加速或替代这些PDE求解过程。

### 2.1 传统PDE方法基础：拉普拉斯、双调和与弹性类比

网格变形技术（MDT）通常基于求解线性或非线性PDE来确定体积位移场。常见的PDE方法包括：

- **拉普拉斯和双调和方程：** 这些方程通过扩散或高阶平滑来传播边界位移。双调和扩展（Biharmonic extension）因其在较大位移下表现出良好的网格质量，而被广泛采用，但其求解涉及的计算量往往很大 2。
    
- **线性弹性类比（Linear Elasticity Analogy, EA）：** 该方法将流体域视为一个具有虚拟弹性属性的介质，通过求解线性弹性方程来获得网格节点的位移场。求解所得的位移场具有良好的物理连续性 2。
    

### 2.2 基于物理信息神经网络（PINN）的PDE代理模型

PINN通过将PDE残差直接嵌入损失函数中，成为替代传统PDE求解器、加速网格变形的有力工具。

#### 2.2.1 PINN架构与输入/输出特征

PINN将网格节点的位移 $\mathbf{u}$ 参数化为一个神经网络 $\hat{\mathbf{u}}(\mathbf{x}; \theta)$。该网络旨在最小化由物理方程（如线性弹性方程）定义的残差损失以及边界条件损失 5。

- **输入特征：** PINN的输入向量 $\mathbf{x}$ 主要是网格节点的空间坐标 $(x, y, z)$。对于涉及时间依赖的瞬态问题，时间 $t$ 也会被纳入输入 5。
    
- **输出变量：** 输出 $\mathbf{u}$ 是该节点相对于其原始位置的位移向量 $^T$，代表了网格的新坐标 5。
    

#### 2.2.2 边界条件（BCs）的硬约束实现

对于移动边界问题，PDE的求解精度高度依赖于边界条件的精确满足。然而，标准的PINN（使用软约束）在满足Dirichlet边界条件时往往存在误差，这对于需要精确边界位置的FSI应用是不可接受的 5。

为解决这一关键挑战，研究人员采用了**硬边界强制（Hard Boundary Condition Enforcement）**方法，通常涉及两步训练 5：

1. **获得特定解：** 首先，使用经典PINN架构（带有软边界条件）训练一个近似模型 $u_{par}(\mathbf{x})$，作为满足边界条件的特定解。
    
2. **残差修正与精确 BCs：** 训练一个新的PINN $\hat{u}$。其最终输出 $\tilde{u}$ 通过一个距离函数 $D(\mathbf{x})$ 进行修正：
    
    $$\tilde{u}(\mathbf{x}; \theta) = u_{par}(\mathbf{x}) + D(\mathbf{x})\hat{u}(\mathbf{x}; \theta)$$
    
    当 $\mathbf{x}$ 位于边界 $\partial\Omega$ 上时，距离函数 $D(\mathbf{x})$ 等于零，从而强制 $\tilde{u}$ 严格等于 $u_{par}$（即精确的边界位移值）。通过这种机制，第二个PINN的训练仅专注于最小化PDE残差损失，同时保证了精确的边界匹配。实验结果表明，这种硬约束方法能够使DL模型的精度达到与传统的有限元（FE）解决方案相当的水平 5。
    

#### 2.2.3 混合PDE-NN方法

除了完全替代PDE求解器，DL模型也可以增强传统的PDE方法。一种混合方法是使用神经网络（NN）来参数化控制网格运动的二阶非线性PDE中的系数 3。这种方法被称为混合PDE-NN方法，它利用神经网络的灵活性来优化网格运动技术，在保证非线性PDE解的存在性的前提下，有助于处理复杂的网格畸变问题 3。

### 2.3 基于操作算子学习（Operator Learning）的PDE代理模型

操作算子学习（如傅里叶神经算子, FNO）旨在学习输入函数空间到输出函数空间的映射，即学习PDE的解算子本身。

- **算子学习的映射：** 在动网格场景中，DL算子模型旨在学习一个映射 $\mathcal{G}: u_{\partial\Omega} \rightarrow u_{\Omega}$，将边界位移（作为输入函数）高效地映射到整个体积网格的位移场（作为输出函数） 4。
    
- **Geo-FNO的应用：** 传统的操作算子（如标准 FNO）通常要求输入域是均匀的，而CFD中使用的网格（尤其是变形后的网格）往往是非结构化的 7。Geo-FNO通过在网络架构中设计一个可学习的变形模块，将不规则的输入域转化为统一的潜在网格，以便利用快速傅里叶变换（FFT）进行高效计算，从而使其适用于网格运动问题 8。
    

## 第三章. 分类 II: 基于径向基函数（RBF）及插值的动网格DL代理模型

径向基函数（RBF）插值是一种强大的网格变形工具，因其网格无关性和节点级精度，在复杂几何和大位移FSI问题中表现出色 9。

### 3.1 传统RBF方法基础与固有挑战

RBF网格变形通过利用表面节点位移作为源点，将插值场传播到体积网格的所有节点 9。RBF方法的优势在于其对网格拓扑的独立性，以及对边界位移的精确复制能力。

然而，RBF方法的固有挑战在于其高昂的计算成本：

- **计算强度与内核选择：** 传统的RBF方法通常需要求解一个稠密的线性系统，计算强度大 11。例如，在最小化网格畸变方面表现优异的双调和样条（Bi-harmonic Spline, BHS）内核，就要求解稠密系统 9。虽然紧支撑的Wendland C2 (WC2) 内核可以产生稀疏系统，计算相对容易，但在平滑度上可能有所欠缺 9。
    
- **数据管理与成本：** 随着表面节点数量的增加，RBF的计算成本（特别是求解线性系统的成本）呈指数级增长，同时带来大规模数据集管理和内存病态（conditioning issues）问题 11。
    

### 3.2 深度学习加速与替代RBF插值策略

DL代理模型旨在通过数据降维、多尺度分析或直接替换插值操作来克服RBF的计算瓶颈。

#### 3.2.1 基于数据降维和多尺度方法的加速

为了解决RBF的 $\mathcal{O}(N^3)$ 缩放问题，DL方法侧重于减少用于插值的源点数量，或优化插值过程：

- **多尺度 RBF（Multiscale RBF）：** 一种新方法通过结合多尺度分析和几何搜索，使用所有表面点构建插值，但避免了传统RBF方法中昂贵的系统求解和病态问题。这种方法将主要的计算成本转化为几何搜索问题，大幅降低了表面网格预处理成本 12。
    
- **动态控制点 RBF（DCP-RBF）：** 在有限体积法（FVM）中，动态控制点RBF技术（DCP-RBF）通过在每个时间步动态更新一组数据缩减的控制点集，来高效更新网格坐标，从而提高了FSI模拟的计算效率 13。
    

在这些加速策略中，DL模型可以用于提高效率。例如，在贪婪算法中选择最优控制点以最小化插值误差是一个计算密集型过程，DL模型（如MLP或GNN）可以被训练来学习最优控制点的映射或特征，从而避免传统方法中昂贵的重复系统求解 12。

#### 3.2.2 DL模型直接替代RBF插值操作

从本质上讲，径向基函数网络（RBFN）本身就是一种人工神经网络，它使用RBF作为激活函数，通常包含三层结构 14。这表明RBF插值可以被视为一种特殊的函数近似任务，为使用更复杂的深度神经网络结构来替代传统RBF核的计算提供了可能性。

在非结构化网格上，图神经网络（GNNs）与RBF的应用场景高度相似。GNNs可以学习节点之间基于距离和位移的关系，高效地将边界位移信息传播到体积域，直接替代传统RBF的插值操作。通过训练GNN，可以实现对高维插值算子的快速推理，从而在保持网格独立性和节点精度的同时，解决RBF方法对计算资源的过度需求。

## 第四章. 动网格DL代理模型的先进架构与特征工程

为了成功替代动网格操作算子，DL模型必须能够处理FSI问题中常见的非结构化、动态变化的网格数据，这推动了图神经网络（GNNs）和先进特征工程的应用。

### 4.1 图神经网络（GNNs）在非结构化网格中的应用

FSI模拟的计算域通常基于非结构化或动态变形的网格，这使得为欧拉网格设计的传统卷积神经网络（CNN）难以直接应用 7。GNNs提供了一种将物理仿真网格转化为图结构的方法，从而有效地处理非均匀数据 15。

- **MeshGraphNet架构：** GNN架构，例如MeshGraphNet，专门用于处理这种非结构化数据。在MeshGraphNet工作流程中，网格顶点或单元中心直接映射为图中的节点 15。
    
    - **节点特征：** 每个节点被赋予一个特征向量，描述其物理状态，包括位置、速度，以及节点类型（如边界节点或内部节点）的一热编码 15。
        
    - **边特征：** 边是根据网格的拓扑连接（例如，三角形或四面体的边）建立的，代表节点间的关系。边特征通常包括连接节点间的位移向量和欧几里得距离 15。
        
- **拓扑驱动的优势：** GNN通过节点和边的特征聚合，能够有效地编码和处理非均匀网格的拓扑信息 7。这种能力使其能够模拟软体和刚体在外部力作用下的动态接触和变形，例如在机器人和生物力学FSI问题中 1。
    

### 4.2 DL模型输入与输出的精确定位

在非端到端的DL代理模型中，关键在于精确定义输入和输出变量，以便将DL模型无缝集成到FSI求解流程中。

- **输入特征的多样性：** 输入不仅包含网格节点的原始空间坐标 $(x, y, z)$，还包括结构求解器提供的精确**边界位移** $\mathbf{u}_{\partial\Omega}$，这些位移充当了DL模型的 Dirichlet 边界条件 5。此外，为了提高模型性能和学习效率，DL模型还利用**静态特征**（如初始位置、面积）和**动态特征**（如 InfoConv 模块聚合的局部信息，甚至包括二面角） 17。
    
- **输出变量：** 代理模型的最终目标输出是体积网格中所有节点的位移向量 $\mathbf{u}_{\Omega} =^T$，或更新后的绝对坐标 $\mathbf{x}'_{\Omega}$ 5。在PINN框架中，输出还可能与距离函数和特定解结合，以确保硬边界条件的满足 5。
    

以下表格总结了基于PDE/算子学习的动网格代理的关键特征定位：

基于PDE/Operator Learning 的动网格代理的输入与输出特征

|**特征类型**|**输入特征 (Input Features)**|**输出变量 (Output Variables)**|**DL模型角色**|
|---|---|---|---|
|几何/空间|空间坐标 $(x, y, z)$|节点位移 $(\Delta x, \Delta y, \Delta z)$|预测位移场 5|
|边界条件|边界位移 $\mathbf{u}_{\partial\Omega}$ 或位移函数|新网格坐标 $\mathbf{x}'_{\Omega}$|施加精确几何约束 5|
|物理状态|节点速度, 节点类型 (GNN)|PDE残差 (PINN)|最小化物理损失 5|
|拓扑/关系|欧氏距离，连接性向量 (GNN Edge Features)|网格质量指标 (如 Jacobian)|保持网格拓扑/质量 2|

## 第五章. 性能评估、鲁棒性约束与挑战

DL代理模型的实际价值在于其在保证网格质量的前提下，带来的计算效率提升和在FSI模拟中的鲁棒性。

### 5.1 计算性能与加速比分析

动网格DL代理模型的核心优势在于其快速推理能力，可替代传统MDT中耗时的迭代求解或密集矩阵运算 18。然而，DL模型的性能评估必须考虑并行计算的背景。

- **传统方法的并行扩展性：** 传统的 PDE 方法，如基于弹性类比（EA）的网格变形方法，虽然在低核心数时计算时间可能长于RBF，但在大规模并行计算环境中表现出优异的可扩展性 19。例如，在一个案例中，EA方法实现了近乎理想的加速比（Speed-up），并行效率持续高于80% 19。相比之下，RBF方法虽然在低核心数下速度较快，但在增加核心数时运行时可能保持不变（例如 $\approx 15$ 秒），表明其并行扩展性受到限制 19。
    
- **DL模型的量化性能：** DL代理模型的推理速度以毫秒计，大幅提高了每次网格更新的效率。然而，DL代理的性能优势需要与其高昂的离线训练成本（数据生成和模型训练）进行权衡 20。为了在工业级应用中超越高度优化的并行EA求解器，DL代理必须证明其在总CPU小时数上的优势，尤其是在需要多次模拟（如优化或长时间瞬态模拟）的场景中。
    

### 5.2 严格的网格质量与非交叉约束

网格质量是动网格技术的生命线。网格单元的严重畸变（Mesh Degeneration），例如负体积或大纵横比，是导致FSI模拟失败的主要原因之一 3。

- **非重叠约束：** 任何物理上不可行的变形，如网格自相交或重叠，都必须严格禁止 21。研究提出，通过设计基于流（Flow-based）的神经网络变形模型，可以在代数上确保变形满足非重叠约束，从而保证物理上的合理性 21。
    
- **质量指标集成：** 为了确保网格质量，DL模型可以整合网格质量度量（如 Jacobian 行列式或单元形状因子）作为损失函数中的正则化项 2。此外，专门的深度学习模型，如用于表面网格质量检测的深度神经网络（DNN）或基于动态图注意力机制的MQENet，可以独立地对网格质量进行预测和评估，辅助MDT的优化 22。
    

### 5.3 泛化能力与模型的鲁棒性挑战

DL代理模型面临的主要挑战之一是其在处理未见或极端工况下的泛化能力。

- **数据依赖与泛化限制：** DL代理的准确性高度依赖于训练数据的覆盖范围。当FSI模拟涉及到训练集范围之外的边界位移或复杂的几何变化时，模型的泛化能力会受到限制 24。
    
- **鲁棒性要求：** 网格运动技术必须在长时间的FSI模拟中保持鲁棒性，以避免位移场中不可逆的累积畸变 2。特别是在复杂的瞬态气动弹性问题中，DL代理必须证明其在预测结构动态响应（如颤振）时的稳定性和物理准确性 26。解决这些挑战需要开发内嵌更强物理信息的DL模型，以提高其在数据稀疏区域的预测可靠性 24。
    

## 第六章. 总结与未来展望

### 6.1 关键发现总结：DL代理模型的优势对比

深度学习代理模型通过替换传统MDT中的 PDE 求解器或 RBF 插值操作，成功实现了FSI模拟流程中的模块化加速。这种非端到端的策略在保证物理准确性的同时，极大地提高了计算效率。

下表总结了目前主流的DL动网格代理方法及其特点：

表格 1：深度学习动网格代理模型方法论对比

|**分类**|**传统基础方法**|**DL 核心架构**|**典型 FSI 应用**|**DL 带来的主要优势**|**关键挑战/约束**|
|---|---|---|---|---|---|
|**PDE-代理**|拉普拉斯, 弹性类比 2|PINN, FNO 4|经典FSI基准, 动翼型 3|强物理约束，保证位移场平滑性；PINN实现PDE加速。|边界硬约束实现复杂 5；对非结构化网格处理需Geo-FNO 8。|
|**RBF-代理**|Bi-harmonic Spline (BHS), WC2 9|GNN, MLP, RBF网络 14|气动弹性，复杂几何变形 10|网格无关性，高精度插值；DL通过降维或多尺度方法加速密集系统求解 12。|传统RBF计算成本高 11；DL模型对训练数据范围依赖性高。|
|**拓扑驱动**|无特定传统方法|GNN (如MeshGraphNet) 15|复杂非结构化网格动态变形|擅长编码和处理非均匀网格拓扑信息；特征聚合能力强 17。|依赖于高质量的图构建和复杂的特征工程 17。|

### 6.2 未来研究方向

未来的研究应集中于增强DL代理模型的鲁棒性、泛化能力以及在高并行环境下的应用：

1. **高维 FSI 中的 GNN 优化：** 迫切需要研究如何优化 GNNs 以高效处理和聚合三维复杂FSI几何（例如风力涡轮机叶片 27）和瞬态问题中的动态特征 15。特别是要确保在复杂、大规模非结构化网格上的推理效率。
    
2. **鲁棒性与泛化能力提升：** 必须开发能够内嵌严格几何约束（如代数上满足非重叠约束 21）和网格质量正则化项的DL模型 25。这将提高模型在处理训练范围外极端变形时的物理合理性和稳定性，防止不可逆的网格畸变累积 2。
    
3. **在线学习与迁移策略：** 为了提高DL代理模型在工业应用中的灵活性，应探索利用迁移学习（Transfer Learning）或在线学习策略，允许对已训练的模型进行在线微调，以快速适应新的FSI工况或几何微小变化 24。
    
4. **混合模型集成：** 结合不同DL架构的优势是提高性能的有效途径。例如，将基于PDE约束的PINN/FNO与擅长处理非结构化拓扑信息的GNNs相结合，可以利用PINN保证解的物理合理性，同时利用GNN高效处理数据和拓扑信息，最终实现更高效、更精确的网格运动操作算子 3。





## 参考文献

1. Learning Fluid-Structure Interaction Dynamics with Physics-Informed Neural Networks and Immersed Boundary Methods - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/html/2505.18565v4](https://arxiv.org/html/2505.18565v4)
    
2. [2006.14051] Mesh deformation techniques in fluid-structure interaction: robustness, accumulated distortion and computational efficiency - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/abs/2006.14051](https://arxiv.org/abs/2006.14051)
    
3. arXiv:2206.02217v2 [math.NA] 3 Sep 2023 - SciSpace, 访问时间为 十一月 6, 2025， [https://scispace.com/pdf/learning-a-mesh-motion-technique-with-application-to-fluid-1pi7riwy.pdf](https://scispace.com/pdf/learning-a-mesh-motion-technique-with-application-to-fluid-1pi7riwy.pdf)
    
4. Mesh motion in fluid-structure interaction with deep operator networks - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/html/2402.00774v1](https://arxiv.org/html/2402.00774v1)
    
5. Physics-Informed Neural Networks for Mesh Deformation with Exact ..., 访问时间为 十一月 6, 2025， [https://arxiv.org/pdf/2301.05926](https://arxiv.org/pdf/2301.05926)
    
6. Neural Network-Based Surrogate Models Applied to Fluid-Structure Interaction Problems - Scipedia, 访问时间为 十一月 6, 2025， [https://www.scipedia.com/wd/images/3/3e/Draft_Content_224647564-3387_paper-3482-document.pdf](https://www.scipedia.com/wd/images/3/3e/Draft_Content_224647564-3387_paper-3482-document.pdf)
    
7. Data-Efficient Time-Dependent PDE Surrogates: Graph Neural Simulators vs Neural Operators - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/html/2509.06154v1](https://arxiv.org/html/2509.06154v1)
    
8. Fourier Neural Operator with Learned Deformations for PDEs on General Geometries - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/html/2207.05209v2](https://arxiv.org/html/2207.05209v2)
    
9. Radial Basis Functions Mesh Morphing: A Comparison Between the Bi-harmonic Spline and the Wendland C2 Radial Function - PMC - PubMed Central, 访问时间为 十一月 6, 2025， [https://pmc.ncbi.nlm.nih.gov/articles/PMC7304786/](https://pmc.ncbi.nlm.nih.gov/articles/PMC7304786/)
    
10. Radial Basis Functions Vector Fields Interpolation for Complex Fluid Structure Interaction Problems - MDPI, 访问时间为 十一月 6, 2025， [https://www.mdpi.com/2311-5521/6/9/314](https://www.mdpi.com/2311-5521/6/9/314)
    
11. Radial Basis Functions Mesh Morphing - The International Conference on Computational Science, 访问时间为 十一月 6, 2025， [https://www.iccs-meeting.org/archive/iccs2020/papers/121420282.pdf](https://www.iccs-meeting.org/archive/iccs2020/papers/121420282.pdf)
    
12. Efficient and exact mesh deformation using multiscale RBF interpolation - ResearchGate, 访问时间为 十一月 6, 2025， [https://www.researchgate.net/publication/317271599_Efficient_and_exact_mesh_deformation_using_multiscale_RBF_interpolation](https://www.researchgate.net/publication/317271599_Efficient_and_exact_mesh_deformation_using_multiscale_RBF_interpolation)
    
13. Radial basis function mesh deformation based on dynamic control points - ResearchGate, 访问时间为 十一月 6, 2025， [https://www.researchgate.net/publication/313228263_Radial_basis_function_mesh_deformation_based_on_dynamic_control_points](https://www.researchgate.net/publication/313228263_Radial_basis_function_mesh_deformation_based_on_dynamic_control_points)
    
14. Radial basis function network - Wikipedia, 访问时间为 十一月 6, 2025， [https://en.wikipedia.org/wiki/Radial_basis_function_network](https://en.wikipedia.org/wiki/Radial_basis_function_network)
    
15. MeshGraphNet: A Practical User Tutorial — NVIDIA PhysicsNeMo Framework, 访问时间为 十一月 6, 2025， [https://docs.nvidia.com/physicsnemo/latest/user-guide/model_architecture/meshgraphnet.html](https://docs.nvidia.com/physicsnemo/latest/user-guide/model_architecture/meshgraphnet.html)
    
16. Physics-Encoded Graph Neural Networks for Deformation Prediction under Contact - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/html/2402.03466v1](https://arxiv.org/html/2402.03466v1)
    
17. InfoGNN: End-to-end deep learning on mesh via graph neural networks - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/html/2503.02414v1](https://arxiv.org/html/2503.02414v1)
    
18. Mesh deep Q network: A deep reinforcement learning framework for improving meshes in computational fluid dynamics - AIP Publishing, 访问时间为 十一月 6, 2025， [https://pubs.aip.org/aip/adv/article/13/1/015026/2871176/Mesh-deep-Q-network-A-deep-reinforcement-learning](https://pubs.aip.org/aip/adv/article/13/1/015026/2871176/Mesh-deep-Q-network-A-deep-reinforcement-learning)
    
19. ACCELERATING THE FLOWSIMULATOR: IMPROVEMENTS IN FSI SIMULATIONS FOR THE HPC EXPLOITATION AT INDUSTRIAL LEVEL, 访问时间为 十一月 6, 2025， [https://elib.dlr.de/205254/1/2023_Cristofaro_Coupled.pdf](https://elib.dlr.de/205254/1/2023_Cristofaro_Coupled.pdf)
    
20. Fast Surrogate Modeling using Dimensionality Reduction in Model Inputs and Field Output: Application to Additive Manufacturing, 访问时间为 十一月 6, 2025， [https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=928052](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=928052)
    
21. ShapeFlow: Learnable Deformations Among 3D Shapes, 访问时间为 十一月 6, 2025， [https://proceedings.neurips.cc/paper/2020/file/6f1d0705c91c2145201df18a1a0c7345-Paper.pdf](https://proceedings.neurips.cc/paper/2020/file/6f1d0705c91c2145201df18a1a0c7345-Paper.pdf)
    
22. A Method of Detecting Surface Mesh Quality Based on Deep Learning - ResearchGate, 访问时间为 十一月 6, 2025， [https://www.researchgate.net/publication/355124760_A_Method_of_Detecting_Surface_Mesh_Quality_Based_on_Deep_Learning](https://www.researchgate.net/publication/355124760_A_Method_of_Detecting_Surface_Mesh_Quality_Based_on_Deep_Learning)
    
23. MQENet: A Mesh Quality Evaluation Neural Network Based on Dynamic Graph Attention, 访问时间为 十一月 6, 2025， [https://www.researchgate.net/publication/373685871_MQENet_A_Mesh_Quality_Evaluation_Neural_Network_Based_on_Dynamic_Graph_Attention](https://www.researchgate.net/publication/373685871_MQENet_A_Mesh_Quality_Evaluation_Neural_Network_Based_on_Dynamic_Graph_Attention)
    
24. Rapid CFD Prediction Based on Machine Learning Surrogate Model in Built Environment: A Review - MDPI, 访问时间为 十一月 6, 2025， [https://www.mdpi.com/2311-5521/10/8/193](https://www.mdpi.com/2311-5521/10/8/193)
    
25. (PDF) A finite volume unstructured mesh approach to dynamic fluid-structure interaction: An assessment of the challenge of predicting the onset of flutter - ResearchGate, 访问时间为 十一月 6, 2025， [https://www.researchgate.net/publication/223027586_A_finite_volume_unstructured_mesh_approach_to_dynamic_fluid-structure_interaction_An_assessment_of_the_challenge_of_predicting_the_onset_of_flutter](https://www.researchgate.net/publication/223027586_A_finite_volume_unstructured_mesh_approach_to_dynamic_fluid-structure_interaction_An_assessment_of_the_challenge_of_predicting_the_onset_of_flutter)
    
26. Deep Learning-Based Surrogate Model for Flight Load Analysis - Tech Science Press, 访问时间为 十一月 6, 2025， [https://www.techscience.com/CMES/v128n2/43929/html](https://www.techscience.com/CMES/v128n2/43929/html)
    
27. Fluid–Structure Interaction Simulations of Wind Turbine Blades with Pointed Tips, 访问时间为 十一月 6, 2025， [https://www.researchgate.net/publication/378513484_Fluid-Structure_Interaction_Simulations_of_Wind_Turbine_Blades_with_Pointed_Tips](https://www.researchgate.net/publication/378513484_Fluid-Structure_Interaction_Simulations_of_Wind_Turbine_Blades_with_Pointed_Tips)
    
28. High Fidelity 2-Way Dynamic Fluid-Structure-Interaction (FSI) Simulation of Wind Turbines Based on Arbitrary Hybrid Turbulence Model (AHTM) - MDPI, 访问时间为 十一月 6, 2025， [https://www.mdpi.com/1996-1073/18/16/4401](https://www.mdpi.com/1996-1073/18/16/4401)
    
29. [2202.02397] Textured Mesh Quality Assessment: Large-Scale Dataset and Deep Learning-based Quality Metric - arXiv, 访问时间为 十一月 6, 2025， [https://arxiv.org/abs/2202.02397](https://arxiv.org/abs/2202.02397)