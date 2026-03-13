# AddForbiddenArea

**服务类型**: `map_manager/srv/AddForbiddenArea`

**服务名**: `/map_manager/add_forbidden_area`

## 描述

添加禁区（keepout zone）到指定楼层。禁区是一个多边形区域，机器人不可进入。
添加禁区时，系统会自动为多边形的每条边创建虚拟墙。

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（必需） |
| area | ForbiddenArea | 禁区数据（必须包含 id 和 boundary） |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| area_id | int32 | 添加的禁区 ID |
| wall_ids | int32[] | 自动生成的虚拟墙 ID 列表 |

## 使用示例

### 命令行调用

```bash
# 添加矩形禁区
ros2 service call /map_manager/add_forbidden_area map_manager/srv/AddForbiddenArea \
  "{floor_id: 'F1', area: {id: 1, boundary: {points: [{x: 0.0, y: 0.0, z: 0.0}, {x: 2.0, y: 0.0, z: 0.0}, {x: 2.0, y: 2.0, z: 0.0}, {x: 0.0, y: 2.0, z: 0.0}]}}}"

# 添加三角形禁区
ros2 service call /map_manager/add_forbidden_area map_manager/srv/AddForbiddenArea \
  "{floor_id: 'F1', area: {id: 2, boundary: {points: [{x: 5.0, y: 5.0, z: 0.0}, {x: 7.0, y: 5.0, z: 0.0}, {x: 6.0, y: 7.0, z: 0.0}]}}}"
```

### Python 示例

```python
from map_manager.srv import AddForbiddenArea
from map_manager.msg import ForbiddenArea
from geometry_msgs.msg import Polygon, Point32
import rclpy
from rclpy.node import Node
import math

class ForbiddenAreaManager(Node):
    def __init__(self):
        super().__init__('forbidden_area_manager')
        self.client = self.create_client(
            AddForbiddenArea, '/map_manager/add_forbidden_area')
        self.client.wait_for_service()
        self.next_id = 1

    def add_area(self, floor_id: str, points: list, area_id: int = None):
        if area_id is None:
            area_id = self.next_id
            self.next_id += 1

        polygon = Polygon(points=[
            Point32(x=float(p[0]), y=float(p[1]), z=float(p[2]) if len(p) > 2 else 0.0)
            for p in points
        ])

        request = AddForbiddenArea.Request()
        request.floor_id = floor_id
        request.area = ForbiddenArea(id=area_id, boundary=polygon)

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()

        if result.success:
            print(f"禁区添加成功: ID={result.area_id}")
            print(f"生成的虚拟墙: {list(result.wall_ids)}")
        else:
            print(f"添加失败: {result.message}")
        return result

    def add_rectangle(self, floor_id: str, x1: float, y1: float,
                      x2: float, y2: float, area_id: int = None):
        """添加矩形禁区"""
        points = [(x1, y1), (x2, y1), (x2, y2), (x1, y2)]
        return self.add_area(floor_id, points, area_id)

    def add_circle_approx(self, floor_id: str, cx: float, cy: float,
                          radius: float, segments: int = 16, area_id: int = None):
        """添加近似圆形禁区"""
        points = []
        for i in range(segments):
            angle = 2 * math.pi * i / segments
            x = cx + radius * math.cos(angle)
            y = cy + radius * math.sin(angle)
            points.append((x, y))
        return self.add_area(floor_id, points, area_id)

rclpy.init()
manager = ForbiddenAreaManager()

# 添加矩形禁区
manager.add_rectangle('F1', 0.0, 0.0, 2.0, 2.0)

# 添加自定义多边形禁区
manager.add_area('F1', [(5.0, 5.0), (7.0, 5.0), (8.0, 7.0), (6.0, 8.0), (4.0, 7.0)])

# 添加近似圆形禁区
manager.add_circle_approx('F1', 10.0, 10.0, 1.5, segments=12)

rclpy.shutdown()
```

### C++ 示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_forbidden_area.hpp>
#include <map_manager/msg/forbidden_area.hpp>
#include <geometry_msgs/msg/polygon.hpp>

class ForbiddenAreaManager : public rclcpp::Node {
public:
    ForbiddenAreaManager() : Node("forbidden_area_manager"), next_id_(1) {
        client_ = create_client<map_manager::srv::AddForbiddenArea>(
            "/map_manager/add_forbidden_area");
        client_->wait_for_service();
    }

    bool add_rectangle(const std::string& floor_id,
                       double x1, double y1, double x2, double y2,
                       int area_id = -1) {
        if (area_id < 0) area_id = next_id_++;

        auto request = std::make_shared<map_manager::srv::AddForbiddenArea::Request>();
        request->floor_id = floor_id;
        request->area.id = area_id;

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
            RCLCPP_INFO(get_logger(), "禁区 %s: ID=%d, 生成 %zu 条虚拟墙",
                result->success ? "添加成功" : "添加失败",
                result->area_id, result->wall_ids.size());
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddForbiddenArea>::SharedPtr client_;
    int next_id_;
};
```

## 禁区与虚拟墙的自动转换

```
禁区 (4 个顶点)              自动生成 4 条虚拟墙
    ┌─────┐              wall_1: P0 → P1
    │     │      →       wall_2: P1 → P2
    │     │              wall_3: P2 → P3
    └─────┘              wall_4: P3 → P0
```

## Costmap 效果

禁区在 costmap 中的效果：
- **边界**: 致命障碍物 (LETHAL_OBSTACLE)
- **内部**: 填充为致命障碍物

## 注意事项

1. **顶点顺序**: 顶点应按顺时针或逆时针顺序排列
2. **最少顶点**: 至少需要 3 个顶点（三角形）
3. **不支持自交**: 多边形边不能自相交
4. **实时生效**: 添加后立即更新 costmap

## 相关接口

- [ListForbiddenAreas](list_forbidden_areas.md) - 列出禁区
- [GetForbiddenArea](get_forbidden_area.md) - 获取禁区
- [UpdateForbiddenArea](update_forbidden_area.md) - 更新禁区
- [RemoveForbiddenArea](remove_forbidden_area.md) - 删除禁区
- [ForbiddenArea](../msg/forbidden_area.md) - 禁区消息定义
- [AddDangerousArea](add_dangerous_area.md) - 添加限速区
