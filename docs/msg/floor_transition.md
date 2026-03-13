# FloorTransition

**消息类型**: `map_manager/msg/FloorTransition`

## 描述

楼层过渡点定义（电梯、楼梯等），用于跨楼层导航

## 字段定义

### 常量

| 名称 | 类型 | 值 | 说明 |
| ---- | ---- | -- | ---- |
| TYPE_ELEVATOR | uint8 | 0 | 电梯 |
| TYPE_STAIRS | uint8 | 1 | 楼梯 |
| TYPE_RAMP | uint8 | 2 | 坡道 |
| TYPE_ESCALATOR | uint8 | 3 | 扶梯 |

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | string | 过渡点唯一标识符 |
| name | string | 楼层可读名称 |
| transition_type | uint8 | 过渡类型：0=ELEVATOR（电梯）, 1=STAIRS（楼梯）, 2=RAMP（坡道）, 3=ESCALATOR（扶梯） |
| from_floor_id | string | 起始楼层 ID |
| from_pose | geometry_msgs/Pose2D | 起始楼层位姿（x, y, theta） |
| to_floor_id | string | 目标楼层 ID |
| to_pose | geometry_msgs/Pose2D | 目标楼层位姿（x, y, theta） |
| bidirectional | bool | 是否双向可通行 |
| cost | float64 | 通过代价（用于路径规划） |
| available | bool | 当前是否可用 |

## 使用示例

*请参考 map_manager API 文档获取使用示例*
## 相关接口

*楼层过渡点通过 CompositeMapInfo 消息访问*
