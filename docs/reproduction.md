# 灵巧手情报复现路线

最后更新：2026-07-24

## 复现优先级定义

- **低难度**：普通相机/PC即可跑通，或仅需读取公开资产。
- **中难度**：需要特定传感器、灵巧手或较重仿真环境。
- **高难度**：需要多机器人、复杂触觉标定、大模型训练或高成本硬件。

## 当前复现队列

| 优先级 | 项目 | 目标 | 最小可行复现 | 难度 | 当前阻塞 |
|---|---|---|---|---|---|
| P0 | Masked Visual Actions | 验证像素轨迹动作在未见机器人上的world-model推理 | 下载作者LoRA，用一个首帧和控制视频完成推理 | 高 | 14B基础模型、显存/磁盘、URDF渲染器待发布 |
| P0 | UHAS | 验证一个策略跨多手型动作空间 | 在Isaac Lab中跑LEAP/Allegro预训练模型并导入新URDF | 中高 | Isaac Sim版本、显存、真机仓库 |
| P0 | Wuji v2026.7.21 | 验证自描述触觉流与校准事件 | 使用官方typed tactile示例记录五指元数据与数据帧 | 中 | 需要Wuji Hand 2 Beta硬件 |
| P0 | MVP-Tac | 验证微型视觉/触觉切换与0–2N力估计 | 制作单个传感头，采集图像—力标定数据 | 中 | 光路装配、涂层一致性、开源仓库核验 |
| P1 | V2F | 验证视觉预测安全抓力 | 深度/双目相机+两指夹爪+load cell采集软物数据 | 中 | 数据与代码未发布、损伤ground truth设计 |
| P1 | SeededGrasp | 验证多本体语言引导抓取 | 数据/代码发布后跑Franka或Allegro评测 | 中高 | 当前仓库仅占位，数据库未下载 |
| P1 | Inspire Hand Retargeting | 建立视觉冗余手势通道 | D405/普通相机跑MediaPipe和6维UDP输出 | 低 | License、真机映射 |
| P1 | XHAND1 Slip Detection | 建立滑移与抓力残差字段 | 在MuJoCo复现抓瓶、抬升、旋转和teacher search | 中 | 真机taxel接口未开放 |
| P1 | LinkerHand/WHATsLAB URDF | 建立跨手型模型注册表 | 下载URDF，计算关节表、耦合、mesh hash和License | 低 | 真机动力学一致性 |
| P2 | Passive Rotating Underactuated Hand | 验证被动侧摆和差动稳定性 | 根据论文重建单指/双指简化机械模型 | 中高 | CAD、尺寸与制造公差未公开 |
| P2 | T-800 | 评估高频手部数据价值 | 用现有编码器/IMU做200/500/800Hz短时采样对比 | 中 | 现有硬件带宽与同步能力 |

## 1. UHAS 复现计划

### 目标

验证现有22路外骨骼数据是否可以先转换为统一动作空间，再映射到不同机器人手。

### 步骤

1. 安装Isaac Sim 4.5与Isaac Lab 2.2.1。
2. 运行仓库自带LEAP、Allegro、Shadow和MANO环境。
3. 使用预训练multi-hand policy进行1000环境评测。
4. 导入MIDAS、Wuji或LinkerHand URDF。
5. 记录自动球面构建是否成功、CIK频率和关节限位违反次数。
6. 比较：直接关节retarget vs UHAS中间表示。

### 指标

- 指尖位置误差
- 关节限位违反率
- CIK运行频率
- 接触/抓取成功率
- 未见手零样本性能
- 映射丢失的人体自由度

## 2. Wuji触觉流复现计划

### 目标

建立可迁移的自描述触觉记录格式。

### 步骤

1. 固件升级到v2.1.0，SDK升级到v2026.7.21。
2. 获取每指`FingertipSensorInfo`。
3. 记录`FingertipSensorData`原始帧与主机/设备时间戳。
4. 执行无载`tactile_calibrate()`并记录校准事件。
5. 断开/重连SDK，检查时间戳是否重新对齐。
6. 模拟通信中断，检查电机故障状态和恢复流程。
7. 转换为MCAP并验证回放。

### 必须保存

- sensor model
- layout
- unit
- finger id
- calibration state
- zero timestamp
- firmware/sdk version
- host/device clock offset

## 3. MVP-Tac 复现计划

### 目标

制作一个低成本微型双模态视觉—触觉原型。

### 最小硬件

- VytaFlex 20
- 半透明镜面涂层
- 圆偏振膜
- 微型USB内窥相机
- LED
- 亚克力间隔件
- 3D打印模具/外壳
- 0–2N参考力传感器或电子秤标定装置

### 实验

1. 视觉模式清晰度与视场。
2. 触觉模式干涉条纹稳定性。
3. 0–2N法向力回归。
4. 反复加载迟滞与漂移。
5. 1–5Hz动态按压。
6. 表面磨损、涂层脱落和污染。
7. 轻微剪切与滑移是否可识别。

### 判定

若法向力误差、迟滞和寿命满足原型需求，再考虑手指尖封装；否则保留为工具端/腕部探针。

## 4. 视觉冗余与滑移软件

### Inspire Retargeting

- 将6维视觉输出与22路编码器对齐。
- 用视觉检测编码器异常、手套脱落和粗姿态不一致。
- 不将归一化MediaPipe输出作为人体学真值。

### XHAND1 Slip Detection

- 先复现MuJoCo能量流与teacher search。
- 输出统一字段：`slip_score`、`grip_residual`、`wrist_residual`。
- 真机前必须建立真实滑移ground truth，例如视觉标记、力传感器或物体IMU。

## 5. 高采样率实验

用现有手套完成同一批快速动作：

- 快速握拳/张开
- 指尖敲击
- 转笔
- 抛接小物体
- 快速换指

分别以100、200、500和可能的800Hz采集，比较：

- 频谱能量
- 峰值速度/加速度
- 相位延迟
- 丢帧
- 总线占用
- 功耗
- 存储速度
- 降采样后retarget误差

## 6. SeededGrasp 复现计划

### 目标

验证语言语义seed与embodiment-specific抓取生成能否接入我们的多手型数据库。

### 等待发布后执行

1. 下载2.56M抓取数据库并核验License、场景/物体划分与位姿坐标系。
2. 先复现Franka或Robotiq三指评测，再检查Allegro数据的关节、碰撞与接触定义。
3. 将同一RGB-D场景绑定多种`embodiment_id`，比较抓取候选分布。
4. 记录语言seed、3D seed和最终6D抓取之间的误差传播。
5. 若有现有LEAP/MIDAS/Wuji模型，测试仅替换目标手时所需的再训练量。

### 指标

- 语义目标命中率
- seed投影误差
- grasp collision rate
- simulation success
- real execution success
- cross-embodiment transfer drop
- tactile/slip failure after geometric success

## 7. V2F最小复现计划

### 目标

建立“视觉预接触抓力预测 + 执行端力/电流修正”的脆弱物体抓取实验。

### 最小硬件

- 一台RGB-D或双目相机
- 可设置抓力的两指夹爪或灵巧手
- load cell/六维力传感器/电子秤标定台
- 纸杯、软水果、泡沫、薄包装等多刚度对象

### 步骤

1. 采集尺寸、曲率、表面状态与破坏/滑移阈值。
2. 建立简单Hertz/线性接触基线。
3. 训练残差模型预测`pre_contact_force_setpoint`。
4. 真机执行并同时记录预测力、实际力、电流、滑移与永久形变。
5. 与固定抓力、触觉试探和Current-as-Touch式在线修正比较。

### 指标

- force prediction error
- slip rate
- damage rate
- permanent deformation
- cycle time
- OOD material performance

## 8. Masked Visual Actions 冒烟测试

### 目标

验证普通人类/机器人视频中的局部像素运动能否作为跨本体world-model动作接口。

### 步骤

1. 按仓库固定commit安装DiffSynth-Studio。
2. 下载作者的高噪声与低噪声LoRA权重。
3. 使用作者示例或自制`reference_image + control_video + prompt`完成一次推理。
4. 制作一个机器人mask和一个物体mask，比较forward与inverse输出。
5. 记录基础模型、LoRA、prompt、seed、分辨率、推理步数和显存。
6. 等待DROID→URDF渲染工具发布后，再接入真实episode。

### 判定

- 动作方向与目标一致性
- 物体/机器人身份保持
- 未见embodiment泛化
- 精细接触伪影
- 生成rollout与真实结果的一致性
- 推理成本是否适合离线数据增强

## 9. 欠驱动机械原型

### 目标

验证被动指根旋转是否能在不增加执行器的情况下扩展抓型，同时评估其对数据采集和独立控制的限制。

### 最小复现

1. 根据论文几何重建一个两节指和被动基部旋转轴。
2. 制作双指差动模型，使用弹簧/绳索或简单滑轮分配力矩。
3. 测试圆柱、球体、板件和平行夹持。
4. 测量被动轴角度、接触力矩、物体挤出条件和重复性。
5. 与固定指根和全主动指根比较。

## 10. 复现结果记录规范

每次复现必须提交：

- 环境与依赖版本
- Git commit
- 硬件BOM与版本
- 固件/SDK版本
- 标定文件hash
- 原始数据链接
- 命令行与配置
- 成功/失败视频
- 定量指标
- 已知失败模式
- 下一步决策