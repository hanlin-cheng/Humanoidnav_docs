# DangerousArea

**消息类型**: `map_manager/msg/DangerousArea`

## 描述

危险/限速区域定义，多边形区域，限制机器人速度

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | int32 | 过渡点唯一标识符 |
| boundary | geometry_msgs/Polygon | 多边形边界（顶点列表） |
| speed_limit | float32 | 速度限制（米/秒） |

## 使用示例

### 在服务中使用

```bash
# 添加限速区（正方形，限速 0.3m/s）
ros2 service call /map_manager/add_dangerous_area map_manager/srv/AddDangerousArea \
  "{floor_id: 'F1',
    area: {
      id: 1,
      boundary: {
        points: [
          {x: 5.0, y: 5.0, z: 0.0},
          {x: 8.0, y: 5.0, z: 0.0},
          {x: 8.0, y: 8.0, z: 0.0},
          {x: 5.0, y: 8.0, z: 0.0}
        ]
      },
      speed_limit: 0.3
    }
  }"
```

### Python 中构造

```python
from map_manager.msg import DangerousArea
from geometry_msgs.msg import Polygon, Point32

area = DangerousArea()
area.id = 1
area.boundary = Polygon()
area.boundary.points = [
    Point32(x=5.0, y=5.0, z=0.0),
    Point32(x=8.0, y=5.0, z=0.0),
    Point32(x=8.0, y=8.0, z=0.0),
    Point32(x=5.0, y=8.0, z=0.0)
]
area.speed_limit = 0.3  # m/s
```

## 相关接口

- [AddDangerousArea](../srv/add_dangerous_area.md) - 添加限速区
- [ListDangerousAreas](../srv/list_dangerous_areas.md) - 列出限速区
- [GetDangerousArea](../srv/get_dangerous_area.md) - 获取限速区
- [UpdateDangerousArea](../srv/update_dangerous_area.md) - 更新限速区
- [RemoveDangerousArea](../srv/remove_dangerous_area.md) - 删除限速区
