# AddPOI

**服务类型**: `map_manager/srv/AddPOI`

**服务名**: `/map_manager/add_poi`

## 描述

添加兴趣点（POI）到指定楼层。POI 可以是充电桩、门、导航点等。

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| poi | POI | POI 数据（必须包含 id, floor_id） |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
# 添加充电桩 POI
ros2 service call /map_manager/add_poi map_manager/srv/AddPOI \
  "{poi: {id: 'charger_01', name: 'Main Charger', type: 'charging_station', floor_id: 'F1', pose: {x: 5.0, y: 3.0, theta: 1.57}}}"

# 添加门 POI
ros2 service call /map_manager/add_poi map_manager/srv/AddPOI \
  "{poi: {id: 'door_101', name: 'Room 101 Door', type: 'door', floor_id: 'F1', pose: {x: 10.0, y: 5.0, theta: 0.0}}}"

# 添加带自定义属性的 POI
ros2 service call /map_manager/add_poi map_manager/srv/AddPOI \
  "{poi: {id: 'station_01', name: 'Work Station 1', type: 'workstation', floor_id: 'F1', pose: {x: 8.0, y: 6.0, theta: 3.14}, properties: '{\"capacity\": 2, \"equipment\": [\"computer\", \"printer\"]}'}}"
```

### Python 示例

```python
from map_manager.srv import AddPOI
from map_manager.msg import POI
from geometry_msgs.msg import Pose2D
import rclpy
from rclpy.node import Node
import json

class POIManager(Node):
    def __init__(self):
        super().__init__('poi_manager')
        self.client = self.create_client(AddPOI, '/map_manager/add_poi')
        self.client.wait_for_service()

    def add_poi(self, poi_id: str, name: str, poi_type: str, floor_id: str,
                x: float, y: float, theta: float, properties: dict = None):
        request = AddPOI.Request()
        request.poi = POI(
            id=poi_id,
            name=name,
            type=poi_type,
            floor_id=floor_id,
            pose=Pose2D(x=x, y=y, theta=theta),
            properties=json.dumps(properties) if properties else ''
        )

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()

        if result.success:
            print(f"POI 添加成功: {poi_id}")
        else:
            print(f"添加失败: {result.message}")
        return result

rclpy.init()
manager = POIManager()

# 添加充电桩
manager.add_poi(
    poi_id='charger_01',
    name='Main Charger',
    poi_type='charging_station',
    floor_id='F1',
    x=5.0, y=3.0, theta=1.57,
    properties={'voltage': 24, 'max_current': 10}
)

# 添加导航点
manager.add_poi(
    poi_id='waypoint_01',
    name='Patrol Point 1',
    poi_type='waypoint',
    floor_id='F1',
    x=10.0, y=8.0, theta=0.0
)

rclpy.shutdown()
```

### C++ 示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_poi.hpp>
#include <map_manager/msg/poi.hpp>

class POIManager : public rclcpp::Node {
public:
    POIManager() : Node("poi_manager") {
        client_ = create_client<map_manager::srv::AddPOI>("/map_manager/add_poi");
        client_->wait_for_service();
    }

    bool add_poi(const std::string& id, const std::string& name,
                 const std::string& type, const std::string& floor_id,
                 double x, double y, double theta) {
        auto request = std::make_shared<map_manager::srv::AddPOI::Request>();
        request->poi.id = id;
        request->poi.name = name;
        request->poi.type = type;
        request->poi.floor_id = floor_id;
        request->poi.pose.x = x;
        request->poi.pose.y = y;
        request->poi.pose.theta = theta;

        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            return future.get()->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddPOI>::SharedPtr client_;
};
```

## 常用 POI 类型

| 类型 | 描述 | 典型用途 |
| ---- | ---- | ---- |
| `charging_station` | 充电桩 | 自动回充 |
| `door` | 门 | 门控导航 |
| `elevator` | 电梯口 | 多楼层导航 |
| `waypoint` | 路径点 | 巡检路线 |
| `workstation` | 工作站 | 物料配送 |
| `rest_area` | 休息区 | 等待位置 |

## 相关接口

- [ListPOIs](list_pois.md) - 列出 POI
- [GetPOI](get_poi.md) - 获取 POI
- [UpdatePOI](update_poi.md) - 更新 POI
- [RemovePOI](remove_poi.md) - 删除 POI
- [POI](../msg/poi.md) - POI 消息定义
