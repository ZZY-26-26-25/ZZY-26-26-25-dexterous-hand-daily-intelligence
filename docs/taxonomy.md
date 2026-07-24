# 灵巧手情报分类体系

最后更新：2026-07-24

## 1. Hardware / 系统形态

- **灵巧手本体**：主动执行的多指机器人手。
- **遥操作（带本体）**：采集时机器人本体在线执行，能记录命令—执行差异和机器人反馈。
- **UMI（无本体）**：采集时不需要真实机器人在线执行，使用手持工具、代理末端或穿戴设备采集。
- **弱本体/代理本体**：采集端有机械约束、读手、代理手或目标手近似结构，但不是完整执行机器人。
- **模块化全身遥操作**：手指、双臂、底盘、头部主动视角、力反馈与记录器可独立替换的系统，例如 ModPack。
- **触觉/力反馈硬件**：触觉皮肤、指尖传感器、F/T、力反馈手套、滑移检测模块。
- **数据集/软件栈**：采集、同步、retarget、训练、评测、world model、VLA 和数据格式工具。
- **视频/公司动态**：新品、SDK、合作、供货、demo 和公司新闻。

## 2. Data Collection / 数据来源

- `robot_teleop`：带本体遥操作。
- `robot_free_umi`：无机器人 UMI。
- `proxy_embodiment`：代理本体、读手或机械同构采集。
- `wearable_fullbody_teleop`：可穿戴双臂/底盘/头部/力反馈系统。
- `human_video`：第一/第三视角人类视频。
- `human_recovery_video`：从预设失败状态开始的人类恢复片段。
- `robot_recovery_demo`：机器人在失败状态下的恢复示教。
- `simulation`：仿真数据。
- `generated`：world model 或生成模型产生的数据。
- `autonomous_robot`：机器人自主执行与回流数据。

所有数据必须记录 `data_source`，不得把人类视频、仿真、生成数据和真机执行混为同一分布。

## 3. Action Representation / 动作表示

- `human_joint_space`：人体或手套关节角。
- `human_fingertip_space`：人体指尖位置/姿态。
- `wrist_6d`：腕部位姿。
- `robot_joint_space`：目标机器人关节命令。
- `eef_6d`：机器人末端位姿。
- `unified_action_space`：跨本体统一动作表示，例如 UHAS 球面形变空间。
- `residual_action`：力控、滑移或触觉策略产生的执行修正。
- `semantic_grasp_seed`：语言/VLM选出的2D或3D目标点，用于条件化后续抓取生成。
- `embodiment_conditioned_grasp`：绑定目标手型的掌部6D与关节抓取候选。
- `masked_visual_action`：在像素空间部分显示机器人或物体轨迹的视频动作表示。
- `object_motion_target`：只描述期望物体运动，用world model或inverse dynamics恢复机器人动作。
- `pre_contact_force_setpoint`：根据视觉、材料和任务预测的接触前抓力目标。
- `task_progress`：人类或机器人轨迹在共享任务流形上的进度。
- `predicted_future_observation`：策略或技能模型预测的未来视觉/状态目标。
- `corrective_intent`：失败恢复中的低维纠错方向、时机和幅度。
- `recovery_gate`：判断何时启用恢复策略的门控信号。
- `haptic_leader_torque`：从机器人F/T或执行状态映射到主设备的力反馈命令。

所有日报应区分：

`operator_input → retargeted_action → nominal_action → residual_action → executed_state`

抓取与世界模型任务还应保留：

`language_instruction → semantic_seed → embodiment_grasp → pre_contact_force → executed_contact → outcome`

生成式动作表示还应保留：

`reference_frame + masked_visual_action → generated_rollout → recovered_robot_action`

单视频技能习得应保留：

`source_video → task_progress → predicted_future_observation → robot_action → observed_progress`

失败恢复应保留：

`failure_state → recovery_clip → corrective_intent → recovery_gate → executed_recovery → recovery_outcome`

## 4. Degrees of Freedom / 自由度组成

所有硬件必须分别记录：

- `active_dof`：独立主动驱动自由度。
- `coupled_dof`：由机械或软件耦合产生的运动自由度。
- `passive_dof`：接触驱动或自由转动的被动自由度。
- `measured_passive_dof`：被动自由度是否有编码器/状态测量。
- `differential_topology`：跨指/关节差动机构的连接方式。
- `actuator_count`：实际执行器数量。

总DoF不能替代上述拆分。欠驱动手可能具有较高运动DoF，但无法提供同等维度的独立动作标签。

## 5. Tactile / Force Data / 触觉与力觉

### 原始层

- `raw_taxel`
- `raw_tactile_image`
- `raw_wrench`
- `motor_current`
- `joint_effort`
- `load_cell_force`
- `leader_follower_tracking_error`

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
- `force_model_version`
- `material_or_object_class`
- `clock_domain`

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
- `pre_contact_force_setpoint`
- `measured_force`
- `damage_result`
- `permanent_deformation`
- `haptic_torque_command`

Wuji v2026.7.21 的自描述指尖触觉流说明：触觉数组必须与传感器元数据绑定，不能假设固定长度和固定布局。

V2F 类方法说明：接触前预测力、接触后实际力和最终损伤/滑移结果必须分开记录。

## 6. Vision / Pose / 视频与位姿

- 原始 RGB / depth / tactile video 必须保存设备时间戳。
- 相机需记录 `intrinsics`、`extrinsics`、`device_serial` 和 `calibration_hash`。
- 腕部位姿需区分视觉 SLAM、外部动捕、XR、VIO 和机器人 FK。
- 视觉与触觉共用相机的传感器应记录 `illumination_state` 和 `sensor_mode`。
- 世界模型数据需保存 `reference_frame`、`mask_video`、`masked_entity_type`、`prompt`、`generation_seed` 和 `checkpoint_hash`。
- 语义抓取数据需保存 `target_object_id`、`seed_pixel`、`seed_point_3d` 和场景点云坐标系。
- 全身遥操作需保存 `operator_head_pose`、`operator_base_pose`、`camera_stream_role` 和主动视角策略。
- 人类恢复数据需保存失败状态出现前后的完整第一视角窗口，而不是只截取成功动作。

## 7. Modular Teleoperation Registry / 模块化遥操作注册表

每个可插拔模块至少记录：

- `module_id`
- `module_role`
- `module_model`
- `hardware_serial`
- `firmware_version`
- `sdk_version`
- `clock_domain`
- `stream_names`
- `nominal_rate_hz`
- `coordinate_frame`
- `calibration_hash`
- `power_source`
- `network_transport`
- `logger_version`

常见 `module_role`：

- `finger_input`
- `leader_arm_left`
- `leader_arm_right`
- `head_tracking`
- `base_tracking`
- `haptic_feedback`
- `camera`
- `compute`
- `power`
- `storage`

## 8. Failure and Recovery / 失败与恢复

### 失败字段

- `failure_state_id`
- `off_nominal_type`
- `failure_onset_frame`
- `failure_source`
- `failure_severity`
- `failure_detected_by`

### 恢复字段

- `recovery_start_frame`
- `recovery_end_frame`
- `corrective_intent`
- `recovery_gate`
- `operator_intervention`
- `recovery_success`
- `recovery_latency`
- `recovery_attempt_count`

### 数据成本字段

- `collection_minutes`
- `valid_episode_count`
- `episodes_per_operator_hour`
- `human_robot_cost_equivalent`

失败episode不得默认删除。应区分：碰撞、漏抓、滑移、掉落、插入偏斜、物体位姿偏差、执行器卡滞、传感器异常、策略停滞和人工接管。

## 9. Embodiment Registry / 目标手注册表

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
- `passive_joint_definition`
- `fingertip_geometry_id`
- `license`

多本体抓取数据库还应记录：

- `source_embodiment`
- `target_embodiment`
- `grasp_representation`
- `mapping_method`
- `cross_embodiment_transfer_result`

## 10. Dataset Release State / 开放状态

- `announced`：论文/项目页声称将开放，但没有可下载数据。
- `placeholder_repo`：仓库已建，但只有README或占位文件。
- `partial_release`：仅权重、推理或部分数据开放。
- `full_release`：数据、代码、配置和License均可用。
- `commercial_sdk`：仅提供闭源/二进制SDK或商业接口。

日报不得把“项目页写release”直接等同于数据已可下载。例如SeededGrasp截至2026-07-24仍属于`placeholder_repo`。

## 11. Evaluation / 评价维度

- 自由度：主动、耦合、被动分别记录。
- 传感器：类型、数量、位置、采样率、单位、量程。
- 手腕6D：来源、频率、漂移和遮挡。
- 时间同步：硬件锁存、设备时钟、主机时钟或后处理对齐。
- SDK开放程度：源码、二进制包、API文档、ROS2、原始数据权限。
- 可复现性：CAD、PCB、BOM、固件、数据、训练代码和真实部署代码。
- 采购状态：公开价、询价、供货、交期和科研支持。
- 数据价值：是否含失败、恢复、触觉、力、命令—执行差异和标定版本。
- 恢复能力：恢复成功率、误触发率、恢复延迟和每小时有效恢复数据量。
- 适配成本：新任务示教数、微调步数、参数是否更新和技能获取时间。
- 模块化：手指、双臂、底盘、头部、力反馈和记录器能否独立替换。
- 抓取安全：滑移、最大力、永久形变、表面损伤和对象类别外泛化。
- 世界模型可信度：动作一致性、身份保持、接触伪影、未见区域伪影和真实结果相关性。

## 12. Reliability / 可靠性

- **高**：论文、项目主页、GitHub、官方文档、公司官网。
- **中**：可靠媒体、作者社交账号、展会材料，但缺技术文档。
- **低**：二手转述、未核验视频、匿名爆料。

同一信息优先使用原始来源；旧项目的新版本应标记“更新项”，不得作为全新项目重复统计。
