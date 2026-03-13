# SwitchFloor

**服务类型**: `map_manager/srv/SwitchFloor`

**服务名**: `/map_manager/switch_floor`

## 描述

切换到指定楼层。支持直接切换和通过过渡点（电梯、楼梯等）切换两种方式。
切换楼层时会自动通知 Lightning-LM 加载对应楼层的定位地图。

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 目标楼层 ID |
| initial_pose | geometry_msgs/Pose2D | 切换后的初始位姿 (x, y, theta) |
| use_transition | bool | 是否使用过渡点切换 |
| transition_id | string | 过渡点 ID（use_transition=true 时必填） |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| current_floor | Floor | 切换后的当前楼层信息 |

## 使用示例

### 命令行调用

```bash
# 直接切换楼层
ros2 service call /map_manager/switch_floor map_manager/srv/SwitchFloor \
  "{floor_id: 'F2', initial_pose: {x: 0.0, y: 0.0, theta: 0.0}, use_transition: false}"

# 使用电梯过渡点切换
ros2 service call /map_manager/switch_floor map_manager/srv/SwitchFloor \
  "{floor_id: 'F2', use_transition: true, transition_id: 'elevator_1'}"
```

### Python 示例

```python
from map_manager.srv import SwitchFloor
from geometry_msgs.msg import Pose2D
import rclpy
from rclpy.node import Node

class FloorSwitcher(Node):
    def __init__(self):
        super().__init__('floor_switcher')
        self.client = self.create_client(SwitchFloor, '/map_manager/switch_floor')
        self.client.wait_for_service()

    def switch_direct(self, floor_id: str, x: float = 0.0, y: float = 0.0, theta: float = 0.0):
        """直接切换楼层"""
        request = SwitchFloor.Request()
        request.floor_id = floor_id
        request.initial_pose = Pose2D(x=x, y=y, theta=theta)
        request.use_transition = False

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()

        if result.success:
            print(f"切换成功: {result.current_floor.name}")
        else:
            print(f"切换失败: {result.message}")
        return result

    def switch_via_transition(self, floor_id: str, transition_id: str):
        """通过过渡点切换楼层"""
        request = SwitchFloor.Request()
        request.floor_id = floor_id
        request.use_transition = True
        request.transition_id = transition_id

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

rclpy.init()
switcher = FloorSwitcher()

# 直接切换到 F2
switcher.switch_direct('F2', x=5.0, y=3.0, theta=1.57)

# 使用电梯切换
switcher.switch_via_transition('F3', 'elevator_main')

rclpy.shutdown()
```

### C++ 示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/switch_floor.hpp>
#include <geometry_msgs/msg/pose2_d.hpp>

class FloorSwitcher : public rclcpp::Node {
public:
    FloorSwitcher() : Node("floor_switcher") {
        client_ = create_client<map_manager::srv::SwitchFloor>(
            "/map_manager/switch_floor");
        client_->wait_for_service();
    }

    bool switch_direct(const std::string& floor_id,
                       double x = 0.0, double y = 0.0, double theta = 0.0) {
        auto request = std::make_shared<map_manager::srv::SwitchFloor::Request>();
        request->floor_id = floor_id;
        request->initial_pose.x = x;
        request->initial_pose.y = y;
        request->initial_pose.theta = theta;
        request->use_transition = false;

        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            RCLCPP_INFO(get_logger(), "切换%s: %s",
                result->success ? "成功" : "失败", result->message.c_str());
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::SwitchFloor>::SharedPtr client_;
};
```

## 楼层切换流程

1. **验证目标楼层存在**
2. **如果使用过渡点**：验证过渡点有效且可用
3. **通知 Lightning-LM** 加载目标楼层的定位地图
4. **更新当前楼层状态**
5. **发布楼层切换事件**

## 注意事项

- 切换楼层前确保目标楼层已加载
- 使用过渡点切换时，机器人应位于过渡点附近
- 切换成功后需要重新定位或使用过渡点提供的目标位姿

## 相关接口

- [ListFloors](list_floors.md) - 列出可用楼层
- [GetCurrentFloor](get_current_floor.md) - 获取当前楼层
- [FloorTransition](../msg/floor_transition.md) - 楼层过渡点消息
