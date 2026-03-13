# ForbiddenArea

**消息类型**: `map_manager/msg/ForbiddenArea`

## 描述

禁止区域定义，多边形区域，机器人不得进入

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | int32 | 过渡点唯一标识符 |
| boundary | geometry_msgs/Polygon | 多边形边界（顶点列表） |

## 使用示例

### 在服务中使用

```bash
# 添加禁区（正方形）
ros2 service call /map_manager/add_forbidden_area map_manager/srv/AddForbiddenArea \
  "{floor_id: 'F1',
    area: {
      id: 1,
      boundary: {
        points: [
          {x: 0.0, y: 0.0, z: 0.0},
          {x: 2.0, y: 0.0, z: 0.0},
          {x: 2.0, y: 2.0, z: 0.0},
          {x: 0.0, y: 2.0, z: 0.0}
        ]
      }
    }
  }"
```

### Python 中构造

```python
from map_manager.msg import ForbiddenArea
from geometry_msgs.msg import Polygon, Point32

area = ForbiddenArea()
area.id = 1
area.boundary = Polygon()
area.boundary.points = [
    Point32(x=0.0, y=0.0, z=0.0),
    Point32(x=2.0, y=0.0, z=0.0),
    Point32(x=2.0, y=2.0, z=0.0),
    Point32(x=0.0, y=2.0, z=0.0)
]
```

## 相关接口

- [AddForbiddenArea](../srv/add_forbidden_area.md) - 添加禁区
- [ListForbiddenAreas](../srv/list_forbidden_areas.md) - 列出禁区
- [GetForbiddenArea](../srv/get_forbidden_area.md) - 获取禁区
- [UpdateForbiddenArea](../srv/update_forbidden_area.md) - 更新禁区
- [RemoveForbiddenArea](../srv/remove_forbidden_area.md) - 删除禁区
