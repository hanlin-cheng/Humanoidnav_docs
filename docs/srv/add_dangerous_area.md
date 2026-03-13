# AddDangerousArea

**服务类型**: `map_manager/srv/AddDangerousArea`

**服务名**: `/map_manager/add_dangerous_area`

## 描述

添加限速区（危险区域）到指定楼层。限速区是一个多边形区域，机器人在此区域内需要降低速度。
与禁区不同，限速区允许机器人通过，但会限制其最大速度。

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（必需） |
| area | DangerousArea | 限速区数据（必须包含 id, boundary, speed_limit） |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| area_id | int32 | 添加的限速区 ID |

## 使用示例

### 命令行调用

```bash
# 添加矩形限速区，限速 0.5 m/s
ros2 service call /map_manager/add_dangerous_area map_manager/srv/AddDangerousArea \
  "{floor_id: 'F1', area: {id: 1, boundary: {points: [{x: 0.0, y: 0.0, z: 0.0}, {x: 3.0, y: 0.0, z: 0.0}, {x: 3.0, y: 3.0, z: 0.0}, {x: 0.0, y: 3.0, z: 0.0}]}, speed_limit: 0.5}}"

# 添加走廊限速区，限速 0.3 m/s
ros2 service call /map_manager/add_dangerous_area map_manager/srv/AddDangerousArea \
  "{floor_id: 'F1', area: {id: 2, boundary: {points: [{x: 10.0, y: 0.0, z: 0.0}, {x: 15.0, y: 0.0, z: 0.0}, {x: 15.0, y: 2.0, z: 0.0}, {x: 10.0, y: 2.0, z: 0.0}]}, speed_limit: 0.3}}"
```

### Python 示例

```python
from map_manager.srv import AddDangerousArea
from map_manager.msg import DangerousArea
from geometry_msgs.msg import Polygon, Point32
import rclpy
from rclpy.node import Node

class DangerousAreaManager(Node):
    def __init__(self):
        super().__init__('dangerous_area_manager')
        self.client = self.create_client(
            AddDangerousArea, '/map_manager/add_dangerous_area')
        self.client.wait_for_service()
        self.next_id = 1

    def add_area(self, floor_id: str, points: list, speed_limit: float,
                 area_id: int = None):
        if area_id is None:
            area_id = self.next_id
            self.next_id += 1

        polygon = Polygon(points=[
            Point32(x=float(p[0]), y=float(p[1]), z=0.0)
            for p in points
        ])

        request = AddDangerousArea.Request()
        request.floor_id = floor_id
        request.area = DangerousArea(
            id=area_id,
            boundary=polygon,
            speed_limit=speed_limit
        )

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()

        if result.success:
            print(f"限速区添加成功: ID={result.area_id}, 限速={speed_limit} m/s")
        else:
            print(f"添加失败: {result.message}")
        return result

    def add_rectangle(self, floor_id: str, x1: float, y1: float,
                      x2: float, y2: float, speed_limit: float,
                      area_id: int = None):
        """添加矩形限速区"""
        points = [(x1, y1), (x2, y1), (x2, y2), (x1, y2)]
        return self.add_area(floor_id, points, speed_limit, area_id)

rclpy.init()
manager = DangerousAreaManager()

# 添加矩形限速区（0.5 m/s）
manager.add_rectangle('F1', 0.0, 0.0, 3.0, 3.0, speed_limit=0.5)

# 添加自定义多边形限速区
manager.add_area('F1',
    [(5.0, 5.0), (8.0, 5.0), (8.0, 8.0), (5.0, 8.0)],
    speed_limit=0.3)

rclpy.shutdown()
```

### C++ 示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_dangerous_area.hpp>
#include <map_manager/msg/dangerous_area.hpp>

class DangerousAreaManager : public rclcpp::Node {
public:
    DangerousAreaManager() : Node("dangerous_area_manager"), next_id_(1) {
        client_ = create_client<map_manager::srv::AddDangerousArea>(
            "/map_manager/add_dangerous_area");
        client_->wait_for_service();
    }

    bool add_rectangle(const std::string& floor_id,
                       double x1, double y1, double x2, double y2,
                       float speed_limit, int area_id = -1) {
        if (area_id < 0) area_id = next_id_++;

        auto request = std::make_shared<map_manager::srv::AddDangerousArea::Request>();
        request->floor_id = floor_id;
        request->area.id = area_id;
        request->area.speed_limit = speed_limit;

        geometry_msgs::msg::Point32 p1, p2, p3, p4;
        p1.x = x1; p1.y = y1; p1.z = 0;
        p2.x = x2; p2.y = y1; p2.z = 0;
        p3.x = x2; p3.y = y2; p3.z = 0;
        p4.x = x1; p4.y = y2; p4.z = 0;

        request->area.boundary.points = {p1, p2, p3, p4};

        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            RCLCPP_INFO(get_logger(), "限速区 %s: ID=%d, 限速=%.2f m/s",
                result->success ? "添加成功" : "添加失败",
                result->area_id, speed_limit);
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddDangerousArea>::SharedPtr client_;
    int next_id_;
};
```

## 限速区与禁区对比

| 特性 | 禁区 (ForbiddenArea) | 限速区 (DangerousArea) |
| ---- | ---- | ---- |
| 机器人可否进入 | 不可以 | 可以 |
| Costmap 效果 | 致命障碍物 (LETHAL) | 高代价（可通行） |
| 速度限制 | N/A | 可配置 |
| 适用场景 | 危险区域、禁止通行 | 人员密集区、狭窄通道 |

## 推荐速度限制值

| 场景 | 推荐限速 (m/s) | 说明 |
| ---- | ---- | ---- |
| 人员密集区 | 0.3 | 安全优先 |
| 狭窄走廊 | 0.5 | 避免碰撞 |
| 门口区域 | 0.4 | 开门安全 |
| 坡道区域 | 0.3 | 防止失控 |
| 玻璃门前 | 0.2 | 传感器盲区 |

## 相关接口

- [ListDangerousAreas](list_dangerous_areas.md) - 列出限速区
- [GetDangerousArea](get_dangerous_area.md) - 获取限速区
- [UpdateDangerousArea](update_dangerous_area.md) - 更新限速区
- [RemoveDangerousArea](remove_dangerous_area.md) - 删除限速区
- [DangerousArea](../msg/dangerous_area.md) - 限速区消息定义
- [AddForbiddenArea](add_forbidden_area.md) - 添加禁区
