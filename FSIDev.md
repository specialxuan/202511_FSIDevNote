# FSI 求解系统搭建与开发清单：OpenFOAM + preCICE + CalculiX

## 阶段一：核心组件与依赖规划

### 1.1 技术选型与版本确认

- [ ] **确定组件版本**：
  - [ ] OpenFOAM: `openfoam2412` (ESi版本，最新稳定版)
  - [ ] preCICE: `v2.5.0` (稳定版)
  - [ ] CalculiX: `2.20` 或 `ccx_2.21`
  - [ ] 编译器: `gcc-13` (Ubuntu 24.04 默认)
  - [ ] CMake: `3.28+` (Ubuntu 24.04 自带)
- [ ] **选择基础 Docker 镜像**： `ubuntu:24.04` (Noble Numbat)

## 阶段二：Docker 环境构建

### 2.1 创建项目结构

- [ ] **创建项目根目录**： `fsi-dev-container/`
- [ ] **初始化子目录**：

```bash
fsi-dev-container/
├── .devcontainer/
│   ├── devcontainer.json
│   ├── Dockerfile
│   └── docker-compose.yml
├── scripts/
│   ├── install_dependencies.sh
│   ├── build_precice.sh
│   └── build_calculix.sh
├── cases/
│   ├── openfoam-case/
│   └── calculix-case/
├── src/
│   ├── openfoam-solver/
│   └── custom-adapter/
└── .vscode/
    ├── settings.json
    └── tasks.json
```

### 2.2 配置 Dev Container

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

## 阶段三：核心组件安装

### 3.1 系统级依赖安装

- [ ] **基础开发工具**：
  - [ ] `build-essential`, `cmake`, `git`, `wget`, `curl`
  - [ ] `gcc-13`, `g++-13`, `gfortran-13`
  - [ ] `python3-dev`, `python3-pip`, `python3-venv`
- [ ] **数学与并行库**：
  - [ ] `openmpi-bin`, `libopenmpi-dev`
  - [ ] `libboost-all-dev`
  - [ ] `libeigen3-dev`
  - [ ] `libscotch-dev`, `libparmetis-dev`

### 3.2 OpenFOAM 2412 安装

- [ ] **添加 OpenFOAM 官方仓库**：

```bash
# 添加 OpenFOAM 官方 GPG 密钥和仓库
sudo sh -c "wget -O - http://dl.openfoam.org/gpg.key | apt-key add -"
sudo add-apt-repository http://dl.openfoam.org/ubuntu
sudo apt-get update
```

- [ ] **安装 OpenFOAM 2412**：
  - [ ] `sudo apt-get install openfoam2412-dev`
  - [ ] 或 `sudo apt-get install openfoam2412`
- [ ] **配置 OpenFOAM 环境**：
  - [ ] 在 `~/.bashrc` 中添加：`source /usr/lib/openfoam/openfoam2412/etc/bashrc`
  - [ ] 验证：`simpleFoam -help`
  - [ ] 检查 `WM_PROJECT_DIR` 环境变量

### 3.3 preCICE 安装与配置

- [ ] **下载 preCICE 源码**：

```bash
git clone https://github.com/precice/precice.git
cd precice && git checkout v2.5.0
```

- [ ] **编译安装**：
  - [ ] 创建构建目录： `build`
  - [ ] 配置 CMake： `-DPETSc=OFF -DMPI=ON -DBUILD_SHARED_LIBS=ON`
  - [ ] 编译： `make -j$(nproc)`
  - [ ] 安装： `sudo make install`
- [ ] **验证安装**：
  - [ ] `pkg-config --modversion precice`
  - [ ] 运行单元测试： `ctest --output-on-failure`

### 3.4 OpenFOAM 适配器集成

- [ ] **安装 OpenFOAM 适配器**：
  - [ ] 下载最新 `precice-openfoam-adapter`
  - [ ] 配置适配器指向 OpenFOAM 2412 安装目录
  - [ ] 编译并与 preCICE 链接
- [ ] **验证适配器**：
  - [ ] 检查 `preciceAdapter` 可执行文件
  - [ ] 运行基础功能测试

### 3.5 CalculiX 安装与配置

- [ ] **安装 CalculiX**：
  - [ ] 通过 `apt` 安装： `calculix-ccx`, `calculix-cgx`
  - [ ] 或从源码编译最新版本
- [ ] **验证安装**：
  - [ ] `ccx -version`
  - [ ] `cgx -version`

## 阶段四：开发环境配置

### 4.1 VS Code 工作区配置

- [ ] **创建 `.vscode/settings.json`**：
  - [ ] 配置 C++ 标准 (C++17)
  - [ ] 设置默认编译器路径
  - [ ] 配置 CMake 构建目录
- [ ] **创建 `.vscode/tasks.json`**：
  - [ ] 定义 preCICE 构建任务
  - [ ] 定义测试用例运行任务
- [ ] **创建 `.vscode/launch.json`**：
  - [ ] 配置 C/C++ 调试器
  - [ ] 设置 FSI 耦合过程调试

### 4.2 环境验证脚本

- [ ] **创建验证脚本**：
  - [ ] `scripts/verify_installation.sh`
  - [ ] 检查所有组件版本
  - [ ] 测试基础功能

```bash
#!/bin/bash
# scripts/verify_installation.sh

echo "=== FSI 环境验证 ==="

# 检查 OpenFOAM
echo "1. 检查 OpenFOAM 2412..."
source /usr/lib/openfoam/openfoam2412/etc/bashrc
foamInstallationTest

# 检查 preCICE
echo "2. 检查 preCICE..."
pkg-config --modversion precice

# 检查 CalculiX
echo "3. 检查 CalculiX..."
ccx -version

echo "=== 环境验证完成 ==="
```

## 阶段五：测试用例搭建

### 5.1 准备 FSI 测试案例

- [ ] **选择标准测试案例**：
  - [ ] OpenFOAM 端：管流或空腔流
  - [ ] CalculiX 端：弹性体变形
- [ ] **配置 preCICE 耦合**：
  - [ ] 创建 `precice-config.xml`
  - [ ] 定义耦合接口和数据交换

### 5.2 案例文件结构

- [ ] **组织案例目录**：

```bash
cases/tutorial-fsi/
├── openfoam/
│   ├── system/
│   │   ├── controlDict
│   │   ├── fvSchemes
│   │   └── fvSolution
│   ├── constant/
│   │   ├── polyMesh/
│   │   └── transportProperties
│   └── 0/
│       ├── U
│       ├── p
│       └── k
├── calculix/
│   ├── model.inp
│   └── material.dat
└── precice-config.xml
```

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

## 阶段七：Ubuntu 24.04 特定优化

### 7.1 利用新版本特性

- [ ] **使用更新的工具链**：
  - [ ] GCC 13 的优化特性
  - [ ] CMake 3.28+ 的新功能
- [ ] **性能优化**：
  - [ ] 利用 Ubuntu 24.04 的内核优化
  - [ ] 配置适当的 CPU 调度策略

### 7.2 包管理优化

- [ ] **利用 Ubuntu 24.04 的软件包**：
  - [ ] 使用更新的 MPI 实现
  - [ ] 利用系统级数学库优化

## 阶段八：验证与测试

### 8.1 组件集成测试

- [ ] **独立组件测试**：
  - [ ] OpenFOAM 2412: `simpleFoam` 空腔流
  - [ ] CalculiX: 静态力学分析
  - [ ] preCICE: 单元测试套件
- [ ] **耦合接口测试**：
  - [ ] 数据传输正确性
  - [ ] 并行计算验证

### 8.2 完整 FSI 流程测试

- [ ] **运行完整案例**：
  - [ ] 启动 OpenFOAM 求解器
  - [ ] 启动 CalculiX 求解器
  - [ ] 监控 preCICE 耦合收敛
- [ ] **结果验证**：
  - [ ] 物理合理性检查
  - [ ] 与基准案例对比

## 阶段九：文档与维护

### 9.1 项目文档

- [ ] **编写使用说明**：
  - [ ] Ubuntu 24.04 特定配置
  - [ ] OpenFOAM 2412 环境设置
  - [ ] 案例运行指南
- [ ] **故障排除文档**：
  - [ ] Ubuntu 24.04 常见问题
  - [ ] OpenFOAM 2412 适配问题

### 9.2 维护计划

- [ ] **更新策略**：
  - [ ] OpenFOAM 版本更新跟踪
  - [ ] Ubuntu LTS 版本兼容性
- [ ] **备份方案**：
  - [ ] 重要配置备份
  - [ ] 案例数据管理

## 阶段十：共享开发文档笔记库建立

### 10.1 Obsidian + Git 知识库搭建

- [ ] **初始化 Obsidian 知识库**：
  - [ ] 创建 `docs/` 目录作为知识库根目录
  - [ ] 初始化 Git 仓库：`git init`
  - [ ] 配置 `.gitignore` 排除临时文件

```bash
fsi-dev-container/
├── docs/                           # Obsidian 知识库
│   ├── .obsidian/                 # Obsidian 配置
│   ├── 01-项目概述/
│   │   ├── 项目介绍.md
│   │   ├── 技术架构.md
│   │   └── 开发团队.md
│   ├── 02-环境搭建/
│   │   ├── 系统要求.md
│   │   ├── 安装指南.md
│   │   └── 环境验证.md
│   ├── 03-开发指南/
│   │   ├── OpenFOAM开发.md
│   │   ├── preCICE配置.md
│   │   └️── CalculiX集成.md
│   ├── 04-案例库/
│   │   ├── 基础案例/
│   │   ├── 进阶案例/
│   │   └── 故障排除.md
│   ├── 05-会议记录/
│   │   ├── 周会记录/
│   │   └── 技术讨论/
│   └── 06-参考资料/
│       ├── 官方文档链接.md
│       └️── 论文收集.md
```

- [ ] **配置 Obsidian 工作区**：
  - [ ] 安装核心插件：日记、模板、反向链接、图谱
  - [ ] 配置主题和外观（如需要）
  - [ ] 设置默认笔记位置和链接方式

- [ ] **建立文档模板系统**：
  - [ ] 创建会议记录模板
  - [ ] 创建技术问题记录模板
  - [ ] 创建案例文档模板

### 10.2 Git 协作流程配置

- [ ] **配置 Git 仓库**：
  - [ ] 设置远程仓库（GitHub/GitLab/Gitee）
  - [ ] 配置分支保护规则
  - [ ] 设置协作权限

- [ ] **建立 Git 工作流**：
  - [ ] 主分支：`main`（稳定版本）
  - [ ] 开发分支：`develop`（集成开发）
  - [ ] 功能分支：`feature/*`（新功能开发）
  - [ ] 文档分支：`docs/*`（文档更新）

- [ ] **配置 Git Hooks**：
  - [ ] 预提交检查：Markdown 格式验证
  - [ ] 自动生成文档索引
  - [ ] 备份重要配置

### 10.3 文档标准化与自动化

- [ ] **建立文档规范**：
  - [ ] Markdown 写作规范
  - [ ] 文件命名约定
  - [ ] 目录结构标准

- [ ] **配置自动化脚本**：
  - [ ] 自动生成目录树：`scripts/generate_toc.sh`
  - [ ] 文档链接检查：`scripts/check_links.py`
  - [ ] 定期备份脚本：`scripts/backup_docs.sh`

### 10.4 团队协作配置

- [ ] **设置协作规则**：
  - [ ] 文档更新流程
  - [ ] 代码审查机制
  - [ ] 冲突解决方案

- [ ] **配置同步机制**：
  - [ ] 定期同步提醒
  - [ ] 变更通知流程
  - [ ] 版本发布流程

### 10.5 知识库维护

- [ ] **定期维护任务**：
  - [ ] 清理过期文档
  - [ ] 更新链接和引用
  - [ ] 优化文档结构

- [ ] **质量保证**：
  - [ ] 文档审查机制
  - [ ] 用户反馈收集
  - [ ] 持续改进流程

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
- **长期支持**：LTS 版本，5年支持周期
- **安全性**：最新的安全补丁和更新
