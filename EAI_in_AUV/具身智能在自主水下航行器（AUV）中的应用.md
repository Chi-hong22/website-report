---
aliases:
  - 具身智能在自主水下航行器（AUV）中的应用
  - 关于水下特殊作业环境中AUV导航、感知与SLAM的前沿调研与思考
tags: [博士/课题, code/AI/具身智能, AUV, 导航, SLAM, BEV, code/AI/强化学习, code/AI/深度强化学习, code/AI/具身智能/VLA, code/AI/具身智能/VLN, code/AI/具身智能/VA]
link: >-
  https://docs.google.com/document/d/1ZTlhS35XQa-FAK93AuYn-eWsvVsZlqKkypzVSOBmPac/edit?usp=sharing
创建时间: 星期三 25日 六月 2025 15:12:31
编辑时间: 星期三 25日 六月 2025 16:23:35
aliases_used: 具身智能在自主水下航行器（AUV）中的应用
---

# 具身智能在自主水下航行器（AUV）中的应用

_——关于水下特殊作业环境中AUV导航、感知与SLAM的前沿调研与思考_

## 1 引言：具身智能与水下自主性的交汇

### 1.1 水下自主性的持久挑战

自主水下航行器（AUV）的运行环境是机器人领域中最具挑战性的领域之一，其固有的物理限制对实现真正的自主性构成了严峻的障碍。这些挑战不仅是技术上的难题，更是定义了AUV导航、感知和测绘（SLAM）问题本质的根本性约束。

1. **GPS信号的缺失**：高频无线电波在水中的衰减极快，导致全球定位系统（GPS）在水下完全失效 ([Nortek, 水下导航指南](https://www.nortekgroup.com/knowledge-center/wiki/new-to-subsea-navigation))。这一事实迫使AUV必须依赖内部传感器进行航位推算（Dead-Reckoning），主要组合是惯性导航系统（INS）和多普勒测速仪（DVL）([Boxfish Robotics, AUV自主导航](https://www.boxfishrobotics.com/autonomy/navigating-the-depths-boxfish-robotics-advanced-auv-navigation-system/))。其核心困境在于"无法解决的漂移问题"。即便是采用光纤陀螺（FOG）或环形激光陀螺（RLG）的高端INS系统，其微小的测量误差在长时间积分后也会导致无界的、持续累积的位置误差([PNI Sensor, AUV导航进展](https://www.pnisensor.com/advancements-in-autonomous-underwater-vehicle-navigation/))。这种漂移的必然性意味着，若无外部信息进行周期性校正，AUV对其自身位置的"信念"将逐渐瓦解，最终在广阔的水域中"迷失方向"([MDPI, AEKF-SLAM算法](https://www.mdpi.com/1424-8220/17/5/1174))。
2. **感知降级的环境**：水下是一个感知被严重降级的环境。与空气中清晰的视野不同，光在水中的传播受到剧烈的吸收和散射影响，导致能见度极低，尤其是在有机物或悬浮物丰富的浑浊水域([ResearchGate, 水下SLAM挑战](https://www.researchgate.net/publication/286724450_Underwater_SLAM_Challenges_state_of_the_art_algorithms_and_a_new_biologically-inspired_approach))。这使得依赖光学相机的视觉方法效果大打折扣。因此，声学传感器，特别是多波束声呐，成为AUV主要的"眼睛"。它能穿透浑浊，获取关键的海底地形和障碍物几何信息([MBARI, 海底测绘AUV](https://www.mbari.org/technology/seafloor-mapping-auv/))。然而，声呐并非完美的替代品，它带来了自身独有的挑战：其数据通常充满散斑噪声、分辨率远低于陆地激光雷达，并且极易因多径效应（声波在海底和水面之间多次反射）产生误导性的伪影([PMC, 水下SLAM与深度学习的结合](https://pmc.ncbi.nlm.nih.gov/articles/PMC12157327/))。
3. **特征稀疏的海底地形**：一个平坦、缺乏显著地貌（如巨石、海沟或人造物）的海底，对传统SLAM算法构成了致命打击([arXiv, AUV不确定性下的实时规划](https://arxiv.org/html/2403.04936v1))。经典SLAM的基石在于提取和匹配独特的、可重复识别的地标来构建地图，并通过"回环检测"——即重新识别到先前访问过的位置——来校正累积的漂移。在一片广阔的沙地或淤泥中，几乎不存在可供稳定追踪的几何特征。这导致了灾难性的**数据关联失败**：算法无法确定当前观测到的一个沙丘波纹是否就是十分钟前看到的那一个([BioRob, 水下SLAM挑战](http://vigir.missouri.edu/~gdesouza/Research/Conference_CDs/BioRob_2014/media/files/0075.pdf))。没有有效的回环检测，定位误差无法被约束，最终导致地图发散，定位完全失效([Missouri University, 水下SLAM挑战](http://docs.google.com/http.vigir.missouri.edu/~gdesouza/Research/Conference_CDs/BioRob_2014/media/files/0075.pdf))。
4. **动态变化的流体环境**：海洋是一个永不停歇的流体介质。洋流、涡流和湍流无时无刻不在对AUV施加着难以精确预测的外部力([PMC, 水下SLAM与深度学习的结合](https://pmc.ncbi.nlm.nih.gov/articles/PMC12157327/))。这些动态扰动不仅给精确的轨迹控制带来了巨大困难，也使得基于简单模型的运动预测变得不可靠，进一步加剧了航位推算的误差累积。

### 1.2 "定位依据"危机：更深层的困境分析

综合上述挑战，AUV面临的根本问题不仅仅是"迷路"，而是一场更深层次的"定位依据"（grounding）的危机。传统方法试图将AUV的位置"锚定"在一个稳定、外部的参照系上——要么是一个绝对坐标系（这在没有GPS或预部署声学信标的情况下是不可能的），要么是地图中的离散几何特征（这在无特征地形中会失败）。当这些外部"锚点"都失效时，AUV的整个"信念状态"就失去了根基，变得无从依托，导致其对自身位置的估计陷入一个不断恶化的恶性循环。

此时，AUV只剩下两种信息来源：其内部运动传感器（通过INS/DVL获得的本体感受，即"我感觉我移动了多少"）和其外部环境传感器（通过多波束声呐获得的外部感受，即"我看到了什么"）([arXiv, 具身智能与通用人工智能](https://arxiv.org/html/2505.06897v1))。没有一个可靠的机制将这两种看似独立的信息源深度耦合起来，无界的漂移问题就是无解的。

### 1.3 范式转移：拥抱具身智能（EAI）

为解决这场"定位依据"危机，具身智能（Embodied Intelligence, EAI）提供了一条革命性的新路径。EAI并非指某一种特定的算法，而是一种根本性的理念框架，它主张真正的智能产生于智能体（agent）的物理身体、传感器、执行器与其所处环境之间持续、双向的交互过程([Encord, 什么是具身AI?](https://encord.com/blog/embodied-ai/))。智能不是在隔离的计算中发生的，而是在与物理世界的动态"对话"中涌现的。

EAI的核心机制是**感知-认知-行动（Perception-Cognition-Action, PCA）循环**([NVIDIA, 什么是具身AI?](https://www.nvidia.com/en-us/glossary/embodied-ai/))。智能体通过传感器**感知**环境，结合任务目标进行**认知**处理，然后执行**行动**。这个行动会改变自身和环境的状态，从而带来新的、可能更有价值的感知，形成一个学习和适应的闭环。正如认知科学家J.J. Gibson的名言——"我们为了行动而感知，我们为了感知而行动"([ResearchGate, 眼-机器人：BC-RL感知-行动循环](https://www.researchgate.net/publication/392629675_Eye_Robot_Learning_to_Look_to_Act_with_a_BC-RL_Perception-Action_Loop))——精辟地指出了行动的主动性角色。

这为AUV带来了深刻的范式转换。与其尝试在全局框架中建立地图，一个具身的AUV可以将知识"锚定"于其自身的**传感器-运动经验**中([Lamarr Institute, 具身AI解释](https://lamarr-institute.org/blog/embodied-ai-explained/))。核心问题从一个静态的、几何学的问题"我在地图上的哪个位置？"，转变为一个动态的、预测性的、更具鲁棒性的问题：

**"这个地方与我交互的方式，是否如我所记忆的那样？"**

例如，AUV可以学习到一个局部的、以自我为中心的、以行动为条件的世界模型："当我声呐看到这个平缓斜坡，并发出向前1米/秒的推力指令时，我的DVL应该记录到约0.9米/秒的速度，并且我的声呐视野应该以一种可预测的方式变化"。当AUV回到一个位置时，它不再是通过匹配稀疏且不可靠的几何特征来识别，而是通过**验证其学习到的交互动力学模型是否仍然成立**来进行识别。这种从**几何匹配**到**动态模型验证**的转变，为在特征稀疏环境中实现可靠的环路闭合和漂移校正提供了强大的新机制，因为它不依赖于任何单一的、静态的地标。

本报告旨在深入探讨并论证，具身智能的原则为解决AUV面临的长期挑战提供了一个强大而统一的理论框架。我们将循序渐进地展开论述，从当前可行的技术（如DRL、主动SLAM）到前瞻性的基础模型，为AUV的未来发展提供一幅清晰的技术路线图。

## 2 具身环境感知：从几何建图到语义理解

传统的AUV感知系统通常扮演着一个被动数据记录者的角色。具身智能范式则要求AUV转变为一个主动的探索者，能够为了更好地"理解"世界而智能地控制其感知过程。

### 2.1 从被动传感到主动感知

主动感知（Active Perception）是具身智能的核心体现之一。智能体能够主动地控制其传感器（通过移动身体）来塑造其观测数据的效用，以最大化地服务于当前任务目标([ResearchGate, 眼-机器人：BC-RL感知-行动循环](https://www.researchgate.net/publication/392629675_Eye_Robot_Learning_to_Look_to_Act_with_a_BC-RL_Perception-Action_Loop))。对于装备了多波束声呐的AUV，主动感知意味着它需要学习如何智能地控制自身的位姿，从而将声呐波束投射到最能解决当前环境不确定性的区域。例如，一个遵循传统"梳状"路径的AUV可能会错过一个被巨大岩石遮挡的重要科学勘测点，而一个具身智能体则可能会在感知到岩石的存在后，自主地规划一条新的路径来"窥探"岩石背后的未知区域。

这种决策过程由严格的数学框架驱动，其核心在于量化**信息增益（Information Gain）**。智能体的目标是选择一个能最大化预期不确定性（通常用信息熵来衡量）降低的行动([University of Michigan, 主动视觉SLAM理论与实验](http://robots.engin.umich.edu/publications/akim-2015a.pdf))。通过这种方式，AUV的行为由内在的"好奇心"驱动，即一种不断寻求信息、消除自身和环境模型不确定性的内在动力。这种策略将感知从一个开放循环的数据收集过程，转变为一个以任务为导向的闭环优化过程。

### 2.2 面向更丰富世界模型的声呐语义理解

传统SLAM构建的纯几何地图在特征稀疏的海底会变得高度相似且缺乏辨识度。为了克服这一局限，我们需要让AUV不仅知道"那里有东西"，更要理解"那是什么东西"。这要求AUV感知世界的方式从几何层面提升到**"功能可见性"（affordances）**——即环境所提供的行动可能性([Encord, 什么是具身AI?](https://encord.com/blog/embodied-ai/))。例如，AUV需要能区分"可安全穿越的沙地"、"可能导致缠绕的海草区"和"具有高度科学价值的珊瑚礁"。

近年来，深度学习技术（特别是CNN）在声呐图像的**语义分割**方面取得了显著进展，能够将声呐图像分割成具有明确语义标签的区域，例如区分出"沙地"、"岩石"、"海草"或"管道"等人造物([ResearchGate, 水下声呐图像的深度学习语义分割](https://www.researchgate.net/publication/337504920_Semantic_Segmentation_of_Underwater_Sonar_Imagery_with_Deep_Learning), [MDPI, 少样本侧扫声呐图像语义分割](https://www.mdpi.com/2079-9292/11/19/3002))。这种从几何到语义的跃升，为AUV的"认知"环节提供了更高层次的输入，为执行更复杂的指令和构建更鲁棒的SLAM系统奠定了基础。然而，这一方向面临的主要挑战是**数据稀缺**。与陆地上拥有海量开源标注图像的自动驾驶领域不同，收集和标注水下声呐数据的成本极高，需要专业的设备和领域专家，这使得大规模、高质量、公开可用的标注声呐数据集目前严重匮乏。

### 2.3 实现方法：鸟瞰图（BEV）表示法

> [[05_AUV SLAM的BEV应用研究|鸟瞰图（BEV）表示法应用于AUV SLAM 中的可行性与机遇分析报告]]

鸟瞰图（Bird's-Eye-View, BEV）表示法是一种强大的技术，它将来自声呐的稀疏、不规则的三维点云，投影到一个或多个密集的、结构化的二维网格上。网格中的每个单元可以存储多个信息通道（如高程、点密度、返回强度），形成一个多通道的"图像"。这个"图像"随后可以被输入到强大的图像处理神经网络（如CNN或Transformer）中进行语义分割等任务([MDPI, 深度学习检测防浪块](https://www.mdpi.com/2072-4292/14/21/5575))。

BEV表示法不仅仅是一种方便的数据结构，它更是一座至关重要的**桥梁**，其内在逻辑如下：

1. AUV自主技术面临"冷启动"问题：研究人员少、数据量不足、工具链不成熟。
2. 自动驾驶领域已围绕 `传感器 -> BEV -> 感知/规划` 这一范式建立了庞大的研发生态系统([arXiv, 端到端自动驾驶深度学习综述](https://arxiv.org/pdf/2311.18636))。这是一个经过数十亿美元投资和数百万小时测试验证的成熟框架。
3. AUV的多波束声呐点云，在经过处理后，其三维结构与自动驾驶的激光雷达点云具有很强的相似性。
4. 因此，通过采用BEV投影这一关键步骤，AUV研究人员可以**立即利用来自自动驾驶领域的成熟架构、预训练模型（通过迁移学习）和开源代码库**，极大地加速研究进程。这不仅是代码层面的复用，更是思想和方法的借鉴，它将问题从"从零开始发明水下感知"转变为一个更易于处理、更具可操作性的"将成熟的陆地感知技术应用于水下领域"。

### 2.4 开源项目参考：语义感知

- **通用分割框架**: **sssegmentation** ([github.com/SegmentationBLWX/sssegmentation](https://github.com/SegmentationBLWX/sssegmentation)) 是一个基于PyTorch的模块化开源语义分割工具箱，支持多种主流模型，是研究者在自定义声呐数据集上进行快速算法验证和二次开发的理想起点。
- **基于Transformer的声呐分割**: **s3Tseg** ([github.com/CIRS-Girona/s3Tseg](https://github.com/CIRS-Girona/s3Tseg)) 是一个专门针对侧扫声呐数据分割的Vision Transformer（ViT）架构实现，代表了将前沿模型应用于声学图像处理的趋势。
- **结合基础模型的分割方法**: **PSIS-ADM** ([github.com/hfarhaditolie/PSIS-ADM](https://github.com/hfarhaditolie/PSIS-ADM)) 展示了如何利用强大的视觉基础模型（如SAM）的零样本分割能力来处理声呐图像，为解决声呐数据稀缺问题提供了一个极具潜力的技术途径。

## 3 具身SLAM：在歧义世界中学习定位

本章通过提出从传统几何SLAM到现代基于学习的、具身化的方法的转变，来解决AUV在无特征环境中的核心导航问题——无界漂移。

### 3.1 主动SLAM：智能地削减不确定性

传统SLAM的失败根源在于其被动性。**主动SLAM（Active SLAM）则从根本上颠覆了这一逻辑**。在主动SLAM框架中，AUV的路径是动态生成的，其唯一或首要目标就是**提升SLAM系统的性能**([arXiv, AUV不确定性下的实时规划](https://arxiv.org/html/2403.04936v1))。SLAM系统自身对位姿和地图的整体不确定性，成为了驱动AUV运动的根本动力。

主动SLAM的决策过程通常旨在选择一个"下一最佳视点"（Next-Best-View, NBV），以最大化预期的信息增益。一个设计良好的效用函数必须巧妙地平衡**探索（Exploration）**——前往未知区域以扩展地图，和**利用（Exploitation）**——重访已知区域以降低定位不确定性([ResearchGate, 主动SLAM探索综述](https://www.researchgate.net/publication/387111017_Active_Perception_in_SLAM_Exploration_Problem_Formulation_and_Methods_Review))。一篇针对AUV的研究提出了一种基于联合熵的效用函数，其计算公式为([MDPI, AUV主动SLAM探索](https://www.mdpi.com/2072-4292/11/23/2827))：

$$
\Delta H(x,m\mid u,z)\approx\Delta H(x\mid u,z)+\alpha(p(x\mid u,z))\Delta H(m\mid\mu x,u,z)
$$

其中，$\Delta H(x|u,z)$ 代表了状态熵的减少，主要通过回环检测带来的信息增益来量化；而 $\Delta H(m|\mu x,u,z)$ 代表了地图熵的减少，通常与从候选视点能够观测到的未知地图单元数量成正比。

这种机制会催生出非常智能的涌现行为：当AUV的定位不确定性很低时（即对自己在哪很自信），效用函数将由地图熵主导，AUV会像一个勇敢的探险家一样，优先选择前往未知区域进行探索；而当AUV在广阔的平坦区域航行过久，定位不确定性累积到较高水平时（即对自己在哪感到困惑），状态熵将占据主导，AUV会像一个谨慎的航海家，自主决策重访一个之前已经精确建图、特征丰富的区域（如一块独特的岩石），以执行回环检测，从而"锚定"自己的位置，消除累积误差。

**另一条已实现的技术路径则不直接依赖于对不确定性的数学建模，而是通过分析已构建的点云地图来直接指导决策**。在这种方法中，AUV的核心任务是判断是否需要回溯到已知的**可航行区（Navigable region）**。系统会持续分析由多波束声呐扫描生成的三维点云地图，当AUV长时间处于特征稀疏或未探索区域，导致定位漂移的风险增大时，它会主动规划路径，返回到之前已经确认的、地图质量高、几何特征丰富的"可航行区"。这种回溯行为同样是一种"利用"策略，其目的在于利用高质量的已知地图区域来重新"锚定"自身的位置，有效抑制漂移。这种基于地图内容直接进行决策的方法，为主动SLAM提供了一条更偏向几何与实践的实现途径。

### 3.2 语义SLAM：克服感知歧义

如果环境中充满了重复的几何结构（如海底的沙丘波纹），即使主动寻找，传统数据关联方法仍会失效，因为它无法区分一个波纹和另一个波纹。**语义SLAM（Semantic SLAM）通过将地图从纯几何基元提升到由语义对象**组成来解决这个难题([Frontiers, 基于多通道CNN的声呐图像分割](https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2022.928206/full))。借助声呐语义分割技术，AUV能够识别出环境中的具体物体（如"独特的Y形岩石"）。此时，SLAM的数据关联和回环检测环节将发生质变：AUV不再是匹配两个模糊的点云，而是在匹配被识别出的、具有唯一标识的语义地标。例如，系统中的逻辑将是："我此刻看到的这块'独特的Y形岩石'，就是我10分钟前在另一个位置看到的那一块'独特的Y形岩石'"。这种基于高级概念的匹配，在那些几何上模糊不清、但可能散布着少量语义上可区分物体的环境中，具有无与伦比的鲁棒性。

### 3.3 范式变革：面向声呐的神经隐式SLAM

**神经隐式地图**从根本上改变了地图的表示方式。它不将地图存储为离散的点或特征集合，而是将其表示为一个连续函数（通常是一个神经网络MLP），该网络将三维坐标$(x,y,z)$映射到一个值，如符号距离函数（SDF）([GitHub, Point-SLAM](https://github.com/eriksandstroem/Point-SLAM))。这个网络通过学习来拟合环境的连续几何形状，而不是简单地记录点的位置。

$$
f(x,y,z)→SDF
$$

这种方法对AUV的优势是革命性的。它天然地对稀疏和嘈杂的声呐数据具有鲁棒性，因为它会对信息进行插值和平滑，形成一个完整的表面。更重要的是，它**重新定义了"环路闭合"**，使其特别适合无特征环境。其内在逻辑如下：

1. 传统环路闭合是**离散匹配问题**：试图在时间t和tk​的两组特征点或点云之间寻找一对一的对应关系。这在无特征环境中几乎是不可能的，就像试图匹配两张沙滩照片中每一粒沙子的位置。
2. 隐式SLAM的环路闭合是**连续函数拟合问题**：寻找一个新的位姿Pt​，使得当前的声呐扫描St​能够最好地与先前学习到的、代表整个区域几何形态的地图函数f的零水平面（即表面）对齐。
3. 一个平缓、无特征的斜坡，虽然没有离散特征，但它仍然可以被学习到的函数f很好地描述。如果AUV返回到同一个斜坡，其新的声呐扫描作为一个整体，将能很好地"嵌入"或"拟合"这个学习到的连续函数，即使没有一个单独的点或特征是唯一可识别的。
4. 这意味着，隐式SLAM通过**改变地图的表示方式**（连续函数 vs. 离散点），直接促成了一种更鲁棒的**定位机制**（函数拟合 vs. 特征匹配），从而从根本上解决了在特征稀疏环境中环路闭合的核心难题。

### 3.4 开源项目参考：现代SLAM

- **隐式SLAM框架**: **PIN-SLAM** ([github.com/PRBonn/PIN_SLAM](https://github.com/PRBonn/PIN_SLAM)) 和 **Point-SLAM** ([github.com/eriksandstroem/Point-SLAM](https://github.com/eriksandstroem/Point-SLAM)) 是专为LiDAR或RGB-D相机设计的先进隐式SLAM框架。它们是改造为声呐输入的理想候选，直接解决了核心漂移问题。
- **主动SLAM框架**: **CERLAB-UAV-Autonomy** ([github.com/Zhefan-Xu/CERLAB-UAV-Autonomy](https://github.com/Zhefan-Xu/CERLAB-UAV-Autonomy)) 是一个功能全面的UAV自主导航模块化框架，其清晰的架构为开发AUV主动SLAM系统提供了极佳的工程参考。
- **语义SLAM基础**: **GTSAM** ([github.com/borglab/gtsam](https://github.com/borglab/gtsam)) 是一个灵活的基于因子图的优化库，是实现语义SLAM后端的理想选择。**SG-SLAM** ([github.com/silencht/SG-SLAM](https://github.com/silencht/SG-SLAM)) 是一个经典的语义视觉SLAM系统，其在成熟SLAM框架上增加语义处理线程的模块化设计思想，为构建水下声呐语义SLAM系统提供了清晰的架构蓝图。

## 4 具身导航：从反应式控制到认知规划

本章聚焦于最高层次的自主性：学习直接从感知到行动的导航策略，体现完整的具身智能循环。AUV导航策略的演进可以看作一条清晰的认知发展路径：从反应式"本能"，到经验式"记忆"，再到抽象式"推理"。

### 4.1 阶段一：基于深度强化学习（DRL）的反应式"本能"

深度强化学习（DRL）是实现具身智能最自然的技术路径之一。智能体通过与环境进行大量的试错交互来学习一个最优策略（从状态到行动的映射），整个学习过程由一个精心设计的奖励信号引导([arXiv, 环境感知AUV的RL框架](https://arxiv.org/html/2506.15082v1))。一个典型的AUV导航DRL问题，其关键要素包括：

- **状态空间 (State)**: 通常包含目标点的相对位置、自身运动学状态，以及来自声呐的障碍物信息([MDPI, 基于DDPG的AUV避碰规划](https://www.mdpi.com/2077-1312/11/12/2258))。
- **动作空间 (Action)**: 通常是连续的，控制纵向推力和偏航力矩([arXiv, AUV自适应编队运动规划](https://arxiv.org/pdf/2304.00225))。
- **奖励函数 (Reward)**: 精心设计的复合函数，鼓励到达目标、惩罚碰撞、并促进平滑和节能的轨迹。

然而，端到端DRL在AUV导航中的成功，其关键不在于算法的选择（SAC、TD3等已相当成熟），而在于**课程设计（curriculum design）**。直接应用DRL很可能因问题过于复杂（高维状态、稀疏奖励）而失败([MDPI, AUV局部运动规划的端到端DRL方法](https://www.mdpi.com/2077-1312/11/9/1796))。通往成功的路径是：问题复杂性 -> DRL失败 -> 课程设计 -> DRL成功。实践者应将精力集中在设计一个有效的训练课程上，将复杂的导航问题分解为一系列可学习的步骤。这包括：

- **通过模仿的课程**：使用**生成对抗模仿学习（GAIL）**等技术，从一个小型专家演示数据集中预训练智能体，让它从"模仿专家"这个简单任务开始，极大地加速学习([PMC, 基于SAC与模仿学习的AUV运动规划](https://pmc.ncbi.nlm.nih.gov/articles/PMC8434076/))。这相当于在教一个新手司机时，先让他跟着教练的车开几圈，而不是直接让他上路。
- **通过奖励塑形的课程**：设计密集的奖励函数，为智能体提供中间"提示"，而非单一的最终奖励([UHR, AUV自适应路径规划的DRL](https://uhra.herts.ac.uk/id/eprint/10051/1/APOR_2022.pdf))。例如，除了在到达终点时给予大奖励，每向终点靠近一米都给予一个小奖励，这能提供更密集的学习信号。
- **通过任务简化的课程**：从简单环境开始训练，然后逐步增加障碍物和动态洋流的复杂性([arXiv, 狭窄环境中的具身逃逸RL](https://arxiv.org/html/2503.03208v1))。这就像在游泳教学中，先在平静的浅水区学习，再到深水区，最后才挑战有浪的海域。

### 4.2 阶段二：基于记忆的经验式"记忆"

纯粹的反应式DRL策略缺乏远见，容易陷入局部最优（例如在U型陷阱中来回徘徊）。

为了解决这个问题，一个前沿的思路是将"经验回放缓冲区"提升为一种主动的**规划机制**([NeurIPS, 带记忆的感知-行动循环](https://proceedings.neurips.cc/paper_files/paper/2022/file/dd7a48c862b800f0537fe1d506e641b5-Paper-Conference.pdf))。当面临一个新的、远距离的导航任务时，智能体可以主动地从这个存储了大量过去成功与失败经验的"记忆库"中，检索并"拼接"出若干段相关的轨迹"片段"，从而组合成一条全新的、连接当前位置和目标点的近似最优路径。

这种方法就像一个经验丰富的老水手，他不需要每次都看海图，而是可以根据记忆中相似的海况和路线片段，快速规划出一条新航线。这种方法巧妙地结合了学习型方法的灵活性和经典规划方法的结构化优势。

### 4.3 阶段三：抽象式"推理"与跨域启示

这代表了最高层次的认知能力，其实现有赖于我们将在下一章讨论的基础模型。此外，来自陆地和空中机器人的成熟技术也为AUV提供了宝贵的参考。特别是UAV（无人机）在三维空间中的**基于前沿的探索（frontier-based exploration）** 与 **主动回环闭合（active loop closing）** 相结合的策略([MDPI, 动态环境中的感知感知规划](https://www.mdpi.com/2072-4292/14/11/2584))，几乎可以直接移植到AUV上。例如，UAV探索建筑物的策略（先探索完一层，再通过楼梯到下一层）可以启发AUV在探索多层沉船或复杂海底峡谷时的分层探索策略，为AUV在广阔未知水域中的高效、精确建图提供了成熟的解决方案。

### 4.4 开源项目参考：导航

- **模块化RL框架**: **Modular-Baselines** ([github.com/TolgaOk/Modular-Baselines](https://github.com/TolgaOk/Modular-Baselines)) 强调组件灵活性，适合算法层面的创新。**rosnav-rl** ([github.com/Arena-Rosnav/rosnav-rl](https://github.com/Arena-Rosnav/rosnav-rl)) 侧重于应用，是连接ROS仿真与真实机器人导航的优秀解决方案。
- **水下目标跟踪RL框架**: **RLforUTracking** ([github.com/imasmitja/RLforUTracking](https://github.com/imasmitja/RLforUTracking)) 是一个完整的、专门为AUV水下目标跟踪任务设计的DRL工具集，是一个端到端的技术解决方案。
- **跨域参考：UAV避障框架**: **UAV_Obstacle_Avoiding_DRL** ([github.com/ZYunfeii/UAV_Obstacle_Avoiding_DRL](https://github.com/ZYunfeii/UAV_Obstacle_Avoiding_DRL)) 项目将环境动态（流场）与RL结合的思路，对开发环境感知的AUV导航策略极具启发性。

## 5 前沿阵地：用于高级AUV自主性的基础模型

近年来，以大型Transformer为核心的基础模型催生了如视觉-语言-行动（ #code/AI/具身智能/VLA ）、视觉-语言-导航（ #code/AI/具身智能/VLN ）和视觉动作 ( #code/AI/具身智能/VA)）等新范式。本章将探讨将这些前沿模型应用于AUV的可行性。

[具身专栏（一）\VLA、VA、VLN概述](https://mp.weixin.qq.com/s/O-eUOCQSAk9NUbElwJ7XdQ)

| 方法                               | 描述                       | 关键模块                   | 典型应用                 |
| -------------------------------- | ------------------------ | ---------------------- | -------------------- |
| Vision-Language-Action (VLA)     | 整合视觉感知、语言理解和动作生成的模型      | 视觉编码器、语言模型、多模态融合、动作生成器 | 机器人操作、家务、人形控制        |
| Vision-Language-Navigation (VLN) | 基于视觉输入和语言指令的导航任务专用模型     | 视觉编码器、语言模型、导航规划器       | 基于视觉输入和语言指令的导航任务专用模型 |
| Vision-Action (VA)               | 将视觉感知直接与动作联系起来的模型，无需语言成分 | 将视觉编码器、策略网络（通常基于扩散）    | 视觉编码器、策略网络（通常基于扩散）   |

### 5.1 从VLA到声呐-语言-行动（SLA）模型

VLA模型是能够接收多模态输入（图像、语言指令、机器人状态）并直接输出底层控制动作的端到端系统([Labellerr, VLA模型如何驱动人形机器人](https://www.labellerr.com/blog/vision-language-action-vla-models-2/))。由于AUV的主传感器是声呐，我们需要将其改编为**声呐-语言-行动（SLA）**模型。

**实现逻辑**：

1. **输入符号化（Tokenization）**：将声呐三维点云投影为二维BEV图，再切分成块（patches）并编码为token。语言指令和本体状态也编码为token。
2. **动作离散化**：使用聚类算法（如K-means）将连续的动作空间离散化为一个"动作词典"。
3. **模型架构**：采用Transformer架构，自回归地预测出代表未来动作的token序列。

**可行性与挑战**：从架构上看，这种改编是可行的。但其面临的**最根本、最严峻的障碍是数据鸿沟（The Data Gap）**。VLA这类模型是数据驱动的，其泛化能力建立在海量、多样化的训练数据之上([arXiv, 可扩展的免训练视觉语言机器人](https://arxiv.org/html/2502.01071v1))。相比于陆地机器人可以轻易获取数百万级别的交互数据，目前全球范围内都**不存在**任何公开的、大规模的、包含（声呐数据、AUV状态、专家动作）三元组的AUV导航数据集([ResearchGate, 水下声呐图像检测结果](https://www.researchgate.net/figure/Detection-results-of-the-underwater-sonar-image-image-size-112117-a-Original-sonar_fig13_331170907))。因此，最现实的路径是通过大规模、高保真的仿真来生成合成数据，并结合少量真实世界数据进行微调。

### 5.2 声呐-行动（SA）模型

与SLA模型相对应，声呐-行动（SA）模型是陆地机器人Vision-Action（VA）范式在水下领域的直接延伸。其核心思想是构建一个更纯粹的、将声呐感知直接映射到行动的端到端策略，完全无需任何语言成分。

核心理念：SA模型旨在学习一个从原始声呐传感器输入直接到低级电机指令（如推进器推力、舵角）的映射函数。这取代了传统的"感知-规划-行动"模块化流水线，代之以一个统一的、通过学习得到的策略网络。这种方法的优势在于，它允许系统对所有组件进行联合优化，能够自主发现传感器数据中与任务最相关的特征，并可能产生比人工设计的模块化系统更具反应性和鲁棒性的行为。

实现逻辑：SA模型的实现逻辑与第四章讨论的端到端DRL导航紧密相关，可以看作是其在基础模型框架下的现代表达。模型（通常是基于Transformer或CNN的编码器-解码器架构）将经过处理的声呐数据（如BEV图）和本体感知状态作为输入，然后直接输出一个动作。

可行性与挑战：SA模型在概念上比SLA模型更简单，但面临着同样严峻的数据鸿沟挑战。因为它是一个纯粹的模仿学习或强化学习问题，其性能完全依赖于训练数据的规模和多样性。在没有大规模（声呐，动作）数据对的情况下，训练一个能够泛化到未见场景的通用SA模型是极其困难的。因此，其发展同样高度依赖于高保真仿真技术的进步和合成数据的生成能力。

### 5.3 声呐-语言-导航（SLN）模型

SLN任务要求智能体根据自然语言指令在环境中导航，例如："**请先勘测前方海底的管道，然后前往并环绕勘察附近的岩石堆。**" 这其核心在于利用感知来"接地"（ground）语言指令中的概念([arXiv, 视觉-语言导航综述](https://arxiv.org/html/2402.14304v1))。

考虑到直接训练端到端SLN模型的困难，一个更为现实的实现路径是采用**分层控制（Hierarchical Control）** 架构：

1. **高层规划器（基于LLM）**：一个大型语言模型（LLM）接收人类指令，并利用其强大的常识推理能力，将抽象指令分解为一个符号化的任务序列，例如：[GOTO(pipeline_start), SURVEY(pipeline), GOTO(rock_pile), CIRCUMNAVIGATE(rock_pile)]。
2. **底层控制器（基于DRL/SLAM）**：一个强大的、已训练好的导航策略（如第3、4章讨论的方法），负责接收并执行来自高层规划器的每一个具体子目标。

这种分层式架构具有很高的可行性，其主要技术难点在于如何实现高层抽象规划与底层物理执行之间的**可靠接地（Robust Grounding）**。也就是说，LLM生成的计划必须符合AUV的物理能力限制和当前环境的实际情况。

例如，当LLM生成"穿越那个拱门"的指令时，底层系统必须能够验证：

1. 语义SLAM系统是否真的在地图中识别出了一个"拱门"实体？
2. 这个"拱门"的宽度是否足够AUV的物理身体安全通过？
3. 环境感知模块预测的拱门区域水流是否在AUV的可控范围内？

这需要高层规划器与底层感知和控制模块之间建立一个紧密的、双向的信息反馈循环。近年来在UAV VLN领域的研究为此提供了重要的参考([arXiv, 面向物流无人机的VLN](https://arxiv.org/abs/2505.03460))。

### 5.4 开源项目参考：基础模型

- **社区驱动的机器人学习框架**: **LeRobot** ([github.com/huggingface/lerobot](https://github.com/huggingface/lerobot)) 是Hugging Face发起的开源计划，其发布的 **SmolVLA** ([Hugging Face, SmolVLA](https://huggingface.co/lerobot/smolvla_base)) 模型是研究如何将VLA范式迁移到新领域（如AUV）的理想起点。
- **大规模开源VLA模型**: **OpenVLA** ([github.com/openvla/openvla](https://github.com/openvla/openvla)) 是由斯坦福大学等机构联合推出的项目，其架构设计和训练策略为构建高性能SLA模型提供了重要参考。

## 6 生态系统、挑战与战略路线图

### 6.1 "仿真到现实"的鸿沟：主要障碍

部署基于学习的AUV控制器的最大障碍是仿真与现实之间的差距（sim-to-real gap）([DSpace@MIT, 机器人学的Sim2Real](https://dspace.mit.edu/bitstream/handle/1721.1/138850/2021-04-Sim2Real_T-ASE.pdf))。对于AUV，这个问题尤为严重，因为精确模拟**复杂的水动力学**和**声呐物理学**都极其困难([arXiv, 水下机器人仿真器综述](https://arxiv.org/html/2504.06245v1))。

### 6.2 主流AUV仿真平台对比分析

选择合适的仿真器对研究成败有决定性影响。一个研究者必须根据其具体目标（侧重高精度动力学还是逼真传感器）做出明智选择。

表1：主流AUV仿真平台对比分析

| 仿真器           | 核心引擎 (物理/渲染)                 | ROS/ROS2 支持 | 关键传感器模型                      | 优势                    | 局限性/侧重点                  |
| :------------ | :--------------------------- | :---------- | :--------------------------- | :-------------------- | :----------------------- |
| UUV Simulator | Gazebo / OGRE                | ROS1 (归档)   | DVL, IMU, 声呐(基础), 水流模型       | 成熟, 社区基础好, 优秀的流体动力学插件 | 项目已归档, 仅支持ROS1, 渲染效果非照片级 |
| Stonefish     | 自定义 / OGRE                   | ROS1        | 声呐, 相机                       | 专为高保真动力学仿真设计          | 渲染真实感非其重点                |
| HoloOcean     | Unreal Engine 4              | ROS1/ROS2   | 声呐(基于八叉树), 相机, DVL, IMU      | 优秀的视觉渲染效果, 支持多智能体     | 基于UE4, 新场景声呐渲染慢, 动力学非重点  |
| OceanSim      | NVIDIA Isaac Sim / Omniverse | ROS2        | GPU加速声呐(光线追踪), DVL, 带水体效应的相机 | 顶级的照片级渲染, 高性能GPU加速传感器 | 侧重感知, 缺乏精确的车辆/流体动力学模型    |
| MARUS         | Unity                        | ROS         | -                            | 侧重于VR增强的遥操作和多机器人系统    | 较少关注自主感知/规划              |

### 6.3 板载计算与功率限制

深度学习模型计算量巨大，而AUV是功率受限的平台。解决方案包括：

- **板载硬件**：配备嵌入式GPU，如NVIDIA Jetson系列([YouTube, DeepVL水下里程计](https://www.youtube.com/watch?v=ctcbrNu_N78))。
- **模型优化**：通过**量化**（降低数字精度）、**剪枝**（移除冗余权重）和**知识蒸馏**（训练小模型模仿大模型）等技术为高效推理进行优化([DTIC, UUV控制的混合精度DRL](https://apps.dtic.mil/sti/trecms/pdf/AD1200547.pdf))。

### 6.4 综合战略研究路径 (参考)

基于以上分析，提出以下分层战略研究路径：

- **第一层（短期，高可行性）**：
  - **改造LiDAR SLAM**：集中精力改造现有的开源隐式LiDAR SLAM系统（如PIN-SLAM），使其能够使用多波束声呐数据。
  - **开发高保真声呐数据集**：收集并发布大型、标注良好的多波束声呐和导航数据集（类似于扩展版的[AURORA数据集](https://github.com/noc-mars/aurora))。
- **第二层（中期，有前景）**：
  - **混合式RL导航**：在高保真仿真器（如[HoloOcean](https://robots.et.byu.edu/holoocean/))中开发基于DRL的导航智能体，重点关注混合模仿/强化学习和课程设计。
  - **Sim-to-Real迁移**：针对AUV进行专注的sim-to-real迁移研究，特别是域随机化策略。
- **第三层（长期，愿景）**：
  - **水下世界模型**：研究开发面向水下领域的大规模、自监督"基础模型"，类似于为语言和视觉领域开发的模型([NVIDIA, 什么是具身AI?](https://www.nvidia.com/en-us/glossary/embodied-ai/))。
  - **端到端具身SLAM**：开发从传感器输入到地图表示和位姿更新完全由单一可微神经网络构成的SLAM系统([GitHub, 主动SLAM论文列表](https://github.com/DoongLi/awesome-Active-SLAM))。

### 6.5 范式与潜力总结

表2：具身智能范式及其在AUV上的应用潜力评估

| 范式                | 核心概念                        | AUV实现的关键促成因素                   | 在AUV导航/SLAM中的应用        | 可行性 & 核心挑战                       |
| :---------------- | :-------------------------- | :----------------------------- | :--------------------- | :------------------------------- |
| 基于DRL的导航          | 通过交互和奖励直接学习状态到动作的映射。        | 高保真度的动力学仿真器，精心设计的奖励函数。         | 反应式避障、目标搜寻、路径跟踪。       | 高可行性。 挑战：Sim-to-Real，奖励函数设计。     |
| 主动SLAM            | 通过动作选择最大化信息增益，最小化状态/地图不确定性。 | 位姿图SLAM后端，信息论规划器。              | 智能地探索特征稀疏的环境。          | 高可行性。 挑战：在线重规划的计算成本。             |
| 语义SLAM            | 使用语义上有意义的对象地图进行SLAM。        | 鲁棒的、实时的声呐数据语义分割模型。             | 在几何上重复但语义上可区分的环境中鲁棒定位。 | 中等可行性。 挑战：缺乏标注声呐数据。              |
| 声呐-行动 (SA) 模型     | 从原始声呐输入到动作输出的端到端学习。         | 极大规模的(声呐, 动作)数据对；高性能仿真器。       | 执行多种简单导航任务的通用策略。       | 低可行性 (当前)。 挑战：数据鸿沟。              |
| 声呐-语言-行动 (SLA) 模型 | 融入自然语言指令的端到端模型。             | 所有SA的促成因素，外加配对的(语言, 声呐, 动作)数据。 | 遵循简单自然语言指令的通用策略。       | 极低可行性 (当前)。 挑战：更巨大的数据需求。         |
| 声呐-语言-导航 (SLN) 模型 | LLM解释指令，底层策略执行的分层系统。        | LLM，语义SLAM (用于接地)，鲁棒的底层控制器。    | 通过自然语言实现复杂的多阶段任务。      | 中高可行性 (分层式)。 挑战：LLM规划与物理世界的可靠接地。 |

## 7 结论与未来展望

### 7.1 结论

本报告的全面分析清晰地表明，具身智能并非一系列新算法的简单集合，而是一种根本性的观念转变，它与水下自主性面临的独特挑战形成了完美的契合。通过将感知-行动循环置于核心，AUV可以从一个遵循预设程序的、脆弱的自动化工具，演变为一个能够在未知、不确定和动态的环境中主动学习和智能适应的真正自主的智能体。  

目前，**主动SLAM的技术路线已经实现**，在此基础上，我初步确定了后续的技术框架：**具身感知采用BEV表示法，具身SLAM采用语义SLAM，而具身规划则采用深度强化学习（DRL）**。这些已实现和选定的技术路线，共同构成了一个从感知到规划的、完整的具身智能解决方案。

在此坚实基础上，未来值得进一步摸索和研究的方向包括：**面向声呐的神经隐式SLAM**，以期从根本上解决特征稀疏环境下的地图构建与定位难题；以及**环境感知的深度强化学习**，旨在将水流等动态环境因素内生地融入到AUV的决策过程中，实现更高层次的适应性和能效。这一系列技术路线的演进，标志着AUV正从遵循预设程序的脆弱系统，向能够在未知和动态环境中主动学习和智能适应的真正自主的智能体演进。

从长远来看，以SLA和SLN为代表的基础模型，为实现真正自然、直观的人-AUV协同作业描绘了激动人心的蓝图，尽管其实现仍面临重大的数据和计算挑战。

### 7.2 弥合数据与仿真的鸿沟

结论是明确的：阻碍具身智能在AUV领域充分释放其潜力的主要瓶颈，是**数据与仿真的鸿沟**。与自动驾驶等领域相比，水下机器人研究所能利用的公开数据和标准化工具链都极为有限。该领域的未来发展，很大程度上取决于两个关键方向的努力：

1. **社区驱动的高保真开源仿真**：大力发展能够同时精确模拟AUV流体动力学和逼真传感器现象学（特别是声呐）的下一代开源仿真平台至关重要([arXiv, 水下机器人仿真器综述](https://arxiv.org/html/2504.06245v1))。一个统一的、可扩展的、经过充分验证的基准仿真环境，将极大地加速整个社区的研究进程，并成为生成大规模合成数据的基础。
2. **数据高效的学习技术研究**：鉴于在真实世界中收集大规模AUV导航数据集的极端困难，研究重心必须向数据高效的学习方法倾斜。自监督学习（从无标签数据中学习表征）、小样本学习（从极少量样本中泛化）以及更先进的Sim-to-Real迁移学习技术（如域随机化），是克服数据稀缺性、将仿真中学习到的知识成功应用于现实世界的关键。

### 7.3 最终展望

随着仿真技术的不断进步和数据高效学习方法的日趋成熟，文中探讨的具身智能的宏伟目标将变得越来越触手可及。从被动的数据记录者到主动的知识寻求者，AUV正在经历一场深刻的智能化变革。通过拥抱具身智能，我们有理由相信，一个由高度自主、智能的AUV进行大规模、精细化探索和作业的全新海洋时代即将到来。

## 8 参考文献

_本章节列出了在报告正文中明确引用的所有文献。_

- [arXiv, AUV不确定性下的实时规划](https://arxiv.org/html/2403.04936v1)  
- [arXiv, AUV自适应编队运动规划](https://arxiv.org/pdf/2304.00225)  
- [arXiv, 端到端自动驾驶深度学习综述](https://arxiv.org/pdf/2311.18636)  
- [arXiv, 环境感知AUV的RL框架](https://arxiv.org/html/2506.15082v1)  
- [arXiv, 具身智能与通用人工智能](https://arxiv.org/html/2505.06897v1)  
- [arXiv, 狭窄环境中的具身逃逸RL](https://arxiv.org/html/2503.03208v1)  
- [arXiv, 可扩展的免训练视觉语言机器人](https://arxiv.org/html/2502.01071v1)  
- [arXiv, 视觉-语言导航综述](https://arxiv.org/html/2402.14304v1)  
- [arXiv, 面向物流无人机的VLN](https://arxiv.org/abs/2505.03460)  
- [arXiv, 水下机器人仿真器综述](https://arxiv.org/html/2504.06245v1)  
- [BioRob, 水下SLAM挑战](http://vigir.missouri.edu/~gdesouza/Research/Conference_CDs/BioRob_2014/media/files/0075.pdf)  
- [Boxfish Robotics, AUV自主导航](https://www.boxfishrobotics.com/autonomy/navigating-the-depths-boxfish-robotics-advanced-auv-navigation-system/)  
- [DSpace@MIT, 机器人学的Sim2Real](https://dspace.mit.edu/bitstream/handle/1721.1/138850/2021-04-Sim2Real_T-ASE.pdf)  
- [DTIC, UUV控制的混合精度DRL](https://apps.dtic.mil/sti/trecms/pdf/AD1200547.pdf)  
- [Encord, 什么是具身AI?](https://encord.com/blog/embodied-ai/)  
- [Frontiers, 基于多通道CNN的声呐图像分割](https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2022.928206/full)  
- [GitHub, AURORA数据集](https://github.com/noc-mars/aurora)  
- [GitHub, 主动SLAM论文列表](https://github.com/DoongLi/awesome-Active-SLAM)  
- [GitHub, CERLAB-UAV-Autonomy](https://github.com/Zhefan-Xu/CERLAB-UAV-Autonomy)  
- [GitHub, GTSAM](https://github.com/borglab/gtsam)  
- [GitHub, HoloOcean](https://robots.et.byu.edu/holoocean/)  
- [GitHub, LeRobot](https://github.com/huggingface/lerobot)  
- [GitHub, Modular-Baselines](https://github.com/TolgaOk/Modular-Baselines)  
- [GitHub, OpenVLA](https://github.com/openvla/openvla)  
- [GitHub, PIN-SLAM](https://github.com/PRBonn/PIN_SLAM)  
- [GitHub, Point-SLAM](https://github.com/eriksandstroem/Point-SLAM)  
- [GitHub, PSIS-ADM](https://github.com/hfarhaditolie/PSIS-ADM)  
- [GitHub, RLforUTracking](https://github.com/imasmitja/RLforUTracking)  
- [GitHub, rosnav-rl](https://github.com/Arena-Rosnav/rosnav-rl)  
- [GitHub, s3Tseg](https://github.com/CIRS-Girona/s3Tseg)  
- [GitHub, SG-SLAM](https://github.com/silencht/SG-SLAM)  
- [GitHub, sssegmentation](https://github.com/SegmentationBLWX/sssegmentation)  
- [GitHub, UAV_Obstacle_Avoiding_DRL](https://github.com/ZYunfeii/UAV_Obstacle_Avoiding_DRL)  
- [Hugging Face, SmolVLA](https://huggingface.co/lerobot/smolvla_base)  
- [Lamarr Institute, 具身AI解释](https://lamarr-institute.org/blog/embodied-ai-explained/)  
- [具身专栏（一）\VLA、VA、VLN概述](https://mp.weixin.qq.com/s/O-eUOCQSAk9NUbElwJ7XdQ)
- [Labellerr, VLA模型如何驱动人形机器人](https://www.labellerr.com/blog/vision-language-action-vla-models-2/)  
- [MBARI, 海底测绘AUV](https://www.mbari.org/technology/seafloor-mapping-auv/)  
- [MDPI, AEKF-SLAM算法](https://www.mdpi.com/1424-8220/17/5/1174)  
- [MDPI, AUV主动SLAM探索](https://www.mdpi.com/2072-4292/11/23/2827)  
- [MDPI, AUV局部运动规划的端到端DRL方法](https://www.mdpi.com/2077-1312/11/9/1796)  
- [MDPI, 基于DDPG的AUV避碰规划](https://www.mdpi.com/2077-1312/11/12/2258)  
- [MDPI, 深度学习检测防浪块](https://www.mdpi.com/2072-4292/14/21/5575)  
- [MDPI, 少样本侧扫声呐图像语义分割](https://www.mdpi.com/2079-9292/11/19/3002)  
- [MDPI, 动态环境中的感知感知规划](https://www.mdpi.com/2072-4292/14/11/2584)  
- [Missouri University, 水下SLAM挑战](http://docs.google.com/http.vigir.missouri.edu/~gdesouza/Research/Conference_CDs/BioRob_2014/media/files/0075.pdf)  
- [NeurIPS, 带记忆的感知-行动循环](https://proceedings.neurips.cc/paper_files/paper/2022/file/dd7a48c862b800f0537fe1d506e641b5-Paper-Conference.pdf)  
- [Nortek, 水下导航指南](https://www.nortekgroup.com/knowledge-center/wiki/new-to-subsea-navigation)  
- [NVIDIA, 什么是具身AI?](https://www.nvidia.com/en-us/glossary/embodied-ai/)  
- [PMC, 基于SAC与模仿学习的AUV运动规划](https://pmc.ncbi.nlm.nih.gov/articles/PMC8434076/)  
- [PMC, 水下SLAM与深度学习的结合](https://pmc.ncbi.nlm.nih.gov/articles/PMC12157327/)  
- [PNI Sensor, AUV导航进展](https://www.pnisensor.com/advancements-in-autonomous-underwater-vehicle-navigation/)  
- [ResearchGate, 水下声呐图像的深度学习语义分割](https://www.researchgate.net/publication/337504920_Semantic_Segmentation_of_Underwater_Sonar_Imagery_with_Deep_Learning)  
- [ResearchGate, 眼-机器人：BC-RL感知-行动循环](https://www.researchgate.net/publication/392629675_Eye_Robot_Learning_to_Look_to_Act_with_a_BC-RL_Perception-Action_Loop)  
- [ResearchGate, 主动SLAM探索综述](https://www.researchgate.net/publication/387111017_Active_Perception_in_SLAM_Exploration_Problem_Formulation_and_Methods_Review)  
- [ResearchGate, 水下SLAM挑战](https://www.researchgate.net/publication/286724450_Underwater_SLAM_Challenges_state_of_the_art_algorithms_and_a_new_biologically-inspired_approach)  
- [ResearchGate, 水下声呐图像检测结果](https://www.researchgate.net/figure/Detection-results-of-the-underwater-sonar-image-image-size-112117-a-Original-sonar_fig13_331170907)  
- [UHR, AUV自适应路径规划的DRL](https://uhra.herts.ac.uk/id/eprint/10051/1/APOR_2022.pdf)  
- [University of Michigan, 主动视觉SLAM理论与实验](http://robots.engin.umich.edu/publications/akim-2015a.pdf)  
- [YouTube, DeepVL水下里程计](https://www.youtube.com/watch?v=ctcbrNu_N78)

## 9 扩展参考

_本章节列出了原始文档中包含但未在本文正文中直接引用的文献，供读者进行扩展阅读。_

- [CACM, 具身人工智能简史及展望](https://cacm.acm.org/blogcacm/a-brief-history-of-embodied-artificial-intelligence-and-its-future-outlook/)  
- [Cambridge University, 具身人工智能](https://www.repository.cam.ac.uk/bitstreams/c2b0c4d6-940f-43e8-9e01-6b49f79a7e09/download)  
- [Built In, 什么是具身AI？](https://builtin.com/artificial-intelligence/embodied-ai)  
- [ResearchGate, 多智能体系统中的具身模仿增强强化学习](https://www.researchgate.net/publication/260066786_Embodied_Imitation-Enhanced_Reinforcement_Learning_in_Multi-Agent_Systems)  
- [MDPI, AUV协同任务的定位、导航与通信](https://www.mdpi.com/2076-3417/10/4/1256)  
- [ResearchGate, 基于惯性导航和声学定位的AUV导航](https://www.researchgate.net/publication/330294959_AUV_Navigation_Based_on_Inertial_Navigation_and_Acoustic_Positioning_Systems)  
- [PMC, 基于多波束瓦片匹配的AUV导航校正](https://pmc.ncbi.nlm.nih.gov/articles/PMC8840710/)  
- [MDPI, 基于位置校正和速度模型的AUV集成导航方法](https://www.mdpi.com/1424-8220/24/16/5396)  
- [NavLab.net, 利用多波束回声测深仪的AUV地形参考导航](https://www.navlab.net/Publications/Terrain_Referenced_Navigation_of_AUVs_and_Submarines_Using_Multibeam_Echo_Sounders.pdf)  
- [MDPI, 基于半直接法的声呐SLAM](https://www.mdpi.com/2077-1312/12/12/2234)  
- [PMC, 水下SLAM传感器融合进展综述](https://pmc.ncbi.nlm.nih.gov/articles/PMC11644431/)  
- [MDPI, 增强水下SLAM导航与感知的深度学习集成综述](https://www.mdpi.com/1424-8220/24/21/7034)  
- [GitHub, SLAM新进展收集](https://github.com/runjtu/awesome-and-novel-works-in-slam)  
- [JST.go.jp, 具身AI基础与核心技术概述](https://www.jst.go.jp/kisoken/boshuu/teian/en/koubo/2025houshin_cemb_en.pdf)  
- [NERC Open Research Archive, AURORA多传感器数据集](https://nora.nerc.ac.uk/532651/1/AURORA__A_multi_sensor_dataset_for_robotic_ocean_exploration.pdf)  
- [Elizabeth Vargas, 融合声学传感的鲁棒水下视觉SLAM](https://evargasv.github.io/publications/conferences/vargas2021_icra.pdf)  
- [GitHub, NeRF与3DGS SLAM资源](https://github.com/3D-Vision-World/awesome-NeRF-and-3DGS-SLAM)  
- [ResearchGate, 基于DRL的AUV在洋流干扰下的路径规划](https://www.researchgate.net/publication/358947954_Path_Planning_based_on_Deep_Reinforcement_Learning_for_Autonomous_Underwater_Vehicles_under_Ocean_Current_Disturbance)  
- [MDPI, 基于DRL的AUV避障规划](https://www.mdpi.com/2077-1312/9/11/1166)  
- [PMC, 基于多源数据辅助的改进人工势场法AUV路径规划](https://pmc.ncbi.nlm.nih.gov/articles/PMC10422249/)  
- [arXiv, 多智能体生成对抗交互自模仿学习用于AUV编队控制](https://arxiv.org/html/2401.11378v1)  
- [PMC, 用于AUV路径规划的带噪声Dueling DDQN算法](https://pmc.ncbi.nlm.nih.gov/articles/PMC11513341/)  
- [Scribd, 基于动作指导的深度交互式强化学习用于AUV路径规划](https://www.scribd.com/document/825970176/Action-Guidance-Based-Deep-Interactive-Reinforcement-Learning-for-AUV-Path-Planning)  
- [ResearchGate, 机器人学强化学习：Sim-to-Real迁移、模仿学习和迁移学习技术探索](https://www.researchgate.net/publication/385885092_Reinforcement_Learning_in_Robotics_Exploring_Sim-to-Real_Transfer_Imitation_Learning_and_Transfer_Learning_Techniques)  
- [PMC, 仿生水下机器人强化学习方法综述](https://pmc.ncbi.nlm.nih.gov/articles/PMC10123646/)  
- [arXiv, 水下机器人领域基于声呐的深度学习：概述、鲁棒性与挑战](https://arxiv.org/html/2412.11840v1)  
- [PMC, 基于射线物理建模的多波束声呐仿真](https://pmc.ncbi.nlm.nih.gov/articles/PMC11902455/)  
- [UUV Simulator Docs, GazeboRosImageSonar](https://uuvsimulator.github.io/packages/uuv_simulator/docs/api/gazebo>::GazeboRosImageSonar/)  
- [HoloOcean Docs](https://byu-holoocean.github.io/holoocean-docs/)  
- [MDPI, UAV辅助边缘计算中基于DRL的计算卸载](https://www.mdpi.com/2504-446X/7/3/213)  
- [The IET Shop, 用于智能6G通信的DRL](https://shop.theiet.org/deep-reinforcement-learning-for-reconfigurable-intelligent-surfaces-and-uav-empowered-smart-6g-communications)  
- [Project DAVE : ROS2 Wiki](https://dave-ros2.notion.site/)  
- [UUV Simulator Intro](https://uuvsimulator.github.io/packages/uuv_simulator/intro/)  
- [GitHub, 开源声呐数据集列表](https://github.com/remaro-network/OpenSonarDatasets)  
- [GitHub, cranfield-navigation-gym](https://github.com/mazqtpopx/cranfield-navigation-gym)  
- [SciOpen, 具身智能综述：进展、挑战与未来展望](https://www.sciopen.com/article/10.26599/AIR.2024.9150042)  
- [GitHub, 端到端自监督SLAM](https://github.com/ivanalberico/End-To-End-Self-Supervised-SLAM)  
- [DiVA portal, 用于藻类养殖场检查的AUV SLAM](https://www.diva-portal.org/smash/get/diva2:1845179/FULLTEXT01.pdf)  
- [Bohrium, 覆盖跟随导航规划](https://bohrium.dp.tech/paper/arxiv/2308.06594)  
- [ResearchGate, 用于GPS拒止环境自主探索的双层路径规划](https://www.researchgate.net/publication/371880458_Dual-Layer_Path_Planning_with_Pose_SLAM_for_Autonomous_Exploration_in_GPS-denied_Environments)  
- [DTIC, 水下环境中的主动视觉SLAM信念空间规划](https://apps.dtic.mil/sti/trecms/pdf/AD1124136.pdf)  
- [CMU, 用于水下体积探索的主动SLAM](https://www.cs.cmu.edu/~kaess/pub/Suresh20icra.pdf)  
- [GitHub, 声呐SLAM](https://github.com/jake3991/sonar-SLAM)  
- [GitHub, Awesome-VLA](https://github.com/Orlando-CS/Awesome-VLA)  
- [arXiv, 视觉-语言-行动模型：概念、进展、应用与挑战](https://arxiv.org/html/2505.04769v1)  
- [MarkTechPost, OceanSim高保真水下模拟器](https://www.marktechpost.com/2025/04/07/university-of-michigan-researchers-introduce-oceansim-a-high-performance-gpu-accelerated-underwater-simulator-for-advanced-marine-robotics/)  
- [ResearchGate, UUV Simulator: A Gazebo-based Package](https://www.researchgate.net/publication/308642839_UUV_Simulator_A_Gazebo-based_Package_for_Underwater_Intervention_and_Multi-Robot_Simulation)  
- [GitHub, lauv_gazebo](https://github.com/uuvsimulator/lauv_gazebo)  
- [GitHub, UUV Simulator](https://github.com/uuvsimulator)  
- [GitHub, uuv_simulator](https://github.com/uuvsimulator/uuv_simulator)  
- [ResearchGate, 用于水下作业的VR增强机器人仿真环境](https://www.researchgate.net/publication/391877669_A_Robot_Simulation_Environment_for_Virtual_Reality_Enhanced_Underwater_Manipulation_and_Seabed_Intervention_Tasks)  
- [Hugging Face Blog, π0 and π0-FAST](https://huggingface.co/blog/pi0)  
- [ResearchGate, 水下应用中前视声呐的旋转目标检测](https://www.researchgate.net/publication/335221706_Rotated_object_detection_with_forward-looking_sonar_in_underwater_applications)  
- [DataCamp, 使用Gymnasium进行强化学习的实用指南](https://www.datacamp.com/tutorial/reinforcement-learning-with-gymnasium)  
- [GitHub, semantic_slam](https://github.com/hridaybavle/semantic_slam)  
- [GitHub, OpenVLA (reazon-research)](https://github.com/reazon-research/openvla)  
- [OpenVLA 官网](https://openvla.github.io/)  
- [arXiv, 视觉-语言导航：综述与分类](https://arxiv.org/pdf/2108.11544)  
- [arXiv, 面向真实UAV视觉-语言导航的平台、基准与方法](https://arxiv.org/abs/2410.07087)  
- [GitHub, Awesome Embodied VLA/VA/VLN](https://github.com/jonyzhang2023/awesome-embodied-vla-va-vln)
