# AddVirtualWall

**服务类型**: `map_manager/srv/AddVirtualWall`

**服务名**: `/map_manager/add_virtual_wall`

## 描述

添加虚拟墙到指定楼层。虚拟墙是一条线段，机器人无法穿越。
虚拟墙会被添加到 costmap 中作为致命障碍物。

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（如果为空，使用 wall.floor_id） |
| wall | VirtualWall | 虚拟墙数据（必须包含 id） |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| wall_id | int32 | 添加的虚拟墙 ID |

## 使用示例

### 命令行调用

```bash
# 添加水平虚拟墙
ros2 service call /map_manager/add_virtual_wall map_manager/srv/AddVirtualWall \
  "{floor_id: 'F1', wall: {id: 1, start: {x: 0.0, y: 5.0, z: 0.0}, end: {x: 10.0, y: 5.0, z: 0.0}}}"

# 添加垂直虚拟墙
ros2 service call /map_manager/add_virtual_wall map_manager/srv/AddVirtualWall \
  "{floor_id: 'F1', wall: {id: 2, start: {x: 5.0, y: 0.0, z: 0.0}, end: {x: 5.0, y: 8.0, z: 0.0}}}"

# 添加斜向虚拟墙
ros2 service call /map_manager/add_virtual_wall map_manager/srv/AddVirtualWall \
  "{floor_id: 'F1', wall: {id: 3, start: {x: 0.0, y: 0.0, z: 0.0}, end: {x: 5.0, y: 5.0, z: 0.0}}}"
```

### Python 示例

```python
from map_manager.srv import AddVirtualWall
from map_manager.msg import VirtualWall
from geometry_msgs.msg import Point
import rclpy
from rclpy.node import Node

class VirtualWallManager(Node):
    def __init__(self):
        super().__init__('virtual_wall_manager')
        self.client = self.create_client(AddVirtualWall, '/map_manager/add_virtual_wall')
        self.client.wait_for_service()
        self.next_id = 1

    def add_wall(self, floor_id: str, start: tuple, end: tuple, wall_id: int = None):
        if wall_id is None:
            wall_id = self.next_id
            self.next_id += 1

        request = AddVirtualWall.Request()
        request.floor_id = floor_id
        request.wall = VirtualWall(
            id=wall_id,
            floor_id=floor_id,
            start=Point(x=start[0], y=start[1], z=start[2] if len(start) > 2 else 0.0),
            end=Point(x=end[0], y=end[1], z=end[2] if len(end) > 2 else 0.0)
        )

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()

        if result.success:
            print(f"虚拟墙添加成功: ID={result.wall_id}")
        else:
            print(f"添加失败: {result.message}")
        return result

rclpy.init()
manager = VirtualWallManager()

# 添加围栏（4条虚拟墙围成矩形）
manager.add_wall('F1', (0, 0), (10, 0))   # 底边
manager.add_wall('F1', (10, 0), (10, 8))  # 右边
manager.add_wall('F1', (10, 8), (0, 8))   # 顶边
manager.add_wall('F1', (0, 8), (0, 0))    # 左边

rclpy.shutdown()
```

### C++ 示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_virtual_wall.hpp>
#include <map_manager/msg/virtual_wall.hpp>

class VirtualWallManager : public rclcpp::Node {
public:
    VirtualWallManager() : Node("virtual_wall_manager"), next_id_(1) {
        client_ = create_client<map_manager::srv::AddVirtualWall>(
            "/map_manager/add_virtual_wall");
        client_->wait_for_service();
    }

    bool add_wall(const std::string& floor_id,
                  double x1, double y1, double x2, double y2,
                  int wall_id = -1) {
        if (wall_id < 0) wall_id = next_id_++;

        auto request = std::make_shared<map_manager::srv::AddVirtualWall::Request>();
        request->floor_id = floor_id;
        request->wall.id = wall_id;
        request->wall.floor_id = floor_id;
        request->wall.start.x = x1;
        request->wall.start.y = y1;
        request->wall.start.z = 0;
        request->wall.end.x = x2;
        request->wall.end.y = y2;
        request->wall.end.z = 0;

        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            RCLCPP_INFO(get_logger(), "虚拟墙 %s: ID=%d",
                result->success ? "添加成功" : "添加失败", result->wall_id);
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddVirtualWall>::SharedPtr client_;
    int next_id_;
};
```

## Costmap 效果

虚拟墙在 costmap 中被标记为致命障碍物：

```
      虚拟墙线段
    ━━━━━━━━━━━
    ▓▓▓▓▓▓▓▓▓▓▓   ← 致命障碍物 (254)
   ░░░░░░░░░░░░░  ← 膨胀区域
```

## 相关接口

- [ListVirtualWalls](list_virtual_walls.md) - 列出虚拟墙
- [GetVirtualWall](get_virtual_wall.md) - 获取虚拟墙
- [UpdateVirtualWall](update_virtual_wall.md) - 更新虚拟墙
- [RemoveVirtualWall](remove_virtual_wall.md) - 删除虚拟墙
- [VirtualWall](../msg/virtual_wall.md) - 虚拟墙消息定义
- [AddForbiddenArea](add_forbidden_area.md) - 添加禁区（多边形，自动生成虚拟墙）
