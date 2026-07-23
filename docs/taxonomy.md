# 灵巧手情报分类体系

最后更新：2026-07-23

## 1. Hardware / 系统形态

- **灵巧手本体**：主动执行的多指机器人手。
- **遥操作（带本体）**：采集时机器人本体在线执行，能记录命令—执行差异和机器人反馈。
- **UMI（无本体）**：采集时不需要真实机器人在线执行，使用手持工具、代理末端或穿戴设备采集。
- **弱本体/代理本体**：采集端有机械约束、读手、代理手或目标手近似结构，但不是完整执行机器人。
- **触觉/力反馈硬件**：触觉皮肤、指尖传感器、F/T、力反馈手套、滑移检测模块。
- **数据集/软件栈**：采集、同步、retarget、训练、评测、world model、VLA 和数据格式工具。
- **视频/公司动态**：新品、SDK、合作、供货、demo 和公司新闻。

## 2. Data Collection / 数据来源

- `robot_teleop`：带本体遥操作。
- `robot_free_umi`：无机器人 UMI。
- `proxy_embodiment`：代理本体、读手或机械同构采集。
- `human_video`：第一/第三视角人类视频。
- `simulation`：仿真数据。
- `generated`：world model 或生成模型产生的数据。
- `autonomous_robot`：机器人自主执行与回流数据。

## 3. Action Representation / 动作表示

- `human_joint_space`：人体或手套关节角。
- `human_fingertip_space`：人体指尖位置/姿态。
- `wrist_6d`：腕部位姿。
- `robot_joint_space`：目标机器人关节命令。
- `eef_6d`：机器人末端位姿。
- `unified_action_space`：跨本体统一动作表示，例如 UHAS 球面形变空间。
- `residual_action`：力控、滑移或触觉策略产生的执行修正。

所有日报应区分：

`operator_input → retargeted_action → nominal_action → residual_action → executed_state`

## 4. Tactile / Force Data / 触觉与力觉

### 原始层

- `raw_taxel`
- `raw_tactile_image`
- `raw_wrench`
- `motor_current`
- `joint_effort`

### 元数据层

- `sensor_model`
- `sensor_metadata_version`
- `finger_id`
- `link_id`
- `taxel_layout`
- `unit`
- `calibration_id`
- `calibration_state`
- `zero_timestamp`
- `sensor_to_link_transform`

### 派生层

- `contact_state`
- `contact_location`
- `normal_force`
- `shear_force`
- `slip_state`
- `slip_score`
- `tactile_latent`
- `future_tactile_target`
- `grip_residual`
- `wrist_residual`

Wuji v2026.7.21 的自描述指尖触觉流说明：触觉数组必须与传感器元数据绑定，不能假设固定长度和固定布局。

## 5. Vision / Pose / 视频与位姿

- 原始 RGB / depth / tactile video 必须保存设备时间戳。
- 相机需记录 `intrinsics`、`extrinsics`、`device_serial` 和 `calibration_hash`。
- 腕部位姿需区分视觉 SLAM、外部动捕、XR、VIO 和机器人 FK。
- 视觉与触觉共用相机的传感器应记录 `illumination_state` 和 `sensor_mode`，例如 MVP-Tac 的视觉/触觉模式切换。

## 6. Embodiment Registry / 目标手注册表

每条机器人数据至少绑定：

- `hand_model`
- `manufacturer`
- `urdf_repo`
- `urdf_commit`
- `mesh_hash`
- `joint_mapping_version`
- `collision_version`
- `firmware_version`
- `sdk_version`
- `handedness`
- `actuated_joint_mask`
- `coupled_joint_definition`
- `license`

## 7. Evaluation / 评价维度

- 自由度：主动、耦合、被动分别记录。
- 传感器：类型、数量、位置、采样率、单位、量程。
- 手腕6D：来源、频率、漂移和遮挡。
- 时间同步：硬件锁存、设备时钟、主机时钟或后处理对齐。
- SDK开放程度：源码、二进制包、API文档、ROS2、原始数据权限。
- 可复现性：CAD、PCB、BOM、固件、数据、训练代码和真实部署代码。
- 采购状态：公开价、询价、供货、交期和科研支持。
- 数据价值：是否含失败、恢复、触觉、力、命令—执行差异和标定版本。

## 8. Reliability / 可靠性

- **高**：论文、项目主页、GitHub、官方文档、公司官网。
- **中**：可靠媒体、作者社交账号、展会材料，但缺技术文档。
- **低**：二手转述、未核验视频、匿名爆料。

同一信息优先使用原始来源；旧项目的新版本应标记“更新项”，不得作为全新项目重复统计。
