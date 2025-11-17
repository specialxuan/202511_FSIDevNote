# FSI 求解系统搭建与开发清单：OpenFOAM + preCICE + CalculiX

## 阶段一：核心组件与依赖规划

### 1.1 技术选型与版本确认

- [x] **确定操作系统版本**： `Ubuntu 24.04 LTS` (Noble Numbat)
- [x] **确定组件版本**：
  - [x] OpenFOAM: `openfoam2412` (ESi 版本，最新稳定版)
  - [x] preCICE: `v3.2.0` (稳定版)
  - [x] CalculiX: `2.20`
  - [x] 编译器: `gcc-13` (Ubuntu 24.04 默认)
  - [x] CMake: `3.28+` (Ubuntu 24.04 自带)
- [x] **选择基础 Docker 镜像**： `ubuntu:24.04` (Noble Numbat)

## 阶段二：核心组件安装

### 2.1 系统级依赖安装

- [x] **基础开发工具**：
  - [x] `build-essential`, `cmake`, `git`, `wget`, `curl`，`pkg-config`
  - [x] `gcc-13`, `g++-13`, `gfortran-13`
  - [x] `python3-dev`, `python3-pip`, `python3-venv`
- [x] **数学与并行库**：
  - [x] `openmpi-bin`,`libarpack2-dev`,`libspooles-dev`,`libyaml-cpp-dev`

### 2.2 OpenFOAM 2412 安装

- [x] **使用 apt 安装 OpenFOAM-v2412：**

```bash
curl -s https://dl.openfoam.com/add-debian-repo.sh | sudo bash
wget -q -O - https://dl.openfoam.com/add-debian-repo.sh | sudo bash
sudo apt-get install ca-certificates
sudo apt-get update
sudo apt-get install openfoam2412-default
```

- [x] **使用 apt 安装 OpenFOAM-v13：**
  - 用于安装 paraview，不参与 preCICE 耦合计算

```bash
sudo sh -c "wget -O - https://dl.openfoam.org/gpg.key > /etc/apt/trusted.gpg.d/openfoam.asc"
sudo add-apt-repository http://dl.openfoam.org/ubuntu
sudo apt update
sudo apt -y install openfoam13
```

- [x] **配置 OpenFOAM 环境选择器：**
  - [x] 在 `~/.bashrc` 中添加：
    - 命令 of13 和 of2412 可以激活对应环境，cleanup_openfoam 可以清除环境
    - of2412 使用 of13 的 paraFoam 路径

```bash
# OpenFOAM Version Management
# ===========================
# OpenFOAM v2412 (ESI Official)
of2412() {
    # 清除可能存在的其他版本环境
    if [ -n "$FOAM_API" ]; then
        echo "Cleaning previous OpenFOAM environment..."
        unset FOAM_API
        cleanup_openfoam
    fi

    # 加载 v2412
    source /usr/lib/openfoam/openfoam2412/etc/bashrc
    # 设置 ParaView 插件路径
    export PV_PLUGIN_PATH="/opt/openfoam13/platforms/linux64GccDPInt32Opt/lib/paraview-5.11"
    export PS1="\[\033[01;32m\](OF2412)\[\033[00m\] \u@\h:\w\$ "
    echo "OpenFOAM v2412 (ESI) activated"
}

# OpenFOAM 13 (Foundation)
of13() {
    # 清除可能存在的其他版本环境
    if [ -n "$FOAM_API" ]; then
        echo "Cleaning previous OpenFOAM environment..."
        unset FOAM_API
        cleanup_openfoam
    fi

    # 加载 13
    source /opt/openfoam13/etc/bashrc
    export PS1="\[\033[01;34m\](OF13)\[\033[00m\] \u@\h:\w\$ "
    echo "OpenFOAM 13 (Foundation) activated"
}

# 清理函数
cleanup_openfoam() {
    # 移除常见的OpenFOAM环境变量
    unset WM_PROJECT WM_PROJECT_VERSION WM_PROJECT_DIR
    unset WM_THIRD_PARTY_DIR FOAM_INST_DIR FOAM_RUN
    # 重置PATH和LD_LIBRARY_PATH（简化版）
    export PATH=$(echo $PATH | sed 's|:/opt/openfoam[^:]*/platforms/[^:]*/bin||g')
    export LD_LIBRARY_PATH=$(echo $LD_LIBRARY_PATH | sed 's|:/opt/openfoam[^:]*/platforms/[^:]*/lib||g')
    # 清理PS1提示符格式 - 恢复默认
    export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
}
```

### 2.3 preCICE 安装

- [x] **安装 preCICE：**

```bash
cd ~
wget https://github.com/precice/precice/releases/download/v3.2.0/libprecice3_3.2.0_noble.deb
sudo apt install ./libprecice3_3.2.0_noble.deb
```

- [x] **安装 OpenFOAM-preCICE adapter:**

```bash
cd ~
wget https://github.com/precice/openfoam-adapter/archive/refs/tags/v1.3.1.tar.gz
tar -xzf v1.3.1.tar.gz
cd openfoam-adapter-1.3.1/
./Allwmake
cd ..
```

- [x] **配置单元测试：**

![[QuickStart_preCICE.png]]

```bash
cd ~
# 下载preCICE教程算例
wget https://github.com/precice/tutorials/archive/refs/tags/v202404.0.tar.gz
tar -xzf v202404.0.tar.gz
cd ~/tutorials-202404.0/quickstart/solid-cpp
cmake . && make
```

- [x] **运行单元测试：**

```bash
# Terminal window 1
cd ~/tutorials-202404.0/quickstart/solid-cpp
./run.sh

# Terminal window 2
of2412
cd ~/tutorials-202404.0/quickstart/fluid-openfoam
./run.sh
# Alternative, in parallel: ./run.sh -parallel
```

### 2.4 CalculiX 安装与配置

- [x] **下载 CalculiX 源码：**

```bash
cd ~
wget http://www.dhondt.de/ccx_2.20.src.tar.bz2
tar xvjf ccx_2.20.src.tar.bz2
```

- [x] **下载适配器源码：**

```bash
cd ~
wget https://github.com/precice/calculix-adapter/archive/refs/heads/master.tar.gz
tar -xzf master.tar.gz
cd calculix-adapter-master
```

- [x] **修改`Makefile`：**

```make
# 删除
FFLAGS = -Wall -O3 -fopenmp $(INCLUDES)
# 添加
FFLAGS = -Wall -O3 -fopenmp -fallow-argument-mismatch $(INCLUDES)
```

- [x] **编译`ccx_preCICE`**

```bash
make clean
make
#移动ccx到系统路径下
sudo cp ./bin/ccx_preCICE /usr/bin/
```

## 阶段三：测试用例搭建

### 3.1 准备 FSI 测试案例

- [x] **选择标准测试案例(Perpendicular flap)：**
  - [x] OpenFOAM 端：二维管流
  - [x] CalculiX 端：弹性体变形

```bash
cd ~/tutorials-202404.0/perpendicular-flap
```

![[QuickStart_FSI.png]]

### 3.2 案例文件结构

- [x] **组织案例目录：**

```bash
./
├── clean-tutorial.sh -> ../tools/clean-tutorial-base.sh
├── fluid-openfoam
│   ├── 0
│   ├── clean.sh
│   ├── constant
│   ├── run.sh
│   └── system
├── metadata.yaml
├── plot-all-displacements.sh
├── plot-displacement.sh
├── precise-config.xml
├── README.md
├── reference-results
├── reference_results.metadata
└── solid-calculix
    ├── all.msh
    ├── clean.sh
    ├── config.yml
    ├── fix1_beam.nam
    ├── flap.inp
    ├── flap_modal.inp
    ├── frequency.inp
    ├── interface_beam.nam
    └── run.sh
```

| **文件/目录**                    | **类型**     | **解释**                                                                                                                                                              |
| -------------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`clean-tutorial.sh`**          | 脚本链接     | 用于**清理整个算例目录**的脚本链接。它会调用 preCICE 教程工具目录中的基础清理脚本，移除所有求解器生成的中间文件和结果文件。                                           |
| **`fluid-openfoam`**             | 目录         | **流体（OpenFOAM）参与者** 的所有文件和配置。OpenFOAM 求解器的标准目录结构（`0`, `constant`, `system`）位于其下。                                                     |
| **`metadata.yaml`**              | 配置文件     | 包含算例的元数据信息，如作者、版本、许可证等。供 preCICE 官方教程管理使用。                                                                                           |
| **`plot-all-displacements.sh`**  | Bash 脚本    | **后处理脚本。** 用于绘制和比较不同求解器组合下的挡板尖端位移结果（例如，OpenFOAM+CalculiX vs. OpenFOAM+FEniCS）。                                                    |
| **`plot-displacement.sh`**       | Bash 脚本    | **后处理脚本。** 用于绘制选定固体参与者（如 `solid-calculix`）的挡板尖端位移（`watchpoint` 监测点）随时间变化的曲线图。                                               |
| **`precise-config.xml`**         | XML 配置文件 | **preCICE 耦合的核心配置文件。** 定义了流体和固体参与者、它们之间的数据交换（力、位移）、接口（耦合面）、时间步长、以及耦合方案（例如，显式或隐式、数据映射方式等）。 |
| **`README.md`**                  | 文本文件     | 包含算例的**基本介绍、设置、配置和运行说明**。这是您现在阅读的指南文档。                                                                                              |
| **`reference-results`**          | 目录         | 包含该算例的**参考结果**文件（通常是 `watchpoint` 日志或绘图数据），用于用户验证自己的模拟结果是否正确。                                                              |
| **`reference_results.metadata`** | 配置文件     | 描述参考结果文件的元数据。                                                                                                                                            |
| **`solid-calculix`**             | 目录         | **固体（CalculiX）参与者** 的所有文件和配置。                                                                                                                         |

- [x] **运行算例**

```bash
# Terminal window 1
of2412
cd tutorials/perpendicular-flap/fluid-openfoam
./run.sh
# Alternative, in parallel: ./run.sh -parallel

# Terminal window 2
cd tutorials/perpendicular-flap/solid-calculix
./run.sh
```

## 阶段四：Docker 环境构建

### 4.1.1 基础 Docker 镜像准备

- [x] **安装 Docker（Linux）或者 Docker Desktop（Windows/Mac）:**
  - 配置镜像源（镜像源可能失效，请根据实际情况调整）：

```json
{
  "registry-mirrors": ["https://docker.xuanyuan.me", "https://docker.1ms.run"]
}
```

- [x] **拉取基础镜像：**

```bash
docker pull ubuntu:24.04
```

- [x] **按照阶段二到阶段三的步骤，安装所有依赖和组件，验证测试案例可运行**
  - 可以不安装`OpenFOAM13`，也就是不安装`ParaView`
- [x] **创建 Docker 镜像：**

```bash
docker commit <container_id> fsi-dev:ubuntu24.04-of2412
```

- [x] **导出 Docker 镜像为 tar 文件（可选）：**

```bash
docker save -o fsi-dev-ubuntu24.04-of2412.tar fsi-dev:ubuntu24.04-of2412
```

### 4.1.2 下载配置好的 Docker 镜像

```bash
docker pull crpi-aa1htdbuqv3dwyf0.cn-beijing.personal.cr.aliyuncs.com/fsi-dev/fsi-openfoam2412-precise-calculix:v1.0
```

### 4.2 配置 VS Code Dev Container

- [ ] **创建 `.devcontainer/devcontainer.json`**：
  - [ ] 定义多阶段构建或服务组合
  - [ ] 配置 VS Code 扩展 (C/C++, CMake Tools, Python)
  - [ ] 设置挂载卷和端口映射
- [ ] **创建 `.devcontainer/Dockerfile`**：
  - [ ] 基于 `ubuntu:24.04`
  - [ ] 安装系统依赖 (build-essential, cmake, git, wget)
  - [ ] 设置非 root 用户
- [ ] **创建 `.devcontainer/docker-compose.yml`** (可选)：
  - [ ] 定义 OpenFOAM 服务
  - [ ] 定义测试用例服务

## 阶段五：VS Code + wmake 二次开发环境配置

### 5.1 项目源码结构规划

- [ ] **创建符合 OpenFOAM 风格的用户源码目录**：
  - [ ] 在 `$FOAM_USER_SRC/` 下建立清晰模块化结构，例如：

    ```bash
    $FOAM_USER_SRC/
    ├── fsi-solvers/          # 自定义 FSI 求解器
    ├── boundaryConditions/   # 自定义边界条件（如流固耦合接口）
    ├── dynamicMesh/          # 自定义动网格方法
    └── utilities/            # 辅助工具（如 preCICE 数据检查器）
    ```

  - [ ] 每个子模块内部包含标准 `.C` / `.H` 文件及 `Make/` 子目录
- [ ] **明确各模块输出类型**：
  - [ ] 求解器 → 可执行程序（`EXE`）
  - [ ] 边界条件/模型/动网格 → 动态库（`LIB`）

### 5.2 wmake 构建系统配置

- [ ] **为每个开发模块编写 `Make/files`**：
  - [ ] 列出所有 `.C` 源文件
  - [ ] 正确指定输出目标：
    - 可执行程序：`EXE = $(FOAM_USER_APPBIN)/<solver-name>`
    - 动态库：`LIB = $(FOAM_USER_LIBBIN)/lib<ModuleName>`
- [ ] **为每个模块编写 `Make/options`**：
  - [ ] 添加必要的头文件路径（`-I$(LIB_SRC)/.../lnInclude`）
    - 如 `finiteVolume`, `dynamicMesh`, `meshTools`, `preciseAdapter` 等
  - [ ] 链接所需 OpenFOAM 库（`-lfiniteVolume -ldynamicMesh ...`）
  - [ ] 若使用 preCICE adapter，确保包含其头文件与库路径

### 5.3 VS Code 工作区设置

- [ ] **配置 `.vscode/settings.json`**：
  - [ ] 启用 C++ 智能感知
  - [ ] 设置默认 C++ 标准为 C++17（与 OpenFOAM 2412 一致）
  - [ ] 指定编译器为 `g++-13`
  - [ ] 启用 `compile_commands.json` 支持以提升代码导航
- [ ] **生成 `compile_commands.json`**：
  - [ ] 在每个模块目录下运行：

    ```bash
    bear -- wmake
    ```

  - [ ] 或使用 `wmake` 插件自动导出（若可用）
- [ ] **配置头文件路径（备用方案）**：
  - [ ] 若无法生成 `compile_commands.json`，在 `c_cpp_properties.json` 中手动添加常用 `lnInclude` 路径

### 5.4 调试环境配置

- [ ] **配置 `.vscode/launch.json`**：
  - [ ] 添加 GDB 调试配置，用于调试自定义求解器
  - [ ] 设置工作目录为测试案例路径（如 `~/tutorials/perpendicular-flap/fluid-openfoam`）
  - [ ] 传递必要参数（如 `-case .`）
  - [ ] 注入 OpenFOAM 环境变量（可通过 `"env"` 字段或启动脚本）
    - 推荐方式：在 `launch.json` 中调用 `of2412` 环境加载脚本
- [ ] **验证调试能力**：
  - [ ] 能在 `.C` 文件中设置断点
  - [ ] 能查看场变量（如 `U`, `p`）、网格信息、preCICE 交换数据等

### 5.5 开发环境验证

- [ ] **执行端到端构建测试**：
  - [ ] 在 `of2412` 环境下，进入任一模块目录执行 `wmake`
  - [ ] 验证可执行文件或动态库成功生成至 `$FOAM_USER_APPBIN` / `$FOAM_USER_LIBBIN`
- [ ] **集成测试验证**：
  - [ ] 在测试案例中加载自定义库（`libs ("libMyBCs.so");`）
  - [ ] 运行案例，确认新组件被正确识别和调用
  - [ ] 更新 `scripts/verify_installation.sh`，加入 `wmake` 编译与基本运行测试
  - [ ] 确保验证脚本可在 Dev Container 或本地一致运行

## 阶段六：开发工作流建立

### 6.1 容器内开发流程

- [ ] **启动开发环境**：
  - [ ] `code fsi-dev-container/`
  - [ ] 在容器中重新打开
- [ ] **验证开发环境**：
  - [ ] 检查所有工具链
  - [ ] 运行简单测试案例

### 6.2 构建与调试配置

- [ ] **CMake 项目配置**：
  - [ ] 创建顶层 `CMakeLists.txt`
  - [ ] 配置组件间依赖关系
- [ ] **调试配置**：
  - [ ] 设置 OpenFOAM 求解器调试
  - [ ] 配置耦合过程监控

## 阶段七：验证与测试

### 7.1 组件集成测试

- [ ] **独立组件测试**：
  - [ ] OpenFOAM 2412: `simpleFoam` 空腔流
  - [ ] CalculiX: 静态力学分析
  - [ ] preCICE: 单元测试套件
- [ ] **耦合接口测试**：
  - [ ] 数据传输正确性
  - [ ] 并行计算验证

### 7.2 完整 FSI 流程测试

- [ ] **运行完整案例**：
  - [ ] 启动 OpenFOAM 求解器
  - [ ] 启动 CalculiX 求解器
  - [ ] 监控 preCICE 耦合收敛
- [ ] **结果验证**：
  - [ ] 物理合理性检查
  - [ ] 与基准案例对比

## 阶段八：文档与维护

### 8.1 项目文档

- [ ] **编写使用说明**：
  - [ ] Ubuntu 24.04 特定配置
  - [ ] OpenFOAM 2412 环境设置
  - [ ] 案例运行指南
- [ ] **故障排除文档**：
  - [ ] Ubuntu 24.04 常见问题
  - [ ] OpenFOAM 2412 适配问题

### 8.2 维护计划

- [ ] **更新策略**：
  - [ ] OpenFOAM 版本更新跟踪
  - [ ] Ubuntu LTS 版本兼容性
- [ ] **备份方案**：
  - [ ] 重要配置备份
  - [ ] 案例数据管理

## 阶段九：共享开发文档笔记库建立

### 9.1 Obsidian + Git 知识库搭建

- [x] **初始化 Obsidian 知识库**：
  - [x] 创建 `docs/` 目录作为知识库根目录
  - [x] 初始化 Git 仓库：`git init`
  - [x] 配置 `.gitignore` 排除临时文件
- [x] **配置 Obsidian 工作区**：
  - [x] 安装核心插件：日记、模板、反向链接、图谱
  - [x] 配置主题和外观（如需要）
  - [x] 设置默认笔记位置和链接方式

### 9.2 Git 协作流程配置

- [x] **配置 Git 仓库**：
  - [x] 设置远程仓库（GitHub）
  - [ ] 配置分支保护规则
  - [ ] 设置协作权限
- [x] **建立 Git 工作流**：
  - [x] 主分支：`main`（稳定版本）
  - [ ] 开发分支：`develop`（集成开发）
  - [ ] 功能分支：`feature/*`（新功能开发）
  - [ ] 文档分支：`docs/*`（文档更新）

---

## 预期协作效益

1. **知识共享**：团队成员可实时访问和更新技术文档
2. **版本控制**：所有文档变更都有完整的历史记录
3. **问题追踪**：技术问题和解决方案可被有效记录和检索
4. **决策追溯**：技术决策过程和讨论有完整记录
5. **新人培训**：新成员可通过文档快速上手项目

通过 Obsidian 的强大链接功能和 Git 的版本控制，建立起一个活的技术知识库，有效支撑团队的协作开发和知识积累。

---

## 版本优势说明

### OpenFOAM 2412 优势

- **最新功能**：包含最新的数值方法和物理模型
- **性能优化**：改进的并行计算和内存管理
- **更好的兼容性**：与现代编译器工具链更好的集成
- **官方支持**：ESi 提供的长期支持

### Ubuntu 24.04 优势

- **更新的工具链**：GCC 13, CMake 3.28+
- **更好的性能**：优化的内核和系统库
- **长期支持**：LTS 版本，5 年支持周期
- **安全性**：最新的安全补丁和更新
