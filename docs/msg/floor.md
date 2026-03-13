# Floor

**消息类型**: `map_manager/msg/Floor`

## 描述

楼层信息定义，包含楼层的基本属性和高度范围

## 字段定义

### 常量

| 名称 | 类型 | 值 | 说明 |
| ---- | ---- | -- | ---- |
| STATUS_UNKNOWN | uint8 | 0 | 未知状态 |
| STATUS_ACTIVE | uint8 | 1 | 活跃状态 |
| STATUS_INACTIVE | uint8 | 2 | 非活跃状态 |

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| floor_id | string | 楼层唯一标识符（如 "1F", "2F", "B1"） |
| name | string | 楼层可读名称 |
| level | int32 | 楼层层数（1=1楼, 0=地面层, -1=地下1层） |
| min_height | float64 | 楼层最小高度（米） |
| max_height | float64 | 楼层最大高度（米） |
| reference_height | float64 | 楼层参考高度（米），用于高度对齐 |
| origin_offset | geometry_msgs/Pose | 楼层坐标原点相对于全局原点的偏移 |
| map_path | string | 地图文件路径 |
| status | uint8 | 楼层状态：0=UNKNOWN, 1=ACTIVE（活跃）, 2=INACTIVE（非活跃） |

## 使用示例

### 在服务中使用

```bash
# 添加楼层时使用 Floor 消息
ros2 service call /map_manager/add_floor map_manager/srv/AddFloor \
  "{floor: {
    floor_id: '1F',
    name: 'First Floor',
    level: 1,
    min_height: 0.0,
    max_height: 3.0,
    reference_height: 0.0,
    status: 1
  }}"
```

### Python 中构造

```python
from map_manager.msg import Floor
from geometry_msgs.msg import Pose

floor = Floor()
floor.floor_id = '1F'
floor.name = 'First Floor'
floor.level = 1
floor.min_height = 0.0
floor.max_height = 3.0
floor.reference_height = 0.0
floor.status = Floor.STATUS_ACTIVE
floor.origin_offset = Pose()  # 默认原点
```

## 相关接口

- [AddFloor](../srv/add_floor.md) - 添加楼层
- [ListFloors](../srv/list_floors.md) - 列出楼层
- [SwitchFloor](../srv/switch_floor.md) - 切换楼层
- [UpdateFloor](../srv/update_floor.md) - 更新楼层
- [RemoveFloor](../srv/remove_floor.md) - 删除楼层
