# ROS2 接口文档

本文档详细描述了 HumanoidNav 导航系统的所有ROS2接口，包括话题、服务和动作。

---

## **特别注意：**

如果算法包在自定义namespace下运行，那么所有的node，topic，service，action都会增加namespace前缀，但是消息体结构不会发生变化。

e.g. 

| **无namespace**                                              | **有namesapce<robot1>**                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /slam/set_mode                                               | /robot1/slam/set_mode                                        |
| ros2 service call /slam/set_mode fourier_msgs/srv/SetMode "{mode: 'mapping'}" | ros2 service call /robot1/slam/set_mode fourier_msgs/srv/SetMode "{mode: 'mapping'}" |
| /slam/mode_status                                            | /slam/mode_status                                            |
| ros2 topic echo /slam/mode_status                            | ros2 topic echo /robot1/slam/mode_status                     |

------

## 1. 建图定位模式切换

### 1.1 /slam/set_mode (服务)

> 在建图模式和定位模式之间切换

**服务类型**: [fourier_msgs/srv/SetMode](srv/set_mode.md)

**调用示例**:
```bash
# 切换到建图模式
ros2 service call /slam/set_mode fourier_msgs/srv/SetMode "{mode: 'mapping'}"

# 切换到定位模式
ros2 service call /slam/set_mode fourier_msgs/srv/SetMode "{mode: 'localization'}"
```

**Python调用**:
```python
from fourier_msgs.srv import SetMode
import rclpy
from rclpy.node import Node

class ModeClient(Node):
    def __init__(self):
        super().__init__('mode_client')
        self.client = self.create_client(SetMode, '/slam/set_mode')
        self.client.wait_for_service()

    def switch_mode(self, mode: str):
        request = SetMode.Request()
        request.mode = mode
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

# 使用示例
rclpy.init()
client = ModeClient()
result = client.switch_mode('localization')
print(f"Success: {result.success}, Message: {result.message}")
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/srv/set_mode.hpp>

class ModeClient : public rclcpp::Node {
public:
    ModeClient() : Node("mode_client") {
        client_ = create_client<fourier_msgs::srv::SetMode>("/slam/set_mode");
        client_->wait_for_service();
    }

    bool switch_mode(const std::string& mode) {
        auto request = std::make_shared<fourier_msgs::srv::SetMode::Request>();
        request->mode = mode;
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            RCLCPP_INFO(get_logger(), "Result: %s", result->message.c_str());
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<fourier_msgs::srv::SetMode>::SharedPtr client_;
};

// 使用示例
int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto client = std::make_shared<ModeClient>();
    client->switch_mode("localization");
    rclcpp::shutdown();
    return 0;
}
```

### 1.2 /slam/mode_status (话题)

> 发布当前系统模式状态

**消息类型**: [std_msgs/msg/String](https://docs.ros2.org/latest/api/std_msgs/msg/String.html)

**消息内容**:
- `"mapping"` - 建图模式
- `"localization"` - 定位模式

**订阅示例**:
```bash
ros2 topic echo /slam/mode_status
```

---

## 2. 建图模式接口

### 2.1 /clear_map (话题)

> 清除当前地图并重启建图

**消息类型**: [std_msgs/msg/String](https://docs.ros2.org/latest/api/std_msgs/msg/String.html)

**功能说明**:
- 如果处于定位模式：切换到建图模式并重置Octomap
- 如果处于建图模式：重启建图，清除所有地图数据

**发布示例**:
```bash
ros2 topic pub /clear_map std_msgs/msg/String "{data: ''}" --once
```

**Python发布**:
```python
from std_msgs.msg import String
import rclpy
from rclpy.node import Node

class ClearMapPublisher(Node):
    def __init__(self):
        super().__init__('clear_map_publisher')
        self.publisher = self.create_publisher(String, '/clear_map', 10)

    def clear(self):
        msg = String()
        msg.data = ''
        self.publisher.publish(msg)
        self.get_logger().info('Clear map command sent')

rclpy.init()
node = ClearMapPublisher()
node.clear()
```

**C++发布**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/string.hpp>

class ClearMapPublisher : public rclcpp::Node {
public:
    ClearMapPublisher() : Node("clear_map_publisher") {
        publisher_ = create_publisher<std_msgs::msg::String>("/clear_map", 10);
    }

    void clear() {
        auto msg = std_msgs::msg::String();
        msg.data = "";
        publisher_->publish(msg);
        RCLCPP_INFO(get_logger(), "Clear map command sent");
    }

private:
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
};
```

### 2.2 /map (话题)

> 发布2D占用栅格地图数据

**消息类型**: [nav_msgs/msg/OccupancyGrid](https://docs.ros2.org/latest/api/nav_msgs/msg/OccupancyGrid.html)

**数据来源**:
- 建图定位都直接订阅(`/map`)

**配置参数**:
| 参数 | 值 | 说明 |
| ---- | ---- | ---- |
| 分辨率 | 0.05m | 每个栅格单元5cm |
| Frame ID | map | 地图坐标系 |

**订阅示例**:
```bash
ros2 topic echo /map
```

### 2.3 /cloud_registered_gravity (话题)

> 发布3D点云地图数据（重力对齐后）

**消息类型**: [sensor_msgs/msg/PointCloud2](https://docs.ros2.org/latest/api/sensor_msgs/msg/PointCloud2.html)

**说明**: 由LIO（激光惯性里程计）处理后输出的点云数据，经过重力方向对齐。

**订阅示例**:
```bash
ros2 topic echo /cloud_registered_gravity
```

### 2.4 /optimize_map (话题)

> 发布优化后的2D地图数据

**消息类型**: [nav_msgs/msg/OccupancyGrid](https://docs.ros2.org/latest/api/nav_msgs/msg/OccupancyGrid.html)

**说明**: 经过降噪处理的2D占用栅格地图，移除了孤立像素和噪声点。仅在定位模式下发布。

**订阅示例**:
```bash
ros2 topic echo /optimize_map
```

### 2.5 /slam/save_map (服务)

> 保存3D点云地图

**服务类型**: [fourier_msgs/srv/SaveMap](srv/save_map.md)

**保存内容**:
- **3D地图**: `global.pcd` - PCL二进制压缩格式点云

**调用示例**:
```bash
# 保存到默认位置 ./data/my_map/
ros2 service call /slam/save_map fourier_msgs/srv/SaveMap "{map_id: 'my_map'}"

# 保存到指定绝对路径
ros2 service call /slam/save_map fourier_msgs/srv/SaveMap "{map_id: '/home/user/maps/office'}"
```

**Python调用**:
```python
from fourier_msgs.srv import SaveMap
import rclpy
from rclpy.node import Node

class MapSaver(Node):
    def __init__(self):
        super().__init__('map_saver')
        self.client = self.create_client(SaveMap, '/slam/save_map')
        self.client.wait_for_service()

    def save(self, map_id: str):
        request = SaveMap.Request()
        request.map_id = map_id
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        return result.response == 0  # 0 表示成功

rclpy.init()
saver = MapSaver()
success = saver.save('/home/user/maps/my_map')
print(f"Save {'succeeded' if success else 'failed'}")
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/srv/save_map.hpp>

class MapSaver : public rclcpp::Node {
public:
    MapSaver() : Node("map_saver") {
        client_ = create_client<fourier_msgs::srv::SaveMap>("/slam/save_map");
        client_->wait_for_service();
    }

    bool save(const std::string& map_id) {
        auto request = std::make_shared<fourier_msgs::srv::SaveMap::Request>();
        request->map_id = map_id;
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            return future.get()->response == 0;
        }
        return false;
    }

private:
    rclcpp::Client<fourier_msgs::srv::SaveMap>::SharedPtr client_;
};
```

### 2.6 nav2_map_server 保存2D地图

> 通过 nav2_map_server 的 map_saver_cli 保存2D占用栅格地图

**命令行工具**: `ros2 run nav2_map_server map_saver_cli`

**保存内容**:
- **2D地图**: `map.pgm` + `map.yaml` - 标准Nav2地图格式

**调用示例**:
```bash
# 指定话题和自由空间阈值
ros2 run nav2_map_server map_saver_cli --free 0.196 -t /map -f /path/to/save/map
```

**参数说明**:
| 参数 | 说明 |
| ---- | ---- |
| `-f` / `--output` | 输出文件路径（不含扩展名） |
| `-t` / `--topic` | 地图话题名称，默认 `/map` |
| `--free` | 自由空间阈值 (0.0-1.0)，默认 0.25 |
| `--occupied` | 占用阈值 (0.0-1.0)，默认 0.65 |

---

## 3. 定位模式接口

### 3.1 /initialpose (话题)

> 设置机器人初始位姿

**消息类型**: [geometry_msgs/msg/PoseWithCovarianceStamped](https://docs.ros2.org/latest/api/geometry_msgs/msg/PoseWithCovarianceStamped.html)

**发布示例**:
```bash
# 设置初始位姿：位置(1.0, 2.0, 0.0)，朝向yaw=90度(四元数表示)
ros2 topic pub /initialpose geometry_msgs/msg/PoseWithCovarianceStamped \
  "{header: {frame_id: 'map'}, pose: {pose: {position: {x: 1.0, y: 2.0, z: 0.0}, orientation: {x: 0.0, y: 0.0, z: 0.707, w: 0.707}}}}" --once
```

**Python发布**:
```python
from geometry_msgs.msg import PoseWithCovarianceStamped
import rclpy
from rclpy.node import Node
import math

class InitialPosePublisher(Node):
    def __init__(self):
        super().__init__('initialpose_publisher')
        self.publisher = self.create_publisher(
            PoseWithCovarianceStamped, '/initialpose', 10)

    def set_pose(self, x: float, y: float, yaw: float):
        msg = PoseWithCovarianceStamped()
        msg.header.frame_id = 'map'
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.pose.pose.position.x = x
        msg.pose.pose.position.y = y
        msg.pose.pose.position.z = 0.0
        # yaw转四元数
        msg.pose.pose.orientation.x = 0.0
        msg.pose.pose.orientation.y = 0.0
        msg.pose.pose.orientation.z = math.sin(yaw / 2.0)
        msg.pose.pose.orientation.w = math.cos(yaw / 2.0)
        self.publisher.publish(msg)

rclpy.init()
node = InitialPosePublisher()
node.set_pose(1.0, 2.0, 1.57)  # x=1, y=2, yaw=90度
```

**C++发布**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <geometry_msgs/msg/pose_with_covariance_stamped.hpp>
#include <cmath>

class InitialPosePublisher : public rclcpp::Node {
public:
    InitialPosePublisher() : Node("initialpose_publisher") {
        publisher_ = create_publisher<geometry_msgs::msg::PoseWithCovarianceStamped>(
            "/initialpose", 10);
    }

    void set_pose(double x, double y, double yaw) {
        auto msg = geometry_msgs::msg::PoseWithCovarianceStamped();
        msg.header.frame_id = "map";
        msg.header.stamp = now();
        msg.pose.pose.position.x = x;
        msg.pose.pose.position.y = y;
        msg.pose.pose.position.z = 0.0;
        // yaw转四元数
        msg.pose.pose.orientation.x = 0.0;
        msg.pose.pose.orientation.y = 0.0;
        msg.pose.pose.orientation.z = std::sin(yaw / 2.0);
        msg.pose.pose.orientation.w = std::cos(yaw / 2.0);
        publisher_->publish(msg);
    }

private:
    rclcpp::Publisher<geometry_msgs::msg::PoseWithCovarianceStamped>::SharedPtr publisher_;
};
```

### 3.2 /slam/load_map (服务)

> 加载地图并切换到定位模式

**服务类型**: [fourier_msgs/srv/LoadMap](srv/load_map.md)

**调用示例**:
```bash
ros2 service call /slam/load_map fourier_msgs/srv/LoadMap \
  "{map_path: '/home/user/maps/office', x: 0.0, y: 0.0, z: 0.0, yaw: 0.0}"
```

**Python调用**:
```python
from fourier_msgs.srv import LoadMap
import rclpy
from rclpy.node import Node

class MapLoader(Node):
    def __init__(self):
        super().__init__('map_loader')
        self.client = self.create_client(LoadMap, '/slam/load_map')
        self.client.wait_for_service()

    def load(self, path: str, x=0.0, y=0.0, z=0.0, yaw=0.0):
        request = LoadMap.Request()
        request.map_path = path
        request.x = x
        request.y = y
        request.z = z
        request.yaw = yaw
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        return result.result == 0  # 0 表示成功

rclpy.init()
loader = MapLoader()
success = loader.load('/home/user/maps/office', x=1.0, y=2.0, yaw=1.57)
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/srv/load_map.hpp>

class MapLoader : public rclcpp::Node {
public:
    MapLoader() : Node("map_loader") {
        client_ = create_client<fourier_msgs::srv::LoadMap>("/slam/load_map");
        client_->wait_for_service();
    }

    bool load(const std::string& path, double x=0.0, double y=0.0,
              double z=0.0, double yaw=0.0) {
        auto request = std::make_shared<fourier_msgs::srv::LoadMap::Request>();
        request->map_path = path;
        request->x = x;
        request->y = y;
        request->z = z;
        request->yaw = yaw;
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            return future.get()->result == 0;
        }
        return false;
    }

private:
    rclcpp::Client<fourier_msgs::srv::LoadMap>::SharedPtr client_;
};
```

### 3.3 /robot_pose (话题)

> 发布机器人3D位姿（6DOF）

**消息类型**: [geometry_msgs/msg/PoseStamped](https://docs.ros2.org/latest/api/geometry_msgs/msg/PoseStamped.html)

**说明**: 由定位系统发布的机器人在map坐标系下的完整3D位姿，包含x, y, z位置和完整的四元数朝向。相比2D导航常用的位姿，此话题提供了完整的6自由度位姿信息。

**发布频率**: 可配置，默认20Hz（通过 `pose_pub_period` 参数设置）

**Frame ID**: `map`

**订阅示例**:
```bash
ros2 topic echo /robot_pose
```

**Python订阅**:
```python
from geometry_msgs.msg import PoseStamped
import rclpy
from rclpy.node import Node

class PoseSubscriber(Node):
    def __init__(self):
        super().__init__('pose_subscriber')
        self.subscription = self.create_subscription(
            PoseStamped, '/robot_pose', self.pose_callback, 10)

    def pose_callback(self, msg):
        pos = msg.pose.position
        ori = msg.pose.orientation
        print(f"Position: x={pos.x:.3f}, y={pos.y:.3f}, z={pos.z:.3f}")
        print(f"Orientation: x={ori.x:.3f}, y={ori.y:.3f}, z={ori.z:.3f}, w={ori.w:.3f}")

rclpy.init()
node = PoseSubscriber()
rclpy.spin(node)
```

**C++订阅**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <geometry_msgs/msg/pose_stamped.hpp>

class PoseSubscriber : public rclcpp::Node {
public:
    PoseSubscriber() : Node("pose_subscriber") {
        subscription_ = create_subscription<geometry_msgs::msg::PoseStamped>(
            "/robot_pose", 10,
            [this](geometry_msgs::msg::PoseStamped::SharedPtr msg) {
                RCLCPP_INFO(get_logger(), "Position: [%.3f, %.3f, %.3f]",
                    msg->pose.position.x, msg->pose.position.y, msg->pose.position.z);
            });
    }

private:
    rclcpp::Subscription<geometry_msgs::msg::PoseStamped>::SharedPtr subscription_;
};
```

### 3.4 /odom (话题)

> 发布机器人里程计数据

**消息类型**: [nav_msgs/msg/Odometry](https://docs.ros2.org/latest/api/nav_msgs/msg/Odometry.html)

**说明**: 包含机器人的位置、姿态和速度信息。

**订阅示例**:
```bash
ros2 topic echo /odom
```

### 3.5 /odom_status_code (话题)

> 发布定位系统状态码

**消息类型**: [std_msgs/msg/Int8](https://docs.ros2.org/latest/api/std_msgs/msg/Int8.html)

**说明**: 实时发布定位系统的运行状态，用于监控定位过程和诊断定位问题。

**状态码定义**:

|| 状态码 | 名称 | 说明 |
|| ------ | ---- | ---- |
|| 0 | IDLE | 空闲状态 |
|| 1 | INITIALIZING | 初始化中 |
|| 2 | GOOD | 正常定位 |
|| 3 | FOLLOWING_DR | 定位异常（降级为航位推算） |
|| 4 | FAIL | 定位失败 |

**订阅示例**:
```bash
ros2 topic echo /odom_status_code
```

**Python订阅**:
```python
from std_msgs.msg import Int8
import rclpy
from rclpy.node import Node

class OdomStatusSubscriber(Node):
    # 状态码常量
    IDLE = 0
    INITIALIZING = 1
    GOOD = 2
    FOLLOWING_DR = 3
    FAIL = 4
    
    STATUS_NAMES = {
        0: "IDLE",
        1: "INITIALIZING", 
        2: "GOOD",
        3: "FOLLOWING_DR",
        4: "FAIL"
    }
    
    def __init__(self):
        super().__init__('odom_status_subscriber')
        self.subscription = self.create_subscription(
            Int8, '/odom_status_code', self.status_callback, 10)
    
    def status_callback(self, msg):
        status_name = self.STATUS_NAMES.get(msg.data, "UNKNOWN")
        print(f"定位状态: {status_name} (code: {msg.data})")
        
        if msg.data == self.GOOD:
            print("定位正常运行")
        elif msg.data == self.FOLLOWING_DR:
            print("警告: 定位异常，使用航位推算")
        elif msg.data == self.FAIL:
            print("错误: 定位失败")

rclpy.init()
node = OdomStatusSubscriber()
rclpy.spin(node)
```

**C++订阅**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/int8.hpp>

class OdomStatusSubscriber : public rclcpp::Node {
public:
    // 状态码常量
    static constexpr int8_t IDLE = 0;
    static constexpr int8_t INITIALIZING = 1;
    static constexpr int8_t GOOD = 2;
    static constexpr int8_t FOLLOWING_DR = 3;
    static constexpr int8_t FAIL = 4;
    
    OdomStatusSubscriber() : Node("odom_status_subscriber") {
        subscription_ = create_subscription<std_msgs::msg::Int8>(
            "/odom_status_code", 10,
            [this](std_msgs::msg::Int8::SharedPtr msg) {
                std::string status_name = getStatusName(msg->data);
                RCLCPP_INFO(get_logger(), "定位状态: %s (code: %d)", 
                    status_name.c_str(), msg->data);
                
                if (msg->data == GOOD) {
                    RCLCPP_INFO(get_logger(), "定位正常运行");
                } else if (msg->data == FOLLOWING_DR) {
                    RCLCPP_WARN(get_logger(), "定位异常，使用航位推算");
                } else if (msg->data == FAIL) {
                    RCLCPP_ERROR(get_logger(), "定位失败");
                }
            });
    }

private:
    std::string getStatusName(int8_t code) {
        switch(code) {
            case IDLE: return "IDLE";
            case INITIALIZING: return "INITIALIZING";
            case GOOD: return "GOOD";
            case FOLLOWING_DR: return "FOLLOWING_DR";
            case FAIL: return "FAIL";
            default: return "UNKNOWN";
        }
    }
    
    rclcpp::Subscription<std_msgs::msg::Int8>::SharedPtr subscription_;
};
```

### 3.6 /odom_status_score (话题)

> 发布定位置信度分数

**消息类型**: [std_msgs/msg/Int8](https://docs.ros2.org/latest/api/std_msgs/msg/Int8.html)

**说明**: 发布定位系统的置信度评分，用于评估定位质量。仅当定位状态为GOOD，FOLLOWING_DR，FAIL时，该话题发布有效的置信度值；其他状态下该值为0。

**数据说明**:
- 当 `/odom_status_code` = （2 || 3 || 4）时：`data` = 定位置信度 (0-100)
- 当 `/odom_status_code` = （0 || 1） 时：`data` = 0

**订阅示例**:
```bash
ros2 topic echo /odom_status_score
```

**Python订阅（联合状态码使用）**:
```python
from std_msgs.msg import Int8, Float32
import rclpy
from rclpy.node import Node

class OdomMonitor(Node):
    def __init__(self):
        super().__init__('odom_monitor')
        self.status_code = 0
        self.status_score = 0.0
        
        self.code_sub = self.create_subscription(
            Int8, '/odom_status_code', self.code_callback, 10)
        self.score_sub = self.create_subscription(
            Float32, '/odom_status_score', self.score_callback, 10)
    
    def code_callback(self, msg):
        self.status_code = msg.data
        self.print_status()
    
    def score_callback(self, msg):
        self.status_score = msg.data
        self.print_status()
    
    def print_status(self):
        if self.status_code == 2:  # GOOD
            print(f"定位正常 - 置信度: {self.status_score:.1f}/100")
            if self.status_score < 50:
                print("警告: 置信度较低")
        else:
            print(f"定位状态异常 (code: {self.status_code}), score: {self.status_score}")

rclpy.init()
node = OdomMonitor()
rclpy.spin(node)
```

**C++订阅（联合状态码使用）**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <std_msgs/msg/int8.hpp>
#include <std_msgs/msg/float32.hpp>

class OdomMonitor : public rclcpp::Node {
public:
    OdomMonitor() : Node("odom_monitor"), status_code_(0), status_score_(0.0) {
        code_sub_ = create_subscription<std_msgs::msg::Int8>(
            "/odom_status_code", 10,
            [this](std_msgs::msg::Int8::SharedPtr msg) {
                status_code_ = msg->data;
                print_status();
            });
        
        score_sub_ = create_subscription<std_msgs::msg::Int8>(
            "/odom_status_score", 10,
            [this](std_msgs::msg::Int8::SharedPtr msg) {
                status_score_ = msg->data;
                print_status();
            });
    }

private:
    void print_status() {
        if (status_code_ == 2) {  // GOOD
            RCLCPP_INFO(get_logger(), "定位正常 - 置信度: %.1f/100", 
                status_score_);
            if (status_score_ < 50) {
                RCLCPP_WARN(get_logger(), "置信度较低");
            }
        } else {
            RCLCPP_INFO(get_logger(), "定位状态异常 (code: %d), score: %.1f",
                status_code_, status_score_);
        }
    }
    
    rclcpp::Subscription<std_msgs::msg::Int8>::SharedPtr code_sub_;
    rclcpp::Subscription<std_msgs::msg::Float32>::SharedPtr score_sub_;
    int8_t status_code_;
    float status_score_;
};
```

---

## 4. 导航相关接口

### 4.1 navigate_to_pose (动作)

> 导航到指定目标点

**动作类型**: nav2_msgs/action/NavigateToPose

#### Goal（目标请求）

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| pose | geometry_msgs/PoseStamped | 目标位姿，包含位置和朝向 |
| behavior_tree | string | 可选，指定自定义行为树XML文件路径，留空使用默认行为树 |

#### Result（执行结果）

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| result | std_msgs/Empty | 空结果（成功时） |
| error_code | uint16 | 错误码 |
| error_msg | string | 错误描述信息 |

**错误码定义**:

| 错误码 | 名称 | 描述 |
| ------ | ---- | ---- |
| 0 | NONE | 无错误，导航成功 |
| 9001 | UNKNOWN | 未知错误 |
| 9002 | FAILED_TO_LOAD_BEHAVIOR_TREE | 加载行为树失败 |
| 9003 | TF_ERROR | TF变换错误 |
| 9004 | GOAL_CHECKER_ERROR | 目标检查器错误 |
| 9005 | PREEMPTED | 被新目标抢占 |
| 9006 | NO_VALID_PATH | 无法找到有效路径 |

#### Feedback（实时反馈）

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| current_pose | geometry_msgs/PoseStamped | 机器人当前位姿 |
| navigation_time | builtin_interfaces/Duration | 已导航时间 |
| estimated_time_remaining | builtin_interfaces/Duration | 预计剩余时间 |
| number_of_recoveries | int16 | 恢复行为执行次数 |
| distance_remaining | float32 | 距目标剩余距离（米） |

#### 调用示例

**命令行**:
```bash
# 导航到位置(1.0, 2.0)，朝向yaw=45度
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
  "{pose: {header: {frame_id: 'map'}, pose: {position: {x: 1.0, y: 2.0, z: 0.0}, orientation: {x: 0.0, y: 0.0, z: 0.383, w: 0.924}}}}"

# 带反馈信息
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
  "{pose: {header: {frame_id: 'map'}, pose: {position: {x: 1.0, y: 2.0, z: 0.0}, orientation: {x: 0.0, y: 0.0, z: 0.383, w: 0.924}}}}" --feedback
```

**Python调用**:
```python
from nav2_simple_commander.robot_navigator import BasicNavigator, TaskResult
from geometry_msgs.msg import PoseStamped
import math

# 初始化导航器
navigator = BasicNavigator()
navigator.waitUntilNav2Active()

# 构造目标位姿
goal_pose = PoseStamped()
goal_pose.header.frame_id = 'map'
goal_pose.header.stamp = navigator.get_clock().now().to_msg()
goal_pose.pose.position.x = 1.0
goal_pose.pose.position.y = 2.0
goal_pose.pose.position.z = 0.0
# yaw=45度转四元数
yaw = 0.785  # 45度 (π/4)
goal_pose.pose.orientation.x = 0.0
goal_pose.pose.orientation.y = 0.0
goal_pose.pose.orientation.z = math.sin(yaw / 2.0)
goal_pose.pose.orientation.w = math.cos(yaw / 2.0)

# 发送导航目标
navigator.goToPose(goal_pose)

# 监控导航进度
while not navigator.isTaskComplete():
    feedback = navigator.getFeedback()
    if feedback:
        print(f"距离目标: {feedback.distance_remaining:.2f}m")
        print(f"已导航时间: {feedback.navigation_time.sec}s")
        print(f"恢复次数: {feedback.number_of_recoveries}")

# 获取结果
result = navigator.getResult()
if result == TaskResult.SUCCEEDED:
    print("导航成功!")
elif result == TaskResult.CANCELED:
    print("导航被取消")
elif result == TaskResult.FAILED:
    print("导航失败")
```

**C++调用（完整示例）**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <rclcpp_action/rclcpp_action.hpp>
#include <nav2_msgs/action/navigate_to_pose.hpp>
#include <cmath>

using NavigateToPose = nav2_msgs::action::NavigateToPose;
using GoalHandleNavigateToPose = rclcpp_action::ClientGoalHandle<NavigateToPose>;

class NavigationClient : public rclcpp::Node {
public:
    NavigationClient() : Node("navigation_client") {
        client_ = rclcpp_action::create_client<NavigateToPose>(
            this, "/navigate_to_pose");
        client_->wait_for_action_server();
    }

    void navigate_to(double x, double y, double yaw) {
        // 构造目标
        auto goal = NavigateToPose::Goal();
        goal.pose.header.frame_id = "map";
        goal.pose.header.stamp = now();
        goal.pose.pose.position.x = x;
        goal.pose.pose.position.y = y;
        goal.pose.pose.position.z = 0.0;
        goal.pose.pose.orientation.x = 0.0;
        goal.pose.pose.orientation.y = 0.0;
        goal.pose.pose.orientation.z = std::sin(yaw / 2.0);
        goal.pose.pose.orientation.w = std::cos(yaw / 2.0);

        // 设置回调
        auto send_goal_options = rclcpp_action::Client<NavigateToPose>::SendGoalOptions();

        // 反馈回调
        send_goal_options.feedback_callback =
            [this](GoalHandleNavigateToPose::SharedPtr,
                   const std::shared_ptr<const NavigateToPose::Feedback> feedback) {
                RCLCPP_INFO(get_logger(), "距离目标: %.2fm, 恢复次数: %d",
                    feedback->distance_remaining, feedback->number_of_recoveries);
            };

        // 结果回调
        send_goal_options.result_callback =
            [this](const GoalHandleNavigateToPose::WrappedResult& result) {
                switch (result.code) {
                    case rclcpp_action::ResultCode::SUCCEEDED:
                        RCLCPP_INFO(get_logger(), "导航成功!");
                        break;
                    case rclcpp_action::ResultCode::ABORTED:
                        RCLCPP_ERROR(get_logger(), "导航中止");
                        break;
                    case rclcpp_action::ResultCode::CANCELED:
                        RCLCPP_WARN(get_logger(), "导航取消");
                        break;
                    default:
                        RCLCPP_ERROR(get_logger(), "未知结果");
                        break;
                }
            };

        // 发送目标
        client_->async_send_goal(goal, send_goal_options);
    }

    void cancel_navigation() {
        client_->async_cancel_all_goals();
    }

private:
    rclcpp_action::Client<NavigateToPose>::SharedPtr client_;
};

int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto client = std::make_shared<NavigationClient>();
    client->navigate_to(1.0, 2.0, 0.785);  // x=1, y=2, yaw=45度
    rclcpp::spin(client);
    rclcpp::shutdown();
    return 0;
}
```

### 4.2 /plan (话题)

> 发布全局规划路径

**消息类型**: [nav_msgs/msg/Path](https://docs.ros2.org/latest/api/nav_msgs/msg/Path.html)

**说明**: 由全局规划器（SmacPlanner2D）生成的路径。

**订阅示例**:
```bash
ros2 topic echo /plan
```

### 4.3 /cmd_vel (话题)

> 机器人速度命令

**消息类型**: [geometry_msgs/msg/Twist](https://docs.ros2.org/latest/api/geometry_msgs/msg/Twist.html)

**订阅示例**:
```bash
ros2 topic echo /cmd_vel
```

### 4.4 cancel_current_action (服务)

> 取消当前正在执行的导航动作

**服务类型**: [fourier_msgs/srv/CancelCurrentAction](srv/cancel_current_action.md)

**调用示例**:
```bash
ros2 service call /cancel_current_action fourier_msgs/srv/CancelCurrentAction
```

**Python调用**:
```python
from fourier_msgs.srv import CancelCurrentAction
import rclpy
from rclpy.node import Node

class ActionCanceller(Node):
    def __init__(self):
        super().__init__('action_canceller')
        self.client = self.create_client(CancelCurrentAction, '/cancel_current_action')
        self.client.wait_for_service()

    def cancel(self):
        request = CancelCurrentAction.Request()
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        return result.success

rclpy.init()
canceller = ActionCanceller()
if canceller.cancel():
    print("Action cancelled successfully")
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/srv/cancel_current_action.hpp>

class ActionCanceller : public rclcpp::Node {
public:
    ActionCanceller() : Node("action_canceller") {
        client_ = create_client<fourier_msgs::srv::CancelCurrentAction>(
            "/cancel_current_action");
        client_->wait_for_service();
    }

    bool cancel() {
        auto request = std::make_shared<fourier_msgs::srv::CancelCurrentAction::Request>();
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            return future.get()->success;
        }
        return false;
    }

private:
    rclcpp::Client<fourier_msgs::srv::CancelCurrentAction>::SharedPtr client_;
};
```

### 4.5 get_current_action (服务)

> 获取当前正在执行的动作信息

**服务类型**: [fourier_msgs/srv/GetCurrentAction](srv/get_current_action.md)

**调用示例**:
```bash
ros2 service call /get_current_action fourier_msgs/srv/GetCurrentAction
```

**Python调用**:
```python
from fourier_msgs.srv import GetCurrentAction
import rclpy
from rclpy.node import Node

class ActionMonitor(Node):
    def __init__(self):
        super().__init__('action_monitor')
        self.client = self.create_client(GetCurrentAction, '/get_current_action')
        self.client.wait_for_service()

    def get_status(self):
        request = GetCurrentAction.Request()
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        if result.success:
            print(f"Action: {result.action_name}")
            print(f"Status: {result.status_description}")
        return result

rclpy.init()
monitor = ActionMonitor()
monitor.get_status()
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/srv/get_current_action.hpp>

class ActionMonitor : public rclcpp::Node {
public:
    ActionMonitor() : Node("action_monitor") {
        client_ = create_client<fourier_msgs::srv::GetCurrentAction>(
            "/get_current_action");
        client_->wait_for_service();
    }

    void get_status() {
        auto request = std::make_shared<fourier_msgs::srv::GetCurrentAction::Request>();
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Action: %s, Status: %s",
                    result->action_name.c_str(),
                    result->status_description.c_str());
            }
        }
    }

private:
    rclcpp::Client<fourier_msgs::srv::GetCurrentAction>::SharedPtr client_;
};
```

### 4.6 action_status (话题)

> 发布当前动作执行状态

**消息类型**: [fourier_msgs/msg/ActionStatus](msg/action_status.md)

**发布频率**: 1 Hz

**订阅示例**:
```bash
ros2 topic echo /action_status
```

---

## 5. 传感器数据接口

### 5.1 /scan (话题)

> 2D激光雷达扫描数据

**消息类型**: [sensor_msgs/msg/LaserScan](https://docs.ros2.org/latest/api/sensor_msgs/msg/LaserScan.html)

**说明**: 由3D点云转换得到的2D激光扫描数据，用于2D导航。

**订阅示例**:
```bash
ros2 topic echo /scan
```

### 5.2 /segmented_groundless_points (话题)

> 3D点云数据（已去除地面）

**消息类型**: [sensor_msgs/msg/PointCloud2](https://docs.ros2.org/latest/api/sensor_msgs/msg/PointCloud2.html)

**说明**: 经过地面分割处理的点云数据，用于障碍物检测。

**订阅示例**:
```bash
ros2 topic echo /segmented_groundless_points
```

### 5.3 /rslidar_points (话题)

> RoboSense激光雷达原始点云数据

**消息类型**: [sensor_msgs/msg/PointCloud2](https://docs.ros2.org/latest/api/sensor_msgs/msg/PointCloud2.html)

**订阅示例**:
```bash
ros2 topic echo /rslidar_points
```

### 5.4 /imu (话题)

> IMU传感器数据

**消息类型**: [sensor_msgs/msg/Imu](https://docs.ros2.org/latest/api/sensor_msgs/msg/Imu.html)

**说明**: 包含加速度计和陀螺仪数据，用于激光惯性里程计融合。

**订阅示例**:
```bash
ros2 topic echo /imu
```

---

## 6. 相机接口

系统集成了深度相机，提供彩色图像、深度图像和点云数据。相机数据经过预处理后可用于感知、建图和导航。

### 6.1 相机配置

系统支持单相机和多相机配置。相机配置通过 launch 文件和参数文件进行管理。

**多相机命名规范**：
- 系统支持配置多个深度相机
- 相机命名依次为：`camera_01`, `camera_02`, `camera_03`, ...
- 每个相机独立配置 USB 端口、参数文件和话题命名空间
- 相机编号从 01 开始递增，使用两位数字格式

**话题命名规则**：
- 相机1话题前缀：`/camera_01/`
- 相机2话题前缀：`/camera_02/`
- 示例：`/camera_01/color/image_raw`, `/camera_02/depth/points`

### 6.2 相机驱动接口（CAMERA_SDK_ROS2）

#### 6.2.1 /camera_01/color/image_raw (话题)

> 发布相机彩色图像数据

**消息类型**: [sensor_msgs/msg/Image](https://docs.ros2.org/latest/api/sensor_msgs/msg/Image.html)

**说明**: 发布相机采集的原始彩色图像（RGB格式）。

**订阅示例**:
```bash
ros2 topic echo /camera_01/color/image_raw
```

#### 6.2.2 /camera_01/color/camera_info (话题)

> 发布彩色相机的标定信息

**消息类型**: [sensor_msgs/msg/CameraInfo](https://docs.ros2.org/latest/api/sensor_msgs/msg/CameraInfo.html)

**说明**: 包含相机的内参矩阵、畸变参数等标定信息。

**订阅示例**:
```bash
ros2 topic echo /camera_01/color/camera_info
```

#### 6.2.3 /camera_01/depth/image_raw (话题)

> 发布相机深度图像数据

**消息类型**: [sensor_msgs/msg/Image](https://docs.ros2.org/latest/api/sensor_msgs/msg/Image.html)

**说明**: 发布相机采集的原始深度图像，每个像素值表示该点到相机的距离（单位：毫米或米，取决于编码格式）。

**订阅示例**:
```bash
ros2 topic echo /camera_01/depth/image_raw
```

#### 6.2.4 /camera_01/depth/camera_info (话题)

> 发布深度相机的标定信息

**消息类型**: [sensor_msgs/msg/CameraInfo](https://docs.ros2.org/latest/api/sensor_msgs/msg/CameraInfo.html)

**说明**: 包含深度相机的内参矩阵、畸变参数等标定信息。

**订阅示例**:
```bash
ros2 topic echo /camera_01/depth/camera_info
```

#### 6.2.5 /camera_01/depth/points (话题)

> 发布深度点云数据

**消息类型**: [sensor_msgs/msg/PointCloud2](https://docs.ros2.org/latest/api/sensor_msgs/msg/PointCloud2.html)

**说明**: 由深度图像生成的3D点云数据，坐标系为相机光学坐标系（camera_01_color_optical_frame）。

**订阅示例**:
```bash
ros2 topic echo /camera_01/depth/points
```

### 6.3 相机预处理接口（camera_preprocessing）

相机预处理节点对原始相机数据进行同步、融合和过滤处理，输出统一的融合数据和过滤后的点云。

#### 6.3.1 /camera_01/fused_data (话题)

> 发布融合的相机数据（RGB + 点云 + 机器人位姿）

**消息类型**: [fourier_msgs/msg/UnifiedCameraData](msg/unified_camera_data.md)

**消息内容**:
- `header`: 消息头，包含时间戳和坐标系
- `camera_name`: 相机名称（用于多相机区分）
- `rgb_image`: RGB 彩色图像
- `point_cloud`: 原始深度点云（camera_link 坐标系）
- `robot_pose`: 机器人位姿（通过时间戳插值得到）
- `camera_transform`: 相机外参（camera_link -> base_link）

**功能说明**:
- 同步采集同一时刻的 RGB 图像和深度点云
- 根据数据时间戳插值获取对应的机器人位姿
- 将所有数据打包成统一的消息格式
- 用于下游的感知、建图等任务

**订阅示例**:
```bash
ros2 topic echo /camera_01/fused_data
```

**Python订阅**:
```python
from fourier_msgs.msg import UnifiedCameraData
import rclpy
from rclpy.node import Node

class CameraDataSubscriber(Node):
    def __init__(self):
        super().__init__('camera_data_subscriber')
        self.subscription = self.create_subscription(
            UnifiedCameraData,
            '/camera_01/fused_data',
            self.callback,
            10)

    def callback(self, msg):
        print(f"Camera: {msg.camera_name}")
        print(f"Image size: {msg.rgb_image.width}x{msg.rgb_image.height}")
        print(f"Point cloud points: {msg.point_cloud.width * msg.point_cloud.height}")
        print(f"Robot pose: x={msg.robot_pose.position.x:.3f}, "
              f"y={msg.robot_pose.position.y:.3f}, "
              f"z={msg.robot_pose.position.z:.3f}")

rclpy.init()
node = CameraDataSubscriber()
rclpy.spin(node)
```

**C++订阅**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/msg/unified_camera_data.hpp>

class CameraDataSubscriber : public rclcpp::Node {
public:
    CameraDataSubscriber() : Node("camera_data_subscriber") {
        subscription_ = create_subscription<fourier_msgs::msg::UnifiedCameraData>(
            "/camera_01/fused_data", 10,
            [this](fourier_msgs::msg::UnifiedCameraData::SharedPtr msg) {
                RCLCPP_INFO(get_logger(), "Camera: %s", msg->camera_name.c_str());
                RCLCPP_INFO(get_logger(), "Image size: %dx%d",
                    msg->rgb_image.width, msg->rgb_image.height);
                RCLCPP_INFO(get_logger(), "Point cloud points: %d",
                    msg->point_cloud.width * msg->point_cloud.height);
                RCLCPP_INFO(get_logger(), "Robot pose: [%.3f, %.3f, %.3f]",
                    msg->robot_pose.position.x,
                    msg->robot_pose.position.y,
                    msg->robot_pose.position.z);
            });
    }

private:
    rclcpp::Subscription<fourier_msgs::msg::UnifiedCameraData>::SharedPtr subscription_;
};
```

#### 6.3.2 /camera_01/filtered_pointcloud (话题)

> 发布过滤处理后的点云数据

**消息类型**: [sensor_msgs/msg/PointCloud2](https://docs.ros2.org/latest/api/sensor_msgs/msg/PointCloud2.html)

**功能说明**:
- 应用可配置的过滤器链进行点云处理：
  - **体素降采样** (VoxelGridFilter): 减少点云密度
  - **区域过滤** (RegionFilter): 保留指定3D区域内的点
  - **地面过滤** (GroundFilter): 移除地面点云
  - **坐标转换** (TransformFilter): 坐标系转换
- 输出经过处理的高质量点云，用于障碍物检测和导航

**坐标系**: `base_link`（或由 TransformFilter 指定的目标坐标系）

**订阅示例**:
```bash
ros2 topic echo /camera_01/filtered_pointcloud
```

**Python订阅**:
```python
from sensor_msgs.msg import PointCloud2
import rclpy
from rclpy.node import Node

class FilteredCloudSubscriber(Node):
    def __init__(self):
        super().__init__('filtered_cloud_subscriber')
        self.subscription = self.create_subscription(
            PointCloud2,
            '/camera_01/filtered_pointcloud',
            self.callback,
            10)

    def callback(self, msg):
        point_count = msg.width * msg.height
        print(f"Filtered point cloud: {point_count} points")
        print(f"Frame ID: {msg.header.frame_id}")
        print(f"Timestamp: {msg.header.stamp.sec}.{msg.header.stamp.nanosec}")

rclpy.init()
node = FilteredCloudSubscriber()
rclpy.spin(node)
```

**C++订阅**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <sensor_msgs/msg/point_cloud2.hpp>

class FilteredCloudSubscriber : public rclcpp::Node {
public:
    FilteredCloudSubscriber() : Node("filtered_cloud_subscriber") {
        subscription_ = create_subscription<sensor_msgs::msg::PointCloud2>(
            "/camera_01/filtered_pointcloud", 10,
            [this](sensor_msgs::msg::PointCloud2::SharedPtr msg) {
                int point_count = msg->width * msg->height;
                RCLCPP_INFO(get_logger(), "Filtered point cloud: %d points", point_count);
                RCLCPP_INFO(get_logger(), "Frame ID: %s", msg->header.frame_id.c_str());
            });
    }

private:
    rclcpp::Subscription<sensor_msgs::msg::PointCloud2>::SharedPtr subscription_;
};
```

---

## 7. 健康状态监控接口

系统提供健康状态监控功能，用于实时监测各组件运行状态并报告错误。

### 7.1 /Humanoid_nav/health (话题)

> 发布聚合后的系统健康状态

**消息类型**: [fourier_msgs/msg/HealthInfo](msg/health_info.md)

**发布频率**: 10 Hz（可配置）

**订阅示例**:
```bash
ros2 topic echo /Humanoid_nav/health
```

错误级别和错误码定义详见 [BaseErrorInfo](msg/base_error_info.md)。

### 7.2 Python 订阅示例

```python
from fourier_msgs.msg import HealthInfo
import rclpy
from rclpy.node import Node

class HealthSubscriber(Node):
    def __init__(self):
        super().__init__('health_subscriber')
        self.subscription = self.create_subscription(
            HealthInfo, '/Humanoid_nav/health', self.health_callback, 10)

    def health_callback(self, msg):
        if msg.has_fatal:
            print("致命错误!")
        elif msg.has_error:
            print("存在错误")
        elif msg.has_warning:
            print("存在警告")
        else:
            print("系统健康")

        for error in msg.errors:
            print(f"  [{hex(error.error_code)}] {error.message}")

rclpy.init()
node = HealthSubscriber()
rclpy.spin(node)
```

### 7.3 C++ 订阅示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/msg/health_info.hpp>

class HealthSubscriber : public rclcpp::Node {
public:
    HealthSubscriber() : Node("health_subscriber") {
        subscription_ = create_subscription<fourier_msgs::msg::HealthInfo>(
            "/Humanoid_nav/health", 10,
            [this](fourier_msgs::msg::HealthInfo::SharedPtr msg) {
                if (msg->has_fatal) {
                    RCLCPP_FATAL(get_logger(), "致命错误!");
                } else if (msg->has_error) {
                    RCLCPP_ERROR(get_logger(), "存在错误");
                } else if (msg->has_warning) {
                    RCLCPP_WARN(get_logger(), "存在警告");
                }

                for (const auto& error : msg->errors) {
                    RCLCPP_INFO(get_logger(), "[0x%08X] %s",
                        error.error_code, error.message.c_str());
                }
            });
    }

private:
    rclcpp::Subscription<fourier_msgs::msg::HealthInfo>::SharedPtr subscription_;
};
```

---

## 8. 事件通知接口

系统提供事件通知功能，用于实时发布导航过程中的重要事件。

### 8.1 /Humanoid_nav/events (话题)

> 发布聚合后的系统事件

**消息类型**: [fourier_msgs/msg/EventsInfo](msg/events_info.md)

**发布频率**: 10 Hz（可配置）

**订阅示例**:
```bash
ros2 topic echo /Humanoid_nav/events
```

完整的事件类型列表和详细说明请参见 [BaseEventInfo](msg/base_event_info.md)。

### 8.2 Python 订阅示例

```python
from fourier_msgs.msg import EventsInfo
import rclpy
from rclpy.node import Node

class EventsSubscriber(Node):
    def __init__(self):
        super().__init__('events_subscriber')
        self.subscription = self.create_subscription(
            EventsInfo, '/Humanoid_nav/events', self.events_callback, 10)

    def events_callback(self, msg):
        for event in msg.events:
            print(f"[{event.event_type}] {event.message} (from: {event.source})")

            # 处理特定事件
            if event.event_type == "obstacle_blocked":
                print("警告: 路径被阻挡，可能需要重新规划")
            elif event.event_type == "near_obstacle":
                print("警告: 检测到近处障碍物")
            elif event.event_type == "out_of_map":
                print("警告: 机器人走出地图边界或进入未知区域")
            elif event.event_type == "enter_forbidden_area":
                print("警告: 机器人进入禁区")
            elif event.event_type == "enter_dangerous_area":
                print("警告: 机器人进入危险区域（限速区）")
            elif event.event_type == "map_loop_closure":
                print("信息: 检测到闭环，地图已优化")

rclpy.init()
node = EventsSubscriber()
rclpy.spin(node)
```

### 8.3 C++ 订阅示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <fourier_msgs/msg/events_info.hpp>

class EventsSubscriber : public rclcpp::Node {
public:
    EventsSubscriber() : Node("events_subscriber") {
        subscription_ = create_subscription<fourier_msgs::msg::EventsInfo>(
            "/Humanoid_nav/events", 10,
            [this](fourier_msgs::msg::EventsInfo::SharedPtr msg) {
                for (const auto& event : msg->events) {
                    RCLCPP_INFO(get_logger(), "[%s] %s (from: %s)",
                        event.event_type.c_str(),
                        event.message.c_str(),
                        event.source.c_str());

                    if (event.event_type == "obstacle_blocked") {
                        RCLCPP_WARN(get_logger(), "路径被阻挡，可能需要重新规划");
                    } else if (event.event_type == "near_obstacle") {
                        RCLCPP_WARN(get_logger(), "检测到近处障碍物");
                    } else if (event.event_type == "out_of_map") {
                        RCLCPP_WARN(get_logger(), "机器人走出地图边界或进入未知区域");
                    } else if (event.event_type == "enter_forbidden_area") {
                        RCLCPP_WARN(get_logger(), "机器人进入禁区");
                    } else if (event.event_type == "enter_dangerous_area") {
                        RCLCPP_WARN(get_logger(), "机器人进入危险区域（限速区）");
                    }
                }
            });
    }

private:
    rclcpp::Subscription<fourier_msgs::msg::EventsInfo>::SharedPtr subscription_;
};
```
---

## 9. 地图管理接口

map_manager 模块提供多楼层复合地图管理功能，包括楼层管理、语义元素（POI、虚拟墙、禁区、限速区、房间、语义对象）管理等。

### 9.0 Namespace 支持

map_manager 支持 ROS2 命名空间，用于多机器人场景。

**服务路径规律**：map_manager 服务使用**相对路径**注册，路径跟随**节点 namespace**（不含节点名称）：

| 启动方式 | 服务路径示例 |
|---|---|
| 无 namespace（默认）| `/add_floor`、`/list_floors` 等 |
| `namespace:=robot1` | `/robot1/add_floor`、`/robot1/list_floors` 等 |

**命令行调用约定（使用 `MM` 变量）：**

```bash
# 单机器人模式（无命名空间）—— 服务在根路径
MM=

# 多机器人模式（有命名空间，如 robot1）
MM=/robot1

# 之后统一使用 ${MM} 调用服务
ros2 service call ${MM}/load_composite_map map_manager/srv/LoadCompositeMap ...
```

> **验证**：`ros2 service list | grep -E "add_floor|list_floors"` 查看实际路径

**话题路径变化：**

| 话题（无命名空间）| 话题（命名空间 robot1）|
| --- | --- |
| `/current_floor` | `/robot1/current_floor` |
| `/costmap_filter_info` | `/robot1/costmap_filter_info` |
| `/keepout_filter_mask` | `/robot1/keepout_filter_mask` |

**启动命令：**

```bash
# 单机器人（默认）
ros2 launch map_manager map_manager.launch.py

# 多机器人（带命名空间）
ros2 launch map_manager map_manager.launch.py namespace:=robot1 use_namespace:=true
```

### 9.1 消息类型定义

#### 9.1.1 Floor 消息

**消息类型**: `map_manager/msg/Floor` | [详细文档](msg/floor.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| name | string | 楼层名称 |
| level | int32 | 楼层层数（如 1, 2, -1） |
| min_height | float64 | 楼层最小高度 (m) |
| max_height | float64 | 楼层最大高度 (m) |
| reference_height | float64 | 参考高度 (m) |
| origin_offset | geometry_msgs/Pose | 原点偏移 |
| map_path | string | 地图文件路径 |
| status | uint8 | 状态 (0=UNKNOWN, 1=ACTIVE, 2=INACTIVE) |

#### 9.1.2 CompositeMapInfo 消息

**消息类型**: `map_manager/msg/CompositeMapInfo` | [详细文档](msg/composite_map_info.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| map_id | string | 地图唯一标识 |
| name | string | 地图名称 |
| version | string | 版本号 |
| created_at | builtin_interfaces/Time | 创建时间 |
| modified_at | builtin_interfaces/Time | 修改时间 |
| origin | geometry_msgs/Pose | 地图原点 |
| floors | Floor[] | 楼层列表 |
| transitions | FloorTransition[] | 楼层过渡点列表 |
| root_path | string | 地图根目录 |

#### 9.1.3 POI 消息

**消息类型**: `map_manager/msg/POI` | [详细文档](msg/poi.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | string | POI 唯一标识 |
| name | string | POI 名称 |
| type | string | POI 类型 (door, room, charging_station 等) |
| floor_id | string | 所属楼层 ID（空表示全局） |
| pose | geometry_msgs/Pose2D | 2D 位姿 (x, y, theta) |
| height | float64 | 距楼层参考高度 (m) |
| properties | string | 自定义属性 (JSON 字符串) |

#### 9.1.4 VirtualWall 消息

**消息类型**: `map_manager/msg/VirtualWall` | [详细文档](msg/virtual_wall.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | int32 | 虚拟墙 ID |
| floor_id | string | 所属楼层 ID |
| start | geometry_msgs/Point | 起点坐标 |
| end | geometry_msgs/Point | 终点坐标 |

#### 9.1.5 ForbiddenArea 消息

**消息类型**: `map_manager/msg/ForbiddenArea` | [详细文档](msg/forbidden_area.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | int32 | 禁区 ID |
| boundary | geometry_msgs/Polygon | 多边形边界 |

#### 9.1.6 DangerousArea 消息

**消息类型**: `map_manager/msg/DangerousArea` | [详细文档](msg/dangerous_area.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | int32 | 限速区 ID |
| boundary | geometry_msgs/Polygon | 多边形边界 |
| speed_limit | float32 | 速度限制 (m/s) |

#### 9.1.7 Room 消息

**消息类型**: `map_manager/msg/Room` | [详细文档](msg/room.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | string | 房间 ID |
| name | string | 房间名称 |
| type | string | 房间类型 |
| boundary | geometry_msgs/Point[] | 边界点 |
| floor_height | float64 | 地板高度 |
| ceiling_height | float64 | 天花板高度 |
| connected_rooms | string[] | 连接的房间 ID 列表 |
| objects | string[] | 包含的对象 ID 列表 |

#### 9.1.8 SemanticObject 消息

**消息类型**: `map_manager/msg/SemanticObject` | [详细文档](msg/semantic_object.md)

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | string | 对象 ID |
| name | string | 对象名称 |
| type | string | 对象类型 |
| category | string | 对象类别 |
| pose | geometry_msgs/Pose | 3D 位姿 |
| dimensions | geometry_msgs/Vector3 | 尺寸 (x, y, z) |
| is_static | bool | 是否静态对象 |

#### 9.1.9 FloorTransition 消息

**消息类型**: `map_manager/msg/FloorTransition`

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| id | string | 过渡点 ID |
| name | string | 过渡点名称 |
| transition_type | uint8 | 类型 (0=电梯, 1=楼梯, 2=坡道, 3=扶梯) |
| from_floor_id | string | 起始楼层 ID |
| from_pose | geometry_msgs/Pose2D | 起始位姿 |
| to_floor_id | string | 目标楼层 ID |
| to_pose | geometry_msgs/Pose2D | 目标位姿 |
| bidirectional | bool | 是否双向 |
| cost | float64 | 通过代价 |
| available | bool | 是否可用 |

---

### 9.2 复合地图管理服务

> **命名空间说明**: 以下所有服务路径均为无命名空间形式（`/xxx`）。使用命名空间时路径变为 `/<namespace>/xxx`。命令行示例中建议使用 `${MM}` 变量。

#### 9.2.1 /load_composite_map (服务)

> 加载复合地图（支持 .mmap 压缩包和目录格式）

**服务类型**: `map_manager/srv/LoadCompositeMap` | [详细文档](srv/load_composite_map.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| map_path | string | 地图路径（.mmap 文件或目录） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| map_info | CompositeMapInfo | 加载的地图信息 |

**调用示例**:
```bash
# 加载目录格式地图
ros2 service call ${MM}/load_composite_map map_manager/srv/LoadCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/office'}"

# 加载压缩包格式地图
ros2 service call ${MM}/load_composite_map map_manager/srv/LoadCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/office.mmap'}"
```

**Python调用**:
```python
from map_manager.srv import LoadCompositeMap
import rclpy
from rclpy.node import Node

class MapLoader(Node):
    def __init__(self):
        super().__init__('map_loader')
        self.client = self.create_client(LoadCompositeMap, '/load_composite_map')
        self.client.wait_for_service()

    def load(self, path: str):
        request = LoadCompositeMap.Request()
        request.map_path = path
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        if result.success:
            print(f"Loaded map: {result.map_info.name}")
            print(f"Floors: {[f.floor_id for f in result.map_info.floors]}")
        return result

rclpy.init()
loader = MapLoader()
loader.load('/opt/fftai/Navigation/Map/office')
```

#### 9.2.2 /save_composite_map (服务)

> 保存复合地图（支持压缩和目录格式）

**服务类型**: `map_manager/srv/SaveCompositeMap` | [详细文档](srv/save_composite_map.md)

**请求**:

| 字段 | 类型 | 默认值 | 说明 |
| ---- | ---- | ------ | ---- |
| map_path | string | - | 保存路径 |
| compress | bool | false | 是否压缩为 .mmap 格式 |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| saved_path | string | 实际保存路径（压缩时可能自动添加 .mmap 后缀） |

**调用示例**:
```bash
# 保存为目录格式
ros2 service call ${MM}/save_composite_map map_manager/srv/SaveCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/my_map', compress: false}"

# 保存为压缩包格式
ros2 service call ${MM}/save_composite_map map_manager/srv/SaveCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/my_map', compress: true}"

# 路径以 .mmap 结尾时自动压缩
ros2 service call ${MM}/save_composite_map map_manager/srv/SaveCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/my_map.mmap'}"
```

#### 9.2.3 /clear_composite_map (服务)

> 清除当前加载的复合地图

**服务类型**: `map_manager/srv/ClearCompositeMap`

---

### 9.3 楼层管理服务

#### 9.3.1 /list_floors (服务)

> 列出所有楼层

**服务类型**: `map_manager/srv/ListFloors` | [详细文档](srv/list_floors.md)

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floors | Floor[] | 楼层列表 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/list_floors map_manager/srv/ListFloors
```

**Python调用**:
```python
from map_manager.srv import ListFloors
import rclpy
from rclpy.node import Node

class FloorLister(Node):
    def __init__(self):
        super().__init__('floor_lister')
        self.client = self.create_client(ListFloors, '/list_floors')
        self.client.wait_for_service()
    
    def list_floors(self):
        request = ListFloors.Request()
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Total floors: {len(result.floors)}")
            for floor in result.floors:
                status = ['UNKNOWN', 'ACTIVE', 'INACTIVE'][floor.status]
                print(f"  {floor.floor_id}: {floor.name} (level {floor.level}, {status})")
                print(f"    Height: {floor.min_height:.2f} ~ {floor.max_height:.2f} m")
        return result

rclpy.init()
lister = FloorLister()
lister.list_floors()
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/list_floors.hpp>

class FloorLister : public rclcpp::Node {
public:
    FloorLister() : Node("floor_lister") {
        client_ = create_client<map_manager::srv::ListFloors>("/list_floors");
        client_->wait_for_service();
    }
    
    void list_floors() {
        auto request = std::make_shared<map_manager::srv::ListFloors::Request>();
        auto future = client_->async_send_request(request);
        
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Total floors: %zu", result->floors.size());
                for (const auto& floor : result->floors) {
                    const char* status[] = {"UNKNOWN", "ACTIVE", "INACTIVE"};
                    RCLCPP_INFO(get_logger(), "  %s: %s (level %d, %s)",
                        floor.floor_id.c_str(), floor.name.c_str(), 
                        floor.level, status[floor.status]);
                    RCLCPP_INFO(get_logger(), "    Height: %.2f ~ %.2f m",
                        floor.min_height, floor.max_height);
                }
            }
        }
    }

private:
    rclcpp::Client<map_manager::srv::ListFloors>::SharedPtr client_;
};
```

#### 9.3.2 /get_current_floor (服务)

> 获取当前楼层

> **注意**: 此服务接口已定义但当前版本未注册，请使用 [8.3.7 /current_floor 话题](#837-current_floor-话题) 订阅当前楼层信息。

**服务类型**: `map_manager/srv/GetCurrentFloor` | [详细文档](srv/get_current_floor.md)

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor | Floor | 当前楼层信息 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/get_current_floor map_manager/srv/GetCurrentFloor
```

**Python调用**:
```python
from map_manager.srv import GetCurrentFloor
import rclpy
from rclpy.node import Node

class CurrentFloorGetter(Node):
    def __init__(self):
        super().__init__('current_floor_getter')
        self.client = self.create_client(GetCurrentFloor, '/get_current_floor')
        self.client.wait_for_service()
    
    def get_current_floor(self):
        request = GetCurrentFloor.Request()
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            floor = result.floor
            print(f"Current floor: {floor.floor_id} ({floor.name})")
            print(f"  Level: {floor.level}")
            print(f"  Height range: {floor.min_height:.2f} ~ {floor.max_height:.2f} m")
        return result.floor

rclpy.init()
getter = CurrentFloorGetter()
current = getter.get_current_floor()
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/get_current_floor.hpp>

class CurrentFloorGetter : public rclcpp::Node {
public:
    CurrentFloorGetter() : Node("current_floor_getter") {
        client_ = create_client<map_manager::srv::GetCurrentFloor>(
            "/get_current_floor");
        client_->wait_for_service();
    }
    
    map_manager::msg::Floor get_current_floor() {
        auto request = std::make_shared<map_manager::srv::GetCurrentFloor::Request>();
        auto future = client_->async_send_request(request);
        
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Current floor: %s (%s), level %d",
                    result->floor.floor_id.c_str(), result->floor.name.c_str(),
                    result->floor.level);
                return result->floor;
            }
        }
        return map_manager::msg::Floor();
    }

private:
    rclcpp::Client<map_manager::srv::GetCurrentFloor>::SharedPtr client_;
};
```

#### 9.3.3 /switch_floor (服务)

> 切换到指定楼层

**服务类型**: `map_manager/srv/SwitchFloor` | [详细文档](srv/switch_floor.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 目标楼层 ID |
| initial_pose | geometry_msgs/Pose2D | 切换后的初始位姿 |
| use_transition | bool | 是否使用过渡点 |
| transition_id | string | 过渡点 ID（use_transition=true 时有效） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| current_floor | Floor | 切换后的楼层信息 |

**调用示例**:
```bash
# 直接切换楼层
ros2 service call ${MM}/switch_floor map_manager/srv/SwitchFloor \
  "{floor_id: 'F2', initial_pose: {x: 0.0, y: 0.0, theta: 0.0}, use_transition: false}"

# 使用电梯过渡点切换
ros2 service call ${MM}/switch_floor map_manager/srv/SwitchFloor \
  "{floor_id: 'F2', use_transition: true, transition_id: 'elevator_1'}"
```

#### 9.3.4 /add_floor (服务)

> 添加新楼层

**服务类型**: `map_manager/srv/AddFloor`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor | Floor | 楼层信息 |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| floor_id | string | 添加的楼层 ID |

**调用示例**:
```bash
ros2 service call ${MM}/add_floor map_manager/srv/AddFloor \
  "{floor: {floor_id: '2F', name: 'Second Floor', level: 2, min_height: 3.0, max_height: 6.0, reference_height: 3.0, status: 1}}"
```

**Python调用**:
```python
from map_manager.srv import AddFloor
from map_manager.msg import Floor
import rclpy
from rclpy.node import Node

class FloorManager(Node):
    def __init__(self):
        super().__init__('floor_manager')
        self.client = self.create_client(AddFloor, '/add_floor')
        self.client.wait_for_service()
    
    def add_floor(self, floor_id: str, name: str, level: int,
                  min_height: float = 0.0, max_height: float = 3.0):
        request = AddFloor.Request()
        request.floor = Floor()
        request.floor.floor_id = floor_id
        request.floor.name = name
        request.floor.level = level
        request.floor.min_height = min_height
        request.floor.max_height = max_height
        request.floor.reference_height = min_height
        request.floor.status = 1  # ACTIVE
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Floor added: {result.floor_id}")
        else:
            print(f"Failed: {result.message}")
        return result.success

rclpy.init()
manager = FloorManager()
manager.add_floor('2F', 'Second Floor', 2, 3.0, 6.0)
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_floor.hpp>
#include <map_manager/msg/floor.hpp>

class FloorManager : public rclcpp::Node {
public:
    FloorManager() : Node("floor_manager") {
        client_ = create_client<map_manager::srv::AddFloor>("/add_floor");
        client_->wait_for_service();
    }
    
    bool add_floor(const std::string& floor_id, const std::string& name, int32_t level,
                   double min_height = 0.0, double max_height = 3.0) {
        auto request = std::make_shared<map_manager::srv::AddFloor::Request>();
        request->floor.floor_id = floor_id;
        request->floor.name = name;
        request->floor.level = level;
        request->floor.min_height = min_height;
        request->floor.max_height = max_height;
        request->floor.reference_height = min_height;
        request->floor.status = 1;  // ACTIVE
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Floor added: %s", result->floor_id.c_str());
                return true;
            }
            RCLCPP_ERROR(get_logger(), "Failed: %s", result->message.c_str());
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddFloor>::SharedPtr client_;
};
```

#### 9.3.5 /remove_floor (服务)

> 删除楼层

**服务类型**: `map_manager/srv/RemoveFloor`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_floor map_manager/srv/RemoveFloor \
  "{floor_id: '2F'}"
```

**Python调用**:
```python
from map_manager.srv import RemoveFloor
import rclpy
from rclpy.node import Node

class FloorRemover(Node):
    def __init__(self):
        super().__init__('floor_remover')
        self.client = self.create_client(RemoveFloor, '/remove_floor')
        self.client.wait_for_service()
    
    def remove_floor(self, floor_id: str):
        request = RemoveFloor.Request()
        request.floor_id = floor_id
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        return result.success

rclpy.init()
remover = FloorRemover()
remover.remove_floor('2F')
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/remove_floor.hpp>

class FloorRemover : public rclcpp::Node {
public:
    FloorRemover() : Node("floor_remover") {
        client_ = create_client<map_manager::srv::RemoveFloor>("/remove_floor");
        client_->wait_for_service();
    }
    
    bool remove_floor(const std::string& floor_id) {
        auto request = std::make_shared<map_manager::srv::RemoveFloor::Request>();
        request->floor_id = floor_id;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            return future.get()->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::RemoveFloor>::SharedPtr client_;
};
```

#### 9.3.6 /save_floor (服务)

> 保存当前楼层数据

**服务类型**: `map_manager/srv/SaveFloor`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示当前楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| saved_path | string | 保存路径 |

**调用示例**:
```bash
# 保存当前楼层
ros2 service call ${MM}/save_floor map_manager/srv/SaveFloor "{floor_id: ''}"

# 保存指定楼层
ros2 service call ${MM}/save_floor map_manager/srv/SaveFloor "{floor_id: '1F'}"
```

**Python调用**:
```python
from map_manager.srv import SaveFloor
import rclpy
from rclpy.node import Node

class FloorSaver(Node):
    def __init__(self):
        super().__init__('floor_saver')
        self.client = self.create_client(SaveFloor, '/save_floor')
        self.client.wait_for_service()
    
    def save_floor(self, floor_id: str = ''):
        """保存楼层（空 floor_id 表示当前楼层）"""
        request = SaveFloor.Request()
        request.floor_id = floor_id
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future, timeout_sec=120.0)
        result = future.result()
        
        if result.success:
            print(f"Floor saved to: {result.saved_path}")
        else:
            print(f"Failed: {result.message}")
        return result.success

rclpy.init()
saver = FloorSaver()
saver.save_floor('1F')  # 保存 1F 楼层
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/save_floor.hpp>

class FloorSaver : public rclcpp::Node {
public:
    FloorSaver() : Node("floor_saver") {
        client_ = create_client<map_manager::srv::SaveFloor>("/save_floor");
        client_->wait_for_service();
    }
    
    bool save_floor(const std::string& floor_id = "") {
        auto request = std::make_shared<map_manager::srv::SaveFloor::Request>();
        request->floor_id = floor_id;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future, 
            std::chrono::seconds(120)) == rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Floor saved to: %s", 
                    result->saved_path.c_str());
                return true;
            }
            RCLCPP_ERROR(get_logger(), "Failed: %s", result->message.c_str());
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::SaveFloor>::SharedPtr client_;
};
```

#### 9.3.7 /load_floor (服务)

> 从磁盘加载楼层数据（包含导航、定位、语义、虚拟层）。若楼层已存在则替换。

**服务类型**: `map_manager/srv/LoadFloor`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 目标楼层 ID（加载后赋予此 ID） |
| input_path | string | 楼层数据所在目录路径 |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| floor_id | string | 实际加载的楼层 ID |

**调用示例**:
```bash
ros2 service call ${MM}/load_floor map_manager/srv/LoadFloor \
  "{floor_id: '1F', input_path: '/data/composite_map/floors/1F'}"
```

**Python调用**:
```python
from map_manager.srv import LoadFloor
import rclpy
from rclpy.node import Node

class FloorLoader(Node):
    def __init__(self):
        super().__init__('floor_loader')
        self.client = self.create_client(LoadFloor, '/load_floor')
        self.client.wait_for_service()

    def load_floor(self, floor_id: str, input_path: str):
        request = LoadFloor.Request()
        request.floor_id = floor_id
        request.input_path = input_path
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future, timeout_sec=60.0)
        result = future.result()
        if result.success:
            print(f"Floor loaded: {result.floor_id}")
        else:
            print(f"Failed: {result.message}")
        return result.success

rclpy.init()
loader = FloorLoader()
loader.load_floor('1F', '/data/composite_map/floors/1F')
```

#### 9.3.8 /clear_floor (服务)

> 清除指定楼层的所有层数据（保留楼层结构，仅清除数据，与 remove_floor 不同）

**服务类型**: `map_manager/srv/ClearFloor`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/clear_floor map_manager/srv/ClearFloor \
  "{floor_id: '1F'}"
```

**Python调用**:
```python
from map_manager.srv import ClearFloor
import rclpy
from rclpy.node import Node

class FloorClearer(Node):
    def __init__(self):
        super().__init__('floor_clearer')
        self.client = self.create_client(ClearFloor, '/clear_floor')
        self.client.wait_for_service()

    def clear_floor(self, floor_id: str):
        request = ClearFloor.Request()
        request.floor_id = floor_id
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        if result.success:
            print(f"Floor {floor_id} cleared")
        else:
            print(f"Failed: {result.message}")
        return result.success

rclpy.init()
clearer = FloorClearer()
clearer.clear_floor('1F')
```

#### 9.3.9 /current_floor (话题)

> 发布当前楼层 ID

**消息类型**: `std_msgs/msg/String`

**QoS**: transient_local, reliable

> **命名空间说明**: 使用命名空间时，话题路径变为 `/<namespace>/current_floor`（相对话题，自动跟随节点命名空间）。

---

### 9.4 POI 管理服务

#### 9.4.1 /add_poi (服务)

> 添加 POI

**服务类型**: `map_manager/srv/AddPOI` | [详细文档](srv/add_poi.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| poi | POI | POI 数据 |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/add_poi map_manager/srv/AddPOI \
  "{poi: {id: 'charging_1', name: 'Charging Station', type: 'charging_station', floor_id: 'F1', pose: {x: 5.0, y: 3.0, theta: 1.57}}}"
```

**Python调用**:
```python
from map_manager.srv import AddPOI
from map_manager.msg import POI
from geometry_msgs.msg import Pose2D
import rclpy
from rclpy.node import Node

class POIManager(Node):
    def __init__(self):
        super().__init__('poi_manager')
        self.client = self.create_client(AddPOI, '/add_poi')
        self.client.wait_for_service()
    
    def add_poi(self, poi_id: str, name: str, poi_type: str, 
                floor_id: str, x: float, y: float, theta: float = 0.0):
        request = AddPOI.Request()
        request.poi = POI()
        request.poi.id = poi_id
        request.poi.name = name
        request.poi.type = poi_type
        request.poi.floor_id = floor_id
        request.poi.pose = Pose2D(x=x, y=y, theta=theta)
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        return result.success

# 使用示例
rclpy.init()
manager = POIManager()
success = manager.add_poi('charging_1', 'Charging Station', 
                          'charging_station', 'F1', 5.0, 3.0, 1.57)
print(f"POI added: {success}")
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_poi.hpp>
#include <map_manager/msg/poi.hpp>

class POIManager : public rclcpp::Node {
public:
    POIManager() : Node("poi_manager") {
        client_ = create_client<map_manager::srv::AddPOI>("/add_poi");
        client_->wait_for_service();
    }
    
    bool add_poi(const std::string& id, const std::string& name,
                 const std::string& type, const std::string& floor_id,
                 double x, double y, double theta = 0.0) {
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
            auto result = future.get();
            RCLCPP_INFO(get_logger(), "%s", result->message.c_str());
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddPOI>::SharedPtr client_;
};
```

#### 9.4.2 /list_pois (服务)

> 列出 POI

**服务类型**: `map_manager/srv/ListPOIs` | [详细文档](srv/list_pois.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| pois | POI[] | POI 列表 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
# 列出所有楼层的 POI
ros2 service call ${MM}/list_pois map_manager/srv/ListPOIs "{floor_id: ''}"

# 列出 F1 楼层的 POI
ros2 service call ${MM}/list_pois map_manager/srv/ListPOIs "{floor_id: 'F1'}"
```

**Python调用**:
```python
from map_manager.srv import ListPOIs
import rclpy
from rclpy.node import Node

class POILister(Node):
    def __init__(self):
        super().__init__('poi_lister')
        self.client = self.create_client(ListPOIs, '/list_pois')
        self.client.wait_for_service()
    
    def list_pois(self, floor_id: str = ''):
        """列出 POI（空 floor_id 表示所有楼层）"""
        request = ListPOIs.Request()
        request.floor_id = floor_id
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Found {len(result.pois)} POIs")
            for poi in result.pois:
                print(f"  - {poi.id}: {poi.name} ({poi.type}) at "
                      f"({poi.pose.x:.2f}, {poi.pose.y:.2f}) on floor {poi.floor_id}")
        return result

# 使用示例
rclpy.init()
lister = POILister()
lister.list_pois('F1')  # 列出 F1 的 POI
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/list_po_is.hpp>

class POILister : public rclcpp::Node {
public:
    POILister() : Node("poi_lister") {
        client_ = create_client<map_manager::srv::ListPOIs>("/list_pois");
        client_->wait_for_service();
    }
    
    void list_pois(const std::string& floor_id = "") {
        auto request = std::make_shared<map_manager::srv::ListPOIs::Request>();
        request->floor_id = floor_id;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Found %zu POIs", result->pois.size());
                for (const auto& poi : result->pois) {
                    RCLCPP_INFO(get_logger(), "  - %s: %s (%s) at (%.2f, %.2f) on floor %s",
                        poi.id.c_str(), poi.name.c_str(), poi.type.c_str(),
                        poi.pose.x, poi.pose.y, poi.floor_id.c_str());
                }
            }
        }
    }

private:
    rclcpp::Client<map_manager::srv::ListPOIs>::SharedPtr client_;
};
```

#### 9.4.3 /get_poi (服务)

> 获取单个 POI

**服务类型**: `map_manager/srv/GetPOI`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| id | string | POI ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| poi | POI | POI 数据 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/get_poi map_manager/srv/GetPOI \
  "{floor_id: 'F1', id: 'charging_1'}"
```

#### 9.4.4 /update_poi (服务)

> 更新 POI

**服务类型**: `map_manager/srv/UpdatePOI`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| poi | POI | 更新后的 POI 数据（必须包含 id 和 floor_id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/update_poi map_manager/srv/UpdatePOI \
  "{poi: {id: 'charging_1', name: 'Main Charger', type: 'charging_station', floor_id: 'F1', pose: {x: 5.5, y: 3.0, theta: 1.57}}}"
```

#### 9.4.5 /remove_poi (服务)

> 删除 POI

**服务类型**: `map_manager/srv/RemovePOI`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| id | string | POI ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_poi map_manager/srv/RemovePOI \
  "{floor_id: 'F1', id: 'charging_1'}"
```

#### 9.4.6 /remove_all_pois (服务)

> 删除所有 POI

**服务类型**: `map_manager/srv/RemoveAllPOIs`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示删除所有楼层的 POI） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_all_pois map_manager/srv/RemoveAllPOIs \
  "{floor_id: 'F1'}"
```

---

### 9.5 虚拟墙管理服务

#### 9.5.1 /add_virtual_wall (服务)

> 添加虚拟墙

**服务类型**: `map_manager/srv/AddVirtualWall` | [详细文档](srv/add_virtual_wall.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |
| wall | VirtualWall | 虚拟墙数据 |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| wall_id | int32 | 添加的虚拟墙 ID |

**调用示例**:
```bash
ros2 service call ${MM}/add_virtual_wall map_manager/srv/AddVirtualWall \
  "{floor_id: 'F1', wall: {id: 1, start: {x: 0.0, y: 0.0, z: 0.0}, end: {x: 2.0, y: 0.0, z: 0.0}}}"
```

**Python调用**:
```python
from map_manager.srv import AddVirtualWall
from map_manager.msg import VirtualWall
from geometry_msgs.msg import Point
import rclpy
from rclpy.node import Node

class VirtualWallManager(Node):
    def __init__(self):
        super().__init__('virtual_wall_manager')
        self.client = self.create_client(AddVirtualWall, '/add_virtual_wall')
        self.client.wait_for_service()
    
    def add_wall(self, floor_id: str, wall_id: int, 
                 start_x: float, start_y: float, 
                 end_x: float, end_y: float):
        request = AddVirtualWall.Request()
        request.floor_id = floor_id
        request.wall = VirtualWall()
        request.wall.id = wall_id
        request.wall.floor_id = floor_id
        request.wall.start = Point(x=start_x, y=start_y, z=0.0)
        request.wall.end = Point(x=end_x, y=end_y, z=0.0)
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Virtual wall added with ID: {result.wall_id}")
        else:
            print(f"Failed: {result.message}")
        return result.success

# 使用示例
rclpy.init()
manager = VirtualWallManager()
# 添加一条从 (0,0) 到 (5,0) 的虚拟墙
manager.add_wall('F1', 1, 0.0, 0.0, 5.0, 0.0)
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_virtual_wall.hpp>
#include <map_manager/msg/virtual_wall.hpp>

class VirtualWallManager : public rclcpp::Node {
public:
    VirtualWallManager() : Node("virtual_wall_manager") {
        client_ = create_client<map_manager::srv::AddVirtualWall>(
            "/add_virtual_wall");
        client_->wait_for_service();
    }
    
    bool add_wall(const std::string& floor_id, int32_t wall_id,
                  double start_x, double start_y, 
                  double end_x, double end_y) {
        auto request = std::make_shared<map_manager::srv::AddVirtualWall::Request>();
        request->floor_id = floor_id;
        request->wall.id = wall_id;
        request->wall.floor_id = floor_id;
        request->wall.start.x = start_x;
        request->wall.start.y = start_y;
        request->wall.start.z = 0.0;
        request->wall.end.x = end_x;
        request->wall.end.y = end_y;
        request->wall.end.z = 0.0;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Virtual wall added with ID: %d", 
                    result->wall_id);
                return true;
            }
            RCLCPP_ERROR(get_logger(), "Failed: %s", result->message.c_str());
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddVirtualWall>::SharedPtr client_;
};
```

#### 9.5.2 /list_virtual_walls (服务)

> 列出虚拟墙

**服务类型**: `map_manager/srv/ListVirtualWalls` | [详细文档](srv/list_virtual_walls.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| walls | VirtualWall[] | 虚拟墙列表 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
# 列出所有楼层的虚拟墙
ros2 service call ${MM}/list_virtual_walls map_manager/srv/ListVirtualWalls "{floor_id: ''}"

# 列出 F1 楼层的虚拟墙
ros2 service call ${MM}/list_virtual_walls map_manager/srv/ListVirtualWalls "{floor_id: 'F1'}"
```

**Python调用**:
```python
from map_manager.srv import ListVirtualWalls
import rclpy
from rclpy.node import Node

class VirtualWallLister(Node):
    def __init__(self):
        super().__init__('virtual_wall_lister')
        self.client = self.create_client(ListVirtualWalls, '/list_virtual_walls')
        self.client.wait_for_service()
    
    def list_walls(self, floor_id: str = ''):
        request = ListVirtualWalls.Request()
        request.floor_id = floor_id
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Found {len(result.walls)} virtual walls")
            for wall in result.walls:
                print(f"  Wall {wall.id}: ({wall.start.x:.2f}, {wall.start.y:.2f}) -> "
                      f"({wall.end.x:.2f}, {wall.end.y:.2f})")
        return result

rclpy.init()
lister = VirtualWallLister()
lister.list_walls('F1')
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/list_virtual_walls.hpp>

class VirtualWallLister : public rclcpp::Node {
public:
    VirtualWallLister() : Node("virtual_wall_lister") {
        client_ = create_client<map_manager::srv::ListVirtualWalls>(
            "/list_virtual_walls");
        client_->wait_for_service();
    }
    
    void list_walls(const std::string& floor_id = "") {
        auto request = std::make_shared<map_manager::srv::ListVirtualWalls::Request>();
        request->floor_id = floor_id;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Found %zu virtual walls", result->walls.size());
                for (const auto& wall : result->walls) {
                    RCLCPP_INFO(get_logger(), 
                        "  Wall %d: (%.2f, %.2f) -> (%.2f, %.2f)",
                        wall.id, wall.start.x, wall.start.y, wall.end.x, wall.end.y);
                }
            }
        }
    }

private:
    rclcpp::Client<map_manager::srv::ListVirtualWalls>::SharedPtr client_;
};
```

#### 9.5.3 /get_virtual_wall (服务)

> 获取单条虚拟墙

**服务类型**: `map_manager/srv/GetVirtualWall`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| id | int32 | 虚拟墙 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| wall | VirtualWall | 虚拟墙数据 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/get_virtual_wall map_manager/srv/GetVirtualWall \
  "{floor_id: 'F1', id: 1}"
```

#### 9.5.4 /update_virtual_wall (服务)

> 更新虚拟墙

**服务类型**: `map_manager/srv/UpdateVirtualWall`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| wall | VirtualWall | 更新后的虚拟墙数据（必须包含 id 和 floor_id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/update_virtual_wall map_manager/srv/UpdateVirtualWall \
  "{wall: {id: 1, floor_id: 'F1', start: {x: 0.0, y: 0.0, z: 0.0}, end: {x: 3.0, y: 0.0, z: 0.0}}}"
```

#### 9.5.5 /remove_virtual_wall (服务)

> 删除单条虚拟墙

**服务类型**: `map_manager/srv/RemoveVirtualWall`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| wall_id | int32 | 虚拟墙 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_virtual_wall map_manager/srv/RemoveVirtualWall \
  "{floor_id: 'F1', wall_id: 1}"
```

#### 9.5.6 /remove_virtual_walls (服务)

> 批量删除虚拟墙（删除指定楼层所有虚拟墙）

**服务类型**: `map_manager/srv/RemoveVirtualWalls`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示删除所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| count | int32 | 删除的虚拟墙数量 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_virtual_walls map_manager/srv/RemoveVirtualWalls \
  "{floor_id: 'F1'}"
```

---

### 9.6 禁区管理服务

#### 9.6.1 /add_forbidden_area (服务)

> 添加禁区（多边形区域）

**服务类型**: `map_manager/srv/AddForbiddenArea` | [详细文档](srv/add_forbidden_area.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |
| area | ForbiddenArea | 禁区数据（多边形边界） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| area_id | int32 | 禁区 ID |
| wall_ids | int32[] | 自动生成的虚拟墙 ID 列表 |

**调用示例**:
```bash
# 添加正方形禁区（2m × 2m）
ros2 service call ${MM}/add_forbidden_area map_manager/srv/AddForbiddenArea \
  "{floor_id: 'F1', area: {id: 1, boundary: {points: [{x: 0.0, y: 0.0, z: 0.0}, {x: 2.0, y: 0.0, z: 0.0}, {x: 2.0, y: 2.0, z: 0.0}, {x: 0.0, y: 2.0, z: 0.0}]}}}"
```

**Python调用**:
```python
from map_manager.srv import AddForbiddenArea
from map_manager.msg import ForbiddenArea
from geometry_msgs.msg import Polygon, Point32
import rclpy
from rclpy.node import Node

class ForbiddenAreaManager(Node):
    def __init__(self):
        super().__init__('forbidden_area_manager')
        self.client = self.create_client(AddForbiddenArea, '/add_forbidden_area')
        self.client.wait_for_service()
    
    def add_forbidden_area(self, floor_id: str, area_id: int, points: list):
        """
        添加禁区
        points: 多边形顶点列表，如 [(x1, y1), (x2, y2), (x3, y3), ...]
        """
        request = AddForbiddenArea.Request()
        request.floor_id = floor_id
        request.area = ForbiddenArea()
        request.area.id = area_id
        request.area.boundary = Polygon()
        request.area.boundary.points = [
            Point32(x=float(p[0]), y=float(p[1]), z=0.0) for p in points
        ]
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Forbidden area added with ID: {result.area_id}")
            print(f"Generated virtual walls: {result.wall_ids}")
        else:
            print(f"Failed: {result.message}")
        return result.success

# 使用示例
rclpy.init()
manager = ForbiddenAreaManager()
# 添加正方形禁区
manager.add_forbidden_area('F1', 1, [(0.0, 0.0), (2.0, 0.0), (2.0, 2.0), (0.0, 2.0)])
# 添加三角形禁区
manager.add_forbidden_area('F1', 2, [(5.0, 5.0), (7.0, 5.0), (6.0, 7.0)])
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_forbidden_area.hpp>
#include <map_manager/msg/forbidden_area.hpp>

class ForbiddenAreaManager : public rclcpp::Node {
public:
    ForbiddenAreaManager() : Node("forbidden_area_manager") {
        client_ = create_client<map_manager::srv::AddForbiddenArea>(
            "/add_forbidden_area");
        client_->wait_for_service();
    }
    
    bool add_forbidden_area(const std::string& floor_id, int32_t area_id,
                           const std::vector<std::pair<double, double>>& points) {
        auto request = std::make_shared<map_manager::srv::AddForbiddenArea::Request>();
        request->floor_id = floor_id;
        request->area.id = area_id;
        
        // 构造多边形边界
        for (const auto& p : points) {
            geometry_msgs::msg::Point32 point;
            point.x = p.first;
            point.y = p.second;
            point.z = 0.0;
            request->area.boundary.points.push_back(point);
        }
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Forbidden area added with ID: %d", 
                    result->area_id);
                RCLCPP_INFO(get_logger(), "Generated %zu virtual walls", 
                    result->wall_ids.size());
                return true;
            }
            RCLCPP_ERROR(get_logger(), "Failed: %s", result->message.c_str());
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddForbiddenArea>::SharedPtr client_;
};

// 使用示例
int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto manager = std::make_shared<ForbiddenAreaManager>();
    // 添加正方形禁区
    manager->add_forbidden_area("F1", 1, {{0.0, 0.0}, {2.0, 0.0}, {2.0, 2.0}, {0.0, 2.0}});
    rclcpp::shutdown();
    return 0;
}
```

#### 9.6.2 /list_forbidden_areas (服务)

> 列出禁区

**服务类型**: `map_manager/srv/ListForbiddenAreas` | [详细文档](srv/list_forbidden_areas.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| areas | ForbiddenArea[] | 禁区列表 |
| floor_ids | string[] | 各禁区对应的楼层 ID |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/list_forbidden_areas map_manager/srv/ListForbiddenAreas \
  "{floor_id: 'F1'}"
```

**Python调用**:
```python
from map_manager.srv import ListForbiddenAreas
import rclpy
from rclpy.node import Node

class ForbiddenAreaLister(Node):
    def __init__(self):
        super().__init__('forbidden_area_lister')
        self.client = self.create_client(ListForbiddenAreas, '/list_forbidden_areas')
        self.client.wait_for_service()
    
    def list_areas(self, floor_id: str = ''):
        request = ListForbiddenAreas.Request()
        request.floor_id = floor_id
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Found {len(result.areas)} forbidden areas")
            for i, area in enumerate(result.areas):
                floor = result.floor_ids[i] if i < len(result.floor_ids) else 'unknown'
                print(f"  Area {area.id} on floor {floor}:")
                print(f"    Vertices: {len(area.boundary.points)}")
                for j, point in enumerate(area.boundary.points):
                    print(f"      {j}: ({point.x:.2f}, {point.y:.2f})")
        return result

rclpy.init()
lister = ForbiddenAreaLister()
lister.list_areas('F1')
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/list_forbidden_areas.hpp>

class ForbiddenAreaLister : public rclcpp::Node {
public:
    ForbiddenAreaLister() : Node("forbidden_area_lister") {
        client_ = create_client<map_manager::srv::ListForbiddenAreas>(
            "/list_forbidden_areas");
        client_->wait_for_service();
    }
    
    void list_areas(const std::string& floor_id = "") {
        auto request = std::make_shared<map_manager::srv::ListForbiddenAreas::Request>();
        request->floor_id = floor_id;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Found %zu forbidden areas", result->areas.size());
                for (size_t i = 0; i < result->areas.size(); ++i) {
                    const auto& area = result->areas[i];
                    const std::string& floor = (i < result->floor_ids.size()) ? 
                        result->floor_ids[i] : "unknown";
                    RCLCPP_INFO(get_logger(), "  Area %d on floor %s: %zu vertices",
                        area.id, floor.c_str(), area.boundary.points.size());
                }
            }
        }
    }

private:
    rclcpp::Client<map_manager::srv::ListForbiddenAreas>::SharedPtr client_;
};
```

#### 9.6.3 /get_forbidden_area (服务)

> 获取单个禁区

**服务类型**: `map_manager/srv/GetForbiddenArea`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| id | int32 | 禁区 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| area | ForbiddenArea | 禁区数据 |
| area_floor_id | string | 禁区所在楼层 ID |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/get_forbidden_area map_manager/srv/GetForbiddenArea \
  "{floor_id: 'F1', id: 1}"
```

#### 9.6.4 /update_forbidden_area (服务)

> 更新禁区

**服务类型**: `map_manager/srv/UpdateForbiddenArea`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（必需） |
| area | ForbiddenArea | 更新后的禁区数据（必须包含 id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| wall_ids | int32[] | 重新生成的虚拟墙 ID 列表 |

**调用示例**:
```bash
ros2 service call ${MM}/update_forbidden_area map_manager/srv/UpdateForbiddenArea \
  "{floor_id: 'F1', area: {id: 1, boundary: {points: [{x: 0.0, y: 0.0, z: 0.0}, {x: 3.0, y: 0.0, z: 0.0}, {x: 3.0, y: 3.0, z: 0.0}, {x: 0.0, y: 3.0, z: 0.0}]}}}"
```

#### 9.6.5 /remove_forbidden_area (服务)

> 删除禁区

**服务类型**: `map_manager/srv/RemoveForbiddenArea`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| area_id | int32 | 禁区 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| walls_removed | int32 | 删除的虚拟墙数量 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_forbidden_area map_manager/srv/RemoveForbiddenArea \
  "{floor_id: 'F1', area_id: 1}"
```

#### 9.6.6 /remove_forbidden_areas (服务)

> 批量删除禁区（删除指定楼层所有禁区）

> **注意**: 此服务接口已定义但当前版本未注册。请使用 [9.6.7 /remove_all_forbidden_areas](#967-remove_all_forbidden_areas-服务) 代替，功能相同。

**服务类型**: `map_manager/srv/RemoveForbiddenAreas`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示删除所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| count | int32 | 删除的禁区数量 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_forbidden_areas map_manager/srv/RemoveForbiddenAreas \
  "{floor_id: 'F1'}"
```

#### 9.6.7 /remove_all_forbidden_areas (服务)

> 删除所有禁区（与 remove_forbidden_areas 功能相同）

**服务类型**: `map_manager/srv/RemoveAllForbiddenAreas`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| count | int32 | 删除的禁区数量 |

---

### 9.7 限速区管理服务

#### 9.7.1 /add_dangerous_area (服务)

> 添加限速区

**服务类型**: `map_manager/srv/AddDangerousArea` | [详细文档](srv/add_dangerous_area.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |
| area | DangerousArea | 限速区数据（包含速度限制） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| area_id | int32 | 限速区 ID |

**调用示例**:
```bash
# 添加限速区（3m × 3m，限速 0.3 m/s）
ros2 service call ${MM}/add_dangerous_area map_manager/srv/AddDangerousArea \
  "{floor_id: 'F1', area: {id: 1, boundary: {points: [{x: 0.0, y: 0.0, z: 0.0}, {x: 3.0, y: 0.0, z: 0.0}, {x: 3.0, y: 3.0, z: 0.0}, {x: 0.0, y: 3.0, z: 0.0}]}, speed_limit: 0.3}}"
```

**Python调用**:
```python
from map_manager.srv import AddDangerousArea
from map_manager.msg import DangerousArea
from geometry_msgs.msg import Polygon, Point32
import rclpy
from rclpy.node import Node

class DangerousAreaManager(Node):
    def __init__(self):
        super().__init__('dangerous_area_manager')
        self.client = self.create_client(AddDangerousArea, '/add_dangerous_area')
        self.client.wait_for_service()
    
    def add_dangerous_area(self, floor_id: str, area_id: int, 
                          points: list, speed_limit: float):
        """
        添加限速区
        points: 多边形顶点列表，如 [(x1, y1), (x2, y2), ...]
        speed_limit: 速度限制（米/秒），如 0.3 表示 0.3 m/s
        """
        request = AddDangerousArea.Request()
        request.floor_id = floor_id
        request.area = DangerousArea()
        request.area.id = area_id
        request.area.boundary = Polygon()
        request.area.boundary.points = [
            Point32(x=float(p[0]), y=float(p[1]), z=0.0) for p in points
        ]
        request.area.speed_limit = speed_limit
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Dangerous area added with ID: {result.area_id}")
            print(f"Speed limit: {speed_limit} m/s")
        else:
            print(f"Failed: {result.message}")
        return result.success

# 使用示例
rclpy.init()
manager = DangerousAreaManager()
# 添加电梯附近限速区（限速 0.2 m/s）
manager.add_dangerous_area('F1', 1, 
    [(5.0, 5.0), (8.0, 5.0), (8.0, 8.0), (5.0, 8.0)], 0.2)
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_dangerous_area.hpp>
#include <map_manager/msg/dangerous_area.hpp>

class DangerousAreaManager : public rclcpp::Node {
public:
    DangerousAreaManager() : Node("dangerous_area_manager") {
        client_ = create_client<map_manager::srv::AddDangerousArea>(
            "/add_dangerous_area");
        client_->wait_for_service();
    }
    
    bool add_dangerous_area(const std::string& floor_id, int32_t area_id,
                           const std::vector<std::pair<double, double>>& points,
                           float speed_limit) {
        auto request = std::make_shared<map_manager::srv::AddDangerousArea::Request>();
        request->floor_id = floor_id;
        request->area.id = area_id;
        request->area.speed_limit = speed_limit;
        
        // 构造多边形边界
        for (const auto& p : points) {
            geometry_msgs::msg::Point32 point;
            point.x = p.first;
            point.y = p.second;
            point.z = 0.0;
            request->area.boundary.points.push_back(point);
        }
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Dangerous area added with ID: %d, "
                    "speed limit: %.2f m/s", result->area_id, speed_limit);
                return true;
            }
            RCLCPP_ERROR(get_logger(), "Failed: %s", result->message.c_str());
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddDangerousArea>::SharedPtr client_;
};

// 使用示例
int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto manager = std::make_shared<DangerousAreaManager>();
    // 添加电梯附近限速区
    manager->add_dangerous_area("F1", 1, 
        {{5.0, 5.0}, {8.0, 5.0}, {8.0, 8.0}, {5.0, 8.0}}, 0.2f);
    rclcpp::shutdown();
    return 0;
}
```

#### 9.7.2 /list_dangerous_areas (服务)

> 列出限速区

**服务类型**: `map_manager/srv/ListDangerousAreas` | [详细文档](srv/list_dangerous_areas.md)

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| areas | DangerousArea[] | 限速区列表 |
| floor_ids | string[] | 各限速区对应的楼层 ID |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/list_dangerous_areas map_manager/srv/ListDangerousAreas \
  "{floor_id: 'F1'}"
```

**Python调用**:
```python
from map_manager.srv import ListDangerousAreas
import rclpy
from rclpy.node import Node

class DangerousAreaLister(Node):
    def __init__(self):
        super().__init__('dangerous_area_lister')
        self.client = self.create_client(ListDangerousAreas, '/list_dangerous_areas')
        self.client.wait_for_service()
    
    def list_areas(self, floor_id: str = ''):
        request = ListDangerousAreas.Request()
        request.floor_id = floor_id
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Found {len(result.areas)} dangerous areas")
            for i, area in enumerate(result.areas):
                floor = result.floor_ids[i] if i < len(result.floor_ids) else 'unknown'
                print(f"  Area {area.id} on floor {floor}:")
                print(f"    Speed limit: {area.speed_limit} m/s")
                print(f"    Vertices: {len(area.boundary.points)}")
        return result

rclpy.init()
lister = DangerousAreaLister()
lister.list_areas('F1')
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/list_dangerous_areas.hpp>

class DangerousAreaLister : public rclcpp::Node {
public:
    DangerousAreaLister() : Node("dangerous_area_lister") {
        client_ = create_client<map_manager::srv::ListDangerousAreas>(
            "/list_dangerous_areas");
        client_->wait_for_service();
    }
    
    void list_areas(const std::string& floor_id = "") {
        auto request = std::make_shared<map_manager::srv::ListDangerousAreas::Request>();
        request->floor_id = floor_id;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Found %zu dangerous areas", result->areas.size());
                for (size_t i = 0; i < result->areas.size(); ++i) {
                    const auto& area = result->areas[i];
                    const std::string& floor = (i < result->floor_ids.size()) ? 
                        result->floor_ids[i] : "unknown";
                    RCLCPP_INFO(get_logger(), 
                        "  Area %d on floor %s: speed_limit=%.2f m/s, %zu vertices",
                        area.id, floor.c_str(), area.speed_limit, 
                        area.boundary.points.size());
                }
            }
        }
    }

private:
    rclcpp::Client<map_manager::srv::ListDangerousAreas>::SharedPtr client_;
};
```

#### 9.7.3 /get_dangerous_area (服务)

> 获取单个限速区

**服务类型**: `map_manager/srv/GetDangerousArea`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| id | int32 | 限速区 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| area | DangerousArea | 限速区数据 |
| area_floor_id | string | 限速区所在楼层 ID |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/get_dangerous_area map_manager/srv/GetDangerousArea \
  "{floor_id: 'F1', id: 1}"
```

#### 9.7.4 /update_dangerous_area (服务)

> 更新限速区

**服务类型**: `map_manager/srv/UpdateDangerousArea`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（必需） |
| area | DangerousArea | 更新后的限速区数据（必须包含 id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/update_dangerous_area map_manager/srv/UpdateDangerousArea \
  "{floor_id: 'F1', area: {id: 1, boundary: {points: [{x: 0.0, y: 0.0, z: 0.0}, {x: 4.0, y: 0.0, z: 0.0}, {x: 4.0, y: 4.0, z: 0.0}, {x: 0.0, y: 4.0, z: 0.0}]}, speed_limit: 0.5}}"
```

#### 9.7.5 /remove_dangerous_area (服务)

> 删除限速区

**服务类型**: `map_manager/srv/RemoveDangerousArea`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| area_id | int32 | 限速区 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_dangerous_area map_manager/srv/RemoveDangerousArea \
  "{floor_id: 'F1', area_id: 1}"
```

#### 9.7.6 /remove_all_dangerous_areas (服务)

> 删除所有限速区

**服务类型**: `map_manager/srv/RemoveAllDangerousAreas`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| count | int32 | 删除的限速区数量 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_all_dangerous_areas map_manager/srv/RemoveAllDangerousAreas \
  "{floor_id: 'F1'}"
```

---

### 9.8 房间管理服务

#### 9.8.1 /add_room (服务)

> 添加房间

**服务类型**: `map_manager/srv/AddRoom`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |
| room | Room | 房间数据（必须包含 room.id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| room_id | string | 添加的房间 ID |

**调用示例**:
```bash
ros2 service call ${MM}/add_room map_manager/srv/AddRoom \
  "{floor_id: 'F1', room: {id: 'room_101', name: 'Office 101', type: 'office', boundary: [{x: 0.0, y: 0.0, z: 0.0}, {x: 5.0, y: 0.0, z: 0.0}, {x: 5.0, y: 4.0, z: 0.0}, {x: 0.0, y: 4.0, z: 0.0}], floor_height: 0.0, ceiling_height: 2.8}}"
```

**Python调用**:
```python
from map_manager.srv import AddRoom
from map_manager.msg import Room
from geometry_msgs.msg import Point
import rclpy
from rclpy.node import Node

class RoomManager(Node):
    def __init__(self):
        super().__init__('room_manager')
        self.client = self.create_client(AddRoom, '/add_room')
        self.client.wait_for_service()
    
    def add_room(self, floor_id: str, room_id: str, name: str, 
                 room_type: str, boundary_points: list,
                 floor_height: float = 0.0, ceiling_height: float = 2.8):
        """
        添加房间
        boundary_points: 房间边界点列表，如 [(x1, y1), (x2, y2), ...]
        """
        request = AddRoom.Request()
        request.floor_id = floor_id
        request.room = Room()
        request.room.id = room_id
        request.room.name = name
        request.room.type = room_type
        request.room.boundary = [
            Point(x=float(p[0]), y=float(p[1]), z=0.0) for p in boundary_points
        ]
        request.room.floor_height = floor_height
        request.room.ceiling_height = ceiling_height
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Room added: {result.room_id}")
        return result.success

rclpy.init()
manager = RoomManager()
# 添加办公室（5m × 4m）
manager.add_room('F1', 'room_101', 'Office 101', 'office',
                 [(0.0, 0.0), (5.0, 0.0), (5.0, 4.0), (0.0, 4.0)])
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_room.hpp>
#include <map_manager/msg/room.hpp>

class RoomManager : public rclcpp::Node {
public:
    RoomManager() : Node("room_manager") {
        client_ = create_client<map_manager::srv::AddRoom>("/add_room");
        client_->wait_for_service();
    }
    
    bool add_room(const std::string& floor_id, const std::string& room_id,
                  const std::string& name, const std::string& type,
                  const std::vector<std::pair<double, double>>& boundary_points,
                  double floor_height = 0.0, double ceiling_height = 2.8) {
        auto request = std::make_shared<map_manager::srv::AddRoom::Request>();
        request->floor_id = floor_id;
        request->room.id = room_id;
        request->room.name = name;
        request->room.type = type;
        request->room.floor_height = floor_height;
        request->room.ceiling_height = ceiling_height;
        
        for (const auto& p : boundary_points) {
            geometry_msgs::msg::Point point;
            point.x = p.first;
            point.y = p.second;
            point.z = 0.0;
            request->room.boundary.push_back(point);
        }
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Room added: %s", result->room_id.c_str());
                return true;
            }
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddRoom>::SharedPtr client_;
};
```

#### 9.8.2 /list_rooms (服务)

> 列出房间

**服务类型**: `map_manager/srv/ListRooms`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| rooms | Room[] | 房间列表 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/list_rooms map_manager/srv/ListRooms \
  "{floor_id: 'F1'}"
```

#### 9.8.3 /get_room (服务)

> 获取房间

**服务类型**: `map_manager/srv/GetRoom`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| id | string | 房间 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| room | Room | 房间数据 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/get_room map_manager/srv/GetRoom \
  "{floor_id: 'F1', id: 'room_101'}"
```

#### 9.8.4 /update_room (服务)

> 更新房间

**服务类型**: `map_manager/srv/UpdateRoom`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |
| room | Room | 更新后的房间数据（必须包含 id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/update_room map_manager/srv/UpdateRoom \
  "{floor_id: 'F1', room: {id: 'room_101', name: 'Meeting Room 101', type: 'meeting_room', boundary: [{x: 0.0, y: 0.0, z: 0.0}, {x: 6.0, y: 0.0, z: 0.0}, {x: 6.0, y: 5.0, z: 0.0}, {x: 0.0, y: 5.0, z: 0.0}]}}"
```

#### 9.8.5 /remove_room (服务)

> 删除房间

**服务类型**: `map_manager/srv/RemoveRoom`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| room_id | string | 房间 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| objects_removed | int32 | 从房间中删除的对象数量 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_room map_manager/srv/RemoveRoom \
  "{floor_id: 'F1', room_id: 'room_101'}"
```

#### 9.8.6 /remove_all_rooms (服务)

> 删除所有房间

**服务类型**: `map_manager/srv/RemoveAllRooms`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| count | int32 | 删除的房间数量 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_all_rooms map_manager/srv/RemoveAllRooms \
  "{floor_id: 'F1'}"
```

---

### 9.9 语义对象管理服务

#### 9.9.1 /add_semantic_object (服务)

> 添加语义对象

**服务类型**: `map_manager/srv/AddSemanticObject`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |
| object | SemanticObject | 语义对象数据（必须包含 object.id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| object_id | string | 添加的对象 ID |

**调用示例**:
```bash
ros2 service call ${MM}/add_semantic_object map_manager/srv/AddSemanticObject \
  "{floor_id: 'F1', object: {id: 'desk_001', name: 'Office Desk', type: 'desk', category: 'furniture', pose: {position: {x: 2.0, y: 3.0, z: 0.0}, orientation: {w: 1.0}}, dimensions: {x: 1.4, y: 0.7, z: 0.75}, is_static: true}}"
```

**Python调用**:
```python
from map_manager.srv import AddSemanticObject
from map_manager.msg import SemanticObject
from geometry_msgs.msg import Pose, Vector3
import rclpy
from rclpy.node import Node

class SemanticObjectManager(Node):
    def __init__(self):
        super().__init__('semantic_object_manager')
        self.client = self.create_client(AddSemanticObject, '/add_semantic_object')
        self.client.wait_for_service()
    
    def add_object(self, floor_id: str, obj_id: str, name: str,
                   obj_type: str, category: str, x: float, y: float, z: float,
                   length: float, width: float, height: float, is_static: bool = True):
        request = AddSemanticObject.Request()
        request.floor_id = floor_id
        request.object = SemanticObject()
        request.object.id = obj_id
        request.object.name = name
        request.object.type = obj_type
        request.object.category = category
        request.object.pose = Pose()
        request.object.pose.position.x = x
        request.object.pose.position.y = y
        request.object.pose.position.z = z
        request.object.pose.orientation.w = 1.0
        request.object.dimensions = Vector3(x=length, y=width, z=height)
        request.object.is_static = is_static
        
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()
        
        if result.success:
            print(f"Semantic object added: {result.object_id}")
        return result.success

rclpy.init()
manager = SemanticObjectManager()
# 添加办公桌（1.4m × 0.7m × 0.75m）
manager.add_object('F1', 'desk_001', 'Office Desk', 'desk', 'furniture',
                   2.0, 3.0, 0.0, 1.4, 0.7, 0.75, True)
```

**C++调用**:
```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/add_semantic_object.hpp>
#include <map_manager/msg/semantic_object.hpp>

class SemanticObjectManager : public rclcpp::Node {
public:
    SemanticObjectManager() : Node("semantic_object_manager") {
        client_ = create_client<map_manager::srv::AddSemanticObject>(
            "/add_semantic_object");
        client_->wait_for_service();
    }
    
    bool add_object(const std::string& floor_id, const std::string& obj_id,
                    const std::string& name, const std::string& type,
                    const std::string& category, double x, double y, double z,
                    double length, double width, double height, bool is_static = true) {
        auto request = std::make_shared<map_manager::srv::AddSemanticObject::Request>();
        request->floor_id = floor_id;
        request->object.id = obj_id;
        request->object.name = name;
        request->object.type = type;
        request->object.category = category;
        request->object.pose.position.x = x;
        request->object.pose.position.y = y;
        request->object.pose.position.z = z;
        request->object.pose.orientation.w = 1.0;
        request->object.dimensions.x = length;
        request->object.dimensions.y = width;
        request->object.dimensions.z = height;
        request->object.is_static = is_static;
        
        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "Semantic object added: %s", 
                    result->object_id.c_str());
                return true;
            }
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::AddSemanticObject>::SharedPtr client_;
};
```

#### 9.9.2 /list_semantic_objects (服务)

> 列出语义对象

**服务类型**: `map_manager/srv/ListSemanticObjects`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |
| object_type | string | 对象类型过滤（空表示所有类型） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| objects | SemanticObject[] | 语义对象列表 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/list_semantic_objects map_manager/srv/ListSemanticObjects \
  "{floor_id: 'F1', object_type: 'desk'}"
```

#### 9.9.3 /get_semantic_object (服务)

> 获取语义对象

**服务类型**: `map_manager/srv/GetSemanticObject`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| id | string | 对象 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| object | SemanticObject | 语义对象数据 |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/get_semantic_object map_manager/srv/GetSemanticObject \
  "{floor_id: 'F1', id: 'desk_001'}"
```

#### 9.9.4 /update_semantic_object (服务)

> 更新语义对象

**服务类型**: `map_manager/srv/UpdateSemanticObject`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID |
| object | SemanticObject | 更新后的对象数据（必须包含 id） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/update_semantic_object map_manager/srv/UpdateSemanticObject \
  "{floor_id: 'F1', object: {id: 'desk_001', name: 'Executive Desk', type: 'desk', category: 'furniture', pose: {position: {x: 3.0, y: 4.0, z: 0.0}, orientation: {w: 1.0}}, dimensions: {x: 1.6, y: 0.8, z: 0.75}, is_static: true}}"
```

#### 9.9.5 /remove_semantic_object (服务)

> 删除语义对象

**服务类型**: `map_manager/srv/RemoveSemanticObject`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示搜索所有楼层） |
| object_id | string | 对象 ID |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_semantic_object map_manager/srv/RemoveSemanticObject \
  "{floor_id: 'F1', object_id: 'desk_001'}"
```

#### 9.9.6 /remove_all_semantic_objects (服务)

> 删除所有语义对象

**服务类型**: `map_manager/srv/RemoveAllSemanticObjects`

**请求**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| floor_id | string | 楼层 ID（空表示所有楼层） |
| object_type | string | 对象类型过滤（空表示所有类型） |

**响应**:

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| success | bool | 是否成功 |
| message | string | 结果消息 |
| count | int32 | 删除的对象数量 |

**调用示例**:
```bash
ros2 service call ${MM}/remove_all_semantic_objects map_manager/srv/RemoveAllSemanticObjects \
  "{floor_id: 'F1', object_type: ''}"
```

---

### 9.10 Python 完整示例

```python
from map_manager.srv import (
    LoadCompositeMap, SaveCompositeMap, ListFloors, SwitchFloor,
    AddPOI, ListPOIs, AddVirtualWall, AddForbiddenArea, AddDangerousArea
)
from map_manager.msg import POI, VirtualWall, ForbiddenArea, DangerousArea
from geometry_msgs.msg import Point, Pose2D, Polygon, Point32
import rclpy
from rclpy.node import Node

class MapManagerClient(Node):
    def __init__(self, namespace: str = ''):
        super().__init__('map_manager_client')

        # 根据命名空间构建服务前缀
        # namespace='' → ''（服务在根路径：/load_composite_map）
        # namespace='robot1' → '/robot1'（服务在：/robot1/load_composite_map）
        prefix = f'/{namespace}' if namespace else ''

        # 创建服务客户端
        self.load_client = self.create_client(LoadCompositeMap, f'{prefix}/load_composite_map')
        self.save_client = self.create_client(SaveCompositeMap, f'{prefix}/save_composite_map')
        self.list_floors_client = self.create_client(ListFloors, f'{prefix}/list_floors')
        self.switch_floor_client = self.create_client(SwitchFloor, f'{prefix}/switch_floor')
        self.add_poi_client = self.create_client(AddPOI, f'{prefix}/add_poi')
        self.list_pois_client = self.create_client(ListPOIs, f'{prefix}/list_pois')
        self.add_wall_client = self.create_client(AddVirtualWall, f'{prefix}/add_virtual_wall')
        self.add_forbidden_client = self.create_client(AddForbiddenArea, f'{prefix}/add_forbidden_area')
        self.add_dangerous_client = self.create_client(AddDangerousArea, f'{prefix}/add_dangerous_area')

    def load_map(self, path: str):
        """加载复合地图"""
        request = LoadCompositeMap.Request()
        request.map_path = path
        future = self.load_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

    def save_map(self, path: str, compress: bool = False):
        """保存复合地图"""
        request = SaveCompositeMap.Request()
        request.map_path = path
        request.compress = compress
        future = self.save_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

    def list_floors(self):
        """列出所有楼层"""
        request = ListFloors.Request()
        future = self.list_floors_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

    def switch_floor(self, floor_id: str, x: float = 0.0, y: float = 0.0, theta: float = 0.0):
        """切换楼层"""
        request = SwitchFloor.Request()
        request.floor_id = floor_id
        request.initial_pose = Pose2D(x=x, y=y, theta=theta)
        request.use_transition = False
        future = self.switch_floor_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

    def add_poi(self, poi_id: str, name: str, poi_type: str, floor_id: str, x: float, y: float, theta: float):
        """添加 POI"""
        request = AddPOI.Request()
        request.poi = POI(
            id=poi_id, name=name, type=poi_type, floor_id=floor_id,
            pose=Pose2D(x=x, y=y, theta=theta)
        )
        future = self.add_poi_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

    def add_virtual_wall(self, floor_id: str, wall_id: int, start: tuple, end: tuple):
        """添加虚拟墙"""
        request = AddVirtualWall.Request()
        request.floor_id = floor_id
        request.wall = VirtualWall(
            id=wall_id,
            start=Point(x=start[0], y=start[1], z=0.0),
            end=Point(x=end[0], y=end[1], z=0.0)
        )
        future = self.add_wall_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

    def add_forbidden_area(self, floor_id: str, area_id: int, points: list):
        """添加禁区"""
        request = AddForbiddenArea.Request()
        request.floor_id = floor_id
        polygon = Polygon()
        polygon.points = [Point32(x=p[0], y=p[1], z=0.0) for p in points]
        request.area = ForbiddenArea(id=area_id, boundary=polygon)
        future = self.add_forbidden_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()

    def add_dangerous_area(self, floor_id: str, area_id: int, points: list, speed_limit: float):
        """添加限速区"""
        request = AddDangerousArea.Request()
        request.floor_id = floor_id
        polygon = Polygon()
        polygon.points = [Point32(x=p[0], y=p[1], z=0.0) for p in points]
        request.area = DangerousArea(id=area_id, boundary=polygon, speed_limit=speed_limit)
        future = self.add_dangerous_client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        return future.result()


# 使用示例
def main():
    rclpy.init()
    # 单机器人模式（默认）
    client = MapManagerClient()
    # 多机器人模式（带命名空间）：client = MapManagerClient(namespace='robot1')

    # 加载地图
    result = client.load_map('/opt/fftai/Navigation/Map/office')
    if result.success:
        print(f"地图加载成功: {result.map_info.name}")
        print(f"楼层数: {len(result.map_info.floors)}")

    # 列出楼层
    floors = client.list_floors()
    for floor in floors.floors:
        print(f"  楼层: {floor.floor_id} ({floor.name})")

    # 切换楼层
    client.switch_floor('F2', x=1.0, y=2.0, theta=0.0)

    # 添加 POI
    client.add_poi('charging_1', '充电站', 'charging_station', 'F1', 5.0, 3.0, 1.57)

    # 添加虚拟墙
    client.add_virtual_wall('F1', 1, (0.0, 0.0), (2.0, 0.0))

    # 添加禁区
    client.add_forbidden_area('F1', 1, [(0.0, 0.0), (2.0, 0.0), (2.0, 2.0), (0.0, 2.0)])

    # 添加限速区
    client.add_dangerous_area('F1', 1, [(5.0, 5.0), (8.0, 5.0), (8.0, 8.0), (5.0, 8.0)], 0.3)

    # 保存地图
    client.save_map('/opt/fftai/Navigation/Map/office_updated', compress=True)

    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## 10. 接口汇总表

### 话题列表

| 话题名 | 消息类型 | 方向 | 说明 |
| ------ | -------- | ---- | ---- |
| /slam/mode_status | std_msgs/String | 发布 | 当前模式状态 |
| /clear_map | std_msgs/String | 订阅 | 清除地图触发 |
| /map | nav_msgs/OccupancyGrid | 发布 | 2D占用栅格地图 |
| /optimize_map | nav_msgs/OccupancyGrid | 发布 | 优化后的2D地图 |
| /cloud_registered_gravity | sensor_msgs/PointCloud2 | 发布 | 3D点云地图 |
| /initialpose | geometry_msgs/PoseWithCovarianceStamped | 订阅 | 初始位姿设置 |
| /robot_pose | geometry_msgs/PoseStamped | 发布 | 机器人3D位姿(6DOF) |
| /odom | nav_msgs/Odometry | 发布 | 里程计数据 |
| /odom_status_code | std_msgs/Int8 | 发布 | 定位状态码 |
| /odom_status_score | std_msgs/Float32 | 发布 | 定位置信度 |
| /plan | nav_msgs/Path | 发布 | 全局规划路径 |
| /cmd_vel | geometry_msgs/Twist | 发布 | 速度命令 |
| /action_status | fourier_msgs/ActionStatus | 发布 | 动作执行状态 |
| /scan | sensor_msgs/LaserScan | 发布 | 2D激光扫描 |
| /segmented_groundless_points | sensor_msgs/PointCloud2 | 发布 | 去地面点云 |
| /imu | sensor_msgs/Imu | 订阅 | IMU数据 |
| /camera_01/color/image_raw | sensor_msgs/Image | 发布 | 彩色图像 |
| /camera_01/color/camera_info | sensor_msgs/CameraInfo | 发布 | 彩色相机标定信息 |
| /camera_01/depth/image_raw | sensor_msgs/Image | 发布 | 深度图像 |
| /camera_01/depth/camera_info | sensor_msgs/CameraInfo | 发布 | 深度相机标定信息 |
| /camera_01/depth/points | sensor_msgs/PointCloud2 | 发布 | 深度点云 |
| /camera_01/fused_data | fourier_msgs/UnifiedCameraData | 发布 | 融合相机数据 |
| /camera_01/filtered_pointcloud | sensor_msgs/PointCloud2 | 发布 | 过滤后的点云 |
| /Humanoid_nav/health | fourier_msgs/HealthInfo | 发布 | 系统健康状态 |
| /Humanoid_nav/events | fourier_msgs/EventsInfo | 发布 | 系统事件通知 |
| /current_floor | std_msgs/String | 发布 | 当前楼层 ID（命名空间时：`/<ns>/current_floor`）|

### 服务列表

#### SLAM 服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /slam/set_mode | fourier_msgs/SetMode | 切换建图/定位模式 |
| /slam/load_map | fourier_msgs/LoadMap | 加载地图 (Lightning-LM) |
| /slam/save_map | fourier_msgs/SaveMap | 保存地图 (Lightning-LM) |

#### 导航动作服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /cancel_current_action | fourier_msgs/CancelCurrentAction | 取消当前动作 |
| /get_current_action | fourier_msgs/GetCurrentAction | 获取当前动作 |

#### Map Manager - 复合地图服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /load_composite_map | map_manager/LoadCompositeMap | 加载复合地图 |
| /save_composite_map | map_manager/SaveCompositeMap | 保存复合地图 |
| /clear_composite_map | map_manager/ClearCompositeMap | 清除复合地图 |

#### Map Manager - 楼层管理服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /list_floors | map_manager/ListFloors | 列出所有楼层 |
| /switch_floor | map_manager/SwitchFloor | 切换楼层 |
| /add_floor | map_manager/AddFloor | 添加楼层 |
| /remove_floor | map_manager/RemoveFloor | 删除楼层 |
| /save_floor | map_manager/SaveFloor | 保存楼层数据 |
| /load_floor | map_manager/LoadFloor | 加载楼层数据 |
| /clear_floor | map_manager/ClearFloor | 清除楼层数据 |

#### Map Manager - POI 管理服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /add_poi | map_manager/AddPOI | 添加 POI |
| /get_poi | map_manager/GetPOI | 获取 POI |
| /list_pois | map_manager/ListPOIs | 列出 POI |
| /update_poi | map_manager/UpdatePOI | 更新 POI |
| /remove_poi | map_manager/RemovePOI | 删除 POI |
| /remove_all_pois | map_manager/RemoveAllPOIs | 删除所有 POI |

#### Map Manager - 虚拟墙管理服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /add_virtual_wall | map_manager/AddVirtualWall | 添加虚拟墙 |
| /get_virtual_wall | map_manager/GetVirtualWall | 获取虚拟墙 |
| /list_virtual_walls | map_manager/ListVirtualWalls | 列出虚拟墙 |
| /update_virtual_wall | map_manager/UpdateVirtualWall | 更新虚拟墙 |
| /remove_virtual_wall | map_manager/RemoveVirtualWall | 删除虚拟墙 |
| /remove_virtual_walls | map_manager/RemoveVirtualWalls | 批量删除虚拟墙 |

#### Map Manager - 禁区管理服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /add_forbidden_area | map_manager/AddForbiddenArea | 添加禁区 |
| /get_forbidden_area | map_manager/GetForbiddenArea | 获取禁区 |
| /list_forbidden_areas | map_manager/ListForbiddenAreas | 列出禁区 |
| /update_forbidden_area | map_manager/UpdateForbiddenArea | 更新禁区 |
| /remove_forbidden_area | map_manager/RemoveForbiddenArea | 删除单个禁区 |
| /remove_all_forbidden_areas | map_manager/RemoveAllForbiddenAreas | 删除所有禁区 |

#### Map Manager - 限速区管理服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /add_dangerous_area | map_manager/AddDangerousArea | 添加限速区 |
| /get_dangerous_area | map_manager/GetDangerousArea | 获取限速区 |
| /list_dangerous_areas | map_manager/ListDangerousAreas | 列出限速区 |
| /update_dangerous_area | map_manager/UpdateDangerousArea | 更新限速区 |
| /remove_dangerous_area | map_manager/RemoveDangerousArea | 删除限速区 |
| /remove_all_dangerous_areas | map_manager/RemoveAllDangerousAreas | 删除所有限速区 |

#### Map Manager - 房间管理服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /add_room | map_manager/AddRoom | 添加房间 |
| /get_room | map_manager/GetRoom | 获取房间 |
| /list_rooms | map_manager/ListRooms | 列出房间 |
| /update_room | map_manager/UpdateRoom | 更新房间 |
| /remove_room | map_manager/RemoveRoom | 删除房间 |
| /remove_all_rooms | map_manager/RemoveAllRooms | 删除所有房间 |

#### Map Manager - 语义对象管理服务

| 服务名 | 服务类型 | 说明 |
| ------ | -------- | ---- |
| /add_semantic_object | map_manager/AddSemanticObject | 添加语义对象 |
| /get_semantic_object | map_manager/GetSemanticObject | 获取语义对象 |
| /list_semantic_objects | map_manager/ListSemanticObjects | 列出语义对象 |
| /update_semantic_object | map_manager/UpdateSemanticObject | 更新语义对象 |
| /remove_semantic_object | map_manager/RemoveSemanticObject | 删除语义对象 |
| /remove_all_semantic_objects | map_manager/RemoveAllSemanticObjects | 删除所有语义对象 |

### 动作列表

| 动作名 | 动作类型 | 说明 |
| ------ | -------- | ---- |
| /navigate_to_pose | nav2_msgs/NavigateToPose | 导航到目标点 |
| /navigate_through_poses | nav2_msgs/NavigateThroughPoses | 经过多点导航 |
| /follow_path | nav2_msgs/FollowPath | 跟随路径 |
| /compute_path_to_pose | nav2_msgs/ComputePathToPose | 计算路径 |

### 消息类型列表

#### Map Manager 消息

| 消息类型 | 说明 |
| -------- | ---- |
| map_manager/msg/Floor | 楼层信息 |
| map_manager/msg/CompositeMapInfo | 复合地图信息 |
| map_manager/msg/FloorTransition | 楼层过渡点 |
| map_manager/msg/POI | 兴趣点 |
| map_manager/msg/VirtualWall | 虚拟墙 |
| map_manager/msg/ForbiddenArea | 禁区 |
| map_manager/msg/DangerousArea | 限速区 |
| map_manager/msg/Room | 房间 |
| map_manager/msg/SemanticObject | 语义对象 |
| map_manager/msg/MapLayer | 地图层 |

---