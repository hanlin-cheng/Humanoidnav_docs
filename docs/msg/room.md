# Room

**消息类型**: `map_manager/msg/Room`

## 描述

房间定义，包含边界、高度和语义信息

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | string | 过渡点唯一标识符 |
| name | string | 楼层可读名称 |
| type | string | POI 类型（如 "charging_station", "waiting_point", "door"） |
| boundary | geometry_msgs/Point[] | 多边形边界（顶点列表） |
| floor_height | float64 | 房间地板高度（米） |
| ceiling_height | float64 | 房间天花板高度（米） |
| connected_rooms | string[] | 连接的房间 ID 列表 |
| objects | string[] | 房间内的对象 ID 列表 |

## 使用示例

### 在服务中使用

```bash
# 添加办公室房间
ros2 service call /map_manager/add_room map_manager/srv/AddRoom \
  "{floor_id: 'F1',
    room: {
      id: 'room_101',
      name: 'Office 101',
      type: 'office',
      boundary: [
        {x: 0.0, y: 0.0, z: 0.0},
        {x: 5.0, y: 0.0, z: 0.0},
        {x: 5.0, y: 4.0, z: 0.0},
        {x: 0.0, y: 4.0, z: 0.0}
      ],
      floor_height: 0.0,
      ceiling_height: 2.8,
      connected_rooms: ['room_102', 'hallway_1'],
      objects: ['desk_001', 'chair_001']
    }
  }"
```

### Python 中构造

```python
from map_manager.msg import Room
from geometry_msgs.msg import Point

room = Room()
room.id = 'room_101'
room.name = 'Office 101'
room.type = 'office'
room.boundary = [
    Point(x=0.0, y=0.0, z=0.0),
    Point(x=5.0, y=0.0, z=0.0),
    Point(x=5.0, y=4.0, z=0.0),
    Point(x=0.0, y=4.0, z=0.0)
]
room.floor_height = 0.0
room.ceiling_height = 2.8
room.connected_rooms = ['room_102', 'hallway_1']
room.objects = ['desk_001', 'chair_001']
```

## 相关接口

- [AddRoom](../srv/add_room.md) - 添加房间
- [ListRooms](../srv/list_rooms.md) - 列出房间
- [GetRoom](../srv/get_room.md) - 获取房间
- [UpdateRoom](../srv/update_room.md) - 更新房间
- [RemoveRoom](../srv/remove_room.md) - 删除房间
