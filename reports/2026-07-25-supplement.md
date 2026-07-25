# 灵巧手每日情报补充｜2026-07-25

> 本补充报告用于收录主日报生成后发现的高价值新增，重点核验过去 24–72 小时内新建立或新公开的开源仓库，以及此前遗漏的真实机器人论文。旧论文对应的新代码发布统一标记为“更新项”，不冒充全新工作。

## 0. 今日一句话判断

本轮增量不在新商业手本体，而在 **仿真—真机接口同构、带掌部自由度灵巧手基准、以及真实人形 VLA 后训练数据工程**。最值得立即关注的是 `G1_BrainCo_hands_Digital_Twin_Mujoco`：它把 Unitree G1 29-DoF 身体与 BrainCo Revo2 型双手放入同一个 MuJoCo 场景，并复用真机 DDS topic、消息结构和控制约定，使控制程序只改网络配置即可在仿真与真机间切换。

`ISyHand RL` 是旧论文对应的新开源更新，公开了带可动掌部的 ISyHand、平掌版本、Allegro 与 LEAP 的统一立方体手内重定向训练和预训练 checkpoint。`DEED` 则给出了 Unitree G1-Edu + Dex-3 在超市场景中从 0% 到 42% 的真实部署改进过程，同时显示第二轮自生成经验加入后性能反而下降到 22%，说明自动回流数据必须记录分布漂移和质量门控。

## 1. 今日高价值新增

| 优先级 | 发布时间 | 类型 | 名称 | 链接 | 所属分类 | 硬件形态 | 自由度/传感 | 反馈类型 | 手指姿态检测 | 手腕6D检测 | 是否开源 | 关键实验/演示 | 为什么重要 | 可靠性 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| P0 | 2026-07-24 新仓库 | Open Source | G1 BrainCo Hands Digital Twin | [GitHub](https://github.com/Humanoid-Team-ISR-Lab/G1_BrainCo_hands_Digital_Twin_Mujoco) | 数据集/软件栈 | Unitree G1 29-DoF 身体 + 双 BrainCo Revo2 型手的 MuJoCo 数字孪生 | 身体 29 主动 DoF；每手 6 主动 / 11 总 DoF；DDS 状态、关节位置/速度/力矩、IMU、头部第一视角相机 | 仿真本体反馈；手部电流估计与真实触觉尚未建模 | 机器人手关节状态 | 机器人 FK / 头部相机 | 是，BSD-3-Clause；部分 BrainCo 参考片段不在主许可证范围 | 仿真与真机使用相同 `rt/lowcmd`、`rt/lowstate`、`rt/brainco/{side}/cmd/state` 接口 | 可在购买或接触真机前验证 G1+BrainCo 控制、记录和 DDS 数据结构 | 高（社区代码，接口依据官方仓库核验） |
| P0 | 2026-07-24 新仓库；论文为 2025 | Open Source | ISyHand RL（更新项） | [GitHub](https://github.com/ISyHand/isyhand-rl) · [项目页](https://isyhand.is.mpg.de/) · [论文](https://arxiv.org/abs/2509.26236) | 数据集/软件栈 | ISyHand 带可动掌部灵巧手、平掌消融、Allegro、LEAP 的 Isaac Gym 统一基准 | 多手型 proprioception；立方体手内重定向；各手预训练 PPO checkpoint | 仿真接触与本体反馈 | 机器人关节状态 | 不涉及独立腕部6D | 是；代码公开，URDF/mesh 需从各官方来源另行下载 | 同一任务对比 articulated palm、flat palm、Allegro 与 LEAP | 可量化“掌部自由度是否真的改善手内操作”，并提供可运行多手型 checkpoint | 高 |
| P0 | 2026-07-22 | Paper | DEED: Closing the Lab-to-Store Gap | [arXiv](https://arxiv.org/abs/2607.20345) | 数据集/软件栈 | Unitree G1-Edu + Dex-3 双手 + 头部/双腕相机；GR00T N1.6 | 三路 RGB；20 维简化动作；81 条遥操作示教、116 条自主 rollout；人工纠错与价值评分 | 真机执行结果、人工纠错、优势标签和 OOD 分析；无明确原始触觉闭环 | 使用机器人 Dex-3 手状态，动作中将手简化为开/闭 | 机器人 FK 和腕部相机 | 未见代码 | 超市薯片补货；naive SFT 0%，数据工程 32%，第一轮经验回流 42%，第二轮下降至 22% | 公开了真实 VLA 从失败到可用、再因分布漂移退化的完整系统经验 | 高 |

## 2. 重点解读

### G1 BrainCo Hands Digital Twin

- **原始链接：** [GitHub](https://github.com/Humanoid-Team-ISR-Lab/G1_BrainCo_hands_Digital_Twin_Mujoco)
- **一句话概括：** 在 MuJoCo 中复现 Unitree G1 29-DoF 身体和 BrainCo 双手，并尽量让仿真与真机使用相同 DDS 传输、topic、消息类型和控制习惯。
- **硬件是什么：** 目标真机组合是 Unitree G1 29-DoF 人形机器人和 BrainCo Revo2 型双手。仓库基于 Unitree 官方 `unitree_mujoco`，补充官方模拟器原本没有的 BrainCo hand bridge、执行器模型和腱耦合关系。
- **采集/控制链路：** 控制程序向 `rt/lowcmd` 发布身体命令并从 `rt/lowstate` 读取身体状态；左右手分别通过 `rt/brainco/left/cmd/state` 与 `rt/brainco/right/cmd/state` 通信。仿真和真机切换主要修改 DDS domain 与网络接口，而不是重写控制层。
- **自由度与传感：** G1 身体 29 个执行关节；每只 Revo2 型手 6 个主动关节、11 个总自由度，额外 5 个关节由 tendon/equality constraint 被动耦合。身体 bridge 发布位置、速度、力矩与 IMU；手 bridge 当前以归一化位置状态为主。
- **反馈方式：** 仿真关节和 IMU 反馈；不含真实高密度触觉。仓库明确说明 `tau_est` 在真机由电机电流推导，但仿真没有电流模型，因此保持为 0，基于电流阈值的抓取/卡滞检测无法直接验证。
- **开源内容：** 自包含场景、mesh、身体与手 DDS bridge、第一视角相机切换、状态监听和全身+双手插值动作示例。主仓库为 BSD-3-Clause，但引用的 BrainCo 模型片段需遵守上游未明确的许可条件。
- **和 DexUMI / DOGlove / UMI-Dex / SynGlove 的关系：** 这些采集端可生成操作者动作；本仓库提供目标人形/手的统一执行后端，可提前测试 joint mapping、消息频率和记录器。它不能替代真实接触、触觉和命令—执行误差数据。
- **对我们搭建平台的启发：** 设备抽象层应保证 sim/real 共用 topic schema，并为每个 stream 保存 `transport`、`topic_name`、`message_schema_hash`、`unit_convention` 和 `simulated_or_real`。仿真缺失的电流、触觉和 tip contact 必须显式标记，不能填充伪值。
- **风险或不确定性：** 指尖被动关节在 BrainCo 左右手参考模型中定义不一致；没有手部电流与触觉模型；社区仓库并非 Unitree 或 BrainCo 官方联合发布，真机前仍需逐字段核验。

### ISyHand RL：带可动掌部的多手型重定向基准

- **原始链接：** [GitHub](https://github.com/ISyHand/isyhand-rl) · [项目页](https://isyhand.is.mpg.de/) · [论文](https://arxiv.org/abs/2509.26236)
- **一句话概括：** 在统一 IsaacGymEnvs 任务中比较带可动掌部 ISyHand、平掌版本、Allegro 和 LEAP，并公开每种手的 PPO 训练配置与预训练 checkpoint。
- **硬件是什么：** 核心对象为 ISyHand 多指手及其 articulated palm；对照包含 ISyHand flat、Allegro 和 LEAP。该仓库是仿真实验代码，真实硬件与模型文件需要从对应官方项目下载。
- **采集/控制链路：** 下载各手 URDF/mesh → 配置 Isaac Gym 立方体初始与目标位置 → PPO 训练手内重定向 → 单环境评测与成功次数统计 → 可保存时间戳图像和评测 dataframe。
- **自由度与传感：** 输入主要为仿真本体和物体状态。价值不在新增触觉传感器，而在把掌部结构变化作为可重复的消融变量。
- **反馈方式：** 仿真接触、本体与任务奖励；无操作者端反馈。
- **实验与开源价值：** 仓库已提供 ISyHand、ISyHand flat、Allegro 和 LEAP 四组预训练策略，可直接检查环境、checkpoint 和评测流程，不必先完整训练。
- **和现有方案的关系：** UHAS、AnyDexRT 等关注跨手型动作映射；ISyHand RL 可作为下游物理任务 benchmark，检验相同人手动作映射到不同掌部结构后是否提升重定向与稳定性。
- **对我们搭建平台的启发：** embodiment registry 不应只有手指 DoF，还要记录 `palm_dof`、`palm_articulation_type`、`palm_configuration` 和 `palm_state`。外骨骼映射评估可加入 articulated-palm 与 flat-palm 对照。
- **风险或不确定性：** URDF/mesh 不随仓库直接分发，存在多上游许可证；代码基于旧版 Isaac Gym；真实触觉、执行器摩擦和 sim-to-real 结果不在该仓库验证范围内。

### DEED：真实零售人形 VLA 后训练与经验回流

- **原始链接：** [arXiv](https://arxiv.org/abs/2607.20345)
- **一句话概括：** 用不到两小时的真实数据和系统级数据工程，将在超市补货任务中完全失败的 GR00T N1.6 策略提升到 42%，同时展示盲目追加自主经验会因分布漂移导致退化。
- **硬件是什么：** Unitree G1-Edu，配 Dex-3 双手、头部相机和双腕相机，任务为从货架附近拾取薯片袋并放回指定位置。
- **采集/控制链路：** 遥操作示教 → 频率对齐、失败尾段裁剪、连续子任务保留、任务相关区域高亮和动作空间简化 → foundation model post-training → 真机自主 rollout → 人工纠错/优势前缀和视觉语言价值函数筛选 → 经验驱动再训练。
- **自由度与传感：** 原始机器人动作被压缩为 20 维：双臂各 7 个关节、左右手二值开合、腰部 1 维和底盘 3 维速度；输入为头部和双腕三路 RGB。该简化牺牲了 Dex-3 手的细粒度动作，不适合作为多指灵巧操作动作格式。
- **数据规模：** 81 条遥操作示教，约 51.5 分钟；116 条自主 rollout，41 成功、75 失败，约 56.9 分钟；总真实数据约 108.4 分钟。
- **方法核心与结果：** naive SFT 成功率 0%；加入频率对齐、数据清洗、视觉高亮和动作简化后达到 32%；第一轮经验驱动 refinement 达到 42%；第二轮继续加入经验后降至 22%，作者通过 latent/OOD 分析将其归因于分布漂移和低质量回流。
- **和 DexUMI / DOGlove / UMI-Dex / SynGlove 的关系：** 这些设备解决示教获取，DEED强调示教到训练之间的数据工程。其经验说明：真实数据量少并非必然失败，频率、失败尾段、恢复动作和动作空间定义可能比简单增加 episode 更重要。
- **对我们搭建平台的启发：** 增加 `camera_rate_hz`、`record_rate_hz`、`policy_rate_hz`、`raw_episode`、`curated_episode`、`trim_reason`、`autonomous_rollout`、`human_correction`、`advantage_label`、`ood_score`、`refinement_round` 与 `policy_checkpoint_parent`。
- **风险或不确定性：** 42%仍不足以直接部署；动作空间将灵巧手简化为开/闭，不能证明多指策略能力；没有公开完整代码和数据；第二轮下降说明自训练数据比例与筛选规则高度敏感。

## 3. 今日开源与可复现价值

### G1 BrainCo Hands Digital Twin

- **链接：** [GitHub](https://github.com/Humanoid-Team-ISR-Lab/G1_BrainCo_hands_Digital_Twin_Mujoco)
- **开源内容：** MuJoCo场景、G1身体bridge、BrainCo双手DDS bridge、mesh、第一视角相机和示例控制程序。
- **License：** BSD-3-Clause；BrainCo参考片段不一定受主许可证覆盖。
- **硬件文件：** 含仿真mesh/MJCF，不含制造CAD/BOM。
- **固件：** 无。
- **数据采集代码：** 提供状态监控和DDS模板，可作为episode logger基础。
- **训练代码：** 无策略训练代码。
- **复现难度：** 低到中；普通PC即可运行MuJoCo，真机切换需Unitree/BrainCo硬件与DDS网络。

### ISyHand RL

- **链接：** [GitHub](https://github.com/ISyHand/isyhand-rl)
- **开源内容：** IsaacGymEnvs任务、PPO训练/评估脚本、四种手的预训练checkpoint。
- **License：** 以仓库`NOTICE.md`和各上游资产许可证为准。
- **硬件文件：** 不直接包含手模型，需从官方ISyHand、IsaacGymEnvs和LEAP仓库下载。
- **固件：** 无。
- **数据采集代码：** 仿真rollout与评测记录。
- **训练代码：** 是。
- **复现难度：** 中；依赖旧版Isaac Gym和多来源模型资产。

### DEED

- **链接：** [arXiv](https://arxiv.org/abs/2607.20345)
- **开源内容：** 当前主要为论文与系统经验总结。
- **License：** 论文采用arXiv对应许可；代码/数据未发布。
- **硬件文件/固件：** 无。
- **数据采集代码：** 未见。
- **训练代码：** 未见。
- **复现难度：** 高；需要G1-Edu、Dex-3、GR00T训练环境和真实零售任务场景。

## 4. 商业产品与采购线索

本轮没有新的商业灵巧手型号或正式报价。新增采购判断如下：

1. **数字孪生接口一致性应成为采购要求。** 厂商若提供仿真模型，应确认仿真与真机是否使用相同topic、消息结构、单位、关节顺序和控制模式，而不只是“外观相同的URDF”。
2. **Revo2与Revo3不可混用。** 本次数字孪生基于Revo2型6主动/11总DoF模型，不能据此推断Revo3触觉版、控制频率或SDK行为。
3. **必须询问电流/力矩/触觉在仿真中的可用性。** 只有位置模型而没有电流、摩擦、tip contact和触觉，不能验证Current-as-Touch、滑移或卡滞检测。
4. **采购人形平台时索取数据工程样例。** 至少应提供相机/记录/推理频率、原始与清洗后episode、故障尾段、人工纠错和自主rollout日志。

## 5. 视频与 demo

### G1 BrainCo数字孪生示例

- **链接：** [GitHub](https://github.com/Humanoid-Team-ISR-Lab/G1_BrainCo_hands_Digital_Twin_Mujoco)
- **展示内容：** G1全身与双手共同运行、DDS状态监听、插值动作和头部第一视角切换。
- **真实机器人还是仿真：** MuJoCo仿真，接口设计面向真机。
- **连续操作：** 支持连续全身+手动作。
- **失败案例：** README明确列出tip关节、电流估计和`arm_sdk`缺失。
- **是否值得深入：** 很值得，适合作为Unitree+BrainCo购置前的软件冒烟测试。

### ISyHand手内重定向

- **链接：** [项目页](https://isyhand.is.mpg.de/) · [GitHub](https://github.com/ISyHand/isyhand-rl)
- **展示内容：** articulated palm、flat palm、Allegro和LEAP的立方体手内重定向。
- **真实机器人还是仿真：** 本仓库为仿真，论文项目包含真实手设计。
- **连续操作：** 是。
- **触觉/力反馈：** 仿真接触，无人端反馈。
- **是否值得深入：** 值得，重点验证掌部自由度对任务可达性和学习效率的影响。

### DEED超市补货

- **链接：** [arXiv](https://arxiv.org/abs/2607.20345)
- **展示内容：** G1-Edu在真实零售货架环境中拾取并补放薯片袋，以及不同后训练阶段的表现。
- **真实机器人还是仿真：** 真实机器人。
- **连续操作：** 是，包含移动、双臂操作和恢复。
- **失败案例：** 论文重点分析失败、自主rollout和第二轮训练退化。
- **是否值得深入：** 非常值得，适合作为真实VLA数据工程反例与经验课。

## 6. 对我们项目的直接建议

1. **立即对G1+BrainCo数字孪生做软件冒烟测试。** 不需要真机即可检查DDS topic、消息结构、手部关节顺序和第一视角记录。
2. **把仿真接口同构度加入硬件评分。** 记录`sim_transport_parity`、`topic_parity`、`message_schema_parity`、`unit_parity`和`missing_physics_channels`。
3. **使用ISyHand评估掌部自由度价值。** 在相同立方体重定向任务中比较articulated palm、flat palm、LEAP和Allegro，再决定外骨骼是否应额外记录掌弓/掌骨运动。
4. **建立原始数据与清洗数据父子链。** 永久保存失败尾段与恢复动作，同时允许生成训练用精炼版本。
5. **严格记录采集、相机、控制与推理频率。** DEED显示频率不一致可能使正确示教变成错误监督。
6. **自动经验回流必须设质量门控。** 每轮保存checkpoint父子关系、数据来源、OOD分数、优势标签与采样比例；性能下降时能够回滚。
7. **不要把Dex-3二值开合数据当成灵巧手全关节数据。** 数据库中应明确`action_abstraction_level`，区分完整关节、低维协同和开/闭命令。

## 7. 待跟踪清单

- **G1 BrainCo Digital Twin：** 真机topic逐字段验证、Revo2具体硬件版本、手部状态频率、tip关节模型、电流/`tau_est`和触觉扩展。
- **ISyHand RL：** `NOTICE.md`许可证、官方URDF下载条款、Isaac Gym版本兼容、真实ISyHand硬件开放程度及触觉扩展。
- **DEED：** 代码、数据、相机频率、推理频率、纠错标注格式、value function、第二轮退化的完整样本分布。
- **BrainCo Revo2/3：** 官方数字孪生、真机DDS topic、原始电流/触觉接口和不同代产品模型差异。
- **过滤说明：** FORGE-plus因仅有纯仿真验证且未主张sim-to-real，本轮不纳入主表；TableVerse为大规模Real2Sim资源，但没有直接硬件闭环，保留为低优先级世界模型线索。

## 8. 明日重点搜索关键词

- `G1 BrainCo hands digital twin DDS real robot`
- `BrainCo Revo2 current tau_est DDS topic`
- `BrainCo Revo3 MuJoCo digital twin tactile`
- `ISyHand articulated palm RL real robot`
- `ISyHand CAD hardware open source`
- `DEED G1 supermarket restocking code dataset`
- `GR00T N1.6 experience driven refinement`
- `robot autonomous rollout distribution drift`
- `humanoid VLA data curation failed tail recovery`
- `灵巧手 数字孪生 真机接口 DDS`
- `灵巧手 掌部自由度 手内操作`
- `人形机器人 自主经验 回流 分布漂移`
