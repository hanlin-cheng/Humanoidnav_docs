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

| 事件类型 | 值 | 说明 | 发布组件 | 触发频率 |
| -------- | -- | ---- | -------- | -------- |
| SystemEventMapLoopClosure | map_loop_closure | 地图闭环检测 | slam | 事件驱动 |
| SystemEventObstacleBlocked | obstacle_blocked | 路径被障碍物阻挡 | collision_monitor | 实时 |
| SystemEventNearObstacle | near_obstacle | 检测到近处障碍物 | collision_monitor | 每1秒 |
| SystemEventNearObstacleWhenStationary | near_obstacle_when_stationary | 机器人静止时检测到近处障碍物 | collision_monitor | 每1秒 |
| SystemEventOutOfMap | out_of_map | 机器人走出地图边界或进入未知区域 | monitor | 实时 |
| SystemEventEnterForbiddenArea | enter_forbidden_area | 机器人进入禁区 | monitor | 每5秒 |
| SystemEventEnterDangerousArea | enter_dangerous_area | 机器人进入危险区域（限速区） | monitor | 每5秒 |

> **注意**:
> - `enter_forbidden_area` 和 `enter_dangerous_area` 在机器人位于对应区域内时，每5秒触发一次
> - 机器人离开区域时会在日志中记录，但不会触发新事件
> - 事件消息中包含区域ID、速度限制（危险区）和机器人位置信息


### 相关消息

- [EventsInfo](events_info.md) - 聚合事件信息
