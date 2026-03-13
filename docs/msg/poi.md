# POI

**消息类型**: `map_manager/msg/POI`

## 描述

兴趣点（Point of Interest）定义，如充电站、等待点等

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | string | 过渡点唯一标识符 |
| name | string | 楼层可读名称 |
| type | string | POI 类型（如 "charging_station", "waiting_point", "door"） |
| floor_id | string | 楼层唯一标识符（如 "1F", "2F", "B1"） |
| pose | geometry_msgs/Pose2D | 2D 位姿（x, y, theta） |
| height | float64 | 距楼层参考高度的偏移（米） |
| properties | string | 自定义属性（JSON 字符串） |

## 使用示例

### 在服务中使用

```bash
# 添加充电站 POI
ros2 service call /map_manager/add_poi map_manager/srv/AddPOI \
  "{poi: {
    id: 'charging_1',
    name: 'Charging Station 1',
    type: 'charging_station',
    floor_id: 'F1',
    pose: {x: 5.0, y: 3.0, theta: 1.57},
    height: 0.0,
    properties: '{\"voltage\": 24, \"current\": 10}'
  }}"
```

### Python 中构造

```python
from map_manager.msg import POI
from geometry_msgs.msg import Pose2D
import json

poi = POI()
poi.id = 'charging_1'
poi.name = 'Charging Station 1'
poi.type = 'charging_station'
poi.floor_id = 'F1'
poi.pose = Pose2D(x=5.0, y=3.0, theta=1.57)
poi.height = 0.0
poi.properties = json.dumps({'voltage': 24, 'current': 10})
```

## 相关接口

- [AddPOI](../srv/add_poi.md) - 添加 POI
- [ListPOIs](../srv/list_pois.md) - 列出 POI
- [GetPOI](../srv/get_poi.md) - 获取 POI
- [UpdatePOI](../srv/update_poi.md) - 更新 POI
- [RemovePOI](../srv/remove_poi.md) - 删除 POI
