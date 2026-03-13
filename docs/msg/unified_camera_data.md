# UnifiedCameraData 消息

> 融合相机数据和机器人位姿的统一消息类型

**消息类型**: `fourier_msgs/msg/UnifiedCameraData`

## 消息定义

```msg
# UnifiedCameraData.msg
# 融合相机数据和机器人位姿的消息类型

std_msgs/Header header
# header.stamp: 使用相机数据的时间戳
# header.frame_id: 点云坐标系（通常是 camera_01_color_optical_frame）

# 相机名称（用于多相机区分）
string camera_name

# RGB 图像数据
sensor_msgs/Image rgb_image

# 点云数据（原始点云）
sensor_msgs/PointCloud2 point_cloud

# 机器人位姿（通过时间戳插值得到）
geometry_msgs/Pose robot_pose
```

## 字段说明

### header (std_msgs/Header)

消息头，包含时间戳和坐标系信息。

- **stamp**: 相机数据采集的时间戳（与 RGB 图像和点云的时间戳一致）
- **frame_id**: 点云的坐标系（从点云 header 中读取，通常是 `camera_01_color_optical_frame`）

### camera_name (string)

相机标识名称，用于区分多个相机。例如：`"camera_01"`, `"camera_02"`。

### rgb_image (sensor_msgs/Image)

彩色图像数据，包含：
- 图像尺寸（width, height）
- 编码格式（通常是 `rgb8` 或 `bgr8`）
- 图像数据

### point_cloud (sensor_msgs/PointCloud2)

深度点云数据，特点：
- **原始点云**：未经过滤处理的完整点云
- **坐标系**：保持在相机坐标系下
- 包含 XYZ 坐标信息

### robot_pose (geometry_msgs/Pose)

机器人位姿，特点：
- 通过时间戳插值得到，与相机数据时间同步
- 包含位置（position）和朝向（orientation）
- 位置：`(x, y, z)` 米
- 朝向：四元数 `(x, y, z, w)`

## 相关话题

- **输入话题**:
  - `/camera/color/image_raw` - RGB 图像源
  - `/camera/depth/points` - 点云数据源
  - `/robot_pose` - 机器人位姿源

- **输出话题**:
  - `/camera/fused_data` - 本消息类型
  - `/camera/filtered_pointcloud` - 过滤后的点云

## 示例代码

详见 [ROS API 文档 - 6.2.1 /camera/fused_data](../ros_api.md#621-camerafused_data-话题)。
