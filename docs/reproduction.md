# 灵巧手情报复现路线

最后更新：2026-07-23

## 复现优先级定义

- **低难度**：普通相机/PC即可跑通，或仅需读取公开资产。
- **中难度**：需要特定传感器、灵巧手或较重仿真环境。
- **高难度**：需要多机器人、复杂触觉标定、大模型训练或高成本硬件。

## 当前复现队列

| 优先级 | 项目 | 目标 | 最小可行复现 | 难度 | 当前阻塞 |
|---|---|---|---|---|---|
| P0 | UHAS | 验证一个策略跨多手型动作空间 | 在Isaac Lab中跑LEAP/Allegro预训练模型并导入新URDF | 中高 | Isaac Sim版本、显存、真机仓库 |
| P0 | Wuji v2026.7.21 | 验证自描述触觉流与校准事件 | 使用官方typed tactile示例记录五指元数据与数据帧 | 中 | 需要Wuji Hand 2 Beta硬件 |
| P0 | MVP-Tac | 验证微型视觉/触觉切换与0–2N力估计 | 制作单个传感头，采集图像—力标定数据 | 中 | 光路装配、涂层一致性、开源仓库核验 |
| P1 | Inspire Hand Retargeting | 建立视觉冗余手势通道 | D405/普通相机跑MediaPipe和6维UDP输出 | 低 | License、真机映射 |
| P1 | XHAND1 Slip Detection | 建立滑移与抓力残差字段 | 在MuJoCo复现抓瓶、抬升、旋转和teacher search | 中 | 真机taxel接口未开放 |
| P1 | LinkerHand/WHATsLAB URDF | 建立跨手型模型注册表 | 下载URDF，计算关节表、耦合、mesh hash和License | 低 | 真机动力学一致性 |
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
4. 执行无载` tactile_calibrate()`并记录校准事件。
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

## 6. 复现结果记录规范

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
