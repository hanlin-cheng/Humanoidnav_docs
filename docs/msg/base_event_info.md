## fourier_msgs/msg/BaseEventInfo

### 文件

fourier_msgs/msg/BaseEventInfo.msg

### 描述

基础事件信息消息，用于描述单个事件的详细信息。

### 定义

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| event_type | string | 事件类型标识符 |
| message | string | 事件描述信息 |
| source | string | 事件来源组件 |
| timestamp | int64 | 时间戳 |

### 已定义的事件类型

| 事件类型 | 值 | 说明 | 发布组件 |
| -------- | -- | ---- | -------- |
| SystemEventMapLoopClosure | map_loop_closure | 地图闭环检测 | slam |
| SystemEventObstacleBlocked | obstacle_blocked | 路径被障碍物阻挡 | is_path_valid |
| SystemEventNearObstacle | near_obstacle | 检测到近处障碍物 | collision_monitor |
| SystemEventNearObstacleWhenStationary | near_obstacle_when_stationary | 机器人静止时检测到近处障碍物 | collision_monitor |
| SystemEventOutOfMap | out_of_map | 机器人走出地图边界或进入未知区域 | monitor |


### 相关消息

- [EventsInfo](events_info.md) - 聚合事件信息
