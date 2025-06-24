---
aliases:
  - null
tags:
  - 博士/课题
  - code/AI/具身智能
  - AUV
  - 导航
  - SLAM
link: null
创建时间: 星期二 24日 六月 2025 16:23:28
编辑时间: Tuesday 24th June 2025 16:27:32
---

## 1 理论基础：具身智能与水下自主性的交汇

本部分旨在建立理论与实践的基础，将具身智能（Embodied Intelligence, EAI）定义为一个明确的科学范式，并将其核心原则与自主水下航行器（AUV）在用户指定场景下面临的根本性、未解决的挑战直接联系起来。

### 1.1 具身智能（EAI）范式

具身智能的核心定义是，智能产生于智能体（agent）的物理身体、其传感器以及其环境之间的连续、动态交互过程 [1]。这与传统的人工智能（AI）形成了鲜明对比，后者通常将感知、规划和行动视为独立的、脱离实体的计算问题。“具身假说”（embodiment hypothesis）进一步提出，认知从根本上是由这些物理交互所塑造的 [1]。

EAI的关键原则包括：

- **感知-行动耦合 (Perception-Action Coupling):** 感知并非一个被动建立世界模型的过程，而是与行动紧密相连。智能体为了行动而感知，并通过行动来优化感知 [5]。这创建了一个闭环，智能体从自身行动所带来的感官结果中学习。

- **多模态学习 (Multimodal Learning):** EAI系统整合来自多种感官（如视觉、触觉、听觉）的信息，以建立一个更丰富、更鲁棒的环境理解 [1]。对AUV而言，这意味着将声呐数据与来自惯性导航系统（INS）的本体感受数据（加速度、角速度）和多普勒测速仪（DVL）的数据进行融合。

- **从经验中学习 (Learning from Experience):** EAI强调通过在真实世界中的试错来学习，而不仅仅是依赖预先加载的数据 [1]。这通常通过强化学习（Reinforcement Learning, RL）和模仿学习（Imitation Learning, IL）等技术实现，智能体在这些过程中逐步提升其任务表现 [2]。

- **环境的角色 (The Role of the Environment):** 环境不仅仅是行动的舞台，更是一个积极的参与者，它塑造了认知发展和策略的形成 [2]。一个具身智能体学会利用其环境的物理属性。

### 1.2 AUV的巨大挑战：在无特征、无GPS的未知水域中导航

水下导航的复杂性源于GPS信号无法穿透水体，这使得AUV不能像陆地或空中载具那样依赖卫星导航系统 [10]。因此，AUV被迫依赖于航位推算（Dead-Reckoning），主要使用由多普勒测速仪（DVL）辅助的惯性导航系统（INS） [10]。

其核心困境在于“无法解决的漂移问题”。即便是采用光纤陀螺（FOG）或环形激光陀螺（RLG）的高端INS系统，其微小的测量误差在长时间积分以计算位置时，也会导致无界的位置误差累积 [12]。没有外部校正，AUV对其自身位置的信念会随着时间的推移变得越来越不准确。

在此特定场景下，AUV的传感器套件及其局限性如下：

- **惯性导航系统/多普勒测速仪 (INS/DVL):** 提供相对于海床的速度（底跟踪模式）和姿态信息。DVL有助于约束INS的漂移率，但误差仍然是累积的 [14]。其精度通常以总航行距离的百分比来衡量（例如，0.05% [14]）。

- **多波束测深声呐 (Multi-Beam Echo Sounder, MBES):** 作为主要的外部感知传感器，MBES能够生成一整片扇形区域的深度测量数据，从而创建海床的三维点云 [14]。这是AUV观察外部世界的“眼睛”。

- **关键的缺失：** 用户明确排除了声学定位系统（如长基线LBL或超短基线USBL）。这些系统通过对声学应答器的信号进行三角测量来提供绝对位置修正，但它们需要预先部署设备，并限制了作业区域 [10]。这一限制使得导航问题变得异常困难。

此外，“无特征”环境带来了双重挑战。首先，海床本身可能在物理上是单调的（如大片的沙地或淤泥），为传统的SLAM算法提供了极少可供重定位的独特地标 [19]。其次，即使存在地貌特征，与陆地上的激光雷达（LiDAR）等传感器相比，声呐数据本身也充满噪声、分辨率低且易产生伪影，这使得特征提取变得非常困难 [20]。

### 1.3 深层分析与范式转换

AUV面临的根本问题不仅仅是“迷路”，而是一场“定位依据”（grounding）的危机。传统方法试图将AUV的位置“锚定”在一个绝对坐标系中（这在没有GPS/LBL的情况下是不可能的），或者通过匹配地图中的离散特征（这在无特征地形中会失败）。具身智能提供了一条新的路径：将AUV的知识“锚定”于其自身的**传感器-运动经验**中。

这种范式转换的逻辑链条如下：

1. AUV的核心问题是整合带有噪声的惯性/速度数据所导致的无界漂移 [12]。

2. 传统的解决方案（基于特征的SLAM、声学信标）在给定场景下被明确排除或无效 [19]。

3. 这意味着AUV只有两种信息来源：其内部运动传感器（通过INS/DVL获得的本体感受）和其外部环境传感器（通过多波束声呐获得的外部感受）。

4. 具身智能的核心原则是感知-行动循环，它将本体感受和外部感受紧密耦合 [2]。一个具身智能体学习一个关于P(next_sensation∣current_sensation, action) 的模型。

5. 因此，与其尝试在全局框架中建立地图，一个具身的AUV可以学习一个**局部的、以自我为中心的、以行动为条件的**世界模型。例如，它可以学习到“当我的声呐看到这个平缓的斜坡，并且我发出向前1米/秒的推力指令时，我的DVL应该记录到0.9米/秒的速度，并且我的声呐视野应该以这种可预测的方式变化”。

6. 这个学习到的模型提供了一个强大的**一致性检查**机制。当AUV回到一个位置时，它不是通过匹配特征来识别它，而是通过测试其学习到的传感器-运动模型是否仍然成立来识别。在**交互动力学**上的匹配为环路闭合和漂移校正提供了一种鲁棒的方法，从根本上将问题从几何匹配转变为动态模型验证。这标志着从“我在哪里？”到“这个地方的行为是否如我所记忆的那样？”的范式转变。

## 2 具身环境感知与世界建模

本部分探讨一个具身的AUV如何超越简单的几何建图，直接从声呐数据中获得对环境更丰富、更有用的理解。

### 2.1 应用构想：从点云到可行动的“功能可见性”

核心思想是，AUV不仅仅是创建一个测深图（一张深度的地图），而是学习以“功能可见性”（affordances）——即环境所提供的行动可能性——来感知环境。这需要一个端到端的模型，将原始声呐数据映射到对海床的语义或功能性理解上。

例如，模型可以将海床区域不仅仅按几何形状分类，而是标记为“高阻力淤泥”、“可通航的硬质沙地”、“具有科学价值的岩石露头”或“潜在的缠绕风险区” [1]。这类似于自动驾驶汽车将道路感知为“可行驶区域”，而不仅仅是一个平面 [1]。

### 2.2 实现方法：借鉴自动驾驶领域的鸟瞰图（BEV）表示法

该方法将稀疏、不规则的点云转换为密集的、结构化的类图像表示，从而能够利用强大的图像处理神经网络。其流程如下：

1. **输入：** 来自多波束声呐的原始三维点云 [14]。

2. **投影：** 将三维点云从自上而下的“鸟瞰视角”（Bird's-Eye-View, BEV）投影到一个二维网格上。网格中的每个单元可以存储多个信息通道，例如平均高程、最大高程、点密度和声呐返回强度。

3. **处理：** 将这个多通道的BEV“图像”输入到一个卷积神经网络（CNN）或视觉变换器（Vision Transformer, ViT）中进行语义分割 [24]。训练网络为BEV地图中的每个像素分配一个类别标签（例如，“沙地”、“岩石”）。

尽管目前没有直接针对AUV声呐到BEV语义分割的开源项目，但整个方法论是陆地激光雷达感知在自动驾驶领域的直接类比。可以参考该领域的项目和技术 [26]。一篇关于使用CNN从水下点云中检测块状物的论文证明了处理此类数据的可行性 [28]。

### 2.3 研究方向：面向声呐的自监督世界模型

“世界模型”是智能体学习到的一个内部模型，使其能够根据当前状态和计划的行动来预测未来的感官输入 [8]。这是高级具身智能的基石。

**实现方式：** 可以通过自监督任务来训练AUV。具体任务是：给定当前的声呐BEV地图和一个计划的电机指令（例如，“向前推进1米”），预测**下一个**声呐BEV地图。通过在大量的真实AUV任务数据（例如，来自AURORA数据集 [30]）上训练一个生成模型（如生成对抗网络GAN或变分自编码器VAE），AUV可以学习到当它在不同类型的地形上移动时，其声呐视图变化的“物理规律”。

**优势：** 这个学习到的世界模型功能强大。它可以用于规划（通过“想象”不同行动序列的后果）、异常检测（如果真实世界突然偏离模型的预测），并为下游任务（如SLAM和导航）提供丰富的、学习到的特征表示。

### 2.4 深层分析与知识迁移

BEV表示法不仅仅是一种方便的数据结构，它是一座关键的桥梁，连接了成熟、资金雄厚的自动驾驶研究领域与小众、数据匮乏的AUV自主技术领域。它实现了直接的知识和架构迁移。

其内在逻辑如下：

1. AUV自主技术面临一个“冷启动”问题：研究人员较少，数据量不足，标准化工具链缺乏。

2. 自动驾驶汽车领域已基本收敛于一个感知范式：使用激光雷达/摄像头创建BEV表示，然后应用强大的CNN/Transformer进行感知和规划 [26]。一个巨大的研发生态系统已经围绕这个范式建立起来。

3. AUV的多波束声呐产生的三维点云，在结构上与激光雷达点云相似 [14]。

4. 因此，通过采用BEV投影这一步骤 [28]，AUV研究人员可以立即利用来自自动驾驶领域的架构、预训练模型（通过迁移学习）乃至开源代码库。

5. 这建立了一条因果链：采用特定的数据表示法（BEV）**解锁**了一个庞大的现有解决方案库，从而极大地加速了AUV感知研究。它将问题从“从零开始发明水下感知”转变为“将成熟的陆地感知技术应用于水下领域”，这是一个更易于处理的问题。

## 3 具身SLAM：在无特征的深海中学习定位与建图

本部分通过提出从传统几何SLAM到现代基于学习的隐式方法的转变，来解决核心的导航问题——无界漂移。

### 3.1 应用构想：面向声呐的神经隐式SLAM

**传统SLAM的问题：** 传统的SLAM方法，如点云配准（例如，ICP）或基于特征的方法（例如，ORB-SLAM），在水下环境中举步维艰。它们需要清晰、可重复的特征，而这在单调的地形中非常稀缺，并且对声呐数据的高噪声和低分辨率非常敏感 [19]。

**隐式表示解决方案：** 神经隐式地图不将地图存储为离散的点或特征集合，而是将其表示为一个连续函数，通常是一个神经网络（多层感知器MLP），该网络将三维坐标 (x,y,z) 映射到一个值（例如，符号距离函数SDF） [33]。网络

**学习环境的形状**。

**对AUV的优势：** 这种方法天然地对稀疏和嘈杂的数据具有鲁棒性，因为网络会对信息进行插值和平滑。它不需要在单个点之间寻找显式的对应关系，而是将传入的点云与学习到的连续场对齐。这对于特征稀疏、噪声大的声呐场景是理想的选择。

### 3.2 实现方法：借鉴最先进的激光雷达SLAM框架

**架构蓝图：** 建议借鉴并改造一个类似**PIN-SLAM** [34]或

**Point-SLAM** [33]的框架。

- **建图 (Mapping):** 系统维护一组“神经点”，每个点都有一个关联的特征向量和位置。在任何查询点的隐式函数（SDF）值是通过对附近神经点的特征进行插值来计算的。随着AUV的移动，新的声呐扫描数据被用来优化这些神经点的位置和特征，从而不断精化地图。

- **跟踪 (Tracking/Localization):** 为了找到自身位姿，AUV通过优化其位置和方向，使得当前声呐扫描中的点能够最好地与学习到的SDF的零水平面（即表面）对齐。这是一个无需特征点对应的优化过程。

- **环路闭合 (Loop Closure):** 当AUV重访一个区域时，系统可以通过比较当前局部地图中神经点的特征与全局地图中的特征来检测到这一点。成功的环路闭合会触发一次全局位姿图优化，该优化会使整个地图变形以校正累积的漂移。

**开源项目参考：** **PIN-SLAM**的GitHub仓库 [34]是进行改造的理想候选。它专为激光雷达设计，能处理稀疏点云，包含ROS接口，并且是开源的。改造任务将集中于调整其输入阶段以处理多波束声呐的数据格式，并针对声呐不同的噪声特性调整超参数。

### 3.3 研究方向：基于强化学习的主动SLAM

**概念：** AUV不再是被动地在其被告知要去的区域进行建图，而是可以利用强化学习来学习一种**探索策略**，主动地寻求改善其自身定位的确定性。这被称为主动SLAM（Active SLAM） [36]。

**实现方式：** RL智能体的目标是最小化其位姿估计的不确定性（例如，来自SLAM滤波器的协方差）。

- **状态 (State):** 当前的地图表示、AUV的位姿不确定性。

- **行动 (Action):** 一个高层次的导航目标（例如，“向北移动50米”）。

- **奖励 (Reward):** 一个奖励函数，该函数惩罚高的位姿不确定性，并鼓励探索新区域。

**预期行为：** AUV会学习到，例如，当它对自己的位置非常不确定时，它应该尝试以一种能够产生重叠声呐扫描条带的方式移动，或者去寻找地形梯度更大的区域，因为这些行动能为SLAM后端提供更多信息以减少漂移。这是一种具身行为，智能体的行动直接旨在改善其感知能力。

### 3.4 深层分析与机制变革

向隐式SLAM的转变从根本上重新定义了“环路闭合”，使其独特地适用于无特征环境。它将问题从“找到相同的离散地标”转变为“找到一个具有相同学习到的几何结构的区域”。

其内在逻辑如下：

1. 在AUV场景下，SLAM的主要失败模式是由于缺乏显著、可重复的地标而无法执行环路闭合 [19]。

2. 传统的环路闭合依赖于数据关联：将时间 t 的一组特征（如SIFT、ORB）或点云扫描与之前时间 t−k 的一组特征进行匹配。这是一个离散匹配问题。

3. 神经隐式方法 [33] 不存储离散特征。它们存储一个由神经网络权重表示的连续函数

   f(x,y,z)→SDF。

4. 因此，环路闭合不再是关于匹配离散点。它是关于找到一个新的位姿 Pt​，使得当前的声呐扫描 St​ 与先前学习到的地图函数 f 高度一致。这种一致性可以通过当用 Pt​ 变换后，St​ 中的点在 f 的零平面上的拟合优度来衡量。

5. 这是一个鲁棒得多的标准。一个平缓、无特征的斜坡仍然可以被学习到的函数 f 很好地描述。如果AUV返回到同一个斜坡，其新的声呐扫描将很好地拟合这个学习到的函数，即使没有一个单独的点或特征是唯一可识别的。

6. 这意味着隐式SLAM可以在传统方法失败的地方取得成功。其因果关系在于，**地图的表示方式**（连续函数 vs. 离散点）直接促成了一种更鲁棒的**定位机制**（函数拟合 vs. 特征匹配），从而解决了在特征稀疏环境中环路闭合的核心难题。

## 4 具身导航与端到端路径规划

本部分聚焦于最高层次的自主性：学习直接从感知到行动的导航策略，体现完整的具身智能循环。

### 4.1 应用构想：基于DRL的“驾驶员”

**核心思想：** 训练一个单一的深度强化学习（DRL）智能体，充当AUV的“驾驶员”。这个智能体学习一个端到端的策略，将原始传感器输入直接映射到低级的电机指令（推进器推力），以导航至目标点，同时避开障碍物并遵守载具的动力学约束 [26]。

**与模块化方法的对比：** 这取代了传统的“感知-规划-行动”流水线（其中独立的模块分别处理感知、路径规划和控制），代之以一个统一的、学习得到的策略。这允许对所有组件进行联合优化，并能产生更具反应性和鲁棒性的行为 [26]。

### 4.2 实现方法：一个详细的AUV DRL框架

- **算法选择：** 适用于连续动作空间的Actor-Critic算法是合适的选择，例如**软演员-评论家（Soft Actor-Critic, SAC）** [38]或

  **双延迟深度确定性策略梯度（Twin-Delayed Deep Deterministic Policy Gradient, TD3）** [39]。这些算法在探索和利用之间取得了很好的平衡，是连续控制任务的当前最佳实践。

- **状态空间（网络输入）：** 状态向量必须为智能体提供做出决策所需的所有信息。一个精心设计的状态应包括（借鉴自 [38]）：

  - **环境感知：** 多波束声呐数据的压缩表示。这可以是来自第二部分BEV感知CNN倒数第二层的特征向量，或者是BEV网格本身的展平向量。一些论文直接使用原始的声呐波束距离 [38]。

  - **任务信息：** 到目标的相对位置向量（距离和方位）。

  - **本体感受状态：** 来自DVL/INS的AUV当前线速度和角速度。这对智能体理解自身动力学至关重要。

- **动作空间（网络输出）：** 一个连续向量，其中每个元素对应于AUV一个执行器的归一化推力（例如，`[纵向推力, 横向推力, 偏航力矩]`）。网络输出值在 [−1,1] 范围内，然后缩放到推进器的物理极限 [38]。

- **奖励函数（关键组成部分）：** 奖励函数必须被精心设计，以引导智能体朝向期望的行为。通常需要一个复合函数 [39]：

  - Rgoal​：到达目标点时给予一个大的正奖励。一个基于到目标距离负变化的塑形项（shaping term）可以鼓励前进。

  - Rcollision​：与障碍物碰撞（或进入最小安全距离）时给予一个大的负惩罚。

  - Renergy​：一个与动作向量大小的平方成正比的小负奖励。这会惩罚大的推进器指令，鼓励节能和平滑控制。

  - Rtime​：每个时间步给予一个小的负奖励，鼓励智能体快速完成任务。

### 4.3 研究方向：通过模仿学习克服稀疏奖励

**问题：** 从零开始训练一个RL智能体可能样本效率极低，特别是当目标很远时（即“稀疏奖励”问题）。智能体可能需要进行数百万步的随机探索，才能偶然碰到目标并获得一次正奖励 [37]。

**解决方案：生成对抗模仿学习（Generative Adversarial Imitation Learning, GAIL）：** 该技术将模仿学习与强化学习相结合 [38]。

1. **收集专家数据：** 首先，收集一个小型的“专家”轨迹数据集（例如，由熟练的人类操作员遥控AUV得到）。

2. **对抗性训练：** 训练一个判别器网络，以区分来自专家数据的状态-动作对和由RL智能体策略生成的状态-动作对。然后，RL智能体（生成器）不仅会因环境的奖励函数而获得奖励，还会因“欺骗”判别器而获得奖励。

3. **优势：** 这提供了一个密集的、连续的学习信号。智能体被持续引导去模仿专家的行为，极大地加速了学习的初始阶段，并确保它能快速发现一个合理的策略。之后，该策略可以通过特定于任务的RL奖励函数进行微调，以超越专家的表现 [44]。研究表明，SAC与GAIL的结合尤其有效 [38]。

### 4.4 深层分析与课程设计

端到端DRL在AUV导航中的成功，其关键不在于算法的选择（TD3、SAC等已相当成熟），而在于**课程设计（curriculum design）**。真正的挑战在于如何构建学习问题，使智能体能够逐步成功。

其内在逻辑如下：

1. AUV任务非常复杂：高维状态空间（声呐）、连续动作空间和稀疏奖励。直接应用DRL很可能失败 [37]。

2. 多篇论文提出的解决方案，本质上都是某种形式的课程学习。

3. **通过模仿的课程：** GAIL预训练 [38] 就是一种课程。它让智能体从一个简单的任务（“模仿专家”）开始，然后再转向更难的任务（“为目标进行优化”）。

4. **通过奖励塑形的课程：** 设计一个复杂的复合奖励函数 [39] 也是一种课程。它为智能体提供中间的“提示”（例如，“你正在靠近目标”、“你消耗了太多能量”），而不仅仅是在最后给予一个单一的奖励。

5. **通过任务简化的课程：** 一些研究从更简单的模型或环境开始，然后逐渐增加复杂性 [46]。对于AUV，可以先在无障碍环境中训练，然后添加静态障碍物，最后再加入动态洋流。

6. **通过人机交互的课程：** 交互式DRL，即人类可以提供偶尔的纠正性反馈，是另一种强大的课程方法 [47]。

因此，通往成功的因果路径并非简单的 `DRL -> 解决方案`，而是 `问题复杂性 -> DRL失败 -> 课程设计（模仿/奖励/任务简化） -> DRL成功`。对于实践者而言，关键的启示是，应将精力集中在设计一个有效的训练课程上，将复杂的AUV导航问题分解为一系列可学习的步骤，而不仅仅是选择一个新颖的RL算法。

## 5 综合、重大挑战与战略路线图

本部分综合前述内容，探讨 overarching 的实践挑战，并为未来的研发提供战略路线图。

### 5.1 一体化的具身AUV架构

一个理想的EAI驱动的AUV架构将呈现出高度整合的特征。多波束声呐数据被送入一个共享的感知编码器（例如，BEV CNN）。该编码器的输出同时作为隐式SLAM模块（用于定位和建图）和DRL导航策略（用于动作选择）的状态表示。这突显了从刚性流水线向更集成化架构的转变，在这种架构中，各个模块共享一个共同的、学习到的世界理解。

### 5.2 仿真到现实的鸿沟：主要障碍

部署基于学习的AUV控制器的最大障碍是仿真与现实之间的差距（sim-to-real gap） [48]。在仿真中训练的策略在真实世界中常常失败，因为仿真器无法完美捕捉现实。

**水下环境的特殊性：** 这个问题对于AUV尤为严重，原因包括：

- **复杂的水动力学：** 精确模拟流体动力学、附加质量、阻力以及推进器冲刷效应，在计算上非常昂贵且难以建模 [51]。

- **声呐物理学：** 模拟声呐比模拟相机或激光雷达更难。它需要对声波传播、吸收、散射、多径反射以及换能器的特定波束模式进行建模，所有这些都高度依赖于水的盐度、温度等特性 [20]。

**仿真器评估：**

- **UUV Simulator (基于Gazebo):** 一个开源、兼容ROS的仿真器。它被广泛使用，但其声呐插件通常基于简化模型，如深度相机的射线投射，可能无法捕捉到必要的物理保真度 [51]。它是控制和动力学研究的一个良好起点，但可能不足以训练感知模型。

- **HoloOcean (基于Unreal Engine):** 一个较新的开源仿真器，优先考虑高保真渲染和传感器模拟。它具有一个新颖的声呐模拟框架，明确地对波束模式和噪声进行建模，使其成为训练“感知在环”策略的更有力候选者 [58]。

**缓解策略：**

- **域随机化 (Domain Randomization):** 在仿真训练期间，系统地随机化物理参数（如水的浊度、洋流强度、传感器噪声水平、载具质量）。这迫使策略对变化具有鲁棒性，减少对任何单一仿真实例特定参数的依赖 [49]。

- **系统辨识 (System Identification):** 使用真实世界的数据来学习一个更准确的AUV动力学模型或声呐噪声模型，然后将这个学习到的模型整合回仿真器中（Real2Sim）。

### 5.3 板载计算与功率限制

**问题：** 深度学习模型，特别是用于感知和RL的大型模型，计算量巨大。而AUV是功率受限的平台，搭载高端GPU的空间有限 [62]。

**解决方案：**

- **边缘计算/卸载：** 尽管一些研究探索了将计算卸载到水面船只或无人机上 [63]，但这对于用户场景中的完全自主、无缆AUV是不可行的。

- **板载硬件：** AUV必须配备嵌入式GPU，例如NVIDIA Jetson系列模块 [66]。

- **模型优化：** 训练好的神经网络必须为高效推理进行优化。这包括以下技术：

  - **量化 (Quantization):** 使用较低精度的数字（例如，用16位或8位整数代替32位浮点数）来减少内存和计算需求 [62]。

  - **剪枝 (Pruning):** 移除网络中冗余的权重或神经元。

  - **知识蒸馏 (Knowledge Distillation):** 训练一个更小、更高效的“学生”网络来模仿一个更大、更强大的“教师”网络的输出。

### 5.4 战略研究路径与建议

**表 1: 用于具身AUV开发的开源资源与类比系统**

| 类别         | 资源名称 / 项目                | 核心技术             | 对AUV的关联性与类比价值                                               | 链接 / 来源                                            |
| ---------- | ------------------------ | ---------------- | ----------------------------------------------------------- | -------------------------------------------------- |
| **仿真器**    | HoloOcean                | Unreal Engine [4]  | 高保真声呐模拟，对sim-to-real至关重要                                    | [59]                                                 |
|            | UUV Simulator            | Gazebo / ROS     | 广泛使用的ROS兼容水下仿真框架，适合控制与动力学开发                                 | [67]                                                 |
| **数据集**    | AURORA Dataset           | 多波束声呐 & 导航数据     | 用于训练/测试感知和SLAM算法的真实世界数据                                     | [30]                                                 |
|            | OpenSonarDatasets        | 数据集集合            | 提供一个集中的目录来查找各种开源声呐数据集                                       | [69]                                                 |
| **SLAM框架** | PIN-SLAM                 | 隐式神经SLAM (LiDAR) | 基于学习的声呐SLAM的直接架构蓝图，适用于稀疏点云                                  | `github.com/PRBonn/PIN_SLAM` [34]                    |
|            | Point-SLAM               | 神经点云SLAM (RGB-D) | 另一种先进的隐式SLAM架构，可作为声呐SLAM的参考                                 | `github.com/eriksandstroem/Point-SLAM` [33]          |
| **RL导航框架** | DRL-robot-navigation     | TD3算法 (LiDAR)    | 从传感器数据到电机指令的端到端DRL导航的参考实现                                   | `github.com/reiniscimurs/DRL-robot-navigation` [40]  |
|            | cranfield-navigation-gym | Gymnasium / ROS  | 提供了ROS/Gazebo与RL框架（如Stable Baselines3）之间的接口模型，可用于构建AUV-声呐接口 | `github.com/mazqtpopx/cranfield-navigation-gym` [70] |
| **RL探索框架** | Active-SLAM-Paper-List   | 主动SLAM / RL探索    | 提供了大量关于基于RL的主动SLAM和探索的论文与代码链接                               | `github.com/DoongLi/awesome-Active-SLAM` [36]        |

基于以上分析，提出以下分层战略研究路径：

- **第一层（短期，高可行性）：**

  - **改造LiDAR SLAM：** 集中精力改造现有的开源隐式LiDAR SLAM系统（如PIN-SLAM [34]），使其能够使用多波束声呐数据。这是一个定义明确的工程和研究任务，直接解决了核心的漂移问题。

  - **开发高保真声呐数据集：** 收集并发布大型、标注良好的多波束声呐和导航数据集（类似于扩展版的AURORA [30]）。这对于训练和基准测试所有基于学习的方法至关重要 [53]。
- **第二层（中期，有前景）：**

  - **混合式RL导航：** 在HoloOcean [59]等高保真仿真器中开发基于DRL的导航智能体。重点关注混合模仿/强化学习 [38]和先进的课程设计，使问题变得易于处理。

  - **Sim-to-Real迁移：** 针对AUV进行专注的sim-to-real迁移研究，特别是比较不同域随机化策略对声呐和水动力学模型的有效性。
- **第三层（长期，愿景）：**

  - **水下世界模型：** 研究开发面向水下领域的大规模、自监督世界模型。这类似于正在为语言和视觉领域开发的“基础模型” [8]，代表了在该领域向通用人工智能（AGI）迈出的一步 [2]。

  - **端到端具身SLAM：** 开发完全端到端的SLAM系统，其中从传感器输入到地图表示和位姿更新的整个过程都是一个单一的可微神经网络，并可能用RL进行训练 [36]。

## 6 参考

[1] What is Embodied AI? A Guide to AI in Robotics - Encord, 访问时间为六月 24, 2025， <https://encord.com/blog/embodied-ai/>
[2] Embodied Intelligence: The Key to Unblocking Generalized Artificial Intelligence - arXiv, 访问时间为六月 24, 2025， <https://arxiv.org/html/2505.06897v1>
[3] A Brief History of Embodied Artificial Intelligence, and its Outlook, 访问时间为六月 24, 2025， <https://cacm.acm.org/blogcacm/a-brief-history-of-embodied-artificial-intelligence-and-its-future-outlook/>
[4] Embodied Artificial Intelligence - University of Cambridge, 访问时间为六月 24, 2025， <https://www.repository.cam.ac.uk/bitstreams/c2b0c4d6-940f-43e8-9e01-6b49f79a7e09/download>
[5] Embodied AI Explained: Principles, Applications, and Future Perspectives, 访问时间为六月 24, 2025， <https://lamarr-institute.org/blog/embodied-ai-explained/>
[6] [Literature Review] Toward Embodied AGI: A Review of Embodied AI and the Road Ahead, 访问时间为六月 24, 2025， <https://www.themoonlight.io/en/review/toward-embodied-agi-a-review-of-embodied-ai-and-the-road-ahead>
[7] What Is Embodied AI? - Artificial Intelligence - Built In, 访问时间为六月 24, 2025， <https://builtin.com/artificial-intelligence/embodied-ai>
[8] What is Embodied AI? | NVIDIA Glossary, 访问时间为六月 24, 2025， <https://www.nvidia.com/en-us/glossary/embodied-ai/>
[9] (PDF) Embodied Imitation-Enhanced Reinforcement Learning in Multi-Agent Systems, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/260066786_Embodied_Imitation-Enhanced_Reinforcement_Learning_in_Multi-Agent_Systems>
[10] Underwater Autonomous Navigation with Boxfish AUVs, 访问时间为六月 24, 2025， <https://www.boxfishrobotics.com/autonomy/navigating-the-depths-boxfish-robotics-advanced-auv-navigation-system/>
[11] A Complete Guide to Underwater Navigation - Nortek, 访问时间为六月 24, 2025， <https://www.nortekgroup.com/knowledge-center/wiki/new-to-subsea-navigation>
[12] Autonomous Underwater Vehicle Navigation Advancements, 访问时间为六月 24, 2025， <https://www.pnisensor.com/advancements-in-autonomous-underwater-vehicle-navigation/>
[13] Autonomous Underwater Vehicles: Localization, Navigation, and Communication for Collaborative Missions - MDPI, 访问时间为六月 24, 2025， <https://www.mdpi.com/2076-3417/10/4/1256>
[14] Seafloor Mapping AUV • MBARI, 访问时间为六月 24, 2025， <https://www.mbari.org/technology/seafloor-mapping-auv/>
[15] (PDF) AUV Navigation Based on Inertial Navigation and Acoustic Positioning Systems, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/330294959_AUV_Navigation_Based_on_Inertial_Navigation_and_Acoustic_Positioning_Systems>
[16] AUV Navigation Correction Based on Automated Multibeam Tile Matching - PMC, 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC8840710/>
[17] An Integrated Navigation Method Aided by Position Correction Model and Velocity Model for AUVs - MDPI, 访问时间为六月 24, 2025， <https://www.mdpi.com/1424-8220/24/16/5396>
[18] Terrain Referenced Navigation of AUVs and Submarines Using Multibeam Echo Sounders - NavLab. net, 访问时间为六月 24, 2025， <https://www.navlab.net/Publications/Terrain_Referenced_Navigation_of_AUVs_and_Submarines_Using_Multibeam_Echo_Sounders.pdf>
[19] Underwater SLAM: Challenges, State of the Art, Algorithms and a New Biologically-Inspired Approach, 访问时间为六月 24, 2025， <http://vigir.missouri.edu/~gdesouza/Research/Conference_CDs/BioRob_2014/media/files/0075.pdf>
[20] Underwater SLAM Meets Deep Learning: Challenges, Multi-Sensor ..., 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC12157327/>
[21] Underwater SLAM: Challenges, state of the art, algorithms and a new biologically-inspired approach - ResearchGate, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/286724450_Underwater_SLAM_Challenges_state_of_the_art_algorithms_and_a_new_biologically-inspired_approach>
[22] Sonar-Based Simultaneous Localization and Mapping Using the Semi-Direct Method - MDPI, 访问时间为六月 24, 2025， <https://www.mdpi.com/2077-1312/12/12/2234>
[23] Advancements in Sensor Fusion for Underwater SLAM: A Review on Enhanced Navigation and Environmental Perception - PubMed Central, 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC11644431/>
[24] Enhancing Underwater SLAM Navigation and Perception: A Comprehensive Review of Deep Learning Integration - ResearchGate, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/385483918_Enhancing_Underwater_SLAM_Navigation_and_Perception_A_Comprehensive_Review_of_Deep_Learning_Integration>
[25] Enhancing Underwater SLAM Navigation and Perception: A Comprehensive Review of Deep Learning Integration - MDPI, 访问时间为六月 24, 2025， <https://www.mdpi.com/1424-8220/24/21/7034>
[26] End-to-end Autonomous Driving using Deep Learning: A Systematic Review - arXiv, 访问时间为六月 24, 2025， <https://arxiv.org/pdf/2311.18636>
[27] runjtu/awesome-and-novel-works-in-slam: collecting some new ideas for slam (Semantic, 3DGS, BEV, Nav, LLM, Multi-session) and will update this repo weekly, for both engineering and academic usings. - GitHub, 访问时间为六月 24, 2025， <https://github.com/runjtu/awesome-and-novel-works-in-slam>
[28] Deep-Learning-Based Three-Dimensional Detection of Individual Wave-Dissipating Blocks from As-Built Point Clouds Measured by UAV Photogrammetry and Multibeam Echo-Sounder - MDPI, 访问时间为六月 24, 2025， <https://www.mdpi.com/2072-4292/14/21/5575>
[29] Fundamentals and Core Technologies for Embodied AI Overview, 访问时间为六月 24, 2025， <https://www.jst.go.jp/kisoken/boshuu/teian/en/koubo/2025houshin_cemb_en.pdf>
[30] AURORA, A multi sensor dataset for robotic ocean exploration - GitHub, 访问时间为六月 24, 2025， <https://github.com/noc-mars/aurora>
[31] AURORA, A multi sensor dataset for robotic ocean exploration - NERC Open Research Archive, 访问时间为六月 24, 2025， <https://nora.nerc.ac.uk/532651/1/AURORA__A_multi_sensor_dataset_for_robotic_ocean_exploration.pdf>
[32] Robust Underwater Visual SLAM Fusing Acoustic Sensing - Elizabeth Vargas, 访问时间为六月 24, 2025， <https://evargasv.github.io/publications/conferences/vargas2021_icra.pdf>
[33] Point-SLAM: Dense Neural Point Cloud-based SLAM - GitHub, 访问时间为六月 24, 2025， <https://github.com/eriksandstroem/Point-SLAM>
[34] PRBonn/PIN_SLAM: PIN-SLAM: LiDAR SLAM Using a Point-Based Implicit Neural Representation for Achieving Global Map Consistency [TRO' 24] - GitHub, 访问时间为六月 24, 2025， <https://github.com/PRBonn/PIN_SLAM>
[35] 3D-Vision-World/awesome-NeRF-and-3DGS-SLAM - GitHub, 访问时间为六月 24, 2025， <https://github.com/3D-Vision-World/awesome-NeRF-and-3DGS-SLAM>
[36] This repository primarily organizes papers, code, and other relevant materials related to Active SLAM and Robotic Exploration. - GitHub, 访问时间为六月 24, 2025， <https://github.com/DoongLi/awesome-Active-SLAM>
[37] End-to-End AUV Local Motion Planning Method Based on Deep Reinforcement Learning, 访问时间为六月 24, 2025， <https://www.mdpi.com/2077-1312/11/9/1796>
[38] End-to-End AUV Motion Planning Method Based on Soft Actor-Critic ..., 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC8434076/>
[39] Deep reinforcement learning for adaptive path planning and control of Autonomous Underwater Vehicle - University of Hertfordshire Research Archive, 访问时间为六月 24, 2025， <https://uhra.herts.ac.uk/id/eprint/10051/1/APOR_2022.pdf>
[40] reiniscimurs/DRL-robot-navigation: Deep Reinforcement ... - GitHub, 访问时间为六月 24, 2025， <https://github.com/reiniscimurs/DRL-robot-navigation>
[41] (PDF) Path Planning Based on Deep Reinforcement Learning for Autonomous Underwater Vehicles Under Ocean Current Disturbance - ResearchGate, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/358947954_Path_Planning_based_on_Deep_Reinforcement_Learning_for_Autonomous_Underwater_Vehicles_under_Ocean_Current_Disturbance>
[42] AUV Obstacle Avoidance Planning Based on Deep Reinforcement Learning - MDPI, 访问时间为六月 24, 2025， <https://www.mdpi.com/2077-1312/9/11/1166>
[43] Improved Artificial Potential Field Algorithm Assisted by Multisource Data for AUV Path Planning - PMC, 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC10422249/>
[44] Multi-Agent Generative Adversarial Interactive Self-Imitation Learning for AUV Formation Control and Obstacle Avoidance - arXiv, 访问时间为六月 24, 2025， <https://arxiv.org/html/2401.11378v1>
[45] Noisy Dueling Double Deep Q-Network algorithm for autonomous underwater vehicle path planning - PubMed Central, 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC11513341/>
[46] Embodied Escaping: End-to-End Reinforcement Learning for Robot Navigation in Narrow Environment - arXiv, 访问时间为六月 24, 2025， <https://arxiv.org/html/2503.03208v1>
[47] Action Guidance-Based Deep Interactive Reinforcement Learning for AUV Path Planning, 访问时间为六月 24, 2025， <https://www.scribd.com/document/825970176/Action-Guidance-Based-Deep-Interactive-Reinforcement-Learning-for-AUV-Path-Planning>
[48] What exactly makes sim to real transfer a challenge in reinforcement learning? : r/robotics, 访问时间为六月 24, 2025， <https://www.reddit.com/r/robotics/comments/1j99vrt/what_exactly_makes_sim_to_real_transfer_a/>
[49] Sim2Real in Robotics and Automation: Applications and Challenges - DSpace@MIT , 访问时间为六月 24, 2025， <https://dspace.mit.edu/bitstream/handle/1721.1/138850/2021-04-Sim2Real_T-ASE.pdf>
[50] Reinforcement Learning in Robotics: Exploring Sim-to-Real Transfer, Imitation Learning, and Transfer Learning Techniques - ResearchGate, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/385885092_Reinforcement_Learning_in_Robotics_Exploring_Sim-to-Real_Transfer_Imitation_Learning_and_Transfer_Learning_Techniques>
[51] Underwater Robotic Simulators Review for Autonomous System Development - arXiv, 访问时间为六月 24, 2025， <https://arxiv.org/html/2504.06245v1>
[52] A Survey on Reinforcement Learning Methods in Bionic Underwater Robots - PMC, 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC10123646/>
[53] Sonar-based Deep Learning in Underwater Robotics: Overview, Robustness and Challenges - arXiv, 访问时间为六月 24, 2025， <https://arxiv.org/html/2412.11840v1>
[54] Ray-Based Physical Modeling and Simulation of Multibeam Sonar for Underwater Robotics in ROS-Gazebo Framework - ResearchGate, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/389457811_Ray-Based_Physical_Modeling_and_Simulation_of_Multibeam_Sonar_for_Underwater_Robotics_in_ROS-Gazebo_Framework>
[55] Ray-Based Physical Modeling and Simulation of Multibeam Sonar for Underwater Robotics in ROS-Gazebo Framework - PMC - PubMed Central, 访问时间为六月 24, 2025， <https://pmc.ncbi.nlm.nih.gov/articles/PMC11902455/>
[56] uuv_gazebo_plugins - Unmanned Underwater Vehicle Simulator Documentation, 访问时间为六月 24, 2025， <https://uuvsimulator.github.io/packages/uuv_simulator/docs/packages/uuv_gazebo_plugins/>
[57] GazeboRosImageSonar - Unmanned Underwater Vehicle Simulator Documentation, 访问时间为六月 24, 2025， <https://uuvsimulator.github.io/packages/uuv_simulator/docs/api/gazebo::GazeboRosImageSonar/>
[58] HoloOcean: A Full-Featured Marine Robotics Simulator for Perception and Autonomy | Request PDF - ResearchGate, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/383187592_HoloOcean_A_Full-Featured_Marine_Robotics_Simulator_for_Perception_and_Autonomy>
[59] HoloOcean - BYU, 访问时间为六月 24, 2025， <https://robots.et.byu.edu/holoocean/>
[60] HoloOcean Underwater Simulator - FRoSt Lab, 访问时间为六月 24, 2025， <https://frostlab.byu.edu/holoocean-underwater-simulator>
[61] Welcome to HoloOcean's documentation! - GitHub Pages, 访问时间为六月 24, 2025， <https://byu-holoocean.github.io/holoocean-docs/>
[62] Mixed Precision Deep Reinforcement Learning for Control of Unmanned Undersea Vehicles - DTIC, 访问时间为六月 24, 2025， <https://apps.dtic.mil/sti/trecms/pdf/AD1200547.pdf>
[63] Deep Reinforcement Learning Based Computation Offloading in UAV-Assisted Edge Computing - MDPI, 访问时间为六月 24, 2025， <https://www.mdpi.com/2504-446X/7/3/213>
[64] (PDF) Deep Reinforcement Learning for Computation Offloading and Resource Allocation in Unmanned-Aerial-Vehicle Assisted Edge Computing - ResearchGate, 访问时间为六月 24, 2025， <https://www.researchgate.net/publication/355221952_Deep_Reinforcement_Learning_for_Computation_Offloading_and_Resource_Allocation_in_Unmanned-Aerial-Vehicle_Assisted_Edge_Computing>
[65] The IET Shop - Deep Reinforcement Learning for Reconfigurable Intelligent Surfaces and UAV Empowered Smart 6G Communications, 访问时间为六月 24, 2025， [https://sh](https://shop.theiet.org/deep-reinforcement-learning-for-reconfigurable-intelligent-surfaces-and-uav-empowered-smart-6g-communications)
[66] DeepVL: Dynamics and Inertial Measurements-based Deep Velocity Learning for Underwater Odometry - YouTube, 访问时间为六月 24, 2025， <https://www.youtube.com/watch?v=ctcbrNu_N78>
[67] Project DAVE : ROS2 Wiki, 访问时间为六月 24, 2025， <https://dave-ros2.notion.site/>
[68] Introduction - Unmanned Underwater Vehicle Simulator Documentation, 访问时间为六月 24, 2025， <https://uuvsimulator.github.io/packages/uuv_simulator/intro/>
[69] remaro-network/OpenSonarDatasets: List of open-source sonar datasets. - GitHub, 访问时间为六月 24, 2025， <https://github.com/remaro-network/OpenSonarDatasets>
[70] mazqtpopx/cranfield-navigation-gym: A ROS-based ... - GitHub, 访问时间为六月 24, 2025， <https://github.com/mazqtpopx/cranfield-navigation-gym>
[71] A Comprehensive Survey on Embodied Intelligence: Advancements, Challenges, and Future Perspectives - SciOpen, 访问时间为六月 24, 2025， <https://www.sciopen.com/article/10.26599/AIR.2024.9150042>
[72] ivanalberico/End-To-End-Self-Supervised-SLAM - GitHub, 访问时间为六月 24, 2025， <https://github.com/ivanalberico/End-To-End-Self-Supervised-SLAM>
[73] [op.theiet.org/deep-reinforcement-learning-for-reconfigurable-intelligent-surfaces-and-uav-empowered-smart-6g-communications](https://shop.theiet.org/deep-reinforcement-learning-for-reconfigurable-intelligent-surfaces-and-uav-empowered-smart-6g-communications)
