# VirtualWall

**消息类型**: `map_manager/msg/VirtualWall`

## 描述

虚拟墙定义，用于在地图上添加不可通行的虚拟障碍

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | int32 | 过渡点唯一标识符 |
| floor_id | string | 楼层唯一标识符（如 "1F", "2F", "B1"） |
| start | geometry_msgs/Point | 虚拟墙起点坐标（x, y, z） |
| end | geometry_msgs/Point | 虚拟墙终点坐标（x, y, z） |

## 使用示例

### 在服务中使用

```bash
# 添加虚拟墙
ros2 service call /map_manager/add_virtual_wall map_manager/srv/AddVirtualWall \
  "{floor_id: 'F1', 
    wall: {
      id: 1, 
      floor_id: 'F1',
      start: {x: 0.0, y: 0.0, z: 0.0}, 
      end: {x: 5.0, y: 0.0, z: 0.0}
    }
  }"
```

### Python 中构造

```python
from map_manager.msg import VirtualWall
from geometry_msgs.msg import Point

wall = VirtualWall()
wall.id = 1
wall.floor_id = 'F1'
wall.start = Point(x=0.0, y=0.0, z=0.0)
wall.end = Point(x=5.0, y=0.0, z=0.0)
```

## 相关接口

- [AddVirtualWall](../srv/add_virtual_wall.md) - 添加虚拟墙
- [ListVirtualWalls](../srv/list_virtual_walls.md) - 列出虚拟墙
- [GetVirtualWall](../srv/get_virtual_wall.md) - 获取虚拟墙
- [UpdateVirtualWall](../srv/update_virtual_wall.md) - 更新虚拟墙
- [RemoveVirtualWall](../srv/remove_virtual_wall.md) - 删除虚拟墙
