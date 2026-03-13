# SemanticObject

**消息类型**: `map_manager/msg/SemanticObject`

## 描述

语义对象定义，如桌子、椅子、门等带语义标签的对象

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | string | 过渡点唯一标识符 |
| name | string | 楼层可读名称 |
| type | string | POI 类型（如 "charging_station", "waiting_point", "door"） |
| category | string | 对象类别（如 "furniture", "equipment"） |
| pose | geometry_msgs/Pose | 2D 位姿（x, y, theta） |
| dimensions | geometry_msgs/Vector3 | 对象尺寸（长×宽×高，米） |
| is_static | bool | 是否静态对象（不可移动） |

## 使用示例

### 在服务中使用

```bash
# 添加办公桌对象
ros2 service call /map_manager/add_semantic_object map_manager/srv/AddSemanticObject \
  "{floor_id: 'F1',
    object: {
      id: 'desk_001',
      name: 'Office Desk',
      type: 'desk',
      category: 'furniture',
      pose: {
        position: {x: 2.0, y: 3.0, z: 0.0},
        orientation: {x: 0.0, y: 0.0, z: 0.0, w: 1.0}
      },
      dimensions: {x: 1.4, y: 0.7, z: 0.75},
      is_static: true
    }
  }"
```

### Python 中构造

```python
from map_manager.msg import SemanticObject
from geometry_msgs.msg import Pose, Vector3

obj = SemanticObject()
obj.id = 'desk_001'
obj.name = 'Office Desk'
obj.type = 'desk'
obj.category = 'furniture'
obj.pose = Pose()
obj.pose.position.x = 2.0
obj.pose.position.y = 3.0
obj.pose.position.z = 0.0
obj.pose.orientation.w = 1.0
obj.dimensions = Vector3(x=1.4, y=0.7, z=0.75)
obj.is_static = True
```

## 相关接口

- [AddSemanticObject](../srv/add_semantic_object.md) - 添加语义对象
- [ListSemanticObjects](../srv/list_semantic_objects.md) - 列出语义对象
- [GetSemanticObject](../srv/get_semantic_object.md) - 获取语义对象
- [UpdateSemanticObject](../srv/update_semantic_object.md) - 更新语义对象
- [RemoveSemanticObject](../srv/remove_semantic_object.md) - 删除语义对象
