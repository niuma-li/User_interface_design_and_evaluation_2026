# 两篇 HOI 论文复现与理解报告

## 一、复现目标

本次尝试复现的两个项目分别是：

1. **Human-Object Interaction from Human-Level Instructions**

   官方代码仓库为 <https://github.com/zhenkirito123/hoifhli_release>。该论文目标是从较抽象的人类级指令出发，生成连续的人-物交互动作，包括人体全身运动、物体运动以及细粒度手指运动。论文提出系统由高层 LLM 规划器和低层运动生成器组成：高层模块理解任务、推理目标物体布局和执行顺序，低层模块生成导航、交互和手部动作。论文摘要中明确说明其目标是同时生成 object motion、full-body human motion 和 finger motion。

2. **ArtHOI: Articulated Human-Object Interaction Synthesis by 4D Reconstruction from Video Priors**

   官方代码仓库为 <https://github.com/Inso-13/ArtHOI>。该论文目标是针对带关节结构的物体，例如橱柜、冰箱、微波炉门等，进行零样本人-物交互合成。它不是直接依赖 3D 标注数据训练一个传统监督模型，而是把视频扩散模型生成的视频作为先验，通过 4D 重建恢复人体运动、物体部件运动和交互关系。论文摘要中强调其核心是从单目视频先验中恢复具有接触、关节运动和时间一致性的 4D 场景。

## 二、复现环境与依赖分析

我阅读了两个项目的 README 和代码结构，二者都依赖 SMPL-X 人体模型文件。

HOIFHLI 的 README 要求将 SMPL-X 文件放入 `data/smpl_all_models/smplx/`，并包含以下文件：

```text
data/
└── smpl_all_models/
    └── smplx/
        ├── SMPLX_FEMALE.npz
        ├── SMPLX_MALE.npz
        ├── SMPLX_NEUTRAL.npz
        ├── SMPLX_FEMALE.pkl
        ├── SMPLX_MALE.pkl
        └── SMPLX_NEUTRAL.pkl
```

该项目还需要安装 `smplx[all]`、`human_body_prior`、BPS、TorchSDF 等依赖，并通过脚本下载额外数据后才能运行采样或训练。

ArtHOI 的 README 也要求在 `assets/body_models/smplx/` 下准备 SMPL-X 模型文件，包括：

```text
assets/
└── body_models/
    └── smplx/
        ├── SMPLX_NEUTRAL.npz
        ├── SMPLX_MALE.npz
        ├── SMPLX_FEMALE.npz
        └── smplx_vert_segmentation.json
```

除此之外，ArtHOI 还依赖 SAM、CoTracker、PyTorch3D、FRNN、diff-gaussian-rasterization、tiny-cuda-nn 等大量视觉和 3D 重建相关组件。

## 三、未能完成复现的原因

本次没有完整跑通两个项目，主要原因不是环境配置或代码理解问题，而是关键数据资源无法下载。两个项目都需要从 <https://smpl-x.is.tue.mpg.de/> 下载 SMPL-X 模型文件，但我在实际尝试时由于网络访问问题，无法稳定访问或下载该网站中的模型文件。因此，项目缺少人体参数化模型，后续的 SMPL-X 解析、人体 mesh 生成、姿态恢复、手部动作建模和渲染步骤都无法执行。

具体来说：

- HOIFHLI 没有 SMPL-X 后，无法构建完整人体模型，也无法生成包含手部和身体的交互动作结果。
- ArtHOI 没有 SMPL-X 后，无法从视频先验中恢复人体 4D 表示，也无法完成与 articulated object 的联合交互重建。
- SMPL-X 是两个项目的人体表达基础，不是可选依赖，因此不能简单跳过。

所以，本次复现失败的直接原因是：**由于网络问题，无法从 SMPL-X 官网下载必要模型文件，导致两个项目均缺少核心人体模型资源，无法进入完整运行阶段。**

## 四、对 HOIFHLI 方法的理解

HOIFHLI 解决的问题是：给定类似“整理卧室中的工作区”这样抽象的人类指令，系统不仅要知道需要移动哪些物体，还要知道物体最后应该放在哪里、人应该怎样走过去、怎样抓取和移动物体、最后怎样放下。

它的核心流程可以理解为两层。

第一层是 **高层规划器**。该模块使用 LLM/VLM 理解场景和任务，不直接让 LLM 输出精确 3D 坐标，而是让它先输出空间关系，例如：

```text
on(object1, object2)
adjacent(object1, object2, direction, distance)
facing(object1, object2)
```

之后系统再把这些关系转换成具体目标位置和朝向。这种设计可以降低 LLM 直接做 3D 空间推理的不稳定性。论文中也说明，高层规划包含空间关系生成、目标布局计算和任务计划生成三个阶段。

第二层是 **低层运动生成器**。它负责把任务计划变成连续动作。系统包含导航模块和交互模块：导航模块负责人在场景中走到目标物体附近，交互模块负责生成搬动、抓取、释放等动作。交互模块又分为 CoarseNet、抓取姿态优化、RefineNet 和 FingerNet。CoarseNet 先生成没有精细手指的身体和物体运动；随后根据物体和手腕相对关系优化抓取姿态；RefineNet 再根据更准确的手腕约束细化全身和物体运动；最后 FingerNet 生成接触前后平滑的手指动作。

我认为这篇工作的关键贡献在于把“高层语义任务理解”和“低层可执行运动合成”连接起来，使系统不仅能理解抽象指令，还能输出包含身体、物体和手部的完整交互序列。

## 五、对 ArtHOI 方法的理解

ArtHOI 关注的是 articulated object，也就是具有可动部件的物体，例如柜门、冰箱门、抽屉等。传统 HOI 方法更多处理刚体物体，而 ArtHOI 的难点在于物体本身会发生关节运动，同时人体还要与这些运动部件产生合理接触。

ArtHOI 的思路是把人-物交互合成转化为一个 4D 重建问题。它先利用视频扩散模型产生人和物体交互的视频先验，然后从这个视频中恢复三维空间和时间维度上的动态场景。论文中强调，它不依赖 3D/4D 监督，而是通过 monocular video prior 进行 inverse rendering 和几何优化。

它的两个关键设计是：

1. **基于光流的部件分割**：利用视频中的运动差异区分静态区域和动态部件，从而识别 articulated object 的可动部分。
2. **解耦式重建流程**：由于单目视频下同时优化人体运动和物体关节运动很不稳定，ArtHOI 先恢复物体关节状态，再基于已恢复的物体状态合成人体运动。

我认为 ArtHOI 的核心价值在于把视频生成模型的语义能力和 4D 几何重建结合起来，使零样本人-物交互不再局限于刚体物体，而能扩展到有门、抽屉、盖子等结构的 articulated objects。

## 六、两篇工作的对比理解

两篇论文都属于人-物交互合成方向，但侧重点不同。

HOIFHLI 更强调从抽象人类指令到完整执行动作的任务规划能力。它关心“人要完成什么任务、移动哪些物体、按什么顺序做、怎样生成全身和手指动作”。

ArtHOI 更强调从视频先验恢复有物理和几何一致性的 4D 交互。它关心“如何在没有 3D 标注的情况下，从生成视频中恢复人体、物体关节和接触关系”。

简单来说，HOIFHLI 偏 **语言规划 + 动作生成**，ArtHOI 偏 **视频先验 + 4D 重建 + articulated object interaction**。

## 七、代码结构理解

### 7.1 HOIFHLI 代码结构

HOIFHLI 的代码主要围绕“扩散模型生成动作”和“采样长序列”展开。仓库中的关键文件和目录如下：

```text
hoifhli_release/
├── sample.py                                  # 长序列交互动作采样入口
├── argument_parser.py                         # 训练和采样参数定义
├── trainer_navigation_motion_diffusion.py     # 导航动作扩散模型训练
├── trainer_interaction_motion_diffusion.py    # 人-物交互动作扩散模型训练
├── trainer_finger_diffusion.py                # 手指动作扩散模型训练
├── grasp_generation/
│   └── gen_grasp.py                           # 抓取姿态生成相关代码
├── physics_tracking/
│   ├── main.py                                # 物理跟踪入口
│   ├── env.py                                 # Isaac Gym 环境封装
│   └── README.md                              # 物理跟踪说明
└── scripts/
    ├── download_data.sh                       # 下载 demo/预处理数据
    ├── download_training_data.sh              # 下载训练数据
    ├── sample.sh                              # 默认采样脚本
    └── train/
        ├── train_coarsenet.sh                 # 训练 CoarseNet
        └── train_refinenet.sh                 # 训练 RefineNet
```

从代码结构可以看出，该项目不是单一模型，而是一个分阶段系统。`sample.py` 是最终生成结果的入口，会调用已经训练好的导航模型、交互模型和手指模型，生成长序列人-物交互动作。`argument_parser.py` 中定义了大量开关，例如 `--use_long_planned_path`、`--add_interaction_root_xy_ori`、`--add_interaction_feet_contact`、`--add_finger_motion`、`--use_guidance_in_denoising`，这些参数对应论文中提到的长序列规划、人体根节点运动、脚接触约束、手指动作和去噪引导。

训练部分主要由三个 trainer 文件负责。`trainer_navigation_motion_diffusion.py` 负责训练人从当前位置走到目标附近的导航动作；`trainer_interaction_motion_diffusion.py` 负责训练人与物体交互时的身体和物体运动，其中 `scripts/train/train_coarsenet.sh` 和 `scripts/train/train_refinenet.sh` 都调用这个文件，只是参数不同；`trainer_finger_diffusion.py` 负责补充手指动作，使最终结果不只是身体靠近物体，而是有更细致的抓取和释放动作。

此外，`grasp_generation/gen_grasp.py` 对应抓取姿态生成，`physics_tracking/` 对应物理跟踪和仿真验证。物理跟踪部分需要 Isaac Gym，说明作者不仅关注视觉上的动作生成，也希望动作能够通过一定的物理约束进行后处理或验证。

### 7.2 ArtHOI 代码结构

ArtHOI 的代码更集中，核心入口是 `src/train.py`。仓库中的主要结构如下：

```text
ArtHOI/
├── requirements.txt
├── src/
│   ├── train.py                    # 主训练/优化入口
│   ├── configs/
│   │   ├── default.yml             # 默认配置
│   │   └── config.py               # 配置读取和合并
│   ├── models/
│   │   ├── humangs.py              # Human Gaussian 表示
│   │   ├── objectgs.py             # Object Gaussian 表示
│   │   ├── renderer.py             # Gaussian 渲染器
│   │   ├── sags.py                 # 分割/高斯相关模块
│   │   └── smpl_x.py               # SMPL-X 人体模型封装
│   └── utils/
│       ├── data_utils.py           # 数据读取、相机、SMPL-X 初始化等
│       ├── loss_utils.py           # 光度、mask、track、碰撞等损失
│       ├── io_utils.py             # 文件读写
│       └── vis_utils.py            # 可视化
```

`src/train.py` 中的 `Trainer` 类基本体现了 ArtHOI 的整体流程。初始化阶段会读取场景数据、相机、SMPL-X 初始人体参数、人体 mask、物体 mask、CoTracker 轨迹、Gaussian mesh 等信息；随后建立 `HumanGaussian`、`ObjectGaussian` 和 `GaussianRenderer`。优化过程分为两个阶段：第一阶段主要优化 articulated object 的运动参数，即物体关节旋转、平移和部件权重；第二阶段在物体状态基础上继续优化人体 SMPL-X 参数，包括身体姿态、相机外参、平移、形状参数等。

`configs/default.yml` 说明了项目的数据格式和训练设置。默认场景名是 `open-cabinet`，结果会保存到 `../results/{scene}/arthoi/`。配置中还定义了两阶段优化迭代次数、学习率、损失权重和数据路径。损失项包括图像重建损失、人体/物体 mask 损失、2D track 损失、人体平滑损失、3D keypoint 损失、脚滑动损失和碰撞损失。这些代码细节和论文方法是一致的：ArtHOI 本质上是把生成视频中的先验信息转化为一个可优化的 4D 人-物场景。

## 八、如果数据可用时的复现流程

### 8.1 HOIFHLI 复现流程

如果能够正常下载 SMPL-X 和项目数据，HOIFHLI 的复现流程大致如下：

```bash
git clone https://github.com/zhenkirito123/hoifhli_release.git
cd hoifhli_release
conda env create -f environment.yml
conda activate hoifhli_env
```

然后安装额外依赖：

```bash
# 安装 TorchSDF，按照 DexGraspNet/TorchSDF 的说明编译

git clone https://github.com/nghorbani/human_body_prior.git
cd human_body_prior
pip install tqdm dotmap PyYAML omegaconf loguru
python setup.py develop
cd ..

pip install git+https://github.com/otaheri/chamfer_distance
pip install git+https://github.com/otaheri/bps_torch
pip install "smplx[all]"
```

接着从 SMPL-X 官网下载人体模型，并整理为：

```text
data/smpl_all_models/smplx/
├── SMPLX_FEMALE.npz
├── SMPLX_MALE.npz
├── SMPLX_NEUTRAL.npz
├── SMPLX_FEMALE.pkl
├── SMPLX_MALE.pkl
└── SMPLX_NEUTRAL.pkl
```

如果 SMPL-X 文件准备完成，就可以下载项目数据：

```bash
bash scripts/download_data.sh
```

运行默认 demo 采样：

```bash
bash scripts/sample.sh
```

`scripts/sample.sh` 实际执行的是：

```bash
python sample.py \
  --window=120 \
  --data_root_folder="./data/processed_data" \
  --project="./experiments" \
  --test_sample_res \
  --use_long_planned_path \
  --add_interaction_root_xy_ori \
  --add_interaction_feet_contact \
  --add_finger_motion \
  --use_guidance_in_denoising \
  --vis_wdir="moving_box"
```

如果要重新训练模型，需要先下载训练数据：

```bash
bash scripts/download_training_data.sh
```

然后分别训练 CoarseNet 和 RefineNet：

```bash
bash scripts/train/train_coarsenet.sh
bash scripts/train/train_refinenet.sh
```

需要注意的是，训练数据下载脚本会删除并重新下载 `data/processed_data`，所以如果里面有自定义数据，需要提前备份。物理跟踪部分还需要安装 Isaac Gym，进入 `physics_tracking/` 后可以运行：

```bash
python main.py config_tracking_seq1_cube.py --ckpt ckpt_seq1_cube/
python main.py config_tracking_seq1_cube.py --ckpt ckpt_seq1_cube/ --test
```

### 8.2 ArtHOI 复现流程

ArtHOI 的复现流程是先安装环境，再准备 `assets/` 和 `data/`，最后运行 `src/train.py`。

```bash
git clone https://github.com/Inso-13/ArtHOI.git
cd ArtHOI
conda create -n arthoi python=3.9
conda activate arthoi
pip install torch==2.0.1+cu117 torchvision==0.15.2+cu117 torchaudio==2.0.2+cu117 -f https://download.pytorch.org/whl/torch_stable.html
pip install -r requirements.txt
```

然后安装项目 README 中列出的 3D/视觉依赖：

```bash
git clone --recursive https://github.com/lxxue/FRNN.git
cd FRNN/external/prefix_sum
pip install .
cd ../../
pip install -e .
cd ..

pip install git+https://github.com/facebookresearch/pytorch3d.git@stable
pip install git+https://github.com/facebookresearch/segment-anything.git
pip install git+https://github.com/facebookresearch/co-tracker.git
pip install git+https://github.com/yzslab/simple-knn.git
pip install git+https://github.com/NVlabs/tiny-cuda-nn.git#subdirectory=bindings/torch
pip install torch-scatter torch-cluster -f https://data.pyg.org/whl/torch-2.0.0+cu117.html
pip install https://github.com/unlimblue/KNN_CUDA/releases/download/0.2/KNN_CUDA-0.2-py3-none-any.whl
pip install git+https://github.com/graphdeco-inria/diff-gaussian-rasterization.git
```

数据准备方面，需要下载 SAM 权重、SMPL-X 模型和 demo 数据。目录结构应为：

```text
assets/
├── sam_vit_h_4b8939.pth
└── body_models/smplx/
    ├── SMPLX_NEUTRAL.npz
    ├── SMPLX_MALE.npz
    ├── SMPLX_FEMALE.npz
    └── smplx_vert_segmentation.json

data/
└── open-cabinet/
    ├── init_params/
    │   ├── align.json
    │   ├── smplx.json
    │   ├── hamer.json
    │   └── camera.json
    ├── init_gaussians/
    │   ├── human_cano.ply
    │   ├── human.ply
    │   ├── object.ply
    │   └── scene.ply
    └── priors/
        ├── images/
        ├── human_masks/
        ├── object_masks/
        ├── cotracker/
        └── hmr4d_results.pt
```

准备完成后运行：

```bash
cd src
conda activate arthoi
python train.py --scene open-cabinet
```

训练和优化结果会保存在：

```text
results/open-cabinet/arthoi/
├── params/     # 优化后的模型参数和中间状态
└── renders/    # 渲染帧和可视化结果
```

如果使用自定义数据，则需要先用 GVHMR 得到人体 4D 初始运动，用 HAMER 得到手部参数，用 SAM/SAM2 得到人体和物体 mask，用 CoTracker 得到视频中的稠密对应关系，再按照 README 中的 `data/{scene_name}/` 格式组织数据。

## 九、结论

虽然由于网络问题无法从 SMPL-X 官网下载必要模型文件，导致两个项目没有完成端到端复现，但我已经完成了对两篇论文和官方代码说明的阅读与分析，理解了它们的研究目标、依赖资源、系统结构和核心技术路线。复现失败的主要原因是外部数据资源获取受阻，而不是没有掌握项目方法。后续如果能够成功获取 SMPL-X 模型文件，就可以继续完成环境配置、数据准备、demo 运行和结果可视化。

## 参考链接

- HOIFHLI 官方代码仓库：<https://github.com/zhenkirito123/hoifhli_release>
- HOIFHLI 论文页面：<https://arxiv.org/abs/2406.17840>
- ArtHOI 官方代码仓库：<https://github.com/Inso-13/ArtHOI>
- ArtHOI 论文页面：<https://arxiv.org/abs/2603.04338>
- SMPL-X 官方网站：<https://smpl-x.is.tue.mpg.de/>
