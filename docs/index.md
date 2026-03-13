# HumanoidNav ROS2 导航系统

HumanoidNav 是一套完整的ROS2机器人导航系统，集成了建图、定位、路径规划和导航控制功能。

## 系统概述

本系统基于ROS2框架开发，主要包含以下核心功能模块：

| 模块 | 说明 | 主要功能 |
| ---- | ---- | ---- |
| SLAM | 激光惯性建图与定位系统 | 3D点云建图、6DOF位姿估计、IMU融合、建图/定位模式切换 |
| Nav2 | 导航规划与控制 | 全局路径规划(SmacPlanner2D)、局部轨迹控制(MPPI)、行为树导航、速度平滑 |
| Octomap | 地图构建与管理 | 3D点云转2D栅格地图、地图保存/加载、地图优化降噪 |
| Perception | 感知与预处理 | 点云地面分割、障碍物检测、3D转2D激光扫描 |
| fourier_msgs | 机器人监控与状态接口 | 导航动作状态监控、动作取消/查询服务、状态发布 |

## 快速开始

### 环境要求

- Ubuntu 22.04
- ROS2 Humble

## 系统架构

```
[传感器输入]
  ├─ IMU (/imu)
  └─ LiDAR (/lidar_points)
        ↓
    [3D激光建图/定位]
        ↓
    [Octomap 2D地图]
        ├─ /map (2D占用栅格)
        └─ /optimize_map (优化后的2D地图)
        ↓
    [Nav2 导航系统]
        ├─ 全局规划 (Planner)
        ├─ 局部控制 (Controller)
        └─ 行为树 (BT Navigator)
        ↓
    [速度输出]
        └─ /cmd_vel_smoothed
```

## 模式说明

系统支持两种运行模式：

- **建图模式 (Mapping)**: 实时构建环境地图
- **定位模式 (Localization)**: 在已有地图上进行定位导航

## 文档导航

- [ROS2 接口文档](ros_api.md) - 完整的接口列表和说明
