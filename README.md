# Awesome Zero-Shot 3D HOI

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://makeapullrequest.com)

A curated list of **papers** on **zero-shot 3D Human-Object Interaction (HOI) motion sequence generation** — synthesizing temporally aligned human and object motion from text instructions, object geometry, and initial states, without relying on paired text-HOI training data.

---

## Contents

- [Taxonomy](#taxonomy)
- [Papers](#papers)
  - [Data-Driven Generation](#data-driven-generation)
    - [Static Grasp & Scene Interaction](#static-grasp--scene-interaction)
    - [Conditional Diffusion for HOI](#conditional-diffusion-for-hoi)
  - [Zero-Shot: LLM-based Planning & Composition](#zero-shot-llm-based-planning--composition)
  - [Zero-Shot: Prior Distillation](#zero-shot-prior-distillation)
  - [Zero-Shot: Inverse Rendering / 4D Reconstruction](#zero-shot-inverse-rendering--4d-reconstruction)
- [Benchmarks & Datasets](#benchmarks--datasets)

---

## Taxonomy

> 三维人-物交互生成的核心挑战可归纳为三个正交维度：**语义对齐**（将自然语言指令转化为具体交互动作）、**物理合理性**（避免穿透、浮空、脚滑动）和**时空协调**（人与物体的精细同步）。

### 1) Data-Driven Generation

在成对 HOI 数据集上训练回归或扩散模型，学习人体姿态与物体位姿的映射关系。条件输入从简单（物体几何+初始姿态）到复杂（文本+场景+路径点）逐步演进。

**核心局限：** 泛化瓶颈（面对分布外物体性能下降）、数据成本（成对 MoCap 数据难以规模化）、任务泛化缺失（无法处理训练集外的交互类型）。

### 2) Zero-Shot: LLM-based Planning & Composition (规划-执行范式)

将复杂交互分解为"LLM 高层规划 → 扩散模型/物理引擎低层执行"的两阶段流程。利用 LLM 的常识推理能力进行任务分解与接触规划。

**核心权衡：** 以工程复杂性换取任务可解释性和执行可行性，支持长程多步骤交互，但多模块串行导致误差累积。

### 3) Zero-Shot: Prior Distillation (先验蒸馏范式)

从预训练的 2D 图像/视频扩散模型或多模态大语言模型中"蒸馏"出 3D/4D 交互先验，避免显式的分步规划，实现端到端生成。

**核心权衡：** 以可控性换取视觉真实感，泛化能力强但对间歇接触和长序列支持较弱。

### 4) Zero-Shot: Inverse Rendering / 4D Reconstruction (逆渲染范式)

将 HOI 生成重构为"从 2D 视频先验反向重建 4D 场景"的逆问题，通过光流分割、铰链约束等重建物体运动，再据此优化人体运动。

**核心权衡：** 以推理速度换取物理约束强度，天然支持铰接物体和开放集物体，但推理慢且依赖生成视频质量。

### Zero-Shot 三层次定义

| 层次 | 定义 | 代表方法 | 难度 |
|------|------|----------|------|
| **物体零样本** | 训练时见过同类交互，测试物体形状未出现 | GOAL, InterDiff, OMOMO, CHOIS, CG-HOI, THOR | 中等 |
| **交互零样本** | 训练时未见过的交互类型 | InterDreamer, PrimHOI | 高 |
| **完全无配对数据** | 训练阶段不使用文本-HOI配对数据 | AnchorHOI, ArtHOI, InteractAnything, AvatarGO, OpenHOI | 最高 |

---

## Papers

> **Format:** **Title** (Venue Year). *Authors*. [[Paper]](…) [[Project]](…) [[Code]](…)

---

### Data-Driven Generation

#### Static Grasp & Scene Interaction

- **GOAL: Generating 4D Whole-Body Motion for Hand-Object Grasping** (CVPR 2022).
  *Omid Taheri, Vasileios Choutas, Michael J. Black, Dimitrios Tzionas (MPI for Intelligent Systems)*.
  首次将全身姿态（含手部和头部）纳入抓握生成框架，通过 GNet 生成目标抓握姿态、MNet 生成运动序列，在 GRAB 数据集上训练。支持物体零样本泛化。
  [[Paper]](https://arxiv.org/abs/2112.11454) [[Project]](https://goal.is.tuebingen.mpg.de/)

- **HUMANISE: Language-conditioned Human Motion Generation in 3D Scenes** (NeurIPS 2022).
  *Zan Wang, Yixin Chen, Tengyu Liu, Yixin Zhu, Wei Liang, Siyuan Huang*.
  提出大规模语义丰富的合成 HSI 数据集（19.6k 运动序列/643 室内场景），实现语言条件驱动的3D场景中人体运动生成，支持"走到物体旁坐下/躺下"等低交互密度动作。
  [[Paper]](https://arxiv.org/abs/2210.09729) [[Project]](https://silverster98.github.io/HUMANISE/)

---

#### Conditional Diffusion for HOI

- **InterDiff: Generating 3D Human-Object Interactions with Physics-Informed Diffusion** (ICCV 2023).
  *Sirui Xu, Zhengyuan Li, Yu-Xiong Wang, Liang-Yan Gui (UIUC)*.
  提出交互扩散框架：扩散模型编码未来 HOI 分布，物理信息校正器利用接触点变换后的相对运动易预测性修正生成结果，实现训练层面的物理先验注入。
  [[Paper]](https://arxiv.org/abs/2308.16905) [[Project]](https://sirui-xu.github.io/InterDiff/)

- **OMOMO: Object Motion Guided Human Motion Synthesis** (SIGGRAPH Asia 2023).
  *Jiaman Li, Jiajun Wu, C. Karen Liu (Stanford University)*.
  以物体运动为条件生成全身人体操作运动，采用双扩散过程——先从物体运动预测手部位置，再据此合成全身姿态，通过手部中间表示显式强制接触约束。附带 15 物体约 10 小时的大规模 HOI 数据集。
  [[Paper]](https://doi.org/10.1145/3618333) [[Project]](https://omomo-iccv2023.github.io/)

- **CG-HOI: Contact-Guided 3D Human-Object Interaction Generation** (CVPR 2024).
  *Christian Diller, Angela Dai (TU Munich)*.
  首个从文本生成动态3D HOI的方法。核心创新是将人体运动、物体运动和接触在联合扩散过程中通过交叉注意力关联，推理时利用学到的接触信息进行引导，可处理给定物体轨迹的条件生成和静态3D场景扫描。
  [[Paper]](https://arxiv.org/abs/2311.16097)

- **THOR: Text to Human-Object Interaction Diffusion via Relation Intervention** (arXiv 2024).
  *Qianyang Wu, Ye Shi, Xiaoshui Huang, Jingyi Yu, Lan Xu, Jingya Wang (ShanghaiTech / Shanghai AI Lab)*.
  提出关系干预机制：在扩散去噪每步中，先用文本引导人体和物体运动，再通过人-物关系表示干预物体运动以增强时空关系一致性。构建了 Text-BEHAVE 数据集。
  [[Paper]](https://arxiv.org/abs/2403.11208)

- **CHOIS: Controllable Human-Object Interaction Synthesis** (ECCV 2024).
  *Jiaman Li, Alexander Clegg, Roozbeh Mottaghi, Jiajun Wu, Xavier Puig, C. Karen Liu (Stanford / Meta FAIR)*.
  以语言描述、初始状态和稀疏路径点为条件，用条件扩散模型同时生成物体和人体运动。引入物体几何损失提升路径点匹配，设计测试时引导项（接触+足部滑动+穿透）强制物理约束，支持与路径规划模块无缝集成生成长程交互。
  [[Paper]](https://arxiv.org/abs/2312.03913)

---

### Zero-Shot: LLM-based Planning & Composition

- **InterDreamer: Zero-Shot Text to 3D Dynamic Human-Object Interaction** (NeurIPS 2024).
  *Sirui Xu, Ziyin Wang, Yu-Xiong Wang, Liang-Yan Gui (UIUC)*.
  开创性工作：首次证明无需文本-HOI配对数据即可生成 3D 动态交互。利用 LLM 将文本改写为 text-to-motion 模型可理解的格式，引入世界模型（STGNN）预测物体动力学，将交互语义与动力学解耦。
  [[Paper]](https://arxiv.org/abs/2403.19652) [[Project]](https://sirui-xu.github.io/InterDreamer/)

- **Human-Object Interaction from Human-Level Instructions** (arXiv 2024).
  *Zhen Wu, Jiaman Li, Pei Xu, C. Karen Liu (Stanford University)*.
  首个完整系统：LLM 解析指令为场景图和执行计划，多阶段扩散生成器依次合成全身、手指与物体的同步运动，最后用 PPO 在 Isaac Gym 中跟踪修正以确保物理合理性。支持长程多步骤物体操作。
  [[Paper]](https://arxiv.org/abs/2406.17840)

- **PrimHOI: Compositional Human-Object Interaction via Reusable Primitives** (ICCV 2025).
  *Kai Jia, Tengyu Liu, Yixin Zhu, Mingtao Pei, Siyuan Huang (BIT / BIGAI / PKU)*.
  将日常交互中的重复局部接触模式（抓取、夹持、支撑）定义为可复用的交互基元，通过时空组合实现复杂 HOI 序列生成。层次化基元规划实现零样本迁移到未见交互任务，无需任务特定训练数据。
  [[Project]](https://kairobo.github.io/PrimHOI/)

---

### Zero-Shot: Prior Distillation

- **AvatarGO: Zero-shot 4D Human-Object Interaction Generation and Animation** (ICLR 2025).
  *Yukang Cao, Liang Pan, Kai Han, Kwan-Yee K. Wong, Ziwei Liu (NTU / Shanghai AI Lab / HKU)*.
  首次实现零样本 4D HOI 动画生成：用 LLM+Lang-SAM 在 3D 人体上分割接触区域（"where"），通过对应感知运动优化利用 SMPL-X 线性混合蒙皮构建人-物运动场（"how"），无需成对 HOI 数据即可生成可动画的交互场景。
  [[Paper]](https://arxiv.org/abs/2410.07164) [[Project]](https://yukangcao.github.io/AvatarGO/)

- **AnchorHOI: Zero-shot Generation of 4D Human-Object Interaction via Anchor-based Prior Distillation** (AAAI 2026).
  *Sisi Dai, Kai Xu (NUDT / CAS)*.
  提出锚点先验蒸馏策略：锚点 NeRF 从图像扩散模型蒸馏静态接触先验，锚点关键点从视频扩散模型蒸馏动态运动先验。锚点提供共享表征空间，使 2D 知识可迁移到 3D SMPL-X 参数优化，混合利用图像和视频扩散模型先验。
  [[Paper]](https://arxiv.org/abs/2512.14095)

- **OpenHOI: Open-World Hand-Object Interaction Synthesis with Multimodal Large Language Model** (NeurIPS 2025).
  *Zhenhao Zhang, Ye Shi, Lingxiao Yang, Suting Ni, Qi Ye, Jingya Wang (ShanghaiTech / Zhejiang Univ.)*.
  首个开放世界 HOI 合成框架：微调 3D MLLM 实现联合功能定位（如杯柄=可抓取区域）和语义任务分解，功能感知扩散模型生成手-物交互序列，无需训练的物理细化减少穿透。支持新物体类别、多阶段任务和自由语言指令。
  [[Paper]](https://arxiv.org/abs/2505.18947) [[Project]](https://openhoi.github.io/)

---

### Zero-Shot: Inverse Rendering / 4D Reconstruction

- **ArtHOI: Articulated Human-Object Interaction Synthesis by 4D Reconstruction from Video Priors** (3DV 2025).
  *Zihao Huang, Tianqi Liu, Zhaoxi Chen, Shaocong Xu, Saining Zhang, Lixing Xiao, Zhiguo Cao, Wei Li, Hao Zhao, Ziwei Liu*.
  首个零样本铰接物体 HOI 框架：将生成任务重构为 4D 重建问题——从视频扩散模型生成的单目视频出发，通过光流分割物体部件、铰链约束重建关节运动，再据此优化 SMPL-X 人体运动。解耦重建流程确保几何一致性和物理合理性。
  [[Paper]](https://arxiv.org/abs/2603.04338) [[Project]](https://arthoi.github.io/)

- **InteractAnything: Zero-shot Human Object Interaction Synthesis via LLM Feedback and Object Affordance Parsing** (ACM MM 2025).
  *Jinlu Zhang, Yixin Chen, Zan Wang, Jie Yang, Yizhou Wang, Siyuan Huang (PKU / BIGAI / BIT)*.
  聚焦开放集未知物体的零样本交互：用预训练 2D 扩散模型生成接触概率图进行功能解析，多视图 SDS 优化初始姿态，LLM 反馈循环提供细粒度人-物关系指导，力闭合约束等损失进行精细优化。手部接触细节表现出色。
  [[Paper]](https://arxiv.org/abs/2505.24315) [[Project]](https://jinluzhang.site/projects/interactanything)

---

## Benchmarks & Datasets

- **InterAct: Advancing Large-Scale Versatile 3D Human-Object Interaction Generation** (CVPR 2025).
  *Sirui Xu, Dongting Li, Yucheng Zhang, Xiyan Xu, Qi Long, Ziyin Wang, Yunzhi Lu, Shuchang Dong, Hezi Jiang, Akshat Gupta, Yu-Xiong Wang, Liang-Yan Gui (UIUC)*.
  大规模 3D HOI 基准：整合标准化 21.81 小时 HOI 数据并丰富文本标注，提出统一优化框架减少伪影并修正手部运动，利用接触不变性扩展至 30.70 小时。定义六个基准任务，从统一视角推进 HOI 生成建模。
  [[Paper]](https://arxiv.org/abs/2509.09555) [[Project]](https://sirui-xu.github.io/InterAct/) [[Code]](https://github.com/wzyabcas/InterAct)

- **Text-BEHAVE** — 文本标注版 BEHAVE 数据集，由 THOR 构建，将文本描述与当前最大公开 3D HOI 数据集无缝整合。
  [[Paper]](https://arxiv.org/abs/2403.11208)

- **BEHAVE** — 大规模人-物交互 3D 扫描数据集，多个方法的基础训练/评估数据。
  [[Website]](https://virtualhumans.mpi-inf.mpg.de/behave/)

- **GRAB** — 全身抓握数据集（51 物体），GOAL 的训练和评估基础。
  [[Website]](https://grab.is.tue.mpg.de/)

---

## 三条技术路线对比

| 维度 | 规划-执行范式 | 先验蒸馏范式 | 逆渲染范式 |
|------|--------------|-------------|-----------|
| **代表方法** | InterDreamer, Human-level Instructions, PrimHOI | AnchorHOI, OpenHOI, AvatarGO | ArtHOI, InteractAnything |
| **核心优势** | 任务可解释性高，支持长程多步骤 | 视觉真实感强，手指精细度高 | 物理约束强，天然支持铰接物体 |
| **核心劣势** | 系统复杂，模块误差累积 | 对间歇接触/长序列支持弱 | 推理速度慢，依赖视频质量 |
| **数据依赖** | LLM + 少量 HOI 数据 | 预训练扩散模型（无 HOI 数据） | 视频扩散模型（无 3D 监督） |
| **适用场景** | 具身智能、任务规划 | 4D 内容生成、AR/VR | 铰接物体、开放集物体 |

> 当前没有一条技术路线在所有维度上占优。未来的统一系统可能需要根据任务需求动态选择或融合多种范式——例如用 LLM 规划任务边界，在精细手指操作中调用先验蒸馏模块，在铰接物体场景切换到逆渲染流程。
