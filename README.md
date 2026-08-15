# FLY_DRONE_330 — 自主无人机飞行系统

基于 fast-lio2 (Mid360  高频版)  Diff-Planner 的室内外自主飞行平台。

## 系统架构

```
Mid360 LiDAR ──(pointcloud+imu)──> fast_lio2 ──(/Odometry)─────────> 
                                                                   │
PX4/MAVROS ────(/livox/imu)────────────────────────────────────────┘
                                                                   │
                                                      (/Odom_high_freq)
                                                             │
                ┌────────────────────────────────────────────┤
                ▼                            ▼               ▼
          diff_planner                    px4ctrl        multipoint
                │                            │               │
                └──(/setpoints_cmd)──────────►               │
                                            │                │
                                PX4 ◄───────┘                │
                                                             │
                         触发: RC CH8 / RViz 2D/3D Nav Goal
```

## 硬件要求

| 组件 | 型号 | 用途 |
|---|---|---|
| 飞控 | PX4 (Pixhawk 系列) | 姿态控制、底层稳定 |
| LiDAR | Livox Mid-360 | 3D 点云建图与定位 |
| 机载电脑 | Intel 13代 i5 Pro | 运行定位、规划、控制算法 |
| 相机 (可选) | Intel RealSense D435i | 视觉辅助 |

## 软件栈

| 模块 | 路径 | 功能 |
|---|---|---|
| **faster_lio** | `src/realflight_modules/mid360_fastlio/` | faster-lio 激光惯性里程计 |
| **ekf_pose** | `src/realflight_modules/ekf_pose/` | EKF 多传感器融合 (LIO + IMU) |
| **px4ctrl** | `src/realflight_modules/px4ctrl/` | 几何控制器，轨迹跟踪 |
| **diff_planner** | `src/diff_planner/` | 差分平坦轨迹规划 + 避障 |
| **multipoint** | `src/user_command/multipoint/` | 多点航点导航 |
| **realsense-ros** | `src/realflight_modules/realsense-ros/` | RealSense 相机驱动 |

## 环境准备

### 依赖

- Ubuntu 20.04
- ROS Noetic
- PX4 Firmware + MAVROS
- [Livox SDK2](https://github.com/Livox-SDK/Livox-SDK2)
- [livox_ros_driver2](https://github.com/Livox-SDK/livox_ros_driver2)

### 编译

```bash
cd ~/fasterlio2diff_ws
catkin_make -j8
```

### 环境变量

每次新终端需 source (顺序固定):

```bash
source /opt/ros/noetic/setup.bash
source ~/fasterlio2diff_ws/devel/setup.bash
source ~/ws_livox/devel/setup.bash
```

## 快速启动

### 一键启动 (5 个终端)

```bash
# 终端A: 传感器 (Mid360 + MAVROS)
bash fly_sensor.sh

# 终端B: 定位 (faster_lio + EKF)
bash fly_mapping.sh

# 终端C: 规划 (diff_planner)
bash fly_planner.sh

# 终端D: 控制 (px4ctrl)
bash fly_run_ctrl.sh

# 终端E: 可视化 (RViz)
bash fly_rviz.sh
```

### 执行飞行任务

1. 编辑航点: `src/user_command/multipoint/config/points.yaml`
2. 设置参数: `src/user_command/multipoint/launch/multipointplan_exp_lio.launch`
3. 起飞后触发: RViz 中点击 **2D Nav Goal**，或拨动遥控器 **CH8**

```bash
bash fly_takeoff.sh    # 起飞
bash fly_trigger.sh    # 开始执行航点
bash fly_back.sh       # 返航
bash fly_land.sh       # 降落
```

> 详细命令参考: [Diff-Planner_命令顺序总结.txt](Diff-Planner_命令顺序总结.txt)

## 录包

```bash
bash "record copy.sh"   # LIO + EKF 里程计 → lio_ekf_<timestamp>.bag
bash fly_record.sh      # 规划 + 可视化   → planner_<timestamp>.bag
```

## 仿真

```bash
roslaunch map_generator random_forest.launch
roslaunch px4ctrl run_ctrl.launch
roslaunch diff_planner run_sim_single.launch
roslaunch multipoint multipointplan_sim.launch
roslaunch rviz rviz -d $(rospack find diff_planner)/launch/include/sim.rviz
```

## 目录结构

```
fasterlio2diff_ws/
├── src/
│   ├── realflight_modules/       # 实飞模块 (LIO, EKF, 控制器, 相机)
│   ├── diff_planner/             # 差分平坦轨迹规划与避障
│   ├── user_command/multipoint/  # 多点航点导航
│   ├── uav_simulator/            # 仿真环境
│   └── utils/                    # 工具包 (可视化, 消息, RViz 插件)
├── fly_*.sh                      # 一键启动脚本 (sensor/mapping/planner/ctrl/rviz)
├── "record copy.sh"              # LIO+EKF 数据录制
├── fly_record.sh                 # 规划可视化数据录制
└── Diff-Planner_命令顺序总结.txt  # 操作手册
```

## 实机成果补充（空中具身智能挑战赛）

- **视频链接（B站）**：https://www.bilibili.com/video/BV1dFVK6CERg/
- **视频说明**：本视频为“空中具身智能挑战赛 / 智能无人机项目”的实机飞行展示。
- **飞行平台**：团队自制四旋翼无人机，搭载 Intel NUC 机载计算机与 Livox Mid-360 激光雷达。
- **核心算法链路**：FAST-LIO2（改）激光惯性里程计定位 + Diff-Planner 局部路径规划 + PX4 / px4ctrl 轨迹跟踪控制。
- **展示内容**：RViz 可视化、实时建图、轨迹规划与第三人称视角实飞过程。
- **任务目标**：面向复杂障碍环境下的空中机器人自主导航，验证结构化/非结构化障碍区中的感知、规划与控制闭环能力。
- **关键词**：无人机、四旋翼、具身智能、自主导航、SLAM、路径规划、ROS、PX4、空中机器人。
- **备注**：视频为项目展示与学习交流用途，不代表最终比赛成绩。

## 致谢

- [faster-lio](https://github.com/hku-mars/FAST_LIO) — HKU Mars Lab
- [Ego-Planner](https://github.com/ZJU-FAST-Lab/ego-planner) — ZJU FAST Lab
- [px4ctrl](https://github.com/ZJU-FAST-Lab/px4ctrl) — ZJU FAST Lab
